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
   ```
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

### creating new Job
	- `New Item` > `Enter an item name = PullingCodeFromGitHub` > `select Free Style Project` > `click on OK` > `Description = Pulling code from Github` > `Source Code Management = select Git = Repository URL = https://github.com/Dhikale27/hello-world-devops.git` > `Apply & Save`
	
### executing the job
	- `Build Now` > `click in Build History` we will get the console with all activity details
