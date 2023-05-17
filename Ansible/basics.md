# Ansible Basics

- [Ansible Basics](#ansible-basics)
  - [Using Ad-hoc Commands](#using-ad-hoc-commands)
  - [Finding Documentation](#finding-documentation)
  - [Ansible Basic Playbook](#ansible-basic-playbook)
  - [Ansible Facts](#ansible-facts)
  - [Default Configurations](#default-configurations)

## Using Ad-hoc Commands

format: `[optional_variable] <cmd> <target> <module>`

eg:

- `ansible_connection=local ansible localhost -m ping`
- `ansible_connection=local ansible localhost -m package -a "name=vim" -u root -k`
- `ansible_connection=local ansible localhost -m package -a "name=vim" -u ansible -b -k -K`
- `ansible_connection=local ansible localhost -m package -a "name=vim" -i inventory`

Here:

- m - module
- u - user account to be used to run
- k - prompt for password
- b - become root user - sudo to root
- K - prompy for previledge escalation password - sudo password
- i - sepecify an inventory file

## Finding Documentation

`ansible-doc -l`
`ansible-doc [module_name]`

## Ansible Basic Playbook

**Basic Structure:**

```yml
## simple-playbook.yml
- name: Simple Playbook      ## list of plays
  hosts: localhost           ## targets
  connection: local          ## Variable
  tasks:                      
      - name : ping server   ## list of tasks
        ping:                ## module

      - name: display OS info
        debug:
          msg: "{{ansible_os_family}}"
```

**Running the Playbook:**

- To tun the playbook:
`ansible-playbook simple-playbook.yml`

**Output:**

```bash
dhayanand@ZenBook-Pro-Duo:~$ ansible-playbook simple-playbook.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Simple Playbook] ********************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************
ok: [localhost]

TASK [ping server] ************************************************************************************************************************************************
ok: [localhost]

TASK [display OS info] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Debian"
}

PLAY RECAP ********************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

dhayanand@ZenBook-Pro-Duo:~$
```

## Ansible Facts

- Ansible Plays have default task `Gathering Facts` to gather facts about the target node
- This default task can be disabled if not needed
- `Facts` can be listed using the `setup` module if needed
- These facts can be used to make decisions on hoe the play should be run
- Facts can also be used as variables inside the playbook

```bash
## to get all the facts of the server
ansible localhost -m setup | less

## to fileter a specific fact
ansible localhost -m setup -a "filter=ansible_os_family"

localhost | SUCCESS => {
    "ansible_facts": {
        "ansible_os_family": "Debian"
    },
    "changed": false
}
```

## Default Configurations

`/etc/ansible/ansible.cfg`

- provides default Ansible configuration as an example setup with default values for ansible & ansible playbooks
- but this is NOT inteneted to be used as a working configuration

```bash
## find the default configuration file
dhayanand@ZenBook-Pro-Duo:~$ ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/dhayanand/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Jun 22 2022, 20:18:18) [GCC 9.4.0]
dhayanand@ZenBook-Pro-Duo:~$
```
