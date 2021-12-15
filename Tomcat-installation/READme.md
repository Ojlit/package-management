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

+ Access tomcat server on the browser with http://publicIP:PortNumber
+ Select "Manager App"
+ Enable external access to Tomcat server 
   + vi /opt/tomcat9/webapps/manager/META-INF/context.xml
   + comment on <Valve className...> to remove restriction for external access
	
+ Add Tomcat users and Configure username and password
   + vi /opt/tomcat9/conf/tomcat-user.xml
   + advance to last line, before the closing tag for tomcat users
   + add the following: <user username="Osazee" password="admin" roles="manager-gui,admin-gui,manager-scipt"/>
   + add more users following the step above
   + Reattempt access to tomcat server on the browser using the credentials created in the tomcat-user.xml file
	
+ Assign Password to ec2-user or any other user
   + This is required for authentication when scping from maven
   + passwd ec2-user
	
+ Enable PasswordAuthentication 
   + sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
   + or vi /etc/ssh/sshd_config and enable PasswordAuthentication
   + set PasswordAuthentication to "yes"
   + sudo service sshd restart 
	
+ Change the Tomcat default port number to increase security:
   + advance to the Tomcat home directory (/opt/tomcat9 or similar)
   + locate and cd into the "conf" directory
   + vi into the server.xml file, locate the "<Connector port"> tag and change port number
   + or vi /opt/tomcat9/conf/server.xml, locate the "<Connector port"> tag and change default port
   + sudo stoptomcat
   + sudo starttomcat
   + Open the new port number in the AWS security group
  

## Step4. Deploying Jobs from Build Server (Maven) to Tomcat
+ For standalone applications built in maven, deployment can only be done in maven using java -jar packageName.jar
+ To deploy standalone applications, Java JDK 1.7+ must be running
+ Tomcat will only deploy maven-web-applications (.war)
+ JBoss/WildFly will deploy both maven-web-applications (.war) and maven-enterprise-applications (.ear)

#### Steps:
 + Assume Maven Server is installed, and configured:
   + create directory for project
   + cd into directory
 + Clone source code and build script from GitHub Repository
   + git clone https://github/githubUserName/project-file-in-github
 + Build Package in maven
   + cd into the project file (cloned from Github)
   + mvn clean package
 + Securely copy build artifcat to the webapps directory in Tomcat or another designated directory
   + copy .war file from maven target directory into Tomcat webapps directory
   + Execute the following secure copy command while in the maven target directory
   + scp war-file-name ec2-user@TomcatIP:/location-of-deployment-in-the-Tomcat-server
   + scp war-file-name ec2-user@TomcatIP:/opt/tomcat9/webapps

====================================================================	
+ How are hotfixes or hot deployments managed in tomcat? ==> Reload
+ How to improve tomcat server performance? ==> 
   + Avoid multiple deployments tomcat server performance.
   + Increase the heap Size from 64MB to 256MB	
====================================================================

	
	
## Security for Tomcat using Proxy - NGINX
	
### Step 1. Prerequisite - Create and Set Up Server
+ AWS Acccount.
+ Create Redhat EC2 T2.micro Instance.
+ Create Security Group and open Required ports 80.
+ Attach Security Group to EC2 Instance.

### Step 2. Prerequisite - Assume Deployment Servers Exist or Create
+ Creat 3 tomcat servers (Redhat8) - EC2 T2.micro Instance in AWS
+ Create Security Group and open Required ports 8080.
+ Attach Security Group to EC2 Instance.
+ Configure all Servers as shown above
+ Log in to all servers using http://publicIP:portNumber
+ Deploy war packages in all servers from maven
	
	
### Step 3. Install and Configure Nginx as Webserver and Load Balancer
On Nginx Server CLI
	
```sh
sudo hostname nginx
#install nginx as root user 
sudu su -
#install epel repo in redhat server to complement the nginx directory system
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest- 8.noarch.rpm
sudo yum install nginx -y
#Enable the nginx service
sudo systemctl enable nginx
#Start the nginx service
sudo systemctl start nginx
#Check the nginx service status
sudo systemctl status nginx
```
	
+ Access the Nginx webserver administrator
+ copy the path to the nginx configuration file

```sh
#create backup by renaming the default nginx configuration file
sudo mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bkp

```
+ Create nginx cont file in default path and define proxy(reverse proxy) rules
+ Use the original conf file path/name create a new configuration file: 
	
```sh
	vi /etc/nginx/nginx.conf
```
	
+ copy and paste the following configuration that describes the expected performance of the created nginx server:

