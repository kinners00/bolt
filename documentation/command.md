Use Bolt commands to connect directly to the systems where you want to execute commands, run scripts, and upload files.


## Running commands on remote targets

When using an inventory file to store connection and target information, we can run a short bolt command:

**Linux**
```shell script
bolt command run 'hostname' --targets linux
```

**Windows**
```shell script
bolt command run 'hostname' --targets windows
```

If the same works on nodes assigned to groups in your inventory file, you can run the command on multiple groups at the same time:

```shell script
bolt command run 'hostname' --targets linux,windows
```

In this example, our inventory file looks like this:

```yaml
---
groups:
- name: linux
  targets:
  - lin.puppet.com 
  config:
    transport: ssh
    ssh:
      host-key-check: false
      user: test
      run-as: root
      private-key: ~/.ssh/test.pem
- name: windows
  targets:
  - win.puppet.com
  config:
    transport: winrm
    winrm:
      ssl: false
      user: test
      password: 'Puppetlabs!'
```

If you're not using an inventory file to store your target connection details, you need to pass those details on your bolt command. Below you can see the `hostname` command running on both linux and windows targets.

**Linux**
```shell script
bolt command run 'hostname' --targets lin.puppet.com --user test --private-key ~/.ssh/test.pem --no-host-key-check
```

**Windows**
```shell script
bolt command run 'hostname' --targets win.puppet.com --user test --password Puppetlabs! --transport winrm --no-ssl
```

   



pause




**Windows**

-   To run a command on WinRM targets, indicate the WinRM protocol in the targets string:
    ```shell script
    bolt command run 'hostname' --targets winrm://win.test.com --user puppet --password puppetlabs
    ```
   
**Linux**

-   To run a command on SSH targets, indicate the SSH protocol in the targets string:
    ```shell script
    bolt command run 'hostname' --targets ssh://lin.test.com --user puppet --password puppetlabs
    ```

NAME
    run

USAGE
    bolt command run <command> [options]

DESCRIPTION
    Run a command on the specified targets.

EXAMPLES
    bolt command run 'uptime' -t target1,target2

INVENTORY OPTIONS
    -t, --targets TARGETS            Identifies the targets of command.
                                     Enter a comma-separated list of target URIs or group names.
                                     Or read a target list from an input file '@<file>' or stdin '-'.
                                     Example: --targets localhost,target_group,ssh://nix.com:23,winrm://windows.puppet.com
                                     URI format is [protocol://]host[:port]
                                     SSH is the default protocol; may be ssh, winrm, pcp, local, docker, remote
                                     For Windows targets, specify the winrm:// protocol if it has not be configured
                                     For SSH, port defaults to `22`
                                     For WinRM, port defaults to `5985` or `5986` based on the --[no-]ssl setting
    -q, --query QUERY                Query PuppetDB to determine the targets
        --rerun FILTER               Retry on targets from the last run
                                     'all' all targets that were part of the last run.
                                     'failure' targets that failed in the last run.
                                     'success' targets that succeeded in the last run.
        --description DESCRIPTION    Description to use for the job

AUTHENTICATION OPTIONS
    -u, --user USER                  User to authenticate as
    -p, --password PASSWORD          Password to authenticate with
        --password-prompt            Prompt for user to input password
        --private-key KEY            Path to private ssh key to authenticate with
        --[no-]host-key-check        Check host keys with SSH
        --[no-]ssl                   Use SSL with WinRM
        --[no-]ssl-verify            Verify remote host SSL certificate with WinRM

ESCALATION OPTIONS
        --run-as USER                User to run as using privilege escalation
        --sudo-password PASSWORD     Password for privilege escalation
        --sudo-password-prompt       Prompt for user to input escalation password
        --sudo-executable EXEC       Specify an executable for running as another user.
                                     This option is experimental.

RUN CONTEXT OPTIONS
    -c, --concurrency CONCURRENCY    Maximum number of simultaneous connections
        --[no-]cleanup               Whether to clean up temporary files created on targets
    -m, --modulepath MODULES         List of directories containing modules, separated by ':'
                                     Directories are case-sensitive
        --boltdir FILEPATH           Specify what Boltdir to load config from (default: autodiscovered from current working dir)
        --configfile FILEPATH        Specify where to load config from (default: ~/.puppetlabs/bolt/bolt.yaml).
                                     Directory containing bolt.yaml will be used as the Boltdir.
    -i, --inventoryfile FILEPATH     Specify where to load inventory from (default: ~/.puppetlabs/bolt/inventory.yaml)
        --[no-]save-rerun            Whether to update the rerun file after this command.

TRANSPORT OPTIONS
        --transport TRANSPORT        Specify a default transport: ssh, winrm, pcp, local, docker, remote
        --ssh-command EXEC           Executable to use instead of the net-ssh ruby library. 
                                     This option is experimental.
        --copy-command EXEC          Command to copy files to remote hosts if using external SSH. 
                                     This option is experimental.
        --connect-timeout TIMEOUT    Connection timeout (defaults vary)
        --[no-]tty                   Request a pseudo TTY on targets that support it

DISPLAY OPTIONS
        --format FORMAT              Output format to use: human or json
        --[no-]color                 Whether to show output in color
    -v, --[no-]verbose               Display verbose logging
        --trace                      Display error stack traces

GLOBAL OPTIONS
    -h, --help                       Display help
        --version                    Display the version
        --debug                      Display debug logging
