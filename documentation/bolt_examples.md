# Automating Windows targets

Bolt lets you automate almost any task you can think of. These are some of the common scenarios we've come across.

If you'd like to share a real-world use case, reach out to us in the #bolt channel on [Slack](https://slack.puppet.com).

For more usage examples, check out the [Puppet blog](https://puppet.com/search/?query=bolt&page=1&configure%5BhitsPerPage%5D=20&refinementList%5Btype%5D%5B0%5D=Post).

## Run a PowerShell script that restarts a service

To show you how you can use Bolt to reuse your existing PowerShell scripts, this guide walks you through running a script with Bolt, and then converting the script to a Bolt task and running that.

> **Before you begin**
> 
> -   Ensure you’ve already [installed Bolt](bolt_installing.md#) on your Windows machine.
> -   Identify a remote Windows target to work with.
> -   Ensure you have Windows credentials for the target.
> -   Ensure you have [configured Windows Remote Management](https://docs.microsoft.com/en-us/windows/desktop/winrm/installation-and-configuration-for-windows-remote-management) on the target.

The example script, called [restart_service.ps1](https://gist.github.com/RandomNoun7/03dfb910e5d93fefaae6e6c2da625c44#file-restart_service-ps1), performs common task of restarting a service on demand. The process involves these steps:

1.  Run your PowerShell script on a Windows target.
1.  Create an inventory file to store information about the target.
1.  Convert your script to a task.
1.  Execute your new task.


### 1. Run your PowerShell script on a Windows target

First, we’ll use Bolt to run the script as-is on a single target.

1.  Create a Bolt project directory to work in, called `bolt-guide`.
2.  Copy the [`restart_service.ps1`](https://gist.github.com/RandomNoun7/03dfb910e5d93fefaae6e6c2da625c44#file-restart_service-ps1) script into `bolt-guide`.
3.  In the `bolt-guide` directory, run the `restart_service.ps1` script:
    ```
    bolt script run .\restart_service.ps1 W32Time --targets winrm://<HOSTNAME> -u Administrator -p 
    ```

    ![](restart_service.png)

    **Note:** The `-p` option prompts you to enter a password.

    By running this command, you’ve brought your script under Bolt control and have run it on a remote target. When you ran your script with Bolt, the script was transferred into a temporary directory on the remote target, it ran on that target, and then it was deleted from the target.


### 2. Create an inventory file to store information about your targets

To run Bolt commands against multiple targets at once, you need to provide information about the environment by creating an [inventory file](inventory_file_v2.md). The inventory file is a YAML file that contains a list of targets and target specific data.

1.  Inside the `bolt-guide` directory, use a text editor to create an `inventory.yaml` file.
2.  Inside the new `inventory.yaml` file, add the following content, listing the fully qualified domain names of the targets you want to run the script on, and replacing the credentials in the `winrm` section with those appropriate for your target:
    ```yaml
    groups:
      - name: windows
        targets:
          - win1.classroom.puppet.com
          - win2.classroom.puppet.com       
        config:
          transport: winrm
          winrm:
            user: Administrator
            password: s3cr3t
    ```

    **Note:** To have Bolt securely prompt for a password, use the `--password-prompt` flag without supplying any value. This prevents the password from appearing in a process listing or on the console. Alternatively you can use the [`prompt` plugin](using_plugins.md) to set configuration values via a prompt.

    You now have an inventory file where you can store information about your targets.

    You can also configure a variety of options for Bolt in `bolt.yaml`, including global and transport options. For more information, see [Bolt configuration options](bolt_configuration_reference.md).


### 3. Convert your script to a Bolt task

To convert the `restart_service.ps1` script to a task, giving you the ability to reuse and share it, create a [task metadata](writing_tasks.md#) file. Task metadata files describe task parameters, validate input, and control how the task runner executes the task.

**Note:** This guide shows you how to convert the script to a task by manually creating the `.ps1` file in a directory called `tasks`. Alternatively, you can use Puppet Development Kit (PDK), to create a new task by using the [`pdk new task` command](https://puppet.com/docs/pdk/1.x/pdk_reference.html#pdk-new-task-command). If you’re going to be creating a lot of tasks, using PDK is worth getting to know. For more information, see the [PDK documentation.](https://puppet.com/docs/pdk/1.x/pdk_overview.html)

1.  In the `bolt-guide` directory, create the following subdirectories:
    ```
    bolt-guide/
    └── modules
        └── gsg
            └── tasks
    ```
2.  Move the `restart_service.ps1` script into the `tasks` directory.
3.  In the `tasks` directory, use your text editor to create a task metadata file — named after the script, but with a `.json` extension, in this example, `restart_service.json`.
4.  Add the following content to the new task metadata file:

    ```json
    {
      "puppet_task_version": 1,
      "supports_noop": false,
      "description": "Stop or restart a service or list of services on a target.",
      "parameters": {
        "service": {
          "description": "The name of the service, or a list of service names to stop.",
          "type": "Variant[Array[String],String]"
        },
        "norestart": {
          "description": "Immediately restart the services after start.",
          "type": "Optional[Boolean]"
        }
      }
    }
    ```

5.  Save the task metadata file and navigate back to the `bolt-guide` directory.

    You now have two files in the `gsg` module’s `tasks` directory: `restart_service.ps1` and `restart_service.json` -- the script is officially converted to a Bolt task. Now that it’s converted, you no longer need to specify the file extension when you call it from a Bolt command.
6.  Validate that Bolt recognizes the script as a task:
    ```
    bolt task show gsg::restart_service
    ```

    ![](bolt_PS_2.png)

    Congratulations! You’ve successfully converted the `restart_service.ps1` script to a Bolt task.

7.  Execute your new task:
    ```
    bolt task run gsg::restart_service service=W32Time --targets windows
    ```

    ![](bolt_PS_3.png)

    **Note:** `--targets windows` refers to the name of the group of targets that you specified in your inventory file. For more information, see [Specify targets](running_bolt_commands.md#adding-options-to-bolt-commands).


## Deploy a package with Bolt and Chocolatey

You can use Bolt with Chocolatey to deploy a package on a Windows node by writing a [Bolt plan](writing_plans.md) that installs Chocolatey and uses Puppet's Chocolatey package provider to install a package. This is all done using content from the [Puppet Forge](https://forge.puppet.com).

> **Before you begin**
>
>- Install [Bolt](https://puppet.com/docs/bolt/latest/bolt_installing.html)
>- Ensure you have Powershell installed and Windows Remote Management (WinRM) access.
>- powershell.exe and pwsh.exe must be available in the system PATH
>- GUI or command line text editor such as [VSCode](https://code.visualstudio.com/download) or [Notepad++](https://notepad-plus-plus.org/downloads).

**Note:** VSCode has extensions for [Puppet](https://marketplace.visualstudio.com/items?itemName=puppet.puppet-vscode) and [YAML](https://marketplace.visualstudio.com/items?itemName=docsmsft.docs-yaml) which can be a useful companions when editing YAML and Puppet code.


In this example, you:

- Build a project-specific configuration using a Bolt project directory.
- Download module content from the Puppet Forge.
- Write a Bolt plan to apply Puppet code and orchestrate the deployment of a package resource using the Chocolatey provider.

### 1. Create a Bolt project

Bolt runs in the context of a [Bolt project directory](bolt_project_directories.md). This directory contains all of the configuration, code, and data loaded by Bolt. Adding a `bolt.yaml` file (even if it's empty) to any directory automatically makes it a Bolt project directory. We could create a directory and `bolt.yaml` file manually but fortunately we can achieve the same result by running one simple command with bolt, as shown below.


1. Create a Bolt project called `bolt_choco_example`

```
bolt project init ./bolt_choco_example
```
### 2. Create a inventory file
   
You can use an inventory file to store information about your targets and arrange them into groups. Grouping your targets lets you aim your Bolt commands at the group instead of having to reference each target individually with it's relevant connection parameters. 

Bolt inventory is typically stored in a `inventory.yaml` file in the root of the project directory. 

1. In the `bolt_choco_example` project directory, create `inventory.yaml` file with the following code, replacing the target and credential information to those appropriate for your target: 

```yaml
groups:
  - name: windows
    targets:
      - win1.classroom.puppet.com
      - win2.classroom.puppet.com
    config:
      transport: winrm
      winrm:
        user: Administrator
        password: s3cr3t
```

2. To make sure that your inventory is configured correctly and that you can connect to all the targets, run the following command from inside the project directory: 

```
bolt command run 'echo hi' --targets windows
```

**Note:** The `--targets windows` argument refers to the target group defined in the inventory file.

You should get the following output:

```
Started on win1.classroom.puppet.com...
Started on win2.classroom.puppet.com...
Finished on win1.classroom.puppet.com:
  STDOUT:
    hi
Finished on win2.classroom.puppet.com:
  STDOUT:
    hi
Successful on 2 targets: win1.classroom.puppet.com,win2.classroom.puppet.com
Ran on 2 targets in 1.11 seconds
```

### 3. Download and install the Chocolatey module

Bolt uses a [Puppetfile](https://puppet.com/docs/pe/latest/puppetfile.html) to install module content from the Forge. A `Puppetfile` is a formatted text file that specifies the modules and data you want in each environment.

1. In the project directory, create a file named `Puppetfile` with the modules needed for this example:

```     
mod 'puppetlabs-chocolatey', '5.0.2'
mod 'puppetlabs-stdlib', '6.3.0'
mod 'puppetlabs-powershell', '3.0.1'
mod 'puppetlabs-registry', '3.1.0'
mod 'puppetlabs-pwshlib', '0.4.1'
 ```

Note that you can install modules from a number of different sources. For more information, see the [Puppetfile README](https://github.com/puppetlabs/r10k/blob/master/doc/puppetfile.mkd#examples).

1. In the project directory, run this command to install the required modules:

```
bolt puppetfile install
```

After it runs, you can see a `modules` directory inside the project directory, containing the modules you specified in the `Puppetfile`.

### 4. Write a Bolt plan to apply Puppet code

Write a Bolt plan to orchestrate the deployment of a package resource using the Chocolatey provider. Plans allow you to run more than one task with a single command, compute values for the input to a task, process the results of tasks, or make decisions based on the result of running a task.

Before you create your bolt plan, you'll need to create the correct folder structure in order to store it. The `site-modules` directory is where you will add all local code and modules. Inside the `site-modules` directory, you'll need to create a local module directory named `puppet_choco_tap` and add a `plans` subdirectory. 

After the next few labs, your site-module folder tree should look like this:

```
bolt_choco_example
└── site-modules
    └── puppet_choco_tap
        └── plans
            └── installer.pp 
```

1. From the `bolt_choco_example` project directory, create the folder tree shown above:

   ```
   mkdir -p site-modules/puppet_choco_tap/plans
   ```


2. Inside the `plans` directory, create a plan called `installer.pp` and add the following code:

```
plan puppet_choco_tap::installer(
  TargetSpec $targets,
  String $package,
  Variant[Enum['absent', 'present'], String ] $ensure = 'present',
){
  apply_prep($targets)

  apply($targets){
    include chocolatey

    package { $package :
      ensure   => $ensure,
      provider => 'chocolatey',
      }
    }
  }
```
Take note of the following features of the plan:

- It has three parameters:
    -  The list of targets to install the package on 
    -  A `package` string for the package name
    -  The `ensure` state of the package which allows for version, absent or present.


- It has the `apply_prep` function call, which is used to install modules needed by `apply` on targets as well as to gather facts about the targets.
- `include chocolatey` installs the Chocolatey package manager. The Chocolatey provider is also deployed as a library with the Puppet agent in `apply_prep`.
- The [package resource](https://puppet.com/docs/puppet/latest/types/package.html) ensures a package's state using the Chocolatey provider.

1. To verify that the `puppet_choco_tap::installer` plan is available, run the following command
   inside the `bolt_choco_example` directory:

```
bolt plan show
```

The output should look like:

```
aggregate::count
aggregate::nodes
aggregate::targets
canary
facts
facts::info
puppet_choco_tap::installer
puppetdb_fact
reboot
```

### 5. Run bolt plan to install package

1. Run the plan with the `bolt plan run` command: 

```
bolt plan run puppet_choco_tap::installer package=frogsay --targets=windows
```

The output looks like this:

```
Starting: plan puppet_choco_tap::installer
Starting: install puppet and gather facts on win1.classroom.puppet.com, win2.classroom.puppet.com
Finished: install puppet and gather facts with 0 failures in 22.11 sec
Starting: apply catalog on win1.classroom.puppet.com, win2.classroom.puppet.com
Finished: apply catalog with 0 failures in 18.77 sec
Starting: apply catalog on win1.classroom.puppet.com, win2.classroom.puppet.com
Finished: apply catalog with 0 failures in 33.74 sec
Finished: plan puppet_choco_tap::installer in 74.63 sec
```

2. To check that the installation worked, run the following `frogsay` command: 

```        
bolt command run 'frogsay ribbit' --targets=windows 
```

The result will vary on each server, and will look something like this:

```
Started on win1.classroom.puppet.com...
Started on win2.classroom.puppet.com...
Finished on win1.classroom.puppet.com:
  STDOUT:
    
            DO NOT PAINT OVER FROG.
            /
      @..@
     (----)
    ( >__< )
    ^^ ~~ ^^
Finished on win2.classroom.puppet.com:
  STDOUT:
    
            TO PREVENT THE RISK OF FIRE OR ELECTRIC SHOCK, DO NOT ENGAGE WITH FROG
            WHILE AUTOMATIC UPDATES ARE BEING INSTALLED.
            /
      @..@
     (----)
    ( >__< )
    ^^ ~~ ^^
Successful on 2 targets: win1.classroom.puppet.com,win2.classroom.puppet.com
Ran on 2 targets in 3.15 seconds
```

That’s it! In this one plan, you have both installed Chocolatey and deployed the package to two targets. You can do the same thing on any number of targets by editing the inventory file. Note that Chocolatey will remain installed on your machine.

After you have installed your package, with the help of Bolt, you can use Chocolatey to automate all of the package management tasks for upgrades or uninstalls. You can use Puppet Enterprise to guarantee state across all of your nodes and handle configuration drift — and make sure no one accidentally uninstalls the package that you just installed.
