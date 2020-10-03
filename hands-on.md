#Spin up the instance with cloudformation templates and adjust to these configurations
```bash
Control-node - AmazonLinux2 Ports: 22
Node1 - AmazonLinux2 Ports: 22, 80
Node2 - AmazonLinux2 Ports: 22, 80
Node3 - Ubuntu Ports: 22, 80
```
Useful Resource:
https://docs.ansible.com/ansible/latest/index.html

```bash
#Login into your control node
ssh -i pem ec2-user@IP
sudo yum update -y
#install the latest ansible version
sudo amazon-linux-extras install ansible2
#get the binary directory
which ansible
#check version
ansible --version
cd /etc/ansible
ll
less ansible.cfg
less hosts

#create our host
sudo vi /etc/ansible/hosts
#add this code into [webservers] - grouping - we're adding alias to the group
[webservers]
node1 ansible_host=ec2-107-21-75-92.compute-1.amazonaws.com ansible_user=ec2-user
#alias name \ host = IP add \ ec2-user
node2 ansible__host=ec2-54-92-219-164.compute-1.amazonaws.com ansible_user=ec2-user
[all:vars]
#tell ansible to supply private key in order to login
ansible_ssh_private_key_file=/home/ec2-user/<pem file>

#exit control nodes and copy the pem key
scp -i qhn.pem qhn.pem ec2-user@ec2-54-160-203-71.compute-1.amazonaws.com:/home/ec2-user/.ssh/
#ssh back into control node
cd .ssh/
ll
chmod 400 qhn.pem
pwd
#change the path of the pem key into hosts (hosts = managed nodes)
sudo vi /etc/ansible/hosts
#list the group of nodes
ansible all --list-hosts
ansible webservers --list-hosts
#ping your server
ansible all -m ping
ansible webservers -m ping
ansible node1 -m ping
#condense output for easy to read
ansible all -m ping -o
#pull up Ansile Docs
ansible-doc ping

#Fix this error
#[WARNING]: Platform linux on host node1 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
sudo vi /etc/ansible/ansible.cfg
    [defaults]
    interpreter_python=auto_silent
#check the uptime of the group
ansible webservers -a "uptime"
```

Resource: https://docs.ansible.com/

Continue...

1. Start up the instances: Control Node, Node1, Node2, Node3 (NOTE the hostname will be different from last class)
2. Log into Control Node and update host file: sudo vi /etc/ansible/hosts
3. In the host file update host file with the new hostname
```bash
#update the new hostname
sudo vi /ect/ansible/hosts

ansible webservers -a "uptime"
ansible webservers -m shell -a "systemctl status sshd"
ansible webservers -m command -a 'df -h'
sudo chown ec2-user:ec2-user us-east-1.pem
ansible webservers -a 'df -h'

vi testfile
    "This is a test file"
/home/ec2-user/.ssh/
ansible webservers -m copy -a "src=/etc/ansible/testfile dest=/home/ec2-user/testfile"
ansible webservers -m copy -a "src=/home/ec2-user/testfile dest=/home/ec2-user/testfile"

ansible node1 -m shell -a "echo Hello Clarusway > /home/ec2-user/testfile2 ; cat testfile2"

ansible webservers -m copy -a "src=/home/ec2-user/testfile dest=/home/ec2-user/testfile"

ansible node1 -m shell -a "echo Hello Clarusway > /home/ec2-user/testfile2 ; cat testfile2"

sudo vi /etc/ansible/ansible.cfg
[defaults]
interpreter_python=auto_silen

[ubuntuserver]
node3 ansible_host=<node3_ip> ansible_user=ubuntu

ansible all --list-hosts
ansible all -m ping -o

ansible all -m shell -a "echo Hello Clarusway > /home/ubuntu/testfile3"

ansible node3 -m shell -a "echo Hello Clarusway > /home/ubuntu/testfile3"

ansible node1:node2 -m shell -a "echo Hello Clarusway > /home/ec2-user/testfile3"

ansible node1 -b -m shell -a "amazon-linux-extras install -y nginx1 ; systemctl start nginx ; systemctl enable nginx"

ansible node3 -b -m shell -a "apt update -y ; apt-get install -y nginx ; systemctl start nginx; systemctl enable nginx"
###Can you see both servers?


ansible webservers -b -m shell -a "yum -y remove nginx"
ansible webservers -b -m yum -a "name=nginx state=present"

ansible all -b -m package -a "name=nginx state=present"

ansible all -b -m shell -a "nginx -v"


```
Resource: https://docs.ansible.com/ansible/2.3/list_of_all_modules.html

