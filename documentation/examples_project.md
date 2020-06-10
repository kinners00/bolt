### Create a Bolt examples project

Before your begin, make sure you've [updated Bolt to version 2.8.0 or
higher](./bolt_installing.md).

To get started with a Bolt project:

1. Create Bolt project:

   ```
   bolt project init bolt_examples
   ```

2. Create a `bolt-project.yaml` file in the root of your Bolt project directory.

   Windows:
   ```
   New-Item -ItemType file bolt-project.yaml
   ```

   Linux:
   ```
   touch bolt-project.yaml
   ```

3. Create `inventory.yaml` file

Windows:
```yaml
groups:
- name: windows
  targets:
  - win1.classroom.puppet.com
  - win2.classroom.puppet.com
  config:
    transport: winrm
    winrm:
      ssl: false
      user: Administrator
      password: 's3cr3t'
```
Linux:
```yaml
- name: linux
  targets:
  - lin1.classroom.puppet.com
  - lin1.classroom.puppet.com
  config:
    transport: ssh
    ssh:
      host-key-check: false
      user: puppet
      run-as: root
      private-key: ~/.ssh/puppet.pem
```

Both target types:
```yaml
groups:
- name: windows
  targets:
  - win1.classroom.puppet.com
  - win2.classroom.puppet.com
  config:
    transport: winrm
    winrm:
      ssl: false
      user: Administrator
      password: 's3cr3t'
- name: linux
  targets:
  - lin1.classroom.puppet.com
  - lin1.classroom.puppet.com
  config:
    transport: ssh
    ssh:
      host-key-check: false
      user: puppet
      run-as: root
      private-key: ~/.ssh/puppet.pem
```

4. Create `tasks` and `plans` directories in the root of the project

    ```
    mkdir tasks plans
    ````


If `bolt-project.yaml` exists at the root of a project directory, Bolt loads the
project as a module. Bolt loads tasks and plans from the `tasks` and `plans`
directories and namespaces them to the project name.

Here is an example of a project using a simplified directory structure:
```console
.
├── bolt.yaml
├── bolt-project.yaml
├── inventory.yaml
├── plans
│   └── myplan.yaml
└── tasks
    └── mytask.yaml
```