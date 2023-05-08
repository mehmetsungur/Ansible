# Hands-on Ansible-05 : Using Roles and Galaxy
The purpose of this hands-on training is to give students knowledge of basic Ansible skills.

## Learning Outcomes

At the end of this hands-on training, students will be able to;

- Explain what is Ansible role

- Baskalari veya kendimiz tarafindan olusturulmus Playbook'lari kullanma durumuna Ansible  Roles diyoruz.  
- Role'ler default olarak  `etc/ansible` in altindaki `roles` klasorunun altina iniyor. Custom bir directory olusturayim role'ler orada depolansin derseniz olusturdugunuz directory'de `tasks`, `vars`, `defaults`(her yerde kullanacagimiz default degiskenler'i tutar), `handlers`(tetikleyici task'lari burada tutariz: su durum gerceklesirse sunu yap durumundaki "su durum" bir tetikleyici aslinda) ve `templates`(template dosyalari tutar keypem'ler gibi) dosya & klasorleri olmali.

- Collection'lar birden fazla role iceren yapilar. 

- Kendimiz role olusturmak icin Ansible Galaxy komutlarini kullaniyoruz. Ansible Galaxy de Docker Hub gibi role paylasmak icin kurulmus ve kullanilan bir websitesi. 

- Ansible Role'lerini terminal'den de aramak mumkun. Mesela nginx icin

```bash
ansible-galaxy search nginx
```

## Outline

- Part 1 - Install Ansible

- Part 2 - Using Ansible Roles 

- Part 3 - Using Ansible Roles from Ansible Galaxy



## Part 1 - Install Ansible


- Spin-up 3 Amazon Linux 2 instances and name them as:
    1. control node -->(SSH PORT 22)(Amazon Linux 2)
    2. web_server_1 ----> (SSH PORT 22, HTTP PORT 80)(Red Hat)
    3. web_server_2 ----> (SSH PORT 22, HTTP PORT 80)(Ubuntu)


- Connect to the control node via SSH and run the following commands.

- Run the commands below to install Python3 and Ansible. 

```bash
$ sudo yum install -y python3 
```

```bash
$ pip3 install --user ansible
```

- Check Ansible's installation with the command below.

```bash
$ ansible --version
```


- Run the command below to transfer your pem key to your Ansible Controller Node.

```bash
scp -i deneme.pem deneme.pem ec2-user@<Control Node IP>:/home/ec2-user
```


- Make a directory named ```working-with-roles``` under the home directory and cd into it.

```bash 
$ mkdir working-with-roles
$ cd working-with-roles
```

- Create a file named ```inventory.txt``` with the command below.

```bash
$ vi inventory.txt
```

- Paste the content below into the inventory.txt file.

- Along with the hands-on, public or private IPs can be used.

```txt
[servers]
web_server_1   ansible_host=54.197.32.221   ansible_user=ec2-user  
web_server_2  ansible_host=54.160.149.180  ansible_user=ubuntu

[servers:vars]
ansible_ssh_private_key_file=~/deneme.pem
```
- Create file named ```ansible.cfg``` under the the ```working-with-roles``` directory.

```cfg
[defaults]
host_key_checking = False
inventory=inventory.txt
interpreter_python=auto_silent
roles_path = /home/ec2-user/ansible/roles/
deprecation_warnings=False
```


- Create a file named ```ping-playbook.yml``` and paste the content below.

```bash
$ touch ping-playbook.yml
```

```yml
- name: ping them all
  hosts: all
  tasks:
    - name: pinging
      ping:
```

- Run the command below for pinging the servers.

```bash
$ chmod 400 /home/ec2-user/deneme.pem
$ ansible-playbook ping-playbook.yml
```

- Explain the output of the above command.



## Part 2 - Using Ansible Roles

- Install apache server and restart it with using Ansible roles.

```bash
# Role directory'si olusturuyoruz
ansible-galaxy init /home/ec2-user/ansible/roles/apache

# Olusturdugumuz directory'ye girelim, tree komutunu yükleyelim ve klasör yapısına tree ile göz atalım
cd /home/ec2-user/ansible/roles/apache
ll
sudo yum install tree
tree
```

- Create tasks/main.yml with the following.

vi tasks/main.yml

