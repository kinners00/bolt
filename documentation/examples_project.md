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

3. Create `inventory.yaml` file with the content below replacing target and credential information with your own. 


> **Note:** If you're only working with one of these groups (linux/windows) you can simply delete the content for said group in your inventory file.

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

4. To make sure that your inventory is configured correctly and that you can connect to all the targets, run the following command from inside the project directory with your target groups: 

    ```
    bolt command run 'echo hi' --targets windows,linux
    ```

5. Create `tasks` and `plans` directories in the root of the project

    ```
    mkdir tasks plans
    ````

You should now have a bolt project directory structure that looks like this:

    ```console
    bolt_examples
    └── bolt.yaml
        bolt-project.yaml
        inventory.yaml
        tasks
        plans
    ```

