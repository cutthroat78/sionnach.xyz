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

## Note

This page is set around using Ansible in Linux and all commands, file paths and information will be based around using Ansible in Linux. I may write about using Ansible on Windows in the future or change this page to suit both using Ansible on Linux and Windows.

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

### Example of An Ansible Config File

```
[defaults]

inventory      = /etc/ansible/hosts
library        = /usr/share/my_modules/
remote_tmp     = $HOME/.ansible/tmp
local_tmp      = $HOME/.ansible/tmp
forks          = 5
poll_interval  = 15
sudo_user      = root

[selinux]
special_context_filesystems=nfs,vboxsf,fuse,ramfs

libvirt_lxc_noseclabel = yes

[colors]
highlight = white
verbose = blue
warn = bright purple
error = red
debug = dark gray
deprecate = purple
skip = cyan
unreachable = red
ok = green
changed = yellow
diff_add = green
diff_remove = red
diff_lines = cyan
```

See a more longer and more complete (default) Ansible config file here: [ansible.cfg Example - Page on RIP Tutorial](https://riptutorial.com/ansible/example/21992/ansible-cfg)

## Ansible Configuration Files (Continued)

You can change the ```/etc/ansible/ansible.cfg``` to change Ansible's behaviour when running any playbooks on the machine, but if you want to change Ansible's behaviour for only certain playbooks, you can copy the ```/etc/ansible/ansible.cfg``` file, paste it into your playbook's directory (e.g. pasting it to ```/home/user/db-playbooks/ansible.cfg```) and make changes to your newly created config file

If you want to specify a specific config file to use when running a playbook you can change the ```ANSIBLE_CONFIG``` environment variable before running your ```ansible-playbook``` command e.g. ```ANSIBLE_CONFIG=/home/user/ansible-web.cfg ansible-playbook playbook.yml```

Ansible config files don't have to have all values. You just need to include the parameters you want to override. The default values for other parameters will come from the next config file(s) in the priority chain. This means you don't have to copy an Ansible config file and change the parameters to what you need, you can just create a new file and change the parameters you want.

### Ansible's Order of What Configuration File to Use First, Second, Third, etc.

1. Parameters set using environment variables
2. The file specified VIA ```ANSIBLE_CONFIG``` environment variable
3. The file in playbook directory e.g. if playbook directory is ```/home/user/network-playbooks/```, then ansible will use the config file at ```/home/user/network-playbooks/ansible.cfg```
4. The file named ```.ansible.cfg``` that is in the user's home directory
5. The file at ```/etc/ansible/ansible.cfg```

### Using Environment Variables to Change Parameters (Instead of Config Files)

Instead of using an Ansible config file to change a parameter, you can use specify an environment variable before executing an Ansible playbook.

#### How to Specify Environment Variables

##### Specifying for The Running of a Singular Ansible Playbook

In Linux/Bash: ```ANSIBLE_GATHER=explicit ansible-playbook playbook.yml```

Remember that with this method, the environment variable will only be specified for this instance of running this playbook

##### Specify for Shell Session

In Linux/Bash:

1. ```export ANSIBLE_GATHERING=explicit```
2. ```ansible-playbook playbook.yml```

Remember that with this method, the environment variable will only be set for this user and will persist until the shell is exited

#### How to Figure Out What The Equivalent Environment Variable is for a Ansible Config File Parameter

For most options options in Ansible config files, the environment variable equivalent will be the name of the parameter (key part) in uppercase with ```ANSIBLE_``` in front of it

Remember to check the documentation or use the ```ansible-config list``` command to see if this is correct before running an Ansible playbooks. The documentation has a section with all Ansible config environment variables here: [Environment Variables Section in Page: "Ansible Configuration Settings" — Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#environment-variables)

##### Example of How to Figure Out What The Equivalent Environment Variable is for a Ansible Config File Parameter

In a Ansible config file we have:
```
gathering    = implicit
```

So, the environment variable equivalent to the above would be ```ANSIBLE_GATHERING``` and to specify it like it says above when executing an Ansible playbook would be done like this: ```ANSIBLE_GATHERING=implicit ansible-playbook playbook.yml```

We can confirm this is correct by looking at the documentation, which has a section for ```ANSIBLE_GATHERING``` here: [```ANSIBLE_GATHERING``` Section in Page: "Ansible Configuration Settings" — Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#envvar-ANSIBLE_GATHERING)

We can also confirm this by running ```ansible-config list``` and going through the output to find if ```ANSIBLE_GATHERING``` is correct. You can pipe the output to ```grep``` or ```less``` to make it easier to find, e.g. for grep: ```ansible-config list | grep ANSIBLE_GATHERING -A 10 -B 20```, for less ```ansible-config list | less``` (in less you can type ```/``` and type in ```ANSIBLE_GATHERING``` and it will take you to the section with ```ANSIBLE_GATHERING```)

### The "ansible-config" Command

#### The "ansible-config list" Command - How to See All Ansible Configuration Info

The command ```ansible-config list``` will display a list of all different configurations, their default values, their environment variable equivalents and the values you can set

#### The "ansible-config view" Command - Show The Contents of the Current Active Config File

The command ```ansible-config view``` shows the contents of the config file that is currently active

#### The "ansible-config dump" Command - Show The Current Settings and Parameters

The command ```ansible-config dump``` shows you a list of all current settings and parameters that Ansible has picked up and where it picked up each setting

##### Example Use of "ansible-config dump" Command

In Linux/Bash:

1. ```export ANSIBLE_GATHERING=explicit```
2. ```ansible-config dump | grep GATHERING```
  - Output: ```DEFAULT_GATHERING(env: ANSIBLE_GATHERING) = explicit```

## Inventory

Ansible can work with one or more systems at the same time. To be able to work with different target systems, ansible needs to connect with them. Ansible can use

- SSH for Linux Systems
- WinRM (Windows Remote Management) for Windows Systems

This method of working with different target systems is what makes ansible agentless (meaning that you don't have to install any software (agents) on target machines to be able to control/work with them).

Information about target systems is stored in an inventory file.

If you don't create a new inventory file, ansible will default to using the inventory file located at ```/etc/ansible/hosts```.

Inventory files can use either (external or internal) IP addresses or hostnames for each target system.

### Inventory Formats

Inventory files can be in an INI, [YAML]({{< ref yaml.md >}}), TOML or JSON format.

#### Why Do We Need Different Inventory Formats?

We need different inventory formats offers flexibility and allows you to group your servers based on their roles, geographical location or any other criteria that makes sense for your use case

#### INI Format

The INI format is very simple and straightforward. You could compare it to a basic organisational chart for a small startup

#### YAML Format

The [YAML]({{< ref yaml.md >}}) format is more flexible and structured than the INI format. You could compare the [YAML]({{< ref yaml.md >}}) format to a complex organisational chart for a large corporation

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

### Grouping in Inventory Files

Servers can be grouped together under a label in the inventory file and groups can be specified using the label to save time and reduce the risk of errors by allowing a user to run a playbook against a group of servers by specifying the group's label as opposed to having to specify each individual server

#### How To Group In An INI Formatted Inventory File

In INI, servers can be grouped by writing the label of the group and surrounding the group label with square brackets (```[``` and ```]```) and then specifying the servers underneath the group label

#### How To Group In An YAML Formatted Inventory File

In [YAML]({{< ref yaml.md >}}), servers can be grouped using the ```hosts:``` keyword and listing the servers underneath the ```hosts:``` keyword

#### Grouping Inventory Example

##### INI

```
server1.company.com

[web]
server2.company.com
server3.company.com

[db]
server4.company.com
server5.company.com
```

##### YAML

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

#### Parent-Child Group Relationships in Inventory Files

Inventory files can have parent groups that can contain child groups. This allows you to specify more specifically which group of servers you want to run a playbook on, whether it is a broader group of servers, which would use a parent group or a smaller group of servers, which would use a child group. Child groups of a parent group can also have their own child groups, allowing for much more flexibility and specificity when running playbooks on groups of server

One good use case for this feature would be for grouping servers by geographical locations (e.g. USA, UK, Europe, North America, etc.) inside a parent group for what type of servers they are (e.g. web, database, etc.)

##### How To Make Parent-Child Group Relationships in INI Formatted Inventory Files

In INI, you can create a parent group by specifying the parent group name and putting ```:children``` after it and surrounding that with square brackets (```[``` and ```]```) and then specifying the child groups and any ungrouped servers underneath the parent group label

##### How To Make Parent-Child Group Relationships in YAML Formatted Inventory Files

In [YAML]({{< ref yaml.md >}}), child groups can be grouped using the ```children:``` keyword and listing the child groups and any servers that don't belong to a children group underneath the ```children:``` keyword

##### Inventory with A Parent Group That Has Two Child Groups Example

###### INI

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

###### YAML

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

##### Inventory with A Parent Group That Has Two Child Groups and Each of Those Child Groups Has Their Own Two Child Groups Example

###### INI

```
[web_UK]
server1.company.com
server2.company.com

[web_USA]
server3.company.com
server4.company.com

[web:children]
web_USA
web_UK

[db_UK]
server5.company.com
server6.company.com

[db_USA]
server7.company.com
server8.company.com

[db:children]
db_USA
db_UK

[all:children]
web
db
```

###### YAML

```
all:
  children:
    db:
      children:
        db_UK:
          hosts:
            server5.company.com:
            server6.company.com:
        db_USA:
          hosts:
            server7.company.com:
            server8.company.com:
    web:
      children:
        web_UK:
          hosts:
            server1.company.com:
            server2.company.com:
        web_USA:
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
  - Example Values: ```ssh```, ```winrm```, ```localhost```
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
  - [Environment Variables Section in Page: "Ansible Configuration Settings" — Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#environment-variables)
    - [ANSIBLE_GATHERING Section in Page: “Ansible Configuration Settings” — Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#envvar-ANSIBLE_GATHERING)
- [Ansible Official Website](https://www.ansible.com/)
- [Ansible - Arch Linux Wiki](https://wiki.archlinux.org/title/Ansible)
- [Ansible in 100 Seconds by Fireship - YouTube Video](https://www.youtube.com/watch?v=xRMPKQweySE) - Good Introduction to Ansible
- [*Ansible for the Absolute Beginner - Hands-On - DevOps* - Udemy Course](https://www.udemy.com/course/learn-ansible/)
- [*Dive Into Ansible - Beginner to Expert in Ansible - DevOps* - Udemy Course](https://www.udemy.com/course/diveintoansible/)
- [*Ansible for DevOps* by Jeff Geerling - Book](https://www.ansiblefordevops.com/)
- [How do you convert ansible ini inventory into json or yaml - StackOverflow](https://stackoverflow.com/questions/57727326/how-do-you-convert-ansible-ini-inventory-into-json-or-yaml)
  - [ansible-inventory tips - dd if=/dev/brain of=/var/log/site (evrard.me)](https://evrard.me/convert-ansible-inventories-with-ansible-inventory-cli/)
- [ansible.cfg Example - Page on RIP Tutorial](https://riptutorial.com/ansible/example/21992/ansible-cfg) - Example of an Ansible config file
