# Ansibe Static Inventory, Dynamic Inventory and Windows Automation with ansible.

# Ansibe Static Inventory Setup 

# https://1.bp.blogspot.com/-dz7PCHIGXhI/X5s7S3esPiI/AAAAAAAADHE/9c3eOitPPkolv8ufrZ0E-yQ0WIKGOH5DwCLcBGAsYHQ/s1062/Ansible%2BArch.png

# Spin up 3 servers with centos 7 AMI, name them as follows ansible-master 
ansible-node1 
ansible-node2 

# Connect into each of the server and change the hostname of your server #sudo hostname ansible-master 

sudo hostname ansible-node1 
sudo hostname ansible-node2 

# Create User ansible in all your 3 servers  
useradd: ansible 
passwd: ansible 

#edit the sshd_config file of all your 3 servers 

sudo vi /etc/ssh/sshd_config 
:/PasswordAuthentication 
:/PermitRootLogin 
PermitRootLogin no to PermitRootLogin yes 
PasswordAuthentication no to PasswordAuthentication yes #sudo service sshd restart

#Add user ansible to the wheel group 
vi /etc/sudoers 
# Add ansible to root, uder "Allow root to run any commands anywhere". This will allow ansible to  work as root without requiring a password 
root ALL=(ALL) ALL
ansible ALL=(ALL) NOPASSWD:ALL  <--

To begin exploring Ansible as a means of managing various servers, we need to install the Ansible software on ansible master. All below commands should be executed on the ansible master node only. To get Ansible for CentOS 7, first ensure that the CentOS 7 EPEL  repository is installed: 

sudo yum install epel-release

Once the repository is installed, install Ansible with yum: #sudo yum install ansible
Ansible keeps track of all of the servers that it knows about through a "hosts" file. We need to set up this file first before we can begin to communicate with our other computers. 
Open the file with root privileges like this:

sudo vi /etc/ansible/hosts 
vim etc/ansible/hosts 
[webserver] 
Ip address of ansible-node1 
Ip address of ansible-node2 

#In your ansible-master, login as user ansible and generate ssh key on ansible control server #exit (to logout of centos user) 
ssh ansible@ip address of the master server

#Create an ssh key in master server and copy it to node servers This will create .ssh folder (/home/ansadm/.ssh). Hit enter all the way  through 
ssh-keygen -t rsa 

This will create .ssh folder (/home/ansible/.ssh). Hit enter all the way  through 

chmod 700 /home/ansible/.ssh 
ssh-copy-id ansible@ip address of you ansible-node1 #ssh-copy-id ansible@ip address of you ansible-node2 
ssh ansible@ip address of you ansible-node1 
ssh ansible@ip address of you ansible-node2

Now all three servers are configured, ansible control server can do ssh on both the servers 
check connectivity of hosts is 

ansible <group> -m ping

# Ansible Dynamic Inventory

https://devopscube.com/wp-content/uploads/2021/07/ansiblee-inventory.png

Pre-requisites:
Ansible Dynamic Inventory
(Ubuntu as Master and Amazon Linux 2 as Nodes)
Create 1 Ubuntu 20.4 EC2 Instance with an IAM "EC2 Full Access" role for and attach to Ansible Master node.
Create 2 security groups 
    Ansible-master-Sg: Allow 22 from my IP --> attach to ansible master node
    Ansible-client-Sg: allow 22 from "Ansible-master-Sg" --> attach to ansible client nodes
Create 3 EC2 Amazon Linux – 2 Instances and attach security group "Ansible-client-Sg" to the instances.

Ubuntu for Ansible Master
Amazon Linux 2 for Ansible Nodes

# Master Setup

SSH into Ansible-Master

    sudo apt update -y
    sudo apt install -y ansible
    sudo apt install python-is-python3
    sudo apt-get install python3-pip -y
    sudo pip3 install boto
    sudo pip3 install boto3
    ansible-galaxy collection install amazon.aws <-- Install Ansible Dynamic Inventory Plugin

