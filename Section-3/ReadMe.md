# Integrating Tomcate server in CI/CD pipeline
- Follow this video lecture in Valaxy Technologies **[YouTube Channel](https://youtu.be/68WNroQBUts)**  
- Complete DevOps Project course on [Udemy](https://www.udemy.com/course/valaxy-devops/?referralCode=8147A5CF4C8C7D9E253F)  
### Pre-requisites
1. EC2 instance with Java 11 
  - we have to create new EC2 instance for tomcate server

### Install Apache Tomcat
1. Install java-11 first
   ```sh
     [root@ip-172-31-36-125 ~]# amazon-linux-extras install java-openjdk11
     [root@ip-172-31-36-125 ~]# java -version
       openjdk version "11.0.13" 2021-10-19 LTS
       OpenJDK Runtime Environment 18.9 (build 11.0.13+8-LTS)
       OpenJDK 64-Bit Server VM 18.9 (build 11.0.13+8-LTS, mixed mode, sharing)
   ```
  
2. Download tomcat packages from  https://tomcat.apache.org/download-80.cgi onto /opt on EC2 instance
   
   > Note: Make sure you change `<version>` with the tomcat version which you download. 
   ```sh 
   # to change dir to opt
   [root@ip-172-31-36-125 ~]# cd /opt
   
   # to download tomcate server in opt
   [root@ip-172-31-36-125 opt]# wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.63/bin/apache-tomcat-9.0.63.tar.gz
   
   #to extract tomcate file and rename file
   [root@ip-172-31-36-125 opt]# tar -xvzf apache-tomcat-9.0.63.tar.gz
   [root@ip-172-31-36-125 opt]# mv apache-tomcat-9.0.63 tomcat
   
   ```
   
3. Starting tomcat server
   ```sh
     # go to bin dir
     [root@ip-172-31-36-125 opt]# cd tomcat
     [root@ip-172-31-36-125 tomcat]# cd bin/
     
     # executing startup.sh file
     [root@ip-172-31-36-125 bin]# ./startup.sh
     
     access tomcat application from browser on port 8080  
     http://<Public_IP>:8080
   ```
4. Changing tomcat server port
   -   Using unique ports for each application is a best practice in an environment. But tomcat and Jenkins runs on ports number 8080. Hence lets change tomcat port number to 8090. Change port number in conf/server.xml file under tomcat home

# ads
