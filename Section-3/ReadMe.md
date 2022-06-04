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
4. Make a tomcate server globally accessible
   - currently our tomcate server work only in folder where its install so to make that server globally available make change in contex.xml file
   ```sh
   #to know the location of contex.xml file
    [root@ip-172-31-36-125 ~]# find / -name context.xml
       /opt/tomcat/conf/context.xml
       /opt/tomcat/webapps/examples/META-INF/context.xml
       /opt/tomcat/webapps/host-manager/META-INF/context.xml
       /opt/tomcat/webapps/manager/META-INF/context.xml
       
   # we have to edit last two file
   # edit "/opt/tomcat/webapps/host-manager/META-INF/context.xml" file
   [root@ip-172-31-36-125 ~]# vi /opt/tomcat/webapps/host-manager/META-INF/context.xml
   
   we have to comment out <!-- <value>...</value> --> tag after editing our file lookes like
   
   <?xml version="1.0" encoding="UTF-8"?>
   <!--
     Licensed to the Apache Software Foundation (ASF) under one or more
     contributor license agreements.  See the NOTICE file distributed with
     this work for additional information regarding copyright ownership.
     The ASF licenses this file to You under the Apache License, Version 2.0
     (the "License"); you may not use this file except in compliance with
     the License.  You may obtain a copy of the License at

         http://www.apache.org/licenses/LICENSE-2.0

     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License.
   -->
   <Context antiResourceLocking="false" privileged="true" >
     <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                      sameSiteCookies="strict" />
   <!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
   allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
     <Manager sessionAttributeValueClassNameFilter="java\.lang\.   (?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
   </Context>
   ~
   ~
   ~
   ~

   :wq
   
   
   #edit "/opt/tomcat/webapps/manager/META-INF/context.xml"
   [root@ip-172-31-36-125 ~]#  vi /opt/tomcat/webapps/manager/META-INF/context.xml
   
   after edit 
   
   <?xml version="1.0" encoding="UTF-8"?>
   <!--
     Licensed to the Apache Software Foundation (ASF) under one or more
     contributor license agreements.  See the NOTICE file distributed with
     this work for additional information regarding copyright ownership.
     The ASF licenses this file to You under the Apache License, Version 2.0
     (the "License"); you may not use this file except in compliance with
     the License.  You may obtain a copy of the License at

         http://www.apache.org/licenses/LICENSE-2.0
 
     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License.
   -->
   <Context antiResourceLocking="false" privileged="true" >
     <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                      sameSiteCookies="strict" />
     <!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
     allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
     <Manager sessionAttributeValueClassNameFilter="java\.lang\.  (?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
   </Context>

   ~
   ~
   ~
   ~

   -- INSERT --  
   
   
   #Chek point
   # shut down and restart tomcate server
   [root@ip-172-31-36-125 ~]# cd /opt/tomcat/bin
   [root@ip-172-31-36-125 bin]# ./shutdown.sh
   [root@ip-172-31-36-125 bin]# ./startup.sh


   ```
   
5. Creating users credentials to access the tomcat server
   - To create new users we have to edit tomcat-user.xml file
     ```sh
      # go to conf dir
      [root@ip-172-31-36-125 tomcat]# cd conf/
      
      # editing tomcat-user.xml file
      [root@ip-172-31-36-125 conf]# vi tomcat-users.xml
      
      after edit 
      
      <?xml version="1.0" encoding="UTF-8"?>
      <!--
      Licensed to the Apache Software Foundation (ASF) under one or more
      contributor license agreements.  See the NOTICE file distributed with
      this work for additional information regarding copyright ownership.
      The ASF licenses this file to You under the Apache License, Version 2.0
      (the "License"); you may not use this file except in compliance with
      the License.  You may obtain a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

      Unless required by applicable law or agreed to in writing, software
      distributed under the License is distributed on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
      See the License for the specific language governing permissions and
      limitations under the License.
      -->
     <tomcat-users xmlns="http://tomcat.apache.org/xml"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
                 version="1.0">
     <!--
     By default, no user is included in the "manager-gui" role required
     to operate the "/manager/html" web application.  If you wish to use this app,
     you must define such a user - the username and password are arbitrary.

     Built-in Tomcat manager roles:
       - manager-gui    - allows access to the HTML GUI and the status pages
       - manager-script - allows access to the HTTP API and the status pages
       - manager-jmx    - allows access to the JMX proxy and the status pages
       - manager-status - allows access to the status pages only

     The users below are wrapped in a comment and are therefore ignored. If you
     wish to configure one or more of these users for use with the manager web
     application, do not forget to remove the <!.. ..> that surrounds them. You
     will also need to set the passwords to something appropriate.
     -->
     <!--
     <user username="admin" password="<must-be-changed>" roles="manager-gui"/>
     <user username="robot" password="<must-be-changed>" roles="manager-script"/>
   -- INSERT --

       # creating shortcut to run start and shutdown tomcat server
       [root@ip-172-31-36-125 conf]# ln -s /opt/tomcat/bin/startup.sh /usr/local/bin/tomcatup
       [root@ip-172-31-36-125 conf]# ln -s /opt/tomcat/bin/shutdown.sh  /usr/local/bin/tomcatdown

     ```
## ----------------------------------------------------------------------------------------

# Integrating tomcat with Jenkins
  1. Install `Deploy to container` without restart 
    - `Manage Jenkins` > `Manage plugin` > `available` > `Deploy to container` 
  
  2. Managing the credentials
     - `Manage Jenkins` > `Manage Credentials` > `Jenkins` > `Globale credentials` > `Add Credentials` 
           - Username : deployer
           - Password : deployer
           - ID : tomacat_deployer
           - Description : tomacat_deployer
  3. Creating new job  
     - `New Item` > `Enter an item name` > `Maven Project` > `OK` > `Description` > `Source Code Management : Git : Repository Url = <address of git repo>` > `Build : goals and option = clean install` > `Post-build Actions : Deploy war/ear to container : WAR/EAR Files = **/*.war` > `Container : Add Container = Tomcat8x : Tomcat Url = http://<public ip add>:<port>/` > `Apply & Save`
  
  4. Build Code
     - Build code using build now and open the console 
     - refresh the tomcate server we will get /webapp option click on it new jsp page will open with form in it

## ---------------------------------------------------------------------------------------------
# Automate building & deploy using Poll SCM
   - We want to automatic building of war files when we make a change in our code. This is possible by making change in `Build Triggers`. In Build Trigger there is lots of option currently we are going to use Poll SCM.
   1. Make change in build job
      - `Go to Job` > `configure` > `Build Triggers = select Poll SCM : in discription box write * * * * *` 
   
   - after making change make change in GitHub repository  
   

# ads
