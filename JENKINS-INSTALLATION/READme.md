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
+ Go to Jenkins-UI
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
 
 4. Install Plugins to communicate with Tomcat
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
 
# Automating build process (jobs) in Jenkins
## FreeStyle Project - 
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



## Pipeline - 
### Jenkins Master-Slave Architecture
+ Jenkins manages SDLC automation
+ Jenkins Master-Slave Architecture:-
   + provides flexiblity to continue CI/CD when the Jenkins server is unavailable
   + makes it possible to execute multiple CI/CD jobs that are required to run concurrently
   + distributes tasks between mutiple slave servers or executors
   + Jenkins Executors are agent that permit Jenkins to run jobs in Jenkins Slaves
   + in the master, it is required to install Jenkins, Java and an SSH Agent Plugin
   + only Java is required in the slave servers to establish communication with the master 
   + cummunication between master and slave is established using TCP/IP 
   + the master and slaves are executors. Hence, at any time, executors = > 2
   + the slaves are referred to as Unix agent
   + TCP - Transfer Control Protocol, a secured protocol is preferreed to UDP - User Datagram Protocol) 
   + TCP => Encrypted data | Secured | 3way handshake   (e.g. https, ssh, scp, rdp)
   + UDP => Clear Text | unsecured | single directional (e.g. http)


#### To install a Jenkins slave, 
   + launch an ec2 instance (t2-micro is ok) and install Java using AWS "User data"
   + "User data" is used to install packages while creating a server (vm or ec2)
   + copy and paste the following in "User data" AWS Step 3: Configure Instance Details
    
```sh
          sudo hostname slave
          sudo su - ec2-user
          sudo yum -y install unzip wget tree git
          sudo wget -c --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm
          sudo yum install jdk-8u131-linux-x64.rpm -y
```
   
   
   + conitnue to the end of ec2/vm launching steps

#### To Add and Configure slave(s) server(s) in Jenkins-UI
  + Jenkins-UI Dashboard
  + Select "Manage Jenkins"
  + Select "Manage Nodes and Clouds"
  + Select "New Node" (to Install sshagent plugin)
  + Add Node Name
  + Check to select "Permanent Agent"...OK
  + Add Desription
  + Add Number of executors
  + Add Remote root directory => location within slave server (/home/ec2-user/node1)
  + Under Launch method, Select "Launch agents via SSH"
  + Enter the public/private IP of the slave
  + Configure credentials
  + Add credentials 
  + For "Kind", Select "SSH Username with private key"
  + Enter Username (ec2-user)
  + Select Enter Private Key directly
  + Copy and paste CI/CD .pem key in dialogue box
  + Add
  + Select "Manually trusted key Verification Strategy" for Host Key Verification Strategy
  + Save
 
Restrict jobs to run on slave

***{SSH into slave from Jenkins master CLI}***






### Jenkins Scripted and Declarative Pipeline
+ Jenkins Pipelines Builds are either Scripted or Declarative:

1. Scripted --> Groovy Script
     - No Gui options
     - ONLY SCRIPTS
     
2. Declarative:
     - It uses some gui options
     - Scripts

### Scripted Pipeline - Jenkinsfile (Infrastructure as a Code - IaC)
+ Jenkinsfile is a pipeline script written in a language called Groovy. Hence it is a Groovy Script.
+ Builds are done in a node. A node is a server. 
+ Jenkins has a master-slave architechture
+ Slaves are also called nodes and agents (slave/agent/node)
+ When creating the project, select "Pipeline" and complete the initial set-up
   + General > Project Description 
   + Select Advanced Project Options
   + Under Pipeline Definition click the drop down to select "Pipeline Script" or "Pipeline Script from SCM"
   + Select "Pipeline"
   + If "Pipeline Script from SCM" was selected:
   + Define the SCM by selecting "Git" from the dopdown
   + Copy and paste the applicable github project repository URL
   + Add the credentials for github repository
   + Script Path - Enter the name of the Jenkinsfile save/created in the SCM
   + Select "Pipeline Syntax to begin development of groovy script
   + Save
+ To update the project/groovy script later, 
   + select the project
   + select "Configure"
   + Select "Pipeline"
+ Typical Groovy Script or Jenkinsfile is as follows:


