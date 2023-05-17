# Ansible Configurations

- [Ansible Configurations](#ansible-configurations)
  - [Hierarchy of Configuration Files](#hierarchy-of-configuration-files)
  - [ansible-config command](#ansible-config-command)
  - [Enforce a Configuration File](#enforce-a-configuration-file)

## Hierarchy of Configuration Files

Hierarchy of configuration files starting from highest priority to least

- environment Variable `ANSIBLE_CONFIG`
- $CWD/ansible.cfg
- $HOME/.ansible.cfg
- /etc/ansible/ansible.cfg

Finding the currently used config file:

```bash
dhayanand@ZenBook-Pro-Duo:~$ ansible --version | grep "config file"
  config file = /etc/ansible/ansible.cfg
dhayanand@ZenBook-Pro-Duo:~$
```

## ansible-config command

```bash
## view the configuration file
ansible-config view

## list all possible configurations along with documentation
ansible-config list

## dump all the configurations without the documentation
ansible-config dump

## dump only the configurations that were changed
ansible-config dump --only-changed
```

## Enforce a Configuration File

Admin can set a specific ansible configuration file for all the users and set it as read-only and prevent the users from changing it.

`declare -xr ANSIBLE_CONFIG=/etc/ansible/ansible.cfg`