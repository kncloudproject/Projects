# Ansible refactoring and static assignments (imports androles)
- In this project I will continue working with `ansible-config-mgt` repository and make some improvements by refactoring the Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook. It allows you to organize your tasks and reuse them when needed.
  
  > Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

### Step 1: Jenkins Job Enhancement
  
  1. `sudo mkdir /home/ubuntu/ansible-config-artifact`

  2. `chmod -R 0777 /home/ubuntu/ansibleconfig-artifact`
  3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins
  4. Create a new Freestyle project and name it `save_artifacts`
  5. This project will be triggered by completion of your existing ansible project. Configure it accordingly.
  6. The main idea of `save_artifacts` project is to save artifacts into `/home/ubuntu/ansible-config-artifact` directory. To achieve this, create a `Build` step and choose Copy artifacts from other project , specify ansible as a source project and `/home/ubuntu/ansible-config-artifact` as a target directory.
  7.  Test your set up by making some change in README.MD Øle inside your  `ansible-config-mgt` repository (right inside master branch)


### Step 2: Refactor Ansible code by importing other playbooks into `site.yml`
```yml
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```

```yml
├── static-assignments
│   └── common.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
```


```yml
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```
- update `site.yml` with `- import_playbook: ../static-assignments/common-del.yml` instead of `common.yml` and run it against dev servers:

### Step 3: Configure uat webservers with a role webserver
```yml
mkdir roles
cd roles
ansible-galaxy init webserver

```

```yml
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```

```yml
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
```
- Update your inventory `ansible-config-mgt/inventory/uat.yml` fle with IP addresses of your 2 UAT Web servers

