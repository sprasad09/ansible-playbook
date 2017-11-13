Ansible Roles:
=============

This repo contains Ansible roles for nginx and mysql. It can be used to spin up new aws instances in your VPC with customized security group  or you can configure an existing aws server with nginx and mysql. 
Nginx is setup to proxy all connection coming from port 80 to port 3000. The application running on port 3000 will connect to mysql on its internal IP. All Mysql server ports are blocked externally and is only accesible through internal IP. There is no firewall restriction for outgoing connection.  

  
### Requirements:

We need Ansible software to be installed in order to use these roles; Roles can be run from any machine that have Ansible installed on.

Here are the steps to install the Ansible on Ubuntu 14.04 LTS:
```bash
sudo apt-add-repository -y ppa:ansible/ansible
sudo apt-get update
sudo apt-get install -y ansible

apt-get install -y git python-pip python-dev build-essential
pip install boto
pip install boto3

Create .boto file in the home dir where ansible is being run and add the following so ansible can connet to your aws servers.
[Credentials]
aws_access_key_id =  
aws_secret_access_key =  
```

### Assumptions:

This playbook is assuming that you are connecting to your AWS environment using VPN and are able to ssh to the servers on the private IP. If that is not the case then you will need to update item.private_ip to item.public_ip in the links below so ansible can ssh and setup/install nginx and mysql to their respective servers once they are up. 

```
dbserver playbook:
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/dbserver.yml#L50
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/dbserver.yml#L51
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/dbserver.yml#L55
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/dbserver.yml#L61

Nginx playbook:
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/webserver.yml#L54
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/webserver.yml#L55
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/webserver.yml#L59
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/webserver.yml#L65

```
Update security group to allow port 22 to connect externally:

```
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/webserver.yml#L20
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/dbserver.yml#L20

add another block like this
            - proto: tcp
              from_port: 22
              to_port: 22
              cidr_ip: 0.0.0.0/0

```
internally all ports are open to talk to each other, please update to point to your CIDR so your app running on port 3000 can talk to mysql internally.  

```
https://github.com/sprasad09/ansible-playbook/blob/2dd8db2798c921cf63a4c95af2d4e3ca940e06b6/playbook/dbserver.yml#L24

            - proto: tcp
              from_port: 0
              to_port: 65535
              cidr_ip: 172.30.0.0/16
```

### Setup new aws servers for web and database with security group: 

1. Install ansible

2. Replace /etc/ansible directory with the git repo 
```
git clone https://github.com/sprasad09/ansible-playbook.git /etc/ansible.   
```
4. Make sure the "private_key_file =" in ansible.cfg is pointing to your .pem file so ansible can login to your server once its up to setup/install application.  

3. Add your aws vpc, subnet, instance size etc... info under ec2-vars directory so the instances are created  with your desired setup. https://github.com/sprasad09/ansible-playbook/tree/master/playbook/ec2-vars. Once you have supplied all the information for webserver.yml and database.yml

Run this from inside the playbook directory:
``` 
ansible-playbook  -v -i localhost, -e "type=webserver" webserver.yml  #for nginx server setup 
ansible-playbook  -v -i localhost, -e "type=dbserver" dbserver.yml   #for dbserver server setup
```

Every server that gets spun up, their private IP address gets added in the "host" file under their respective hostgroups. For the next configuration push to the hostgroup, Ansible will look at the host file to pick the appropriate ip's from the hostgroup to push to moving forward. 


### Setup nginx and mysql to an existing aws servers:  

Edit the `hosts` file, enter the server ip/name of your existing web and db server under their respective hostgroups.  Run this command for pushing roles to all server:
```
ansible-playbook -i hosts  site.yml
```

### For individual role deploy:
```
ansible-playbook webserver.yml --tags "nginx"  # nginx deploy
ansible-playbook dbserver.yml --tags "mysql"   # mysql deploy
```
 
Ansible requires nothing more than a password or SSH key in order to start managing systems. Ansible is easy to install and the playbook based roles can be run from any computer/server that has ansible installed. 

Ansible  pushes out small programs, called "Ansible Modules", by connecting to the nodes over SSH. Because these modules are simple Python scripts, and Ansible is agent-less, the target hosts only require an SSH connection and Python installed. Besides simplicity, which is another advantage of using Ansible vs Chef or Puppet, there are no servers, daemons, or databases required - all that is needed is to install Ansible on the host, Python on the targets, and have SSH access.
