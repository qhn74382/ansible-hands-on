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