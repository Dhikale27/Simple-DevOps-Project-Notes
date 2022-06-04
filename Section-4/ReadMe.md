# Integrating Docker in CI/CD pipeline
   #### Docker Environment Setup
   - In this section we are going to instal docker in our EC2 instance
   
   1. Creating EC2 instance on AWS name as Docker-Host
   
   2. Installing docker 
      ```sh
      # in terminal type
      [root@ip-172-31-39-151 ~]# yum install docker -y
      
      # to check docker install or not
      [root@ip-172-31-39-151 ~]# docker -v
      
      # to start docker
      [root@ip-172-31-39-151 ~]# service docker start
      
      # to check status 
      [root@ip-172-31-39-151 ~]# service docker status
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
      - command to create new container for tomcat image
          ```sh [root@docker-host ~]# docker run -d --name tomcat-container -p 8081:8080 tomcat```
       
      
   
