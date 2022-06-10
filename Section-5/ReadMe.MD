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

4. Eable password based authentication
   ```sh
     # edit file
     [root@ansible-server ~]# vi /etc/ssh/sshd_config
     
     # make change as mension below
       PasswordAuthentication yes
       #PermitEmptyPasswords no
       #PasswordAuthentication no
     
     #after edit reload
     [root@ansible-server ~]# service sshd reload
    
5. Creating the ssh key for ansadmin user
   ```sh
     # login as ansadmin
     [ec2-user@ansible-server ~]$ sudo su - ansadmin
     [ansadmin@ansible-server ~]$
     
     # creating new ssh key
     [ansadmin@ansible-server ~]$ ssh-keygen
       Generating public/private rsa key pair.
       Enter file in which to save the key (/home/ansadmin/.ssh/id_rsa):
       Created directory '/home/ansadmin/.ssh'.
       Enter passphrase (empty for no passphrase):
       Enter same passphrase again:
       Your identification has been saved in /home/ansadmin/.ssh/id_rsa.
       Your public key has been saved in /home/ansadmin/.ssh/id_rsa.pub.
       The key fingerprint is:
       SHA256:Gcn+gyL18TtzN2T0S0tzV71DRKTE1h0Ej4QskDttnhg ansadmin@ansible-server
       The key's randomart image is:
       +---[RSA 2048]----+
       |       .o . o+*=o|
       |       o o ooo+o.|
       |        * . .o...|
       |       E =    ..o|
       |      . S .  ...o|
       |     . o B    o*+|
       |    . . o +  oo B|
       |     . .  oo. oo |
       |          .+ . . |
       +----[SHA256]-----+

5. Install ansible
   - To install insible we required python install. In our ec2 instace having linux operating system and oython is bydefault present in linux system.
   ```sh
     # install ansible
     [root@ansible-server ~]# amazon-linux-extras install ansible2
     
     # To check installation
     [root@ansible-server ~]# python --version
     [root@ansible-server ~]# ansible --version
     
     
### ------------------------------------------------------------------------------------------------------

## Integrating Docker with ansible
1. Create new user ansadmin in docker-host > add usr to sudoerce file

2. Add docker-host manager node in ansible-server
   ```sh
     #edit hosts file
     [root@ansible-server ~]# vi /etc/ansible/hosts
     
     #in above file add private ip add of docker-host
     
3. Adding ssh key to docker-host 
   ```sh
      [ansadmin@ansible-server .ssh]$ ssh-copy-id 172.31.33.185
      
4. Creating connection 
   ```sh
     [ansadmin@ansible-server home]$ ansible all -m  ping
     
     # above command meaning : try to connect all hosts which present in inventory file
     
     # to check the connection
     [ansadmin@ansible-server home]$ ansible all -m command -a uptime

     
### -------------------------------------------------------------------------------------------------------------

## Integrating ansible with jenkins
- Integration of ansible with jenkins is almost same as with docker
1. make change in configure system of jenkins
   - `Manage jenkins` > `configure jenkins` > `ssh server : ADD : Server Name = ansible-server & Hostname = <private ip address of ansible server> & username = ansadmin & password = ansadmin123` > `Test Configuration` > `Apply & Success`

2. Creating new job to build and copy artifects on ansible
   - `New Item` > `Enter New Item Name` > `PullBuildDeployOnConatiner` > `OK` > `Post-Build-Action : SSH Server name = ansible-server (other setting will same, We have to create remote directory /opt/docker on ansible and also giv ansadmin permission to access docker folder)` > `Apply & Save`
   
3. Build Now the job

