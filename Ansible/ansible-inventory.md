# Ansible Inventory

- Ansible inventory has the list of nodes that we want to manage using Ansible
- The inventory can be saved in ini or yaml or json formats

**Default Inventory:**

- default inventory is defined in ansible.cfg
- usually it is `/etc/ansible/hosts`, but this file is ment to be used as reference to create our own inventory

```bash
ansible-inventory  --host localhost
{
    "ansible_connection": "local",
    "ansible_python_interpreter": "/usr/bin/python3"
}

####### default host file ##########
cat /etc/ansible/hosts

# Ex 1: Ungrouped hosts, specify before any group headers.

green.example.com
blue.example.com
192.168.100.1
192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

[webservers]
alpha.example.org
beta.example.org
192.168.1.100
192.168.1.110

# If you have multiple hosts following a pattern you can specify them like this:

www[001:006].example.com


### custom hosts file ###

[rhel]
192.168.1.200

[centos]
192.168.1.201

## adding varibles to host in inventory
[ubuntu]
ubuntu1 ansible_port=22 ansible_host=192.168.1.301 
ubuntu2 ansible_port=22 ansible_host=192.168.1.302

## create nested groups using the keyword `children`
[redhat:childern]
rhel
centos

```

`ansible --list redhat` - list the host groups

## Inventory Formats

- `.ini` is the simplest inventory format
- `ansible-inventory --list` the default listing prints the inventory in `json` format
- `ansible-inventory --list -y`  - `-y` camn be used to print in `yaml` format

## Inventory Variables

- Inventory variables allow for more customizations
- We can have host variables under the `host_vars` directory and group variables under `group_vars` directory

```bash
echo "ansible_connection: local" > ./host_vars/192.168.1.100
```

## Dynamic Inventory

- simplest dynamic inventory can be created using `nmap` to discover ssh hosts on network and `awk` to create inventory from the nmap Output

```bash
## scan for servers in network 192.168.1.0/24 with port 22 open
sudo nmap -Pn -p22 -n 192.168.1.0/24 --open 
sudo nmap -Pn -p22 -n 192.168.1.0/24 --open -oG -
sudo nmap -Pn -p22 -n 192.168.1.0/24 --open -oG - | awk '/22\/open/{print $2}'

```