```sh

events{
worker_connections 1024;
}
http { keepalive_timeout 5;
upstream tomcatServers {
  keepalive 50;
  
  server Tomcat-1-IP:8080;
  server Tomcat-2-IP:8080;
  server Tomcat-3-IP:8080;

}
server {
   listen 80;
location / {
        proxy_set_header  X-Real-lP $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header  X-Forwarded-Proto $scheme;
        proxy_set_header        Host $host;
        proxy_pass http://tomcatServers;
}
}
}
	
```
Validate the nginx.conf by using below command TWICE
```sh 	
  nginx -t
```
	
+ Reload nginx configurations
```sh
  nginx -s reload
```

+ Restart nginx Service to Reflect the proxy rules.
```sh
    systemctl restart nginx
```
+ Access the nginx server on the browser:
	
+ Note: If you get 502 Bad Gateway, disable the existing Security-Enhanced Linux (SELinux) using the following on nginx CLI:
```sh
	setsebool -P httpd_can_network_connect true
```
+ Access the nginx server on the browser wit the context path (http://ngixIP:portNumber/artifactName)

+ NB: If there is a change in the distribution of the traffic to the servers, perhapse due to maintenance, upgrade or patching, the "Validate", "Reload" and "Restart" as shwon above must be executed

## Additional Study for Nginx

### Choosing a Load-Balancing Method:

+ Round Robin - Requests are distributed evenly across the servers, with server weights taken into consideration. This method is used by default

+ upstream backend { no load balancing method is specified for Round Robin server landmarktechtechnologies.app1.com;
server   landmarktechtechnologies.app2.com;
}

+ Least Connections - A request is sent to the server with the least number of active connections, again with server weights taken into consideration:

+ upstream backend { least_conn; server landmarktechnologies.app1.com; server landmarktechnologies.app2.com;
}

+ IP Hash - The server to which a request is sent is determined from the client IP address. In this case, either the first three octets of the IPv4 address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the same server unless it is not available.

+ upstream backend { ip_hash; server landmarktechnologies.app1.com; server landmarktechnologies.app2.com;
}


+ If one of the servers needs to be temporarily removed from the load-balancing rotation, it can be marked with the down parameter in order to preserve the current hashing of client IP addresses. Requests that were to be processed by this server are automatically sent to the next server in the group:

+ upstream backend { server landmarktechnologies.app1.com; server landmarktechnologies.app2.com; server  landmarktechnologies.app3.com down;
}

### Server Weights:
+ By default, NGINX distributes requests among the servers in the group according to their weights using the Round Robin method. The weight parameter to the server directive sets the weight of a server; the default is 1:

+ upstream backend {
server landmarktechnologies.app1.com weight=5; server landmarktechnologies.app2.com;
server landmarktechnologies.app3.com backup;
}
+ In the example, landmarktechnologies.app1.com has weight 5; the other two servers have the default weight (1), but the one with domain name landmarkntechnologies.app3.com is marked as a backup server and does not receive requests unless both of the other servers are unavailable. With this configuration of weights, out of every 6 requests, 5 are sent to landmarktechnologies.app1.com and 1 to landmarktechnologies.app2.com.

### Tuning Your NGINX Configuration

+ The following are some NGINX directives that can impact performance. As stated above, we only discuss directives that are safe for you to adjust on your own. We recommend that you not change the settings of other directives without direction from the NGINX team.

#### Worker Processes

+ NGINX can run multiple worker processes, each capable of processing a large number of simultaneous connections. You can control the number of worker processes and how they handle connections with the following directives:

+ worker_processes - The number of NGINX worker processes  (the default is 1). In most cases, running one worker process per CPU core works well, and we recommend setting this directive to auto to achieve that. There are times when you may want to increase this number, such as when the worker processes have to do a lot of disk 1/0.


+ worker connections - The maximum number of connections that each worker process can handle simultaneously. The default is 512, but most systems have enough resources to support a larger number. The appropriate setting depends on the size of the server and the nature of the traffic, and can be discovered through testing.


#### Keepalive Connections

+ Keepalive connections can have a major impact on performance by reducing the CPU and network overhead needed to open and close connections. NGINX terminates all client connections and creates separate and independent connections to the upstream servers. NGINX supports keepalives for both clients and upstream servers. The following directives relate to client keepalives:

+ keepalive_requests - The number of requests a client can make over a single keepalive connection. The default is 100, but a much higher value can be especially useful for testing with a load-generation tool, which generally sends a large number of requests from a single client. keepalive_timeout - How long an idle keepalive connection remains open. The following directive relates to upstream keepalives:

+ keepalive - The number of idle keepalive connections to an upstream server that remain open for each worker process. There is no default value.





	


