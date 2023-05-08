# Hands-on Ansible-03 : Using vars, facts and secrets

Purpose of the this hands-on training is to give students the knowledge of ansible facts gathering and working with secret values.

## Learning Outcomes

At the end of this hands-on training, students will be able to;

- Explain how and what facts gathering is and how to use it in the playbook
- Learn how to deal with secret values with ansible-vault

## Outline

- Part 1 - Ansible Variables

- Part 2 - Ansible Facts

- Part 3 - Working with sensitive data


## Part 0 - Install Ansible


- Spin-up 3 Amazon Linux 2 instances and name them as:
    1. control node
    2. node1 ----> (SSH PORT 22, HTTP PORT 80)
    3. node2 ----> (SSH PORT 22, HTTP PORT 80)


- Connect to the control node via SSH and run the following commands.

```bash
sudo yum update -y
sudo amazon-linux-extras install ansible2
```

### Confirm Installation

- To confirm the successful installation of Ansible, run the following command.

```bash
$ ansible --version
```
Stdout:
```
ansible 2.9.12
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ec2-user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.18 (default, Aug 27 2020, 21:22:52) [GCC 7.3.1 20180712 (Red Hat 7.3.1-9)]
```
- Explain the lines above:
    1. Version Number of Ansible
    2. Path for the Ansible Config File
    3. Modules are searched in this order
    4. Ansible's Python Module path
    5. Ansible's executable file path
    6. Ansible's Python version with GNU Compiler Collection for Red Hat

### Configure Ansible on the Control Node

- Connect to the control node for building a basic inventory.

- Edit ```/etc/ansible/hosts``` file by appending the connection information of the remote systems to be managed.

- Along with the hands-on, public or private IPs can be used.

```bash
$ sudo su
$ cd /etc/ansible
$ ls
$ vim hosts
[webservers]
node1 ansible_host=<node1_ip> 

[dbservers]
node2 ansible_host=<node2_ip> 

[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=/home/ec2-user/<pem file>
```

- Explain what ```ansible_host```, ```ansible_user``` and ansible_ssh_key_file parameters are. For this reason visit the Ansible's [inventory parameters web site](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters).

- Explain what an ```alias``` (node1 and node2) is and where we use it.

