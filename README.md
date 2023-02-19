# Ansibe Static Inventory, Dynamic Inventory and Windows automation with ansible.

# Ansibe Static Inventory Setup 

# ![CompleteCICDProject!](https://1.bp.blogspot.com/-dz7PCHIGXhI/X5s7S3esPiI/AAAAAAAADHE/9c3eOitPPkolv8ufrZ0E-yQ0WIKGOH5DwCLcBGAsYHQ/s1062/Ansible%2BArch.pn)

# Spin up 3 servers with centos 7 AMI, name them as follows ansible-master 
- ansible-node1 
- ansible-node2
- ansible-node3

# Connect into each of the server and change the hostname of your server 
- sudo hostname ansible-master 

- sudo hostname ansible-node1 
- sudo hostname ansible-node2 

# Create User ansible in all your 3 servers  
- useradd: ansible 
- passwd: ansible 

#edit the sshd_config file of all your 3 servers 

- sudo vi /etc/ssh/sshd_config 
- :/PasswordAuthentication 
- :/PermitRootLogin 
- PermitRootLogin no to PermitRootLogin yes 
- PasswordAuthentication no to PasswordAuthentication yes
- sudo service sshd restart

# Add user ansible to the wheel group 
nano /etc/sudoers

- Add ansible to root, uder "Allow root to run any commands anywhere". This will allow ansible to  work as root without 	  requiring a password 
          root ALL=(ALL) ALL
          ansible ALL=(ALL) NOPASSWD:ALL  <--

To begin exploring Ansible as a means of managing various servers, we need to install the Ansible software on ansible master. All below commands should be executed on the ansible master node only. To get Ansible for CentOS 7, first ensure that the CentOS 7 EPEL  repository is installed: 

- sudo yum install epel-release

Once the repository is installed, install Ansible with yum: 
- sudo yum install ansible

Ansible keeps track of all of the servers that it knows about through a "hosts" file. We need to set up this file first before we can begin to communicate with our other computers. 
Open the file with root privileges like this:

- sudo nano /etc/ansible/hosts 
- nano etc/ansible/hosts 

[webserver] 

- Ip address of ansible-node1 
- Ip address of ansible-node2 

#In your ansible-master, login as user ansible and generate ssh key on ansible control server #exit (to logout of centos user) 
- ssh ansible@ip address of the master server

#Create an ssh key in master server and copy it to node servers This will create .ssh folder (/home/ansadm/.ssh). Hit enter all the way  through 
ssh-keygen -t rsa 

This will create .ssh folder (/home/ansible/.ssh). Hit enter all the way through 

- chmod 700 /home/ansible/.ssh 
- ssh-copy-id ansible@ip address of you ansible-node1 
- ssh-copy-id ansible@ip address of you ansible-node2 
- ssh ansible@ip address of you ansible-node1 
- ssh ansible@ip address of you ansible-node2

Now all three servers are configured, ansible control server can do ssh on both the servers 
check connectivity of hosts is 

- ansible <group> -m ping

# Ansible Dynamic Inventory

![CompleteCICDProject!](https://devopscube.com/wp-content/uploads/2021/07/ansiblee-inventory.png)

Pre-requisites:
Ansible Dynamic Inventory (Ubuntu as Master and Amazon Linux 2 as Nodes)
- Create 1 Ubuntu 20.4 EC2 Instance with an IAM "EC2 Full Access" role and attach to Ansible Master node.
- Create 2 security groups 
	- Ansible-master-Sg: Allow 22 from my IP and 80 open to the world 0.0.0.0/0 --> attach to ansible master node
	- Ansible-client-Sg: allow 22 from "Ansible-master-Sg" --> attach to ansible client nodes
Create 3 EC2 Amazon Linux – 2 Instances and attach security group "Ansible-client-Sg" to the instances.

Ubuntu for Ansible Master
Amazon Linux 2 for Ansible Nodes

# Master Setup

SSH into ansible-Masterand install the following packages
- sudo apt update -y
- sudo apt install -y ansible
- sudo apt install python-is-python3
- sudo apt-get install python3-pip -y
- sudo pip3 install boto
- sudo pip3 install boto3
- ansible-galaxy collection install amazon.aws <-- Install Ansible Dynamic Inventory Plugin

Add a tag to the Ansible client Instances
    Env:dev

copy the file "aws_ec2" for Ansible Dynamic Inventory Plugin to "/etc/ansible/"


Edit "/etc/ansible/ansible.cfg" file, and uncomment
- “sudo_user”, "host_key_checking" and enable the plugins "enable_plugins = aws_ec2"
   nano /etc/ansible/ansible.cfg

Run the command below to test dynamic Inventory
- ansible-inventory -i aws_ec2.yaml --list

Create a pem key(Private key) in the master node and apply the required permissions -> this key will be used by ansible to manage client nodes.
- sudo nano ansible.pem
- chmod 400 ansible.pem

Run below command to check ping for Ansible Nodes
- ansible aws_ec2 -i aws_ec2.yaml -m ping --private-key=ansible.pem --user ec2-user

Run below command to install git on Ansible nodes using dynamic inventory
- sudo ansible aws_ec2 -i aws_ec2.yaml -m yum -a 'name=git state=present' --private-key=ansible.pem --become --user ec2-user

# Dynamic Inventory Demo
Install Apache on client nodes by running the command below
- sudo ansible-playbook -i aws_ec2.yaml -l aws_ec2 test-playbook.yaml --private-key=ansible.pem --user ec2-user

For verification, go to the url
- http://public-ip:80 --> ansible client public ip address

# Ansible and Windwos Demo
Automate windows server deployments using Ansible.

![CompleteCICDProject!](https://miro.medium.com/max/1200/1*X1hqJo4uXBhvmZgMPQ0U3Q.jpeg)

# Install chokolatey windows package manager on a windows os using ansible.

Basic reminders and requirements
- Ansible uses WinRM protocol to automate windows servers.
- Ansible is written in python thus we need the latest version of python3 and pywinrm installed.
- The control node must be a linux server.

# Provision 2 instances 
- control node --> Ubuntu server
- client node --> windows sever

# Control node Setup (Ubuntu)
- sudo apt update -y
- sudo apt install ansible -y / ansible --version --> check the version of python ansible is using.
- sudo apt install python3 & python3 --version
- sudo apt install pip3 -y
- pip3 install pywinrm>=0.3.0

On control node copy the ansible configurations in the "inventory" file to your /etc/ansible/hosts file.

# Client node Setup (Windows)
Ansible requires Powershell 3.0  or newer and atleast .Net 4.0 to installed on windows host.
A WinRM lister should be create and activated.
- Create an ansible admin user
- Open powershell as admin and run the following commands "win_setup.txt" file.

Test connectivity
- ansible win -m win_ping
	
- Run the playbook "choco_install.yml" on control node to install chocolatey  
$ ansible-playbook choco_install.yml

Verify if chocolatey is installed on windows server
- Run powershell as administrator then type the command 
- "choco search tomcat"

Refer to documentation for more details.

- https://docs.ansible.com/ansible/latest/os_guide/windows_setup.html#winrm-setup
- https://docs.ansible.com/ansible/latest/reference_appendices/python_3_support.html
