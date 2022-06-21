# Integrating Docker in CI/CD pipeline
   #### Docker Environment Setup
   - In this section we are going to instal docker in our EC2 instance
   
   1. Creating EC2 instance on AWS name as Docker-Host
   
   2. Installing docker 
      - installing docker
        ```sh
        [root@ip-172-31-39-151 ~]# yum install docker -y
        ```
      - to check docker installation
        ```sh
        [root@ip-172-31-39-151 ~]# docker -v
        ```
      - to start and check status 
        ```sh
        [root@ip-172-31-39-151 ~]# service docker start
        
        [root@ip-172-31-39-151 ~]# service docker statu
        ```
      
   #### -------------------------------------------------------------------------------------
   
   #### Create the tomcat container
   
   - we are going to use docker hub to get image of tomcat
   - before pulling the image let change the name of our docker host
     ```sh
     [root@ip-172-31-39-151 ~]# vi /etc/hostname 
     make change in above file as docker-host
     
     #reboot our terminal
     [root@ip-172-31-39-151 ~]# init 6

     ```
   1. pull docker image
      ```sh
      # start docker service
      [root@docker-host ~]# service docker start
      
      # pulling docker image
      [root@docker-host ~]# docker pull tomcat
      
      # check docker images
      [root@docker-host ~]# docker images
      
      ```
   2. creating the docker container
      ```sh
          # command to create new container for tomcat image
          [root@docker-host ~]# docker run -d --name tomcat-container -p 8081:8080 tomcat
       ```
       
   #### ------------------------------------------------------------------------------------------------
   
   #### Fixing Tomcat Container Issue
   - Tomcat 8x and onward facing issue i.e. on given port 8081 we are not able to access our tomcat. This problem occure due to webapp folder which is include in our container is empty. All the required file are is in webapps.dist directory. So to solve the problem we have to copy all data from webapp.dist to uor webapp folder
     ```sh
     # to see the container file we have to login in that
     [root@docker-host ~]# docker exec -it 72d /bin/bash
       o/p>> root@72dffa9ca590:/usr/local/tomcat#
     
     # TO check the content 
     root@72dffa9ca590:/usr/local/tomcat# ls
       o/p>> BUILDING.txt     NOTICE         RUNNING.txt  lib             temp          work
             CONTRIBUTING.md  README.md      bin          logs            webapps
             LICENSE          RELEASE-NOTES  conf         native-jni-lib  webapps.dist
             
     # To check content of webapps.dist
     root@72dffa9ca590:/usr/local/tomcat/webapps.dist# ls
      o/p>> ROOT  docs  examples  host-manager  manager
    
    #To check containt of webapps
    root@72dffa9ca590:/usr/local/tomcat/webapps.dist# cd ..
    root@72dffa9ca590:/usr/local/tomcat# cd webapps
    root@72dffa9ca590:/usr/local/tomcat/webapps# ls
       o/p>> 
       
    #To copy containt from webapp.dist to webapp
    root@72dffa9ca590:/usr/local/tomcat/webapps.dist# cp -R * ../webapps/
      
    # To get out of container
    root@72dffa9ca590: exit

     ```
   - whatewher changes we made is valid for only given container noly if we create new conatiner by using the same image then once again the proble will resume. To overcome this problem we have to take help of docker file.
   
   #### ----------------------------------------------------------------------------------------
   
   #### Creating First Docker file
   - we are going to create our own image of tomcat server by using dockerfile.
   - Dockerfile name should be start with D.
   1. Creating docker file
      ```sh
      # go to root dir and create Docker file
      [root@docker-host ~]# vi Dockerfile
     
      #Type below line of code
     
      FROM centos:latest
      RUN cd /etc/yum.repos.d/
      RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
      RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
 
      RUN yum install java -y
      RUN mkdir /opt/tomcat
      WORKDIR /opt/tomcat
      ADD https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.63/bin/apache-tomcat-9.0.63.tar.gz .
      RUN tar -xvzf apache-tomcat-9.0.63.tar.gz
      RUN mv apache-tomcat-9.0.63/* /opt/tomcat
      EXPOSE 8080
      CMD ["/opt/tomcat/bin/catlina.sh", "run"]
      ```
     
   #### Customise Docker file for Tomcate
   - in tis section we create a docker to create image of tomcat and to cofigure for globally access
   1. Create Docker file
      ```sh
        # creating docker file
        [root@docker-host ~]# vi Dockerfile
        
        in above file add 
          FROM tomcat:latest
          RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
          ~
          ~
          ~
          
        # run the docker file
        [root@docker-host ~]# docker build -t demotomcat .
        
        #demotomcat docker image will get generate
        #to check the docker images
        [root@docker-host ~]# docker images
        
        #creating the container
        [root@docker-host ~]# docker run -d --name demotomcat -p 8085:8080 demotomcat
        
        [root@docker-host ~]# docker ps 
        
        # to run tomcate we have to open the port after that we can run the tomcat

      ```
   ### ------------------------------------------------------------------------------------------ 
   #### Integrating Docker with jenkins
   - Before integrating with docker we must have saperate user. We can use ec2-user but its not recomended
   1. To create new User
      ```sh
        # to check list of user
        [root@docker-host ~]# cat /etc/passwd
        
        # to add new user
        [root@docker-host ~]# useradd dockeradmin
        
        # To set the password for dockeradmin
        [root@docker-host ~]# passwd dockeradmin
      
      ```
   2. Add this user into docker group
      ```sh
        # to add user in docker group
        [root@docker-host ~]# usermod -aG docker dockeradmin
      ```
   3. Enbling password based authentication
      - Currently we are using key base authentication to enable the password base authentication we have to configure sshd_config file
      ```sh
        # To edit the sshd_config file
        [root@docker-host ~]# vi /etc/ssh/sshd_config
        
        in above file just change below things
        # To disable tunneled clear text passwords, change to no here!
        PasswordAuthentication yes
        #PermitEmptyPasswords no
        #PasswordAuthentication no
        
        # After editing we have to reload
        [root@docker-host ~]# service sshd reload
      ```
   4. Adding docker host plugin to jenkin server
      - to connect our docker host with jenkin server we required to add docker host plugins(Publish Over SSH) in jenkins server
      - `Manage jenkins` > `Manage plugins` > `Available` > `Publish Over SSH` > `Install without restart`
      - `Manage jenkins` > `Configure system` > `Publish Over SSH : Add SSH : Name = dockerhost & Hostname= <private ip add> & Username=dockeradmin & Password = dockeradmin123` > `Test Configuration`

   5. Creating new Job to build and deploy artifect on docker container
      - we already create job for pull build deoply code so we will use thae job and creat new job
      - `New Item` > `Enter Item Name` > `Copy From = <old job>` > `Ok` > `Post Build Action : Send Build artifects over SSh : ssh server Name = dockerhost & Source file = webapp/target/*.war & Remove prefix = webapp/target` > `Apply & Save`.
      - after saving Build will happen automatically.


   ### -----------------------------------------------------------------------------------------------------------
   #### Updating the tomcat docker file to automate deployment process
   - let us create a separate dirctory for dockerfile and artifects so we can easily run the dockerfile
   1. Creating directory
      ```sh
        # exite first from dockeradmin
        [dockeradmin@docker-host ~]$ exit
           logout
       [ec2-user@docker-host ~]$
       [ec2-user@docker-host opt]$ sudo su -
       [root@docker-host ~]#
       
       # create dir
       [root@docker-host ~]# cd /opt
       [root@docker-host opt]# mkdir docker
       
       # we have to give permission to dockeradmin for accesing the docker dir
       [root@docker-host opt]# chown -R dockeradmin:dockeradmin docker
       
       # after giving persmission we will get
       [root@docker-host opt]# ll
         total 0
         drwxr-xr-x 4 root        root        33 Apr 28 19:54 aws
         drwx--x--x 4 root        root        28 Jun  8 09:01 containerd
         drwxr-xr-x 2 dockeradmin dockeradmin  6 Jun  8 13:05 docker
         drwxr-xr-x 2 root        root         6 Aug 16  2018 rh
         
       # move Dockerfile to docker dir
       [root@docker-host ~]# mv Dockerfile /opt/docker/
       
       #give persion to dockeradmin to acess the adocker file
       [root@docker-host docker]# chown -R dockeradmin:dockeradmin /opt/docker/
       
   2. Changes In Jenkin Job and Building it
      ```sh  
       # after this we have make change in jenkin job 
       go to "jenkin job configuration >> change Remote directory  = //opt//docker >> Apply & Save"
       
       # after this build job and check docker dir
       [root@docker-host docker]# ls
       Dockerfile  webapp.war
   
   3. Editing Docker file and executing it
      - to create the docker container with war file we have change the DOckerfile
      ```sh
        # To edit the docker file
        [root@docker-host docker]# vi Dockerfile
        
        # After editing Docker file
        FROM tomcat:latest
        RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
        COPY ./*.war /usr/local/tomcat/webapps
        ~
        ~
        
        # Execute docker file
        [root@docker-host docker]# docker build -t tomcat:v1 .
        
        # check for images
        [root@docker-host docker]# docker images
        
        # creating container from image
        [root@docker-host docker]# docker run -d --name tomcatev1 -p 8086:8080 tomcat:v1
        
        here v1 is important
        
        # check for container running
        [root@docker-host docker]# docker ps
        
        # check the address whtether we can access or not
        - open browser and go given address http://<public-ip- address>/webapp/
   
   ### -----------------------------------------------------------------------------------------------
   #### Automate build and deployment on docker container
   - After commiting code in git then that code shold automatically pull by jenkins then its artifect shoild be get created by maven build tool, once artifect generate then the docker image shold be get create by automatic and once docker image get create container should run automatically. This part we will see this section
   - go to out old job (PullBuildDeployCodeOnContainer) `Configure` > `Post-Build Action : Exec command = cd /opt/docker; docker build -t regapp:v1; docker run -d --name registerapp -p 8087:8080 regapp:v1` > `Apply & Save`
   - after saving build the job we will see conatine run automatically.
   - Now go to our source code and make change in that then push the code in repository then container will run automatically.
   1. Resolving Automatic build issue: 
      - If we have already container present in docker host then we face error while next building.
      - go to out old job (PullBuildDeployCodeOnContainer) `Configure` > `Post-Build Action : Exec command = cd /opt/docker; docker build -t regapp:v1 .; docker stop registerapp; docker rm registerapp; docker run -d --name registerapp -p 8087:8080 regapp:v1` > `Apply & Save`. 

        
        
    
        
        

        
 


     
      
      
  
