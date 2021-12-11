#  **<span style="color:green">Landmark Technologies.</span>**
### **<span style="color:green">Contacts: +1437 215 2483<br> WebSite : <http://mylandmarktech.com/></span>**
### **Email: mylandmarktech@gmail.com**



## Jenkins Installation And Setup In AWS EC2 Redhat Instance.
### Prerequisite
+ AWS Acccount.
+ Create Redhat EC2 t2.medium Instance with 4GB RAM.
+ Create Security Group and open Required ports.
   + 8080 got Jenkins, ..etc
+ Attach Security Group to EC2 Instance.
+ Install java openJDK 1.8+ for SonarQube version 7.8

## Install Java JDK 1.8+ as Jenkins pre-requisit
## Install other softwares - git, unzip and wget

``` sh
sudo hostname ci
sudo su - ec2-user
sudo yum -y install unzip wget tree git
sudo wget -c --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm
sudo yum install jdk-8u131-linux-x64.rpm -y
```
##  Add Jenkins Repository and key
```sh
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
cd /etc/yum.repos.d/
sudo curl -O https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

## Install Jenkins
```sh
sudo yum -y install jenkins  --nobest
```
## start Jenkins  service and verify Jenkins is running
```sh
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```
## Access Jenkins from the browser
```sh
#public-ip:portNumner (8080)
curl ifconfig.co 
```
## get jenkins initial admin password to access browser
```sh
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

## Customize Jenkins on Browser
+ Select "install suggested plugins"
+ Create First Admin User

# Integrate Jenkins with build and deployment tools
+ Create project in Jenkins. On Jenkins dashboard:
  + Click "New item"
  + Enter project name
  + Select Freestyle or Pipeline and click "OK"
  + On General Tab, enter project decriciption
  + Select "Save"
  
## Integrate Jenkins with GitHub
+ Go to JEnkins-UI
+ In the created project, select "Configure"
+ Complete Source Code Manaagement definition:
 + Copy project URL from GitHub and past in Jenkins  SCM Reposirory URL
   + For Private GitHub Repositories, Set up authentication by entering existing GitHub credentials (username, password or PA Tokens)
 + Select github branch from which to "clone" (master by default)
 + "Save"

## Integrate Jenkins with Maven
+ Go to Jenkins-UI
+ On the Dashboard, select "Manage Jenkins"
+ Select "Global Tools Configuration"
+ Select "Add Maven" and set name to match the preferred version of Maven
+ Save
 
+ In the created project, select "Configure"
+ Select "Build"
+ Select "Add Build Step"
+ Select "Invoke top-level Maven targets"
+ Set Maven Version 
+ Set Goal for build --> clean package
+ save
+ Build Now...
    
## Integrate Jenkins with SonaQube
+ NB: SonaQube server must be running and SonaQube Service must be running
+ Open required ports (9000) in SonarQube server firewall or SG to allow traffic from Jenkins server
+ Run the following on the SonaQube server CLI
```sh
sudo hostname -i | telnet 9000
```
+ Go to project repository in GitHub
+ Open the project file pom.xml
+ Scroll to "Properties" tag
  + Update URL with the current SonaQube server ==> http:public/privateIP:9000
  + Update login username and password to be same as that used for Sonaube-UI
+ Commit the changes.

+ Go to Jenkins-UI 
+ Select "Configure"
+ Select "Build"
+ Select "Add Build Step"
+ Select "Invoke top-level Maven targets"
+ Set Maven Version 
+ Set Goal for build --> sonar:sonar
+ save
+ Build Now...

## Integrate Jenkins with Nexus
1. Create repos in Nexus-UI to upload artifacts
+ Go to Nexus-UI
+ Create repos (snapshot and release) in nexus-UI to upload artifacts

2. Modify "distributionManagement" tag in pom.xml
+ Go to project repository in GitHub
+ Open the project file pom.xml
+ Modify 'distributionManagement' tag with nexus repos details in pom.xml
+ Commit changes

3. Authenticate Maven/Jenkins in Nexus server by Modifying Settings.xml file:
+ Go to Jenkins CLI
+ Access Maven installed within Jenkins:

``` sh
cd /home/tools/hudson.tasks.Maven_MavenInstallation/maven3.8.2/conf/settings.xml
```
+ Search for "servers" tag
+ Just before the closing of servers tag, insert the following:

``` sh
<server>
     <id>nexus</id>
     <username>admin</username>
     <password>admin@123</password>
   </server> 
 ```
4. Set Default Goal in Jenkins-UI
+ Go to Jenkins-UI 
+ Select "Configure"
+ Select "Build"
+ Select "Add Build Step"
+ Select "Invoke top-level Maven targets"
+ Set Maven Version 
+ Set Goal for build --> deploy
+ save

5. Confirm required ports are open in Nexus server to allow traffic from Jenkins server.

+ Build Now...   


## Integrate Jenkins with Tomcat