```yml
- name: installing apache
  yum:
    name: httpd
    state: latest

- name: index.html
  copy:
    content: "<h1>Hello World!</h1>"
    dest: /var/www/html/index.html

- name: restart apache2
  service:
    name: httpd
    state: restarted
    enabled: yes
```

- Create a playbook named "role1.yml".

```bash
cd /home/ec2-user/working-with-roles/
nano role1.yml
```

```yml
---
- name: Install and Start apache
  hosts: web_server_1
  become: yes
  roles:
    - apache
```

- Run the command below 
```bash
$ ansible-playbook role1.yml
```
- Check the web_server_1 public IP to view the web page

- (if you will use same server, uninstall the apache)


## Part 3 - Using Ansible Roles from Ansible Galaxy

- Go to Ansible Galaxy web site (https://galaxy.ansible.com/search?deprecated=false&keywords=nginx%20geerlingguy&order_by=-relevance&page=1)

- Click the Search option

- Write nginx

- Go to command line and write:

```bash
$ ansible-galaxy search nginx
```

Stdout:
```
Found 1687 roles matching your search. Showing first 1000.

 Name                                                         Description
 ----                                                         -----------
 0x0i.prometheus                                              Prometheus - a multi-dimensional time-series data mon
 0x5a17ed.ansible_role_netbox                                 Installs and configures NetBox, a DCIM suite, in a pr
 1davidmichael.ansible-role-nginx                             Nginx installation for Linux, FreeBSD and OpenBSD.
 1it.sudo                                                     Ansible role for managing sudoers
 1mr.zabbix_host                                              configure host zabbix settings
 1nfinitum.php                                                PHP installation role.
 2goobers.jellyfin                                            Install Jellyfin on Debian.
 2kloc.trellis-monit                                          Install and configure Monit service in Trellis.
 ```


 - there are lots of roles. Lets filter them.

 ```bash
 $ ansible-galaxy search nginx --platform EL
```
- **`EL`** is for centos 

- Lets go more specific :

```bash
$ ansible-galaxy search nginx --platform EL | grep geerl

Stdout:
```
geerlingguy.nginx                                            Nginx installation for Linux, FreeBSD and OpenBSD.
geerlingguy.php                                              PHP for RedHat/CentOS/Fedora/Debian/Ubuntu.
geerlingguy.pimpmylog                                        Pimp my Log installation for Linux
geerlingguy.varnish                                          Varnish for Linux.

```
- Install it:

$ ansible-galaxy install geerlingguy.nginx

Stdout:(CIKTI BURASI)
```
- downloading role 'nginx', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-nginx/archive/2.8.0.tar.gz
- extracting geerlingguy.nginx to /home/ec2-user/.ansible/roles/geerlingguy.nginx
- geerlingguy.nginx (2.8.0) was installed successfully
```

- Inspect the role:

$ cd /home/ec2-user/ansible/roles/geerlingguy.nginx

$ ls
defaults  handlers  LICENSE  meta  molecule  README.md  tasks  templates  vars

$ cd tasks
$ ls

main.yml             setup-Debian.yml   setup-OpenBSD.yml  setup-Ubuntu.yml
setup-Archlinux.yml  setup-FreeBSD.yml  setup-RedHat.yml   vhosts.yml

$ vi main.yml

```yml
---
# Variable setup.
- name: Include OS-specific variables.
  include_vars: "{{ ansible_os_family }}.yml"

- name: Define nginx_user.
  set_fact:
    nginx_user: "{{ __nginx_user }}"
  when: nginx_user is not defined

# Setup/install tasks.
- include_tasks: setup-RedHat.yml
  when: ansible_os_family == 'RedHat'

- include_tasks: setup-Ubuntu.yml
  when: ansible_distribution == 'Ubuntu'

- include_tasks: setup-Debian.yml
  when: ansible_os_family == 'Debian'

- include_tasks: setup-FreeBSD.yml
  when: ansible_os_family == 'FreeBSD'

- include_tasks: setup-OpenBSD.yml
  when: ansible_os_family == 'OpenBSD'

- include_tasks: setup-Archlinux.yml
  when: ansible_os_family == 'Archlinux'

# Vhost configuration.
- import_tasks: vhosts.yml

# Nginx setup.
- name: Copy nginx configuration in place.
  template:
    src: "{{ nginx_conf_template }}"
    dest: "{{ nginx_conf_file_path }}"
    owner: root
    group: "{{ root_group }}"
    mode: 0644
  notify:
    - reload nginx
```

- # use it in playbook:

- Create a playbook named "playbook-nginx.yml"

```yml
- name: use galaxy nginx role
  hosts: web_server_2
  become: true
  roles:
    - role: geerlingguy.nginx
```

- Run the playbook.

```bash
$ ansible-playbook playbook-nginx.yml
$ ansible-playbook playbook-nginx.yml -v # duruma gore
```
- Ubuntu makinasinin IP sine baglan ve `Welcome to NGINX` mesajini gor.

- List the roles you have:

```bash
$ ansible-galaxy list
```

Stdout:
```
# /home/ec2-user/ansible/roles
- apache, (unknown version)
- geerlingguy.nginx, 3.1.3
```

# Ansible Role'u Kullanarak EC2 Instance Image'i Olusturma

* As a company we need to create an instance image. At this image we want to use some software such as docker and prometheus node exporter. So every instance will be created with this instance image. We are also planning to update this image every 6 months. So we can update docker and prometheus software versions later 6 months. We need to have re-usable configs to do that. Lets talk about this situation.

* First create a new instance with **Ubuntu 20.04 instance image**

* Olusturdugun makinenin IP sini `inventory.txt` ye ekle:

```txt
[servers]
web_server_1   ansible_host=54.197.32.221   ansible_user=ec2-user  
web_server_2  ansible_host=54.160.149.180  ansible_user=ubuntu
instance_image_2023 ansible_host=172.31.88.66 ansible_user=ubuntu

[servers:vars]
ansible_ssh_private_key_file=~/deneme.pem
```


* We will create declarative file to download ansible roles, lets start!

* First install git with:
```bash
sudo yum install git
```

* Create a file whose name is `role_requirements.yml` under the `/home/ec2-user/working-with-roles`:

```
- src: git+https://github.com/geerlingguy/ansible-role-docker
  name: docker
  version: 2.9.0

- src: git+https://github.com/geerlingguy/ansible-role-ntp
  version: 2.1.0
  name: ansible-role-ntp

- src: git+https://github.com/UnderGreen/ansible-prometheus-node-exporter
  version: master
```

* We will use prometheus node exporter at next session to monitor our intances, and NTP is Network Time Protocol. [For more information](https://en.wikipedia.org/wiki/Network_Time_Protocol)

- Network Time Protocol (NTP) is a protocol that allows the synchronization of system clocks (from desktops to servers). Having synchronized clocks is not only convenient but required for many distributed applications.

Then run this command:

```bash
ansible-galaxy install -r role_requirements.yml
```

* Check all the roles are created.

```bash
$ ansible-galaxy list
```
```output
# /home/ec2-user/ansible/roles
- apache, (unknown version)
- geerlingguy.nginx, 3.1.3
- docker, 2.9.0
- ansible-role-ntp, 2.1.0
- ansible-prometheus-node-exporter, master
- UnderGreen.prometheus-exporters-common, v1.2.0
```



* `working-with-roles` un altina `common.yml` isimli yaml dosyasini olustur:

```bash
# working-with-roles in altindasin
nano common.yml
```
* Then create a playbook file to create instance image.

```yml
---
-
  hosts: instance_image_2023
  become: yes
  become_method: sudo  

  roles:
    - common
    - { role: ansible-role-ntp, ntp_timezone: UTC }
    - docker
    - ansible-prometheus-node-exporter

```
* Additionally create a role named with common:

```
ansible-galaxy init /home/ec2-user/ansible/roles/common
```

* To apply this first you need to configure your Inventory file, so add your instance private ip address to instance_image.

* Go and customize common role from `ansible/roles/common/tasks/main.yml`

```yml
---
# tasks file for /home/ec2-user/ansible/roles/common/tasks/main.yml:
- name: Common Tasks
  debug:
    msg: Common Task Triggered

- name: Fix dpkg
  command: dpkg --configure -a

- name: Update apt
  apt:
    upgrade: dist
    update_cache: yes

- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - ntp
```
* Run the `common.yml`

```bash
ansible-playbook common.yml
```

- Finally docker, prometheus and NTP server are installed on our last Ubuntu 20.04 instance

_ Using this instance you can create an image.


* Check the latest host whether Docker is installed.


