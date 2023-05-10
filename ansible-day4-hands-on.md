
# Hands-on Ansible-04: Provisioning a Web Server and a Database Server with a Dynamic Website Using Ansible

The purpose of this hands-on training is to give students the knowledge of provisioning a web and database server with a dynamic website.

## Learning Outcomes

At the end of this hands-on training, students will be able to;

- Explain how to provision a web server using Ansible
- Explain how to provision a database server using Ansible


## Part 1 - Build the Infrastructure

- Get to the AWS Console and spin-up 3 EC2 Instances with Amazon Linux 2 AMI.

- Configure the security groups as shown below:

    - Controller Node ----> Port 22 SSH

    - Target Node1 -------> Port 22 SSH, Port 80, 3306 MYSQL/Aurora

    - Target Node2 -------> Port 22 SSH, Port 80 HTTP

## Part 2 - Install Ansible on the Controller Node

- Connect to your ```Controller Node```.

- update packages and install Ansible

```bash
sudo yum update -y
sudo amazon-linux-extras install -y ansible2
```

- Check Ansible's installation with the command below.

```bash
$ ansible --version
```

## Part 3 - Pinging the Target Nodes

- Run the command below to transfer your pem key to your Ansible Controller Node.

```bash
$ scp -i <PATH-TO-PEM-FILE> <PATH-TO-PEM-FILE> ec2-user@<CONTROLLER-NODE-IP>:/home/ec2-user

$ chmod 400 /home/ec2-user/key.pem
```

- Make a directory named ```Ansible-Website-Project``` under the home directory and cd into it.

```bash 
$ mkdir Ansible-Website-Project
$ cd Ansible-Website-Project
```

- Create a file named ```inventory``` with the command below.

```bash
$ nano inventory
```

- Paste the content below into the inventory file.

- Along with the hands-on, public or private IPs can be used. 

```txt
[servers]
db_server   ansible_host=35.181.155.102    
web_server  ansible_host=13.38.120.143

[servers:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=/home/ec2-user/key.pem
```

- Create a file named ```ping-playbook.yml``` and paste the content below.

```bash
$ nano ping-playbook.yml
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
$ ansible-playbook ping-playbook.yml -i inventory
```

- Create another file named ```ansible.cfg``` in the project directory.

```
[defaults]
host_key_checking = False
inventory = inventory
deprecation_warnings=False
interpreter_python=auto_silent
```

- Run the command below again.

```bash
$ chmod 400 /home/ec2-user/tevfikkey.pem
$ ansible-playbook ping-playbook.yml
$ ansible all -m ping
```

## Part4 - Install, Start and Enable MariaDB

- Create another file named ```playbook.yml``` under the ```Ansible-Website-Project``` directory.

- Paste the content below into the ```playbook.yml``` file.

```yml
- name: db configuration
  hosts: db_server
  tasks:
    - name: install mariadb and PyMySQL
      become: yes
      yum:
        name: 
            - mariadb-server
            - MySQL-python
        state: latest

    - name: start mariadb
      become: yes  
      command: systemctl start mariadb

    - name: enable mariadb
      become: yes
      systemd: 
        name: mariadb
        enabled: true
```

- Run the playbook.yaml

```bash
ansible-playbook playbook.yml
```

- Open up a new Terminal or Window and connect to the ```DBServer``` instance and check if ```MariaDB``` is installed, started, and enabled. 

```bash
mysql --version
```

- Go back to control machine window

- Create a file named ```db-load-script.sql``` under CONTROL MACHINE "project" folder, copy and paste the content below.

```bash
vi db-load-script.sql
```

- Here with this sql script we will create table and items inside it for the database

```txt
USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");
```

- Append the content below into ```playbook.yml``` file for transferring the sql script into database server.

```yml
    - name: copy the sql script
      copy:
        src: ~/project/db-load-script.sql
        dest: ~/
```

- Run the command below.

```bash
$ ansible-playbook playbook.yml
```
- Check if the file has been sent to the database server.

- Append the content below into ```playbook.yml``` file in order to login to db_server and set a root password.

```yml
    - name: Create password for the root user
          mysql_user:
            login_password: ''
            login_user: root
            name: root
            password: "Techpro1234"
```

- Create a file named ```.my.cnf``` under **project directory on controller node.**

- Paste the content below.