Add a tag to the Ansible client Instances
    Env : dev

Create a file for Ansible Dynamic Inventory Plugin
sudo vi aws_ec2.yaml
    plugin: aws_ec2 
        regions:
            - us-east-1
        filters:
            tag:Env:
                - dev

Goto /etc/ansible/ansible.cfg, here everything is commented so uncomment
“sudo_user” and "host_key_checking" and enable ec2 plugins "enable_plugins = aws_ec2"
    sudo vi /etc/ansible/ansible.cfg and uncomment

To test dynamic Inventory
    ansible-inventory -i aws_ec2.yaml --list

Create the pem key file which is used for Ansible Node EC2 Instances 
    sudo vi avi-jes-oregon.pem
    chmod 400 avi-jes-oregon.pem

Run below command to check ping for Ansible Nodes
    ansible aws_ec2 -i aws_ec2.yaml -m ping --private-key=avi-jes-oregon.pem --user ec2-user

Run below command to install git on Ansible nodes using dynamic inventory
    sudo ansible aws_ec2 -i aws_ec2.yaml -m yum -a 'name=git state=present'
    --private-key=avi-jes-oregon.pem --become --user ec2-user

# Demo

Create a Ansible Playbook
    sudo vi test-playbook.yaml

        ---
        - name: sample playbook
          hosts: all
          become: true
          tasks:
            - name: install httpd
              yum:
                name: httpd
                state: latest
            - name: run httpd
              service:
                    name: httpd
                        state: started
            - name: create content
              copy:
                content: "GM! Hello from TECHWORLD"
                dest: /var/www/html/index.html

To test ansible playbook run below command
    sudo ansible-playbook -i aws_ec2.yaml -l aws_ec2 test-playbook.yaml --private-key=<your Key> --user ec2-user

To test whether playbook is executed successfully. 
Copy the public ip of Ansible node and search on google.

# Demo project. Automate windows server deployments using Ansible.

https://miro.medium.com/max/1200/1*X1hqJo4uXBhvmZgMPQ0U3Q.jpeg

# Install chokolatey windows package manager on a windows os using ansible.

Basic reminders and requirements
- Ansible uses WinRM protocol to automate windows servers.
- Ansible is written in python thus we need the latest version of python3 and pywinrm installed.
- The control node must be a linux server.

# Provision 2 instances 
- control node --> Ubuntu server
- client node --> windows sever

# Control node Setup (Ubuntu)
sudo apt update -y
sudo apt install ansible -y / ansible --version --> check the version of python ansible is using.
sudo apt install python3 & python3 --version
sudo apt install pip3 -y
pip3 install pywinrm>=0.3.0

nano /etc/ansible/hosts
[win]
10.0.0.216 --> Ip address of windows server

[win:vars]
ansible_user=ansible_user
ansible_password=welcome1234
ansible_connection=winrm


# Client node Setup (Windows)
Ansible requires Powershell 3.0  or newer and atleast .Net 4.0 to installed on windows host.
A WinRM lister should be create and activated.

- Create an ansible admin user
- login as ansible user
- Open powershell as admin and run the following commands one after the other.
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
        $url = "https://raw.githubusercontent.com/jborean93/ansible-windows/master/scripts/Install-WMF3Hotfix.ps1"
        $file = "$env:temp\Install-WMF3Hotfix.ps1"
        (New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
        powershell.exe -ExecutionPolicy ByPass -File $file -Verbose

Test connectivity
- ansible win -m win_ping

playbook to install chocolatey
nano choco_install.yml
- hosts win
  gather_facts no
  tasks 
	- name Install chocolates on windows 
	  win_chocolatey  name=procexp state=present

To run playbook use 
- ansible-playbook choco_install.yml

To verify if chocolatey is installed on windows server
Run powershell as administrator then type the command 
"choco search tomcat"

Refer to documentation for more details.

https://docs.ansible.com/ansible/latest/os_guide/windows_setup.html#winrm-setup
https://docs.ansible.com/ansible/latest/reference_appendices/python_3_support.html