### Playbook
```bash
vi playbook1.yml

---
- name: Test Connectivity
  hosts: all
  tasks:
   - name: Ping test
     ping:

ansible-playbook playbook1.yml

vi playbook2.yml

---
- name: Copy for linux
  hosts: webservers
  tasks:
   - name: Copy your file to the webservers
     copy:
       src: /home/ec2-user/testfile1
       dest: /home/ec2-user/testfile1
- name: Copy for ubuntu
  hosts: ubuntuservers
  tasks:
   - name: Copy your file to the ubuntuservers
     copy:
       src: /home/ec2-user/testfile1
       dest: /home/ubuntu/testfile1

mv testfile testfile1


### create new file
---
- name: Copy for linux
  hosts: webservers
  tasks:
   - name: Copy your file to the webservers
     copy:
       src: /home/ec2-user/testfile1
       dest: /home/ec2-user/testfile1
       mode: u+rw,g-wx,o-rwx
- name: Copy for ubuntu
  hosts: ubuntuservers
  tasks:
   - name: Copy your file to the ubuntuservers
     copy:
       src: /home/ec2-user/testfile1
       dest: /home/ubuntu/testfile1
       mode: u+rw,g-wx,o-rwx
- name: Copy for node1
  hosts: node1
  tasks:
   - name: Copy using inline content
     copy:
       content: '# This file was moved to /etc/ansible/testfile1'
       dest: /home/ec2-user/testfile2
   - name: Create a new text file
     shell: "echo Hello World > /home/ec2-user/testfile3"



---
- name: Apache installation for webservers
  hosts: webservers
  tasks:
   - name: install the latest version of Apache
     yum:
       name: httpd
       state: latest
   - name: start Apache
     shell: "service httpd start"
- name: Apache installation for ubuntuserver
  hosts: ubuntuserver
  tasks:
   - name: install the latest version of Apache
     apt:
       name: apache2
       state: latest


vi playbook4_remove.yml

---
- name: Remove Apache from webservers
  hosts: webservers
  tasks:
   - name: Remove Apache
     yum:
       name: httpd
       state: absent
- name: Remove Apache from ubuntuserver
  hosts: ubuntuserver
  tasks:
   - name: Remove Apache
     apt:
       name: apache2
       state: absent
   - name: Remove unwanted Apache2 packages from the system
     apt:
       autoremove: yes
       purge: yes

ansible-playbook -b playbook4_remove.yml

---
- name: play 4
  hosts: ubuntuservers
  tasks:
   - name: installing apache
     apt:
       name: apache2
       state: latest
   - name: index.html
     copy:
       content: "<h1>Hello Clarusway</h1>"
       dest: /var/www/html/index.html
   - name: restart apache2
     service:
       name: apache2
       state: restarted
       enabled: yes
- name: play 5
  hosts: webservers
  tasks:
    - name: installing httpd and wget
      yum:
        pkg: "{{ item }}"
        state: present
      with_items:
        - httpd
        - wget

ansible node3 -m shell -a "systemctl status apache2.service"

ansible node3 -m shell -a "sudo systemctl stop nginx"

---
- name: play 4
  hosts: ubuntuserver
  become: yes
  tasks:
   - name: installing apache
     apt:
       name: apache2
       state: latest
   - name: index.html
     copy:
       content: "<h1>Hello Clarusway</h1>"
       dest: /var/www/html/index.html
   - name: restart apache2
     service:
       name: apache2
       state: restarted
       enabled: yes
- name: play 5
  hosts: webservers
  become: yes
  tasks:
    - name: installing httpd and wget
      yum:
        pkg: "{{ item }}"
        state: present
      with_items:
        - httpd
        - wget

```

