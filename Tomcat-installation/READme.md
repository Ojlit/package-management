## Apache Tomcat Installation And Setup In AWS EC2 Redhat Instance.
### Step 1. Prerequisite - Create and Set Up Server
+ AWS Acccount.
+ Create Redhat EC2 T2.micro Instance.
+ Create Security Group and open Tomcat ports or Required ports.
   + 8080 ..etc
+ Attach Security Group to EC2 Instance.
+ Install java openJDK 1.8+

### Step 2. Install Java JDK 1.8+ & Tomcat version 9.0.55

``` sh
#!/bin/bash
# TOMCAT.SH
# Steps for Installing tomcat9 on rhel7&8
# install Java JDK 1.8+ as a pre-requisit for tomcat to run.
# https://github.com/LandmakTechnology/package-management/tree/main/Tomcat-installation
cd /opt 
sudo yum install tree git wget -y
sudo yum install java-1.8.0-openjdk-devel -y
# Download and extract tomcat software
sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.55/bin/apache-tomcat-9.0.55.tar.gz
sudo tar -xvf apache-tomcat-9.0.55.tar.gz
sudo rm apache-tomcat-9.0.55.tar.gz
# Rename Tomcat software for easy of use and good naming convention
sudo mv apache-tomcat-9.0.55 tomcat9
#Assign executable persmissions recursively to access the bin directory
sudo chmod 777 -R /opt/tomcat9
sudo sh /opt/tomcat9/bin/startup.sh
# create a soft link to start and stop tomcat from anywhere 
sudo ln -s /opt/tomcat9/bin/startup.sh /usr/bin/starttomcat
sudo ln -s /opt/tomcat9/bin/shutdown.sh /usr/bin/stoptomcat
sudo starttomcat
echo "end on tomcat installation"
```
chech that Tomcat is running  as follows:
```sh
ps -ef | grep tomcat
```

### Step3. Tomcat Server Configuration 

+ access tomcat server on the browser with http://publicIP:PortNumber
+ select "Manager App"
+ Enable external access to Tomcat server 
   + vi /opt/tomcat9/webapps/manager/META-INF/context.xml
   + comment on <Valve className...> to remove restriction for external excess
+ Add Tomcat users and Configure username and password
   + vi /opt/tomcat9/conf/tomcat-user.xml
   + advance to last line, before the closing tag for tomcat users
   + add the following: <user username="Osazee" password="admin" roles="manager-gui,admin-gui,manager-scipt"/>
   + add more users following the step above
   + Reattempt access to tomcat server on the browser using the credentials created in the tomcat-user.xml file
+ To change the Tomcat default port number:
   + advance to the Tomcat home directory (/opt/tomcat9 or similar)
   + locate and cd into the "conf" directory
   + vi into the server.xml file and change port number
   + or vi /opt/tomcat9/conf/server.xml and change port number
   + sudo stoptomcat
   + sudo starttomcat
   + Open the new port number in the AWS security group
  
+ To enable PasswordAuthentication 
   + sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
   + or vi /etc/ssh/sshd_config and enable PasswordAuthentication
   + set PasswordAuthentication to "yes"
   + sudo service sshd restart 
  
  

	
	

/opt/tomcat9/conf/context.xml

 vi /opt/tomcat9/webapps/manager/META-INF/context.xml
  
  vi /opt/tomcat9/conf/tomcat-user.xml  # to add user
  
	
	username YourName password=PassWord   roles=manager-gui

### Step4. Deploying Jobs from Build Server (Maven) to Tomcat
+ For standalone applications built in maven, deployment can only be done in maven using java -jar packageName.jar
+ To deploy standalone applications, Java JDK 1.7+ must be running
+ Tomcat will only deploy maven-web-applications (.war)
+ JBoss/WildFly will deploy both maven-web-applications (.war) and maven-enterprise-applications (.ear)

#### Steps:
 + Assume Maven Server is installed, and configured:
 + Clone source code and build script from GitHub Repository
 + Build Package in maven 
 + Securely copy build artifcat to the webapps directory in Tomcat or another designated directory
 + 



### Step5. Security for Tomcat using Proxy

	
