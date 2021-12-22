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
   
   
   + continue to the end of ec2/vm launching steps

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
    + NB: SonarQube server details must be configured in the pom.xml file within the "properties" tag
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
   
//Declarative Pipeline
pipeline {
agent any
tools {
   maven "maven3.8.4"
   //JDK "openJDK"
   //Git "git31.1"
      }
/*
options {}
   */
triggers {
   pollSCM'*****'
   }
stages {
  stage('1.CodeClone'){
   agent {slave#, master, none, docker...and enter version}
   steps {
        git branch: 'stage', credentialsId: 'GitHubCredentials', url: 'https://github.com/LandmakTechnology/web-app'
       }
    }
  stage('2.MavenBuild'){
   agent {slave#, master, none, docker...and enter version}
   steps {
          sh "mvn clean package"
        }
    }
  stage('3.CodeQuality'){
   agent {slave#, master, none, docker...and enter version}
   steps {
          sh "mvn sonar:sonar"
       }
    }
  stage('4.uploadToNexus'){
   agent {slave#, master, none, docker...and enter version}
   steps {
          sh "mvn deploy"
       }
    }
  stage('5.Deploy2Tomcat'){
   agent {slave#, master, none, docker...and enter version}
   steps{
    sshagent(['32d5fb4f-d92f-4a10-9f12-2738eab55fcc']) {
    sh "scp -o StrictHostKeyChecking=no target/*war ec2-user@172.31.15.31:/opt/tomcat9/webapps/app"
    }
   }
   }
   post {
     always  {//use email syntax to generate email notification to be sent always}
     success {//use email syntax to generate email notification to be sent when ci/cd is successful}
     failure {//use email syntax to generate email notification to be sent when ci/cd fails and not successful}
   }
   }
   
```
+ Agent can be "any", "none", jenkins "master" or "slave"
+ Tools refers to the build tool version, the java version, the Git version etc (see the Global Tool Configuration in Jenkins-UI)
+ Options directive is applied where there are nuances when adding an agent to the top level or a stage level
+ "Step" is the most fundamental part of a Pipeline. Steps tell Jenkins what to do and serve as the basic building block for both Declarative and Scripted Pipeline syntax
+ Triggers - Use Declarative Directive Generator to select the appropraite build trigger. Then copy and paste the generated syntax
+ Stages are the same as in Scripted Pipeline
+ SSHagent for Tomcat  is a plugin for authentication and not a build agent like exit in the preceeding stages
+ Post lists the actions to be taken after CI/CD...typically Email Notification...configure email notification for each post action scenerios
   
   
## Jenkins Shared Libraries
+ A concept of having a common pipeline code in the version control system that can be used by any number of pipelines just by referencing it. 
+ Can be referenced by multiple projects that have similar ci/cd structure or steps that intersects. 
+ Eliminates code repetition
+ It optimises resources by leveraging exisiting defined pipelines for executing multiple concurent project without having to develop individual pipelines for each project.
+ Process Steps:
   + Step 1 - Create Shared Library and Directory Structure
   + Step 2 - Configure Shared Library
   + Step 2 - Use library in pipeline script
   
### Creating Shared Library and Directory Structure
+ Typical SCM directory structure to use Jenkins Shared Library
+ root 
   + src       #Groovy source files 
   + vars      #variable files to be referenced or called in the groovy pipeline script
   + resources #resource files (external libraries only)
   

### Configuring Jenkins Shared Libraries
+ Access Jenkins server UI Dashboard
+ Select "Manage Jenkins" 
+ Select "Configure Systems"
+ Scroll to locate "Global Pipeline Libraries"
+ Add preferred Library Name
+ Select Default Version/Branch
+ Check "Allow default version to be overridden"
+ Check "Include @Library changes in job recent changes"
+ Select Retrieval Method - "Modern SCM - recommended"
+ Select "Git" for Source Code Management
+ Paste the URL for the Jenkins Shared Library (the "vars" directory)
+ Add Git Credentials 
+ Save

   
### Using Jenkins Shared Libraries
+ Create a project on Jenkins-UI ("enter new item")
+ Select Pipeline
+ Select OK
+ Select "Pipeline Tab" in Configure to begin building pipeline
  + NB: JenkinsFile begins with "@Library('Project/LibraryName') _"
+ Create Declarative Pipeline
+ Typical Jenkins Shared Library has the following content/function:

```sh

def call(String stageName){
  
  if ("${stageName}" == "Build")
     {
       sh "mvn clean package"
     }
  else if ("${stageName}" == "SonarQube Report")
     {
       sh "echo Running Code Quality Report analysis"
       sh "mvn clean sonar:sonar"
     }
  else if ("${stageName}" == "Upload Into Nexus")
     {
       sh "mvn clean deploy"
     }
}

```
   
+ Typical Declarative Pipeline (Jenkins File) using Jenkins Shared Library
```sh
 @Library('LandmarkSS-sharedlibs') _
//common.groovy is the file name containing the Jenkins Shared Library being referenced. "common" is used in the steps below to correspond to the filename of the referenced library.
pipeline {
agent any 
tools {
    maven "maven3.8.4"

  }
stages{
stage('CheckoutCode'){
  steps{
    git 'https://github.com/LandmakTechnology/web-app'
}
}
stage("Build"){ 
  steps{
    common("Build")
}
}

stage("Execute SonarQube Report"){ 
  steps{
    common("SonarQube Report")
}
}
stage("Upload Artifacts Into Nexus"){ 
  steps{
    common("Upload Into Nexus")
}
}
//=========================
}// Stages Close
} // Pipeline Close
   
```
   

   
   
