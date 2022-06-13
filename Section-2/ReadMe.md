# Install Jenkins on AWS EC2
Jenkins is a self-contained Java-based program, ready to run out-of-the-box, with packages for Windows, Mac OS X and other Unix-like operating systems. As an extensible automation server, Jenkins can be used as a simple CI server or turned into the continuous delivery hub for any project.

### Follow this artical On [On YouTube Channel](https://youtu.be/ERR7cqW28FY)

### Prerequisites
1. EC2 Instance
    - With Internet Access
    - Security Group with Port 8080 open for internet
2. Java 11 should be installed

# Install Jenkins
You can install jenkins using the rpm or by setting up the repo. We will set up the repo so that we can update it easily in the future.

1. Get the latest version of jenkins from https://pkg.jenkins.io/redhat-stable/ and install
   ```sh
     sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
     sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
     amazon-linux-extras install epel 
     amazon-linux-extras install java-openjdk11
     yum install jenkins
   
     #on RedHat/CentOs 
     #yum install epel-release # repository that provides 'daemonize'
     #yum install java-11-openjdk-devel
     #yum install jenkins
  
### Start Jenkins
   ```sh
   # check jenkins service status
   service jenkins status
   
   # Start jenkins service
   service jenkins start
   
   # Stop jenkins service
   service jenkins stop

   # Setup Jenkins to start at boot,
   chkconfig jenkins on
   ```
### Accessing Jenkins
   By default jenkins runs at port `8080`, You can access jenkins at
   ```sh
   http://YOUR-SERVER-PUBLIC-IP:8080
   ```

http://YOUR-SERVER-PUBLIC-IP:8080

  #### Configure Jenkins
- The default Username is `admin`
- Grab the default password 
- Password Location:`/var/lib/jenkins/secrets/initialAdminPassword`
- to get password use: cat /var/lib/jenkins/secrets/initialAdminPassword
- `Skip` Plugin Installation; _We can do it later_
- Change admin password
   - `Admin` > `Configure` > `Password`
- Configure `java` path
  - `Manage Jenkins` > `Global Tool Configuration` > `JDK`  
- Create another admin user id


### Test Jenkins Jobs
1. Create “new item”
1. Enter an item name – `My-First-Project`
   - Chose `Freestyle` project
1. Under the Build section
	Execute shell: echo "Welcome to Jenkins Demo"
1. Save your job 
1. Build job
1. Check "console output"


# --------------------------------------------------------------

# Intigrating Git with Jenkins
Git is one of the most popular tools for version control system. you can pull code from git repositories using jenkins if you use github plugin. 


#### Prerequisites
1. Jenkins server 

#### Install Git on Jenkins server
1. Install git packages on jenkins server
   ```sh
   yum install git -y
   ```

#### Setup Git on jenkins console
- Install git plugin without restart  
  - `Manage Jenkins` > `Jenkins Plugins` > `available` > `github`

- Configure git path
  - `Manage Jenkins` > `Global Tool Configuration` > `git`
  - In Path to Git executable
   - we can use `git` only
   - if above not work then add git path (to get git path use comand in linux terminal # `whereis git`)
 
 
# Run Jenkin Job to Pull Code from GitHub

#### creating new Job
   - `New Item` > `Enter an item name = PullingCodeFromGitHub` > `select Free Style Project` > `click on OK` > `Description = Pulling code from Github` > `Source Code Management = select Git = Repository URL = https://github.com/Dhikale27/hello-world-devops.git` > `Apply & Save`
	
#### executing the job
   - `Build Now` > `click in Build History` we will get the console with all activity details

## -----------------------------------------------------------------------------------------

#  Install & configure Maven build tool on Jenkins
Maven is a code build tool which used to convert your code to an artifact. this is a widely used plugin to build in continuous integration


#### Prerequisites
1. Jenkins server


