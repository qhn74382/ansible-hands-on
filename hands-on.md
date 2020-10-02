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