```conf
[client]
user=root
password=Techpro1234

[mysqld]
wait_timeout=30000
interactive_timeout=30000
bind-address=0.0.0.0
```

- Append the content below into ```playbook.yml``` file.

```yml
    - name: copy the .my.cnf file
          copy:
            src: ~/project/.my.cnf
            dest: ~/
```


- Kodun son hali:

```yml
- name: db configuration
  hosts: db_server
  tasks:
    - name: install mariadb and PyMySQL
      become: yes
      yum:
        name: 
            - mariadb-server
            - MySQL-python
        state: latest

    - name: start mariadb
      become: yes  
      command: systemctl start mariadb

    - name: enable mariadb
      become: yes
      systemd: 
        name: mariadb
        enabled: true

    - name: copy the sql script
      copy:
        src: ~/project/db-load-script.sql
        dest: ~/

    - name: Create password for the root user
      mysql_user:
        login_password: ''
        login_user: root
        name: root
        password: "Techpro1234"

    - name: copy the .my.cnf file
      copy:
        src: ~/project/.my.cnf
        dest: ~/

```

## Part 6 - Install, Start and Enable Apache Web Server and Other Dependencies

- Append the content below into the ```playbook.yml``` file 

```yml
    - name: web server configuration
      hosts: web_server
      become: yes
      tasks: 
        - name: install the latest version of Git, Apache, Php, Php-Mysqlnd
          package:
            name: 
              - git
              - httpd
              - php
              - php-mysqlnd
            state: latest
```

- Append the content below into the ```playbook.yml``` file.

```yml
    - name: start the server and enable it
      service:
        name: httpd
        state: started
        enabled: yes
```



## Part 7 - Pull the Code and Make Necessary Changes

- Append the content below into the ```playbook.yml``` file.

```yml
    - name: clone the repo of the website
      shell: |
        if [ -z "$(ls -al /var/www/html | grep .git)" ]; then
          git clone https://github.com/mefekax/learning-app-ecommerce.git /var/www/html/
          echo "ok"
        else
          echo "already cloned..."
        fi
      register: result

    - name: DEBUG
      debug:
        var: result
```
- run the playbook

- Now go to your web_server machine and cd to /var/www/html, and then edit index.php

- find the line 

<?php

                        $link = mysqli_connect('172.31.87.105', 'ecomuser', 'ecompassword', 'ecomdb');

- change ip address to db_server private ip save and quit

- now go to db_server machine

- let's create a database, a user and a password

```bash
mysql
```

MariaDB > CREATE DATABASE ecomdb;
MariaDB > CREATE USER 'ecomuser'@'web_serverIP' IDENTIFIED BY 'ecompassword';
MariaDB > GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'web_serverIP';
MariaDB > FLUSH PRIVILEGES;

- now, run the sql script

```bash
mysql < db-load-script.sql
```

- restart mariadb and apache services

- Finally check the web_server Public IP on the borsewr to see the application running

- Congratulations!!

- Final version of the Project playbook code


```yaml
- name: db configuration
  hosts: db_server
  tasks:
    - name: install mariadb and PyMySQL
      become: yes
      yum:
        name: 
            - mariadb-server
            - MySQL-python
        state: latest

    - name: start mariadb
      become: yes  
      command: systemctl start mariadb

    - name: enable mariadb
      become: yes
      systemd: 
        name: mariadb
        enabled: true

    - name: copy the sql script
      copy:
        src: ~/project/db-load-script.sql
        dest: ~/
    - name: Create password for the root user
      mysql_user:
        login_password: ''
        login_user: root
        name: root
        password: "techpro1234"
    - name: copy the .my.cnf file
      copy:
        src: ~/project/.my.cnf
        dest: ~/

- name: web server configuration
  hosts: web_server
  become: yes
  tasks: 
    - name: install the latest version of Git, Apache, Php, Php-Mysqlnd
      package:
        name: 
          - git
          - httpd
          - php
          - php-mysqlnd
        state: latest
    - name: start the server and enable it
      service:
        name: httpd
        state: started
        enabled: yes
    - name: clone the repo of the website
      shell: |
        if [ -z "$(ls -al /var/www/html | grep .git)" ]; then
          git clone https://github.com/mefekax/learning-app-ecommerce.git /var/www/html/
          echo "ok"
        else
          echo "already cloned..."
        fi
      register: result

    - name: DEBUG
      debug:
        var: result
```
