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
