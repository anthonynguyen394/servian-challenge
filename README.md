# servian-challenge

This is a guide on how to deploy Servian tech stack using Ansible scripts, there are 3 Ansible playbooks to run 3 parts of this process:
1. Running ec2.yaml will provision an ec2 instance on AWS (either Ubuntu or Centos).
2. Running install_docker.yaml will install and enable Docker on the instance and the required dependencies.
3. Running deploy.yaml will deploy the Servian Postgres database container, the Servian web app container and the process to populate the database. 

Prerequisites:
- This guide assume that you have an AWS account with administrator permissions and no infrastructure provisioned yet, you will need your AWS access key and secret to provision the instance. The AWS account can be a service account.
- You will also need to have your SSH public key ready for instance access to run remote Ansible deployment.
    

# I/ Provision EC2 Instance with ec2.yaml:

1.1. You will need Ansible installed and python dependencies on your personal computer, for example, on Ubuntu:
$ sudo apt install python
$ sudo apt install python-pip
$ pip install boto boto3 ansible

1.2. Have your public SSH key available, if not yet generated, use the below to generate SSH key pair:
$ ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_aws

1.3. Clone this project onto your local machine:
$ git clone <project_url>

1.4. Run the following to create a safe location with encryption password so you can put your AWS key pairs.
$ ansible-vault create group_vars/all/pass.yaml

1.5. Edit the pass.yaml file to save your AWS key pairs:
$ ansible-vault edit group_vars/all/pass.yaml 
Vault password:
ec2_access_key: AAAAAAAAAAAAAABBBBBBBBBBBB                                      
ec2_secret_key: afjdfadgf$fgajk5ragesfjgjsfdbtirhf

1.6. Run the playbook with the password to provision the instance, note that you can change the ec2.yaml file for the Image ID and instance type if you prefer to use others:
$ ansible-playbook ec2.yaml --ask-vault-pass --tags create_ec2 -i inventories/hosts 

The script will run and gives the public DNS at the end for next step

# II/ Install Docker and dependencies for the provisioned EC2 Instance with install_docker.yaml:

2.1. From the previous step (1.6) note down the public DNS of the instance. Edit the file hosts at inventories/hosts and put the public DNS under [webserver] section, this is used by the script deploy to the remote server. 
 
[webserver]
ec2-xx-xxx-xxx-x.ap-southeast-2.compute.amazonaws.com

2.2. Edit the config file under ~/.ssh/config to set the default user and SSH private key for access, if you are using Centos AMI, use Centos for the user login, and ubuntu for ubuntu AMI. For example: 

Host *.ap-southeast-2.compute.amazonaws.com
 ProxyCommand ssh sydbstpd.aiam-dh.com -W %h:%p
 User centos
 IdentityFile ~/.ssh/my_aws

2.3. After editting the file, run the ansible playbook to install and run Docker and its dependencies:
$ ansible-playbook install_docker_centos.yaml -i inventories/hosts --ask-vault-pass

# III/ Install the Servian application stack with deploy.yaml:

3.1 Run the following playbook to deploy Servian application and database with test data:
$ ansible-playbook deploy.yaml -i inventories/hosts --ask-vault-pass

3.2 There are some issues under investigation with the application being accessed publicly, use SSH tunnel to access the application through http://localhost:3000 on web browser:
$ ssh ec2-xx-xxx-xxx-x.ap-southeast-2.compute.amazonaws.com -L 3000:localhost:3000 -N
