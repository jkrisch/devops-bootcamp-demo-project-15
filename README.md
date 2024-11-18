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


## Modules and Collections

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


## Variables
can be defined within the playbook using the 
``` yaml
vars:
    - variable1: value1
    - variable2: value2
```
syntax
or by passing them when executing ansible-playbook using the
``` bash
--extra-vars "variable1=value1 variable2=value2"
```
parameter

or (what is always preferred) use a vars file:



## Demo Project - Deploy Nexus

we can follow all commands from the previous installation.
We only have to transform them into the respective modules.

To avoid that we overwrite the existing nexus folder we check it and include a when-clause (conditional).


## Demo Project - Deploy docker and start applications on ec2 instance and docker compose
see [ansible files](./deploy-2-ec2-ansible/) and [terraform-files](./deploy-2-ec2-terraform/).
Using the remote_exec in terraform you can integrate the playbooks within the terraform process:
[see](./deploy-2-ec2-tf-ansible-integration/)
We can either define the remote_exec within the resource which is affected or we use the resource_null resource


## Dynamic Inventory
Manage an inventory which changes over time.
A reason for this could be scaling up and down your cloud infrastructure (IPs change everytime.)

Before being able to create the dynamic inventory we need to connect to the aws account and fetch server informations.
For this there are two options:

1. Inventory Plugins
2. Inventory Scripts

Redhat recommends using Plugins (as they integrate better with ansible).
So for aws we need to use the aws_ec2 Inventory plugin (https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html)-
Using
```bash
ansible-inventory -i inventory_aws_ec2.yaml --graph
```
or
```bash
ansible-inventory -i inventory_aws_ec2.yaml --list
```
We can see the information we get from our aws account.
But we get the private DNS names and not the public ones which we could use to connect to the server.
Therefore we need to do the following:
add the line 
``` json
enable_dns_hostnames = true
```

In case we want to create environment based inventories we can use the 'filter' attribute in the dynamic inventory file.


## Ansible - K8s
we use localhost as host, as we run the tasks locally but connect to the k8s cluster using the kubeconfig file.
To deploy apps or other k8s components using ansible we use the *kubernetes.core.k8s* module.
We can either pass config files:
```yaml
    - name: Deploy nginx app
      kubernetes.core.k8s:
        src: nginx-config.yaml
        state: present
        kubeconfig: /home/jonas/devopsbootcamp/devops-bootcamp-lecture-12-tf-learn/kubeconfig_myapp-eks-cluster
        namespace: myapp-namespace
```
or pass the whole config using
```yaml
    - name: Deploy nginx app
      kubernetes.core.k8s:
        state: present
        kubeconfig: /home/jonas/devopsbootcamp/devops-bootcamp-lecture-12-tf-learn/kubeconfig_myapp-eks-cluster
        definition:
            apiVersion: v1
            kind: Service
            ...
```


## Run Ansible from Jenkins Pipeline
With other tools like helm, kubectl etc. we installed them on the jenkins server (or in my case I installed them during the container build).
With Ansible we follow a different approach:
* Create a dedicated server for Ansible (Ansible Control Node)
* Install Ansible on that server
This is a common practice

After deploying the droplet:
* install ansible-core
* install python3-boto3 package
Then create [Jenkinsfile](./ansible-jenkins/Jenkinsfile).
 

## Ansible Roles
With Roles you can structure your playbooks into smaller more maintanable files.
It also makes reusability much easier as you can refer to specific tasks which are in certain roles within different plays.
One can look at roles like "packages".

A Role can also contain files which you need like specific files or templates, or role-specific variables and even custom modules (library/my_module.py).

Roles have a standard file/folder structure.

```
roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case
```
[see: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)