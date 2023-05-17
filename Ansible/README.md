# Ansible-Notes

**Intro:**

- Ansible is an opensource configuration management tool.
- The code written in YAML to perform tasks is called a "Playbook"
- The Playbook consists of ordered list of tasks which are called as Plays
- Ansible is "Idempotent"

**How it works:**

- Ansible runs each task in parallel across hosts
- Waits until a task in all hosts have been completed before moving to next task
- Runs tasks in the order as written in Playbook
- We can also check Playbook sanity and dry run before executing the Playbook

**When the Playbook is run:**

- Ansible generates python script based on the module that is invoked
- Copies and executes the python script to the remote host

[Basics](Ansible/basics.md)
[Configuration](ansible-configuration.md)
[Inventory](ansible-inventory.md)