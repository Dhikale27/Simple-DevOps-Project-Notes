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
   
   