```sh

node('master')                                     //ensure build and release is taking place in the right slave/node/agent
  {
   def mavenHome = tool name: 'maven3.6.3'   
        /* Defining the maven Home and version. ("def" = defines functions in Groovy Script). Would apply to all stages below
        */
  stage('1.git clone')
        {
        git credentialsId: 'GitCredentials', url: 'https://github.com/LandmakTechnology/maven-web-app'
        }                                               //from "Pipeline Syntax" for "Git"
  stage('2.maven-Build')
        { 
        sh '${mavenHome}/bin/mvn clean package'         //using the absolute path for executing the mvn goal
        }
 stage('3.CodeQualityReport')
        {
        sh '${mavenHome}/bin/mvn sonar:sonar'           //using the absolute path for executing the mvn goal
        }
 stage('4.UploadWarNexus')
        {
        sh '${mavenHome}/bin/mvn clean deploy'          //using the absolute path for executing the mvn goal
        }
 stage('5.DeployTomcat')
        {
        deploy adapters: [tomcat9(credentialsId: 'Tomcat_Credentials', path: '', url: 'http://3.85.28.18:7777/')], contextPath: null, war: '**/*.war'
        }                                           //from "Pipeline Syntax" for "Deploy to Container" using UDP
        
 stage('6.EmailN') 
        {
        emailtext body: '''Hello Everyone,
        Build from Shaphothan Pipeline Project.
        Osazee''', subject: 'Build Status', to: 'developers'
        
        }
 
  } 
  
  ```



Another example of Scripted Pipeline:

```sh

//scripted pipeline
node{
 def MHD = tool name: "maven3.8.4"
    stage('1.Initiation'){
    sh "echo Start of td deployment"
    }
    stage('2.CloneCode'){
    git branch: 'stage', credentialsId: 'GitHubCredentials', url: 'https://github.com/LandmakTechnology/web-app'
    }
    stage('3.buildMaven'){
    sh "${MHD}/bin/mvn package"
    }
    stage('4.CodeQuality'){
    //sh "${MHD}/bin/mvn sonar:sonar"
    }
    stage('5.UploadArtifacts'){
    //sh "${MHD}/bin/mvn deploy"
    }
    stage('6.Deploy2Stage'){
    sshagent(['32d5fb4f-d92f-4a10-9f12-2738eab55fcc']) {
    sh "scp -o StrictHostKeyChecking=no target/*war ec2-user@172.31.15.31:/opt/tomcat9/webapps/app"
    }
    }                                                                      //deploy using TCP

    stage('7.Approval'){
    timeout(time:5, unit:'DAYS'){
 			input message: 'Approval for production'
    }
    }
    stage('8.deployToProd'){
    sshagent(['32d5fb4f-d92f-4a10-9f12-2738eab55fcc']) {
    sh "scp -o StrictHostKeyChecking=no target/*war ec2-user@172.31.15.31:/opt/tomcat9/webapps/app.war"
    }                                                                     
    
    }                                                                    //deploy using TCP
}

```

Prepare or update groovy script as follows:
+ For Code Clone from Git, 
    + Select Pipeline Syntax  ==> 
    + Select "Git" in the "Sample Step" dropdown menu, 
    + Enter the GitHub Repo URL and complete other required info. 
    + Generate the Pipeline Script and copy the output. 
    + Paste the copied output inside the curly bracket for the respective node stage
+ For Build using maven, 
    + Write script command to execute "mvn clean package" as an absolute path for executing the mvn goal. 
    + The function for maven home directory and the version for build must be defined before the first stage action
+ For Code Quality using SonaQube, 
    + Write script command to execute "mvn sonar:sonar" as an absolute path for executing the mvn goal. 
    + NB: SonarQube server details must be configured in the pom.xml file within the <properties> tag
+ For Artifact Upload in Nexus, 
    + Write script commaand to execute "mvn sonar:sonar" as an absolute path for executing the mvn goal. 
    + NB: Nexus server details must be configured in the pom.xml file within the <distribution management> tag
