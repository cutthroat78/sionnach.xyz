---
title: "Ansible"
tags: [
  "FOSS",
  "computing",
  "software-engineering",
  "devops",
  "ansible",
]
---

## Intro to Ansible

### What is Ansible

Ansible is an open-source automation tool that can describe and build IT infrastructure with [YAML]({{< ref yaml.md >}}). It is one of the most popular tools for configuration. It's use of declarative push-based [YAML]({{< ref yaml.md >}}) is what allows it to be able to automate servers with more efficiency and less chance of human error occurring.

### Things Ansible Can Do

- Provisioning
- Configuration Management
- Continuous Delivery
- Application Deployment
- Security Compliance

### Ansible vs. Scripting

Scripting often requires:
- Time
- Coding Skills
- Maintenance

In comparison, Ansible (is):
- Simple
- Agentless
- Doesn't require coding skills
- Doesn't require maintenance
- Often has fewer lines of code than a script doing the same task would have
- Powerful enough to build and automate even the most complex deployments

![Scripts vs Ansible Playbook](/images/ansible-vs-scripts.png)

## Installing Ansible

Always check the [Official Documentation's Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) to get the latest info on how to install ansible

## Ansible Configuration Files

When Ansible is installed it creates a default config file at the default path ```/etc/ansible/ansible.cfg```. This file controls Ansible's default behaviour like where the default inventory files is, where to write Ansible's log file, the timeout time for SSH connections, etc.

The ```ansible.cfg``` is made up of sections with headings like ```[section_name]```, e.g. ```[privilege_escalation]```. The options for each section are in a key value structure that look like this ```key  = value``` e.g. ```inventory = /etc/ansible/hosts```

You can change the ```/etc/ansible/ansible.cfg``` to change Ansible's behaviour when running any playbooks on the machine, but if you want to change Ansible's behaviour for only certain playbooks, you can copy the ```/etc/ansible/ansible.cfg``` file, paste it into your playbook's directory (e.g. pasting it to ```/home/user/db-playbooks/ansible.cfg```) and make changes to your newly created config file

If you want to specify a specific config file to use when running a playbook you can change the ```ANSIBLE_CONFIG``` environment variable before running your ```ansible-playbook``` command e.g. ```$ANSIBLE_CONFIG=/home/user/ansible-web.cfg ansible-playbook playbook.yml```

### Ansible's Order of What Configuration File to Pick

1. The file specified VIA ```ANSIBLE_CONFIG``` environment variable
2. The file in playbook directory e.g. if playbook directory is ```/home/user/network-playbooks/```, then ansible will use the config file at ```/home/user/network-playbooks/ansible.cfg```
3. The file named ```.ansible.cfg``` that is in the user's home directory
4. The file at ```/etc/ansible/ansible.cfg```

<!--Video 6 at 04:54-->

## Inventory

Ansible can work with one or more systems at the same time. To be able to work with different target systems, ansible needs to connect with them. Ansible can use

- SSH for Linux Systems
- WinRM (Windows Remote Management) for Windows Systems

This method of working with different target systems is what makes ansible agentless (meaning that you don't have to install any software (agents) on target machines to be able to control/work with them).

Information about target systems is stored in an inventory file.

If you don't create a new inventory file, ansible will default to using the inventory file located at ```/etc/ansible/hosts```.

Inventory files can stored in an INI or [YAML]({{< ref yaml.md >}}) format.

Inventory files can use either (external or internal) IP addresses or hostnames for each target system.

### Basic Inventory Example

#### INI

```
server1.company.com
server2.company.com
123.123.123.123
192.168.1.1
```

#### YAML

```
all:
  hosts:
    server1.company.com:
    server2.company.com:
    123.123.123.123:
    192.168.1.1:
```

### Grouping Inventory Example

#### INI

```
server1.company.com

[web]
server2.company.com
server3.company.com

[db]
server4.company.com
server5.company.com
```

#### YAML

```
all:
  children:
    ungrouped:
      hosts:
        server1.company.com:
    web:
      hosts:
        server2.company.com:
        server3.company.com:
    db:
      hosts:
        server4.company.com:
        server5.company.com:
```

### Inventory Example with Groups Inside of A Group

#### INI

```
[web]
server1.company.com
server2.company.com

[db]
server3.company.com
server4.company.com

[all:children]
web
db
```

#### YAML

```
all:
  children:
    web:
      hosts:
        server1.company.com:
        server2.company.com:
    db:
      hosts:
        server3.company.com:
        server4.company.com:
```

### Inventory Parameters

#### Inventory with Alias Parameter Example

##### INI

```
web ansible_host=server1.company.com
db ansible_host=server2.company.com
mail ansible_host=server3.company.com
web2 ansible_host=server4.company.com
```

##### YAML

```
all:
  children:
    ungrouped:
      hosts:
        web:
          ansible_host: server1.company.com
        db:
          ansible_host: server2.company.com
        mail:
          ansible_host: server3.company.com
        web2:
          ansible_host: server4.company.com
```

#### Inventory With Connection Parameter Example

##### INI

```
server1.company.com ansible_connection=ssh
server2.company.com ansible_connection=winrm
server3.company.com ansible_connection=ssh
server4.company.com ansible_connection=winrm
```

##### YAML

```
all:
  children:
    ungrouped:
      hosts:
        server1.company.com:
          ansible_connection: ssh
        server2.company.com:
          ansible_connection: winrm
        server3.company.com:
          ansible_connection: ssh
        server4.company.com:
          ansible_connection: winrm
```

#### Inventory With Connection and Alias Parameters Example

##### INI

```
web ansible_host=server1.company.com ansible_connection=ssh
db ansible_host=server2.company.com ansible_connection=winrm
mail ansible_host=server3.company.com ansible_connection=ssh
db2 ansible_host=server4.company.com ansible_connection=winrm
```

##### YAML

```
all:
  children:
    ungrouped:
      hosts:
        web:
          ansible_connection: ssh
          ansible_host: server1.company.com
        db:
          ansible_connection: winrm
          ansible_host: server2.company.com
        mail:
          ansible_connection: ssh
          ansible_host: server3.company.com
        db2:
          ansible_connection: winrm
          ansible_host: server4.company.com
```

#### Other Inventory Parameters

- ansible_connection
  - Defines what connection to use to connect to the target system
  - Example Values: ```ssh```, ```winrm```, ```localhost```:w
- ansible_port
  - Defines what port to use to connect to the target system
  - Example Values: ```22```, ```5986```, ```8922```
- ansible_user
  - Defines what user to use to connect to the target system
  - Example Values: ```root```, ```admin```, ```johnmoe```
- ansible_password
  - Defines what password to use to connect to the target system
  - **Warning:** This method is very insecure
    - I really only recommend using this method for testing on machines that don't really matter
  - Example Values: ```password```, ```P@ssw0rd```, ```4^^#Dv7M-kJ3bNSN```
- ansible_ssh_pass
  - Defines what ssh password to use to connect to the target system
  - **Warning:** This method is very insecure and it is recommend to use SSH keys instead of this method
    - I really only recommend using this method for testing on machines that don't really matter
  - Example Values: ```password```, ```P@ssw0rd```, ```4^^#Dv7M-kJ3bNSN```

There are more parameters and you can read about them [here](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters)

#### Inventory with Many Parameters Example

##### INI

```
web ansible_host=server1.company.com ansible_connection=ssh ansible_user=root
db ansible_host=server2.company.com ansible_connection=winrm ansible_user=admin
mail ansible_host=server3.company.com ansible_connection=ssh ansible_ssh_pass=P@ssw0rd
db2 ansible_host=server4.company.com ansible_connection=ssh ansible_port=8922

localhost ansible_connection=localhost
```

##### YAML

```
all:
  children:
    ungrouped:
      hosts:
        web:
          ansible_connection: ssh
          ansible_host: server1.company.com
          ansible_user: root
        db:
          ansible_connection: winrm
          ansible_host: server2.company.com
          ansible_user: admin
        mail:
          ansible_connection: ssh
          ansible_host: server3.company.com
          ansible_ssh_pass: P@ssw0rd
        db2:
          ansible_connection: ssh
          ansible_host: server4.company.com
          ansible_port: 8922
        localhost:
          ansible_connection: localhost
```

### Convert Inventory to YAML, JSON or TOML

Replace anything inside ```{inventory file}``` with (path to) your own inventory file that you are looking to convert, which can be in any format (INI, YAML, JSON or TOML)

I have not found a way so far to convert **to INI** format

#### YAML

- Convert Inventory File to YAML: ```ansible-inventory -i {inventory file} -y --list```
  - Save Output to File: ```ansible-inventory -i {inventory file} -y --list --output inventory.yaml```

#### JSON

- Convert Inventory File to JSON: ```ansible-inventory -i {inventory file} --list```
  - Save Output to File: ```ansible-inventory -i {inventory file} --list --output inventory.json```

#### TOML

- Convert Inventory File to TOML: ```ansible-inventory -i {inventory file} --toml --list```
  - Save Output to File: ```ansible-inventory -i {inventory file} --toml --list --output inventory.toml```

## Check if Ansible Can Connect to Target Machine

Replace ```{target system}``` with the name of your target system and replace ```{inventory file}``` with (path to) your own inventory file

```ansible {target system} -m ping -i {inventory file}```

## Links

- [Ansible - Wikipedia](https://en.wikipedia.org/wiki/Ansible_(software))
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Official Website](https://www.ansible.com/)
- [Ansible - Arch Linux Wiki](https://wiki.archlinux.org/title/Ansible)
- [Ansible in 100 Seconds by Fireship - YouTube Video](https://www.youtube.com/watch?v=xRMPKQweySE) - Good Introduction to Ansible
- [*Ansible for the Absolute Beginner - Hands-On - DevOps* - Udemy Course](https://www.udemy.com/course/learn-ansible/)
- [*Dive Into Ansible - Beginner to Expert in Ansible - DevOps* - Udemy Course](https://www.udemy.com/course/diveintoansible/)
- [*Ansible for DevOps* by Jeff Geerling - Book](https://www.ansiblefordevops.com/)
- [How do you convert ansible ini inventory into json or yaml - StackOverflow](https://stackoverflow.com/questions/57727326/how-do-you-convert-ansible-ini-inventory-into-json-or-yaml)
  - [ansible-inventory tips - dd if=/dev/brain of=/var/log/site (evrard.me)](https://evrard.me/convert-ansible-inventories-with-ansible-inventory-cli/)
