## Lecture 15 Ansible

# Inventory

In ansible the [host file](./first_ansible_usage/hosts) is also called inventory.
The default location for this would be /etc/ansible/hosts
This is the collection of hosts which needs to be managed by ansible

Authentication is done via ssh key

# Playbooks

* Ordered list of tasks 
* Plays & tasks run in order from top to bottom 
* Written in yaml 
* A Playbook can have multiple plays 
* A play is a group of tasks 


# Modules and Collections

## What is a collection

* Packaging and distribution format of Ansible content
    * Playbooks
    * Modules
    * Plugins
    * ...
* can be released and installed independent of other collections


## Plugins

Pieces of code that add to ansible's functionality or modules


## Ansible Galaxy

Hub for Ansible collections.
ansible-galaxy cli tool to install a specific collection

Example:
``` bash
ansible-galaxy collection install amazon.aws==7.5.0
```

In case a collection gets an update it is recommended to use a specific version.

We can also create our own collection (for bigger Ansible projects) where you consolidate plugins, modules and playbooks needed to execute specific orchestration tasks.

Collections have a simple data structure.
required:
* galaxy.yml file (at the root level of the collection) containing all the metadata of the collection

Every collection name is made up of namespace-name.collection-nam:
amazon.aws
amazon=namespace
aws=collection

FQCN = fully qualified collection name