### ---------------------------------------------------------------------------------------------------
## Building image and container on ansible
- Creating image is almost same as we see in docker
1. install docker on ansible
2. add ansadmin to docker group
   ```sh 
     # add user in docker
     [root@ansible-server docker]# usermod -aG docker ansadmin
   
3. Creating docker file
   ```sh
     [root@ansible-server docker]# vi Dockerfile
     
     # In above file add
       FROM tomcat:latest
       RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
       COPY ./*.war /usr/local/tomcat/webapps
       
     # Giving permission to ansadmin to read the dockerfile
     [root@ansible-server docker]# chmod 777 /var/run/docker.sock
     
     # Create image
     [root@ansible-server docker]# docker build -t regapp:v1 .
     
     #crteate container
     [root@ansible-server docker]# docker run -t --name regapp-server -p 8081:8080 regapp:v1
     
### ----------------------------------------------------------------------------------------------------------

## Ansible playbook to create the image and container
- we will create the images and container by using ansible playbook for this we use ansible-playbook command
1. Adding ansible ip address in invetory file
   - To execute ansible playbook we have to add private ip address to ansible invetory(/etc/ansible/hosts) file.
   ```sh
     # edit inventory file
     [root@ansible-server docker]# vi /etc/ansible/hosts
     
     # In above file add
       [dockerhost]
       172.31.33.185

      [ansible]
      172.31.5.102
2. copy ssh key to ansible-server
   ```sh
     [ansadmin@ansible-server docker]$ ssh-copy-id 172.31.5.102
     
     # check uptime
     [ansadmin@ansible-server docker]$ ansible all -a uptime

3. write ansible playbook
   ```sh
     #create new yml file
     [ansadmin@ansible-server docker]$ vi regapp.yml
     
     # above file add below data
       ---
       - hosts: ansible

         tasks:
         - name: create docker image
           command: docker build -t regapp:lates .
           args:
            chdir: /opt/docker
            
     # check before execute the yml file
     [ansadmin@ansible-server docker]$ ansible-playbook regapp.yml --check

     # execute the yml file
     [ansadmin@ansible-server docker]$ ansible-playbook regapp.yml

    # check images
    [ansadmin@ansible-server docker]$ docker images
     
### -----------------------------------------------------------------------------------------
## Copy image to dockerhub
- to push images on dockerhub we required a dockerhub account
1. Docker Login
   ```sh
     # to login
     [ansadmin@ansible-server docker]$ docker login
     
     #provide username and password
     
2. Push image to dockerhub
   ```sh
     # giving tad to image
     [ansadmin@ansible-server docker]$ docker tag b1a397acf8fb sachin27021992/regapp:latest
     
     #Push image 
     [ansadmin@ansible-server docker]$ docker push sachin27021992/regapp:latest

### -------------------------------------------------------------------------------------------------------------------

## Jenkin job to push an image onto ansible

1. Add image push code in ansible playbook
   ```sh
     # edit yml file
     [ansadmin@ansible-server docker]$ vi regapp.yml
     
     # above file shoid lool like
       ---
       - hosts: ansible

       tasks:
       - name: create docker image
         command: docker build -t regapp:lates .
         args:
          chdir: /opt/docker

       - name: create tag push image onto dockerhub
         command: docker tag regapp:latest sachin27021992/regapp:latest

       - name: push docker image
         command: docker push sachin27021992/regapp:latest

2. Executing ansible playbook file
   ```sh
     # to check
     [ansadmin@ansible-server docker]$ ansible-playbook regapp.yml --check
     
     # to execute
     [ansadmin@ansible-server docker]$ ansible-playbook regapp.yml
     
     
3. Change in Jenkins job 
   - to automate the process of push image on dockerhub we have to configure our old jinkins job.
   - go to `CopyArtifectOntoAnsible : configure` > `activate Poll SCM` > `Post-Build Action : Exec command = ansible-playbook /opt/docker/regapp.yml` > `Apply & Save`. 
   - after configuration make chnage in our source code, we will see automatic building of job and new updated image will get upload.
   
### ----------------------------------------------------------------------------------------------------------------------

## Create Container using Ansible Playbook
- we have already created container for push image to dockerhub. Now we will create another playbook for pull image from dockerhub and create container of it
1. create ansible playbook
   ```sh
     # create new playbook
     [ansadmin@ansible-server docker]$ vi deploy_regapp.yml
     
     # In above file add below data
       ---
       - hosts: dockerhost

       tasks:
       - name: create container
         command: docker run -d --name regapp-server -p 8082:8080 sachin27021992/regapp:latest

     
2. To execute the ansible playbook
   - before executing remove previous images and container
   ```sh
     # give permission
     [root@docker-host ~]# chmod 777 /var/run/docker.sock
     
     # executing command
     [ansadmin@ansible-server ~]# ansible-playbook deploy_regapp.yml
     
     #check docker images
     [root@docker-host ~]# docker images
     
     #check on wbsite
     http://<public ip add of docker host>/8082/webapp
     
3. Continuous deployment of docker container
   - we create playbook to pull image from dockerhub and create container from that. Now If container is already running or exist in system then we face error when we try to run ansible playbook(which cointain same container creation task)
   - To solve above issue we will edit ansible playbook file and add task to remove old container and task to remove old image as if image present in our system it not pull latest image.
   
   ```sh
     # edit ansible playbook of deployment
     [ansadmin@ansible-server docker]$ vi deploy_regapp.yml

     
     # after editing our file should look like this
       ---
       - hosts: dockerhost

       tasks:
       - name: stop existing container
         command: docker stop regapp-server
         ignore_errors: yes
         
       - name: remove the container
         command: docker rm regapp-server
         ignore_errors: yes

       - name: remove image
         command: docker rmi sachin27021992/regapp:latest
         ignore_errors: yes

       - name: create container
         command: docker run -d --name regapp-server -p 8082:8080 sachin27021992/regapp:latest
         
### -------------------------------------------------------------------------------------------------------

## Automat Everything
- Now we are going to automate everything i.e. pulling code from github by using jenkins, building code i.e. creating artifects by using maven, creating image and deploying that image on dockerhub and Pulling image from dockerhub and crating container with help of ansible and docker.
- go to `CopyArtifectOntoAnsible job` > `Configur` > `Post-Build Action : Exec Comand = ansible-playbook /opt/docker/regapp.yml; sleep 10; ansible-playbook /opt/docker/deploy_regapp.yml;` > `Apply &  Save`.
- after  that go to source code and make some chnage so we test our application.
- meanwhile during code we create different job, just remove Poll SCM for them so the will not trigger automatically

### ---------------------------------------------------- END --------------------------------------------------