- Explain what ```[webservers] and [all:vars]``` expressions are. Elaborate the concepts of [group name](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups), [group variables](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#assigning-a-variable-to-many-machines-group-variables) and [default groups](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#default-groups). 

- Visit the above links for helping to understand the subject. 

- Copy your pem file to the /home/ec2-user directory. First, go to your pem file directory on your local PC and run the following command.

```bash
$ scp -i <pem file> <pem file> ec2-user@<public DNS name of Control Node>:/home/ec2-user
```
- Check if the file is transferred to the remote machine. 

- As an alternative way, create a file on the control node with the same name as the <pem file> in ```/etc/ansible``` directory. 

- Then copy the content of the pem file and paste it in the newly created pem file on the control node.

- To make sure that all our hosts are reachable, we will run various ad-hoc commands that use the ping module.

```bash
$ chmod 400 <pem file>
```

```bash
$ ansible all -m ping -o
# hata alirsan once ansible dbservers -m ping -o sonra ansible all -m ping -o yaz.
```

## Part 1 - Ansible Variables

In Ansible a value is assigned to a variable that can be then referenced in a playbook or on command line during playbook runtime. Variables can be used in playbooks, inventories, and at the command line as we have just mentioned.

Variables in Playbooks

var1.yaml

```bash
---
- hosts: all
  vars:
    greetings: Hello everyone!

  tasks:
  - name: Ansible Simple Variable Example Usage
    debug:
      msg: "{{ greetings }}, Let’s learn Ansible variables"
```

var2.yaml

```bash
---
- hosts: all
  vars:
    students:
      - Alice
      - Mark
      - Peter

  tasks:
  - name: Ansible Array Usage Example
    debug:
      msg: "{{ students }}"
```

var3.yaml

```bash
---
- hosts: all
  tasks:
    - name: Create new users
      user:
        name: '{{ item }}'
        state: present

      loop:
        - mike
        - sandra
```

- variables can be assigned using external files

varsfile.yaml

```bash
  username: johnwayne
  password: wayne123
```

var4.yaml

```bash
---
- hosts: all
  vars_files:
    - ~/varsfile.yaml
  tasks:
  - name: user name and password from external var file
    debug:
      msg: "{{ username}}, {{ password }}"
```

## Part 2 - Ansible Facts

- Gathering Facts:

```bash
$ ansible node1 -m setup
```
```
ec2-34-201-69-79.compute-1.amazonaws.com | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.31.20.246"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::88c:37ff:fe8f:3b71"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "08/24/2006",
        "ansible_bios_vendor": "Xen",
        "ansible_bios_version": "4.2.amazon",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "NA",
        "ansible_board_serial": "NA",
```
- create a playbook named "facts.yml"

```yml
- name: show facts
  hosts: all
  tasks:
    - name: print facts
      debug:
        var: ansible_facts
```
- run the play book

```bash
$ ansible-playbook facts.yml
```

- create a playbook named "ipaddress.yml"

```yml
- hosts: all
  tasks:
  - name: show private IP address
    debug:
      msg: >
       This host uses private IP address {{ ansible_facts.default_ipv4.address }}

```
- run the playbook

```bash
ansible-playbook ipaddress.yml 
```

- create another yaml file for another fact

```yaml
- hosts: all
  tasks:
  - name: show architecture
    debug:
      msg: >
       This host uses {{ ansible_facts.architecture }} architecture
```






- dump the facts of a target host

```bash
ansible -m setup node1
```

- the result has 3 types of data

- dictionary
- list
- ansibleunsafetext

- dictionary 
"ansible_all_ipv4_addresses": [
            "172.31.95.236"
        ], 

- list or array
"ansible_apparmor": {
            "status": "disabled"
        }, 

- ansibleunsafetext
"ansible_architecture": "x86_64", 
"ansible_bios_date": "08/24/2006", 
"ansible_bios_version": "4.11.amazon",



- now let's create another yaml file to get those facts from the remote host

facts2.yaml

```yaml
---
- name: Ansible Variable Example Playbook
  hosts: node1
  tasks:
    # display the variable data type
    - debug:
        msg: 
          - " Data type of 'ansible_architecture'  is {{ ansible_architecture | type_debug }} "
          - " Data type of 'ansible_apparmor' is {{ ansible_apparmor | type_debug }} "
          - " Data type of 'ansible_all_ipv4_addresses' is {{ ansible_all_ipv4_addresses | type_debug }} "

    # Simply printing the value of fact which is Ansible UnSafe Text type
    - debug:
        msg: "{{ ansible_architecture }}"


    # Accessing an element of dictionary
    - debug:
        msg: "{{ansible_apparmor.status}}"

    # Accessing the list
    - debug:
        msg: "{{ansible_all_ipv4_addresses}}"

    # Accessing the Second Element of the list
    - debug:
        msg: "{{ansible_all_ipv4_addresses[0]}}"

```


- this exercise shows how to parse through a variable dictionary and how to run a command on a specific host

- parse.yaml

```yaml
---
- name: Ansible Variable Example Playbook
  hosts: node1
  tasks:

- name: Ansible Variable Example Playbook
  hosts: node1
  tasks:

    # Print the Dictionary
    - debug:
        msg: "{{ansible_mounts}}"

    # Parsing through Variable Dictionary
    - debug:
        msg: "Mount Point {{item.mount}} is at {{item.block_used/item.block_total*100}} percent "
      loop: "{{ansible_mounts}}"

    # Execute Host based task using variable
    - name: Execute the command only node2 server
      become: yes
      become_user: root 
      shell: "uname -a"
      when: "ansible_facts['os_family'] == 'RedHat'"
```


## Part 3 - Working with sensitive data

- Create encypted variables using "ansible-vault" command

```bash
ansible-vault create secret.yml
```

New Vault password: xxxx
Confirm New Vault password: xxxx

```yml
username: tony
password: 123456a
```

- look at the content

```bash
$ cat secret.yml
```
```
33663233353162643530353634323061613431366332373334373066353263353864643630656338
6165373734333563393162333762386132333665353863610a303130346362343038646139613632
62633438623265656330326435646366363137373333613463313138333765366466663934646436
3833386437376337650a636339313535323264626365303031366534363039383935333133306264
61303433636266636331633734626336643466643735623135633361656131316463
```
- how to use it:

- create a file named "create-user"

```bash
$ nano create-user.yml

```

```yml
- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret.yml
  tasks:
    - name: creating user
      user:
        name: "{{ username }}"
        password: "{{ password }}" # | password_hash('sha512')
```

- run the plaaybook

```bash
ansible-playbook create-user.yml
```
```bash
ERROR! Attempting to decrypt but no vault secrets found
```
- Run the playbook with "--ask-vault-pass" command:

```bash
$ ansible-playbook --ask-vault-pass create-user.yml
```
Vault password: xxxx

```
PLAY RECAP ******************************************************************************************
node1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node2                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- To verify it

```bash
ansible all -b -m command -a "grep tony /etc/shadow"
```
```
node1 | CHANGED | rc=0 >>
tony:99abcd:18569:0:99999:7:::
```

- Create another encypted variables using "ansible-vault" command but this time use SHA (Secure Hash Algorithm) for your password:

```bash
ansible-vault create secret-1.yml
```

New Vault password: xxxxx
Confirm Nev Vault password: xxxxx

```yml
username: Erling
pwhash: 06dmpnr
```

- look at the content

```bash
$ cat secret-1.yml
```
```
33663233353162643530353634323061613431366332373334373066353263353864643630656338
6165373734333563393162333762386132333665353863610a303130346362343038646139613632
62633438623265656330326435646366363137373333613463313138333765366466663934646436
3833386437376337650a636339313535323264626365303031366534363039383935333133306264
61303433636266636331633734626336643466643735623135633361656131316463
```
- how to use it:

- create a file named "create-user-1"

```bash
$ nano create-user-1.yml

```

```yml
- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret-1.yml
  tasks:
    - name: creating user
      user:
        name: "{{ username }}"
        password: "{{ pwhash | password_hash ('sha512') }}"     
``` 

- run the plaaybook


```bash
$ ansible-playbook --ask-vault-pass create-user-1.yml
```
Vault password: xxxxx

```
PLAY RECAP ******************************************************************************************
node1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node2                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- to verrify it:

```bash
ansible all -b -m command -a "grep Erling /etc/shadow"
```
```
node1 | CHANGED | rc=0 >>
tyler:#665fffgkg6&fkg689##2£6466?%^^+&%+:18569:0:99999:7:::
```