#### Install Maven on Jenkins
1. Download maven packages https://maven.apache.org/download.cgi onto Jenkins server. In this case, I am using /opt/maven as my installation directory
 - Link : https://maven.apache.org/download.cgi
   ```sh
   - Go to linux terminal (in my case MobaXterm terminal)
        [root@ip-172-31-34-251 ~]# cd /opt
        [root@ip-172-31-34-251 opt]# ll
          total 0
          drwxr-xr-x 4 root root 33 Apr 28 19:54 aws
          drwxr-xr-x 2 root root  6 Aug 16  2018 rh
     
     - downloading maven version 3.8.5 in this dir
       [root@ip-172-31-34-251 opt]# wget https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz
     
       [root@ip-172-31-34-251 opt]# ll
         total 8472
         -rw-r--r-- 1 root root 8673123 Mar  5 15:51 `apache-maven-3.8.5-bin.tar.gz`
         drwxr-xr-x 4 root root      33 Apr 28 19:54 aws
         drwxr-xr-x 2 root root       6 Aug 16  2018 rh
     
     - extract file and rename that file
     
       [root@ip-172-31-34-251 opt]# tar -xvzf apache-maven-3.8.5-bin.tar.gz
     
       [root@ip-172-31-34-251 opt]# mv apache-maven-3.8.5 maven
       [root@ip-172-31-34-251 opt]# ll
         total 8472
         -rw-r--r-- 1 root root 8673123 Mar  5 15:51 apache-maven-3.8.5-bin.tar.gz
         drwxr-xr-x 4 root root      33 Apr 28 19:54 aws
         drwxr-xr-x 6 root root      99 Jun  3 04:51 maven
         drwxr-xr-x 2 root root       6 Aug 16  2018 rh
       [root@ip-172-31-34-251 opt]#
       [root@ip-172-31-34-251 opt]#


	
2. Setup M2_HOME and M2 paths in .bash_profile of the user and add these to the path variable
  ```sh
   # to go root dir
   [root@ip-172-31-34-251 opt]# cd ~
   
   # to get hidden files
   [root@ip-172-31-34-251 ~]# ll -a
    
       total 20
       dr-xr-x---  3 root root 103 Jun  3 02:37 .
       dr-xr-xr-x 18 root root 257 Jun  3 02:37 ..
       -rw-r--r--  1 root root  18 Oct 18  2017 .bash_logout
       -rw-r--r--  1 root root 176 Oct 18  2017 .bash_profile
       -rw-r--r--  1 root root 176 Oct 18  2017 .bashrc
       -rw-r--r--  1 root root 100 Oct 18  2017 .cshrc
       -drwx------  2 root root  29 Jun  3 02:37 .ssh
       -rw-r--r--  1 root root 129 Oct 18  2017 .tcshrc
  
 
   
   # to get the path ex java-11
     [root@ip-172-31-34-251 jvm]# find / -name java-11*
       /usr/lib/jvm/java-11-openjdk-11.0.13.0.8-1.amzn2.0.3.x86_64
       /usr/share/doc/java-11-openjdk-11.0.13.0.8-1.amzn2.0.3.x86_64
     [root@ip-172-31-34-251 jvm]#

   
   # to edit the .bash_profile file
     [root@ip-172-31-34-251 jvm]# vi .bash_profile
   
   - to edit in vi editor we have to use command like
   	- to insert data : o (press o)
   	- to change command : press Esc 
	- to save data : w (press shift + : + w)
	- to save & quite : wq (press shift + : + wq)
	- to quite w/o save : q! (press shift + : + q!)
	
	
   
   - we get bash file after edit file look like as below
     ```sh
     # .bash_profile

     # Get the aliases and functions
     if [ -f ~/.bashrc ]; then
        . ~/.bashrc
     fi
     M2_HOME=/opt/maven
     M2=/opt/maven/bin
     JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.13.0.8-1.amzn2.0.3.x86_64

     # User specific environment and startup programs

     PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2`

     export PATHG

     ~
     ~
     ~
     ~
     ~
     ~
     ~
     -- INSERT --   
     
 
   
   - to load new edited file
     [root@ip-172-31-34-251 ~]# source .bash_profile
     
   
   - to check our changes
     [root@ip-172-31-34-251 ~]# echo $PATH
       /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/root/bin:/usr/lib/jvm/java-11-openjdk-11.0.13.0.8-1.amzn2.0.3.x86_64:/opt/maven:/opt/maven/bin
     [root@ip-172-31-34-251 ~]#
   
   
  - or we can do as below
      1. logoff and login to check maven version
  
      ```sh
        mvn --version
	```
```

    
    
So far we have completed the installation of maven software to support maven plugin on the jenkins console. Let's jump onto Jenkins to complete the remaining steps. 

#### Setup maven on Jenkins console

1. Install maven plugin without restart  
   - `Manage Jenkins` > `Jenkins Plugins` > `available` > `Maven Integration`

2. Configure maven path
   - `Manage Jenkins` > `Global Tool Configuration` > `Add JDK : Name = java-11 & JAVA_HOME = /usr/lib/jvm/java-11-openjdk-11.0.13.0.8-1.amzn2.0.3.x86_64` > `Add Maven : Name = maven-3.8.5 & MAVEN_HOME= /opt/maven` > `Apply & Save`


## -----------------------------------------------------------------------------------------------

# Building Java Project using Maven

- Creating new job
  - `New Item` > `Enter item Name` > `Maven Project` > `description` > `git: Repository Url = <repository url>` > ` Build : Gola and Options = clean install` > `Apply and Save`

- Building project (or executing the job)
  - `Build Now` > `Build history : click here to open the console` >  console will open with all activity detail