1. Add a Tomcat user in /tomcat9/conf/tomcat-users.xml
 + Go to Tomcat CLI
 ```sh
  sudo vi /opt/tomcat9/conf/tomcat-users.xml
 ```
 + add the following user and roles to the tomcat-users.xml file...before the closing tag
 ```sh
 <user username="Osazee" password="admin123" roles="manager-gui,admin-gui,manager-script"/>
 <user username="class26" password="admin123" roles="manager-gui,admin-gui"/>
  ```
2. Change Tomcat port number if required within:
  ```sh
  vi /opt/tomcat9/conf/server.xml
  ```
3. Enable application deployment in Tomcat i.e. allow external access to Tomcat-UI
  ```sh
  vi /opt/tomcat9/webapps/manager/META-INF/context.xml
  ```
  + comment on "Valve className" tag

  + Go to Tomcat UI using publicIP:PortNumber using username and password from Step 1 above to view deployment status
 
2. Install Plugins to communicate with Tomcat
 + Go to Jenkins-UI
 + Select "Manage Jenkins"
 + Select "Manage Plugins"
 + Select "Deploy to Container" under "Available" tab 
 + Select "Install without restart"
 + Select project/job
 + Select "Configure"
 + Select "Post-Build Action" tab
 + Select "Add post-build action" dropdown
 + Select "Deploy war/ear to a container"
 + Set location of file to deploy by entering "target/*war"
 + Select "Add Containers"
 + Choose the version of Tomcat installed
 + Add Tomcat credentials - Username and password set in Step 1 above (must have "manager-script" role => RBAC Role Based access Control)
 + Add Tomcat URL htttp://publicIP:PortNumber
 + Save
 + Build Now...

+ View deployed application using the contextpath "htttp://publicIP:PortNumber/artifactName"

## Set up Email Notification
1. Install and Configure Email Plugins
 + Log into Jenkins server UI  = http://18.212.53.48:8080/
 + Select "Manage Jenkins" 
 + Select "Manage Plugins"
 + Select "Available" or "Installed" tab to Install "Email Extension Plugin" 
 + Select "Manage Jenkins"
 + Select "Configure System"
 + Select "Extended Email Notification"
 + Fill in required info to complete configuration
     + Set enterprise SMTP or use smtp.gmail.com 
       + set project email: td-project@gmail.com 
       + set project email password: admin@123
       + set email content using variables
       + set receipients: all Developers 
     + save 
     
2. Configure Editable Email Notification
 + Go to Jenkins-UI
 + Select project/job
 + Select "Configure"
 + Select "Post-Build Action" tab
 + Select "Add post-build action" dropdown
 + Select "Editable Email Notification" from dropdown
 + Fill in required info for outgoing, incoming email, attached logs, add triggers etc
 + Save
 
## Automating build process (jobs) in Jenkins
Builds can be accomplished in 6 ways (1 Manual and 5 Automated):
  + Build Now -- Botton on Jenkins UI -- This process is manual
  + Trigger Builds Remotely -- Say from a bash shell script
  + Build Periodically -- Entire CI/DC proceses are scheduled based on specific time interval/timer set using crontable. Usually do not relate to the application being developed. Suitable for database backup, server monitoring, installing updates, server patching to check if servers are running optimally, etc
     + Go to Jenkins-UI
     + Select project/job
     + Select "Configure"
     + Select "Build Triggers" tab
     + Check "Build Periodically"
     + Enter schedule in dialog box using definition from crontable 
     + Save
  + Poll SCM ---> Jenkins will query Github project repository at specific time interval and check for new commits/versions. Jenkins pull changes and trigger the entire CI/DC proceses if new commits/versions exist in GitHub (SCM). Build is initiated based on timer/schedule and SCM changes
     + Configure Github Webhook as follows:Go to Jenkins-UI
      + Select project/job
      + Select "Configure"
      + Select "Build Triggers" tab
      + Check "Poll SCM"
      + Enter schedule to define frequency of Jenkins query of GitHub in the dialog box using definition from crontable 
      + Save
  + Github-Webhook -- Github will push to changes to jenkins when a new branch created or new commits / version are made in GitHub. Entire CI/DC proceses is initiated by changes in the SCM
     +  Configure Github Webhook as follows:
      +  Go to SCM (GitHub)
      +  Open project repository
      +  Go to Settings
      +  Select "Webhoocks" from settings options
      +  Click "Add webhook"
      +  Set Payload URL as: http://JenkinsIPAddress:portNumber/github-webhook/
      +  Set Content type to "application/json"
      +  Select "Just the push event" or any other applicable
      +  Select "Add webhook"
  + Build other projects -- The end of one job triggers the start of another



## Updating Credentials in Jenkins
+ Go to Jenkins Dashboard
+ Select "Manage Jenkins"
+ Select "Manage Credentials"
+ Select the applicable tool for which credential update is required.
+ For GitHub, select change password and enter Personal Access Token (PAT) created in GitHub
