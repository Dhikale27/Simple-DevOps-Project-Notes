# Integrating Ansible in CI/CD Pipeline

## Installing Ansible
- Ansible is an open-source automation platform. It is very, very simple to set up and yet powerful. Ansible can help you with configuration management, application deployment, task automation.

### Pre-requisites

1. An AWS EC2 instance (on Control node)

### Instalation

1. Creating EC2 instance

2. Create Session and change host name.

3. Create new user
   ```sh
     # Creating new user
     [root@ansible-server ~]# useradd ansadmin
     
     #Set password
     [root@ansible-server ~]# passwd ansadmin
     
     # Add user to visudo file
     [root@ansible-server ~]# visudo
     
     # in above file add ansadmin ALL=(ALL)         ALL   
     
