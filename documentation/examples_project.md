### Create a Bolt examples project

Before your begin, make sure you've [updated Bolt to version 2.8.0 or
higher](./bolt_installing.md).

To get started with a Bolt project:


1. Create Bolt project called `bolt_examples`:

```
bolt project init bolt_examples
```

Make sure your current working directory is now `bolt_examples`

2. Create directories to hold local tasks and plans in `bolt_examples` project directory

```
mkdir -p site-modules/tasks site-modules/plans
```

3. Create `inventory.yaml` file with the content below replacing target and credential information with your own. 


> **Note:** If you're only working with one of these groups (linux/windows) you can simply delete the content for said group in your inventory file.

```yaml
groups:
- name: windows
  targets:
  - win.classroom.puppet.com
  config:
    transport: winrm
    winrm:
      user: Administrator
      password: 's3cr3t'
- name: linux
  targets:
  - lin.classroom.puppet.com
  config:
    transport: ssh
    ssh:
      host-key-check: false
      user: puppet
      run-as: root
      private-key: ~/.ssh/puppet.pem
```

4. To make sure that your inventory is configured correctly and that you can connect to all the targets, run the following command from inside the project directory with your target groups: 

```
bolt command run 'echo hi' --targets windows,linux
```

**Note:** The `--targets windows` argument refers to the target group defined in the inventory file.

You should get the following output:

```
Started on win.classroom.puppet.com...
Started on lin.classroom.puppet.com...
Finished on win.classroom.puppet.com:
  STDOUT:
    hi
Finished on lin.classroom.puppet.com:
  STDOUT:
    hi
Successful on 2 targets: win.classroom.puppet.com,lin.classroom.puppet.com
Ran on 2 targets in 0.97 sec
```

You should now have a bolt project directory structure that looks like this:

```console
bolt_examples
└── bolt.yaml
    inventory.yaml
    site-modules
    └── tasks
        plans      
```