###  Continue...
```bash
#Update the hostname everytime you restart the ec2
sudo vi /etc/ansible/hosts

sudo vi /etc/ansible/hosts

ansible-playbook playbook1.yml

#Clean up
systemctl status nginx
systemctl stop nginx
apt-get remove nginx

pkg: ['httpd', 'wget']

#Create playbook5.yml
vi playbook5.yml

---
- name: play 4
  hosts: ubuntuserver
  become: yes
  tasks:
   - name: installing apache
     apt:
       name: apache2
       state: latest
   - name: index.html
     copy:
       content: "<h1>Hello Clarusway</h1>"
       dest: /var/www/html/index.html
   - name: restart apache2
     service:
       name: apache2
       state: restarted
       enabled: yes
- name: play 5
  hosts: webservers
  become: yes
  tasks:
    - name: installing httpd and wget
      yum:
        pkg: ['httpd', 'wget']
        state: present

#Create playbook6.yml
vi playbook6.yml

---
- name: play 6
  hosts: ubuntuservers
  tasks:
   - name: Uninstalling Apache
     apt:
       name: apache2
       state: absent
       update_cache: yes
   - name: Remove unwanted Apache2 packages
     apt:
       autoremove: yes
       purge: yes
- name: play 7
  hosts: webservers
  tasks:
   - name: removing apache and wget
     yum:
       pkg: ['httpd', 'wget']
       state: absent

#
ansible-playbook -b playbook5_remove.yml

---
- name: Create users
  hosts: "*"
  tasks:
    - user:
        name: "{{ item }}"
        state: present
      loop:
        - joe
        - matt
        - james
        - oliver
      when: ansible_os_family == "RedHat"
    - user:
        name: "{{ item }}"
        state: present
      loop:
        - david
        - tyler
      when: ansible_os_family == "SUSE"
    - user:
        name: "{{ item }}"
        state: present
      loop:
        - john
        - aaron
      when: ansible_os_family == "Debian" or ansible_os_family == "20.04"

ansible-playbook -b playbook6.yml

#Role in Ansible is to give specific permission to do certain task or job

#In a real world, we need to build a directory structure in Ansible. There's a recommend way to create file structure (not require), but use mostly

#Create a role in the Path of apache
ansible-galaxy init /home/ec2-user/roles/apache
#install tree for us better view of our structure
sudo yum install tree -y

tree roles/

cd roles/
cd apache/
ll
tree
  #We will see main.yml, this is where our code will be in
  #handlers - set trigger - look for changes in a specific file
  #README - instructions
  #tasks - jobs
  #template - 
  #vars - variables go here

cd tasks/
#pwd
#/home/ec2-user/roles/apache/tasks
ll
vi main.yml

- name: installing apache
  apt:
    name: apache2
    state: latest
- name: index.html
  copy:
    content: "<h1>Hello Clarusway</h1>"
    dest: /var/www/html/index.html
- name: restart apache2
  service:
    name: apache2
    state: restarted
    enabled: yes

cd
#read playbook.yml to see we will change the task section in the playbook
cat playbook.yml

cd -
vi main.yml

#go to apache folder to look for the handlers/main.yml
cd ..

vi handlers/main.yml

- name: restart apache
  service: name=apache2 state=restarted
#go back to home folder /home/ec2-user
cd ../..
pwd

vi playbook7.yml

---
- name: Install and Start Apache
  hosts: ubuntuserver
  roles:
    - apache

less roles/apache/tasks/main.yml
#run playbook
ansible-playbook -b playbook7.yml

#Role is becoming a collection of tasks

#This is a different way to create inventory file
mkdir Ansible-Website-Project
cd Ansible-Website-Project/

vi inventory.txt

#get the node1 hostname = db_server and node2 hostname = web_server
[servers]
db_server   ansible_host=<YOUR-DB-SERVER-IP>   ansible_user=ec2-user  ansible_ssh_private_key_file=~/<YOUR-PEM-FILE>
web_server  ansible_host=<YOUR-WEB-SERVER-IP>  ansible_user=ec2-user  ansible_ssh_private_key_file=~/<YOUR-PEM-FILE>

#Spin up 2 new instance with Redhat Enterprise Linux 8
db_server
webserver

#Securitry Group
Allow SSH
MYSQL/Aurora 3306

Target DB_server -------> Port 22 SSH, Port 3306 MYSQL/Aurora
Target Webserver -------> Port 22 SSH, Port 80 HTTP

#In Cloudformation, remove node1 node2 and node3, and keep the control-node

vi ping-playbook.yml

- name: ping them all
  hosts: all
  tasks:
    - name: pinging
      ping:

ansible-playbook ping-playbook.yml -i inventory.txt

touch ansible.cfg
[defaults]
host_key_checking = False

vi playbook.yml

- name: db configuration
  hosts: db_server
  tasks:
    - name: install mariadb and PyMySQL
      become: yes
      yum:
        name: 
            - mariadb-server
            - python3-PyMySQL
        state: latest
    - name: start mariadb
      become: yes  
      command: systemctl start mariadb
    - name: enable mariadb
      become: yes
      systemd: 
        name: mariadb
        enabled: true

ll
#run playjbook
ansible-playbook playbook.yml -i inventory.txt

#start the db-server
ansible db_server -m shell -a "mysql --version" -i inventory.txt

#Create sql script file
vi db-load-script.sql

USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;
INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");

vi playbook.yml
#add this to the end of tasks
    - name: copy the sql script
      copy:
        src: ~/Ansible-Website-Project/db-load-script.sql
        dest: ~/

vi ansible.cfg
# add this line
inventory=inventory.txt

#build more tasks
vi playbook.yml

- name: db configuration
  hosts: db_server
  tasks:
    - name: install mariadb and PyMySQL
      become: yes
      yum:
        name:
            - mariadb-server
            - python3-PyMySQL
        state: latest
    - name: start mariadb
      become: yes
      systemd:
        name: mariadb
        state: started
    - name: enable mariadb
      become: yes
      systemd:
        name: mariadb
        enabled: true
    - name: copy the sql script
      copy:
        src: ~/Ansible-Website-Project/db-load-script.sql
        dest: ~/
    - name: Create password for the root user
      mysql_user:
        login_password: ''
        login_user: root
        name: root
        password: "clarus1234"

#run playbook
ansible-playbook playbook.yml

vi my.cnf

[client]
user=root
password=clarus1234
[mysqld]
wait_timeout=30000
interactive_timeout=30000
bind-address=0.0.0.0
    
#build up more task, open up playbook.yml
vi playbook.yml
#add this task to playbook
- name: copy the my.cnf file
      copy:
        src: ~/Ansible-Website-Project/my.cnf
        dest: ~/.my.cnf

#build more task
#add this task to playbook
    - name: Create db user with name 'remoteUser' and password 'clarus1234' with all database privileges
      mysql_user:
        name: remoteUser
        password: "clarus1234"
        login_user: "root"
        login_password: "clarus1234"
        priv: '*.*:ALL,GRANT'
        state: present
        host: "{{ hostvars['web_server'].ansible_host }}"
#run it
ansible-playbook playbook.yml


#How to handle errors like this
fatal: [db_server]: FAILED! => {"changed": true, "cmd": "echo \"USE ecomdb; show tables like 'products'; \" | mysql\n", "delta": "0:00:00.015959", "end": "2020-10-03 19:59:19.212972", "msg": "non-zero return code", "rc": 1, "start": "2020-10-03 19:59:19.197013", "stderr": "ERROR 1049 (42000) at line 1: Unknown database 'ecomdb'", "stderr_lines": ["ERROR 1049 (42000) at line 1: Unknown database 'ecomdb'"], "stdout": "", "stdout_lines": []}

stderr": "ERROR 1049 (42000) at line 1: Unknown database 'ecomdb'", "stderr_lines": ["ERROR 1049 (42000) at line 1: Unknown database 'ecomdb'"]



#Playbook should be like this at this time
- name: db configuration
  hosts: db_server
  tasks:
    - name: install mariadb and PyMySQL
      become: yes
      yum:
        name:
            - mariadb-server
            - python3-PyMySQL
        state: latest
    - name: start mariadb
      become: yes
      systemd:
        name: mariadb
        state: started
    - name: enable mariadb
      become: yes
      systemd:
        name: mariadb
        enabled: true
    - name: copy the sql script
      copy:
        src: ~/Ansible-Website-Project/db-load-script.sql
        dest: ~/
        #    - name: Create password for the root user
        # mysql_user:
        # login_password: ''
        # login_user: root
        # name: root
        # password: "clarus1234"
    - name: copy the my.cnf file
      copy:
        src: ~/Ansible-Website-Project/my.cnf
        dest: ~/.my.cnf
    - name: Create db user with name 'remoteUser' and password 'clarus1234' with all database privileges
      mysql_user:
        name: remoteUser
        password: "clarus1234"
        login_user: "root"
        login_password: "clarus1234"
        priv: '*.*:ALL,GRANT'
        state: present
        host: "{{ hostvars['web_server'].ansible_host }}"
    - name: Create database schema
      mysql_db:
        name: ecomdb
        login_user: root
        login_password: "clarus1234"
        state: present
    - name: check if the database has the table
      shell: |
        echo "USE ecomdb; show tables like 'products'; " | mysql
      register: resultOfShowTables
    - name: DEBUG
      debug:
        var: resultOfShowTables
    - name: Import database table
      mysql_db:
        name: ecomdb   # This is the database schema name.
        state: import  # This module is not idempotent when the state property value is import.
        target: ~/db-load-script.sql # This script creates the products table.
      when: resultOfShowTables.stdout == "" # This line checks if the table is already imported. If so this task doesn't run.
    - name: restart mariadb
      become: yes
      service:
        name: mariadb
        state: restarted
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


```