+ For Deploying to Tomcat, 
   + Select Pipeline Syntax  ==> 
   + Select "Deploy to a Container" in the "Sample Step" dropdown menu, 
   + Enter the source of file to deploy ("target/*.war) and 
   + Select "Add Container" to Specify the Tomcate Version 
   + Add Tomcat credentials
   + Add Tomcat URL (http://ipAddress:PortNumber)
   + Generate the Pipeline Script and copy the output
   + Paste the copied output inside the curly bracket for the respective node stage
   + NB: When using the scp to copy package to Tomcat, an sshagent is added to the groovy script to provide authentication into Tomcat server
   + NB: When using the scp to copy package to Tomcat, go to Tomcat server CLI and change permissions recursively to the tomcat9 or webapps directory 
+ For Email Notification, 
   + Select Pipeline Syntax  ==> 
   + Select "Extended Email" in the "Sample Step" dropdown menu, 
   + Enter or Add email recipients, Email Subject, Email Body
   + Generate the Pipeline Script and copy the output
   + Paste the copied output inside the curly bracket for the respective node stage

   
Select/Apply the appropriate Build Trigger
   
   

### Declarative Pipeline
+ Declarative Pipeline also use the groovy script for set up. 
+ Declarative Pipeline is so powerful that you can use multiple agents in one pipeline script

```sh 
pipeline {
agent any
tools {
   maven "maven3.8.4"
   //JDK "openJDK"
   //Git "git31.1"
      }
options {
     triggers
      }
stages {
  stage('1.CodeClone'){
    steps{
     agent ([//enter agent name if different from from the agent defined above, i.e if "none" is specified for agent above])
        {
    git branch: 'stage', credentialsId: 'GitHubCredentials', url: 'https://github.com/LandmakTechnology/web-app'
        }
      }
    }
  stage('2.MavenBuild'){
      steps{
     agent ([//enter agent name if different from from the agent defined above, i.e if "none" is specified for agent above])
        {
     sh "mvn clean package"
        }
      }
    }
  stage('3.CodeQuality'){
     steps{
     agent ([//enter agent name if different from from the agent defined above, i.e if "none" is specified for agent above])
        {
     sh "mvn sonar:sonar"
        }
      }
    }
  stage('4.uploadToNexus'){
   steps{
     agent ([//enter agent name if different from from the agent defined above, i.e if "none" is specified for agent above])
        {
     sh "mvn deploy"
        }
      }
    }
  stage('5.Deploy2Tomcat'){
   steps{
    sshagent(['32d5fb4f-d92f-4a10-9f12-2738eab55fcc']) {
    sh "scp -o StrictHostKeyChecking=no target/*war ec2-user@172.31.15.31:/opt/tomcat9/webapps/app"
    }      
     }
   }
   post {
     always
     success
     failure
   }
   }
   
```
+ agent can be  ==? any, none, jenkins master or slave
+ tools refers to the build tool version, the java version, the Git version etc (see the Global Tool Configuration in Jenkins-UI)
+ Options lists the build triggers
+ Stages are the same as in Scripted Pipeline
+ Post lists the actions to be taken after CI/CD...typically Email Notification...states whaen email notification should be sent

## Updating Credentials in Jenkins
+ Go to Jenkins Dashboard
+ Select "Manage Jenkins"
+ Select "Manage Credentials"
+ Select the applicable tool for which credential update is required.
+ For GitHub, select change password and enter Personal Access Token (PAT) created in GitHub



## Other Configurations in Jenkins
+ Discard Old Build
+ Disable this project 
    + when there is scheduled maintainance of servers 
    + when database backup is scheduled
+ Delete workspace before build starts 
+ Add timestamps to the Console Output 
    + run the following in the Jenkins CLI:
    ```sh
    sudo timedatectl set-timezone America/New_York
    ```
    
    
### Plugin Management
+ Plugins extends the functionality of Jenkins server
+ Jeknins Dashboard ---> Manage Plugins ---> Install Plugins ---> Configure System  or Project then Configure ---> etc
+ For JaCoCO Plugin:
  + Jeknins Dashboard ---> Manage Plugins ---> Install Plugins ---> Configure ---> Post-Build Actions ---> Add Post-Build Actions ---> "Record JaCoCo coverage report" ---> Update Post-Build Actions



### Plugins in Jenkins
+ JACOCO ---> Java Code Coverage, similar to Code Coverage quality gate in SonaQube
+ Deploy to container  --- > deploys applications in Tomcat/GlassFish/JBoss servers
+ eploy WebLogic ---> deploys applications in WebLogic servers
+ Maven Integration (Maven Project)
+ Safe Restart  ---> would not intrupt when there are jobs running.  
+ Next Build Number
+ Build Name Setter
+ SSH Agent
+ Email Extension
+ SonarQube Scanner
+ Audit Trail Plugin ---> set up in JHD (/var/lib/jenkins/)
+ Schedule Build
+ Artifactory Plugin
+ Cloud Foundry
+ Blue Ocean
+ Publish Over SSH :   jenkins --> ansible --deployment
+ ThinBackup
+ Convert To Pipeline
+ Job import plugin --->  jenkins migration 


### Jenkins Variables and Parameter
+ Variables:
  + boolean variables (True or False) ---> Jenkins_installed=true
  + intergers are numbers --->  port=80 
  + float (integer plus decimal) --->  price=100.50
  + string variables -->   name='simon'
  + list --->  team_numbers = ['simon','paul','john']

+ Build with parameters  :
  + $Name 
  + $branchName
  
  + Create Project in Jenkins Dashboard
  + Select Configure
  + Under General Tab, Select "This project is parameterized"
  + Select dropdown to choose parameter
  + Select "Choice Parameter" -- (1)
  + Enter "branchName" for Choice Parameter "Name"
  + Enter master, dev, and stage in Choice Parameter "Choices" 
  + Enter Description  - "This will dynamically allows selection of branch for build purposes"
  + Select "String Parameter" --- (2)
  + Enter "Name" and Default Value
  + Enter Description - "Please Identify Yourself"


## Jenkins Home Directory (JHD)
/var/lib/jenkins 
+ jobs --->  Config.xml/logs 
+ workspace 
+ tools -->     maven3.6.0 /  maven3.8.4
+ plugins 
+ secrets 
+ users --> users.xml 



## How to troubleshoot issues with Jenkins server
  + To trace issues within the jenkins server:
  ```sh 
       cat  /var/log/jenkins/jenkins.log 
  ```
  
  + When switching user to Jenkins and "sudo su -jenkins" doesn't work, run the following commands and update the shell for Jenkins user from bin/false to bin/bash
  ```sh
       sudo vi /etc/passwd 
  ```
 
+ To verify the Linus distibution 
 ```sh
      hostnamectl
 ```
+ Changing default port on RHEL or CentOS
   ```sh
     sudo vi /etc/sysconfig/jenkins     
   ```
+ Changing default port on UBUNTU   
  ```sh
     sudo vi /etc/default/jenkins
  ```  
  

    
    
## Jenkins Security
The seurty of Jenkins can be enhanced or maintained through the following:

1. Changing jenkins default configurations (port number, JHD)
    + JHD = /var/lib/jenkins 
          + /app/jenkins 
  
2. Using a proxy (HAProxy) server to access jenkins 
     + Team--> HAProxy/Nginx --> Jenkins 
     
3. Maintaining a strong password Policy 
     + special characters, expiry date, case sensitive (lower, upper letters), numbers

4. Role Based Access Control (RBAC) for User mgt:

5. Using an LDAP server for user mgt 


## RESTFUL API 
APM ==> Application Performance Monitoring
  + Monitoring and learning from 'live site'
    - Diagnostics and error reporting
    - usage = zelle / intertact = 30 millions 
    - Notifications on application performance
  + Rules for application performance and availability
    - High availability
    - Automated scale up/down or out/in


APM Tools ==> 
 + APM tools allow you to target bottleneck swith your applications framework
 + New Relic is the reigning market leader which lets you pinpoints precisely where and when bottlenecks are occurring
 + AppDynamics is also a great tool, enabling you to monitor Java, .NET, PHP, and Nodejs applications
 + Compuware APM & Boundary are enterprise-geared APM tools which give you a clear view of the user experience, offering metrics like data transactions performance and user requests
    + Dynatrace
    + CloudWatch and SNS 

Application Monitoring ==>
   + Hypothesis driven development  requires telemetry = servers (cpu <70% and memory >75%)         
   + Proactive (not reactive) action

Types of monitoring ===>
  - Usage
  - Availability
  - Performance
  - Custom telemetry

      
