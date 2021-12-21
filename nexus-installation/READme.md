## Nexus Installation And Setup In AWS EC2 Redhat Instance
##### Pre-requisite
+ AWS Acccount.
+ Create Redhat EC2 t2.medium Instance with 4GB RAM.
+ Create Security Group and open Required ports.
   + 8081 ..etc
+ Attach Security Group to EC2 Instance.
+ Install java openJDK 1.8+ for Nexus version 3.15

### Create nexus user to manage the Nexus server
```sh
#As a good security practice, Nexus is not advised to run nexus service as a root user, 
# so create a new user called nexus and grant sudo access to manage nexus services as follows. 
sudo hostname nexus
sudo useradd nexus
# Grant sudo access to nexus user
sudo echo "nexus ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/nexus
sudo su - nexus
```

### Install Java as a pre-requisit for nexus and other softwares

``` sh
cd /opt
sudo yum install wget git nano unzip -y
sudo yum install java-11-openjdk-devel java-1.8.0-openjdk-devel -y
```
### Download nexus software and extract it (unzip)
```sh
sudo wget http://download.sonatype.com/nexus/3/nexus-3.15.2-01-unix.tar.gz 
sudo tar -zxvf nexus-3.15.2-01-unix.tar.gz
sudo mv /opt/nexus-3.15.2-01 /opt/nexus
sudo rm -rf nexus-3.15.2-01-unix.tar.gz
```

### Grant permissions for nexus user to start and manage nexus service
```sh
# Change the owner and group permissions to /opt/nexus and /opt/sonatype-work directories.
sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work
sudo chmod -R 775 /opt/nexus
sudo chmod -R 775 /opt/sonatype-work
```
###  Set User to "Nexus"
+ Open /opt/nexus/bin/nexus.rc file
+ uncomment "run_as_user" parameter and set as nexus user. i.e.
+ Change from [ #run_as_user="" ] to [ run_as_user="nexus" ]

```sh
vi /opt/nexus/bin/nexus.rc
```

###  Configure Nexus to Run as a Service 
+ First create a symbolic link:

```sh
sudo ln -s /opt/nexus/bin/nexus /etc/init.d/nexus

#9 Enable and start the nexus services
sudo systemctl enable nexus
sudo systemctl start nexus
sudo systemctl status nexus
echo "end of nexus installation"
```

### Nexus directory structure
+ bin 
  + binary files. Most important is "nexus"
+ lib 
  + jar and library files             
+ deploy  
  + uploaded artifacts
+ etc (conf)
  + Configuration files  
  + nexus-default.properties ==> to change PortNumber:
  + sudo vi /opt/nexus/etc/nexus-default.properties 
  + If PortNumber is changed, restart ssdh (sudo service nexus restart)


### Configure Nexus Server on the UI
+ Access the Nexus server using the URL --> IP:PortNumber
+ Log in using the default credentials => userName:admin; PAssWd:admin123
+ Create Remote repositories:
   + maven2-hosted-snapshot and 
   + maven2-hosted-release repositories

### Integrate Nexus with Maven
#### Step 1 - Update Nexus Repositories in Build Script (pom.xml)
+ On the maven-CLI, vi into pom.xml file or open from the SCM
+ Locate the "distributionManagement" tag
+ Update the respective URL for SnapShot and Releases using the URL copied from Nexus Server UI

#### Step 2 - Configure Maven Authentication to Access Nexus Server
+ Authenticate maven in Nexus server 
  + within maven project folder CLI 
  + locate the settings.xml file (conf/settings.xml)
  + sudo vi /opt/maven/conf/settings.xml
  + define the nexus server credentials in the settings.xml file
  + copy and paste the following nexus server details to the server tag in the settings.xml file. Ensure included credentials for nexus are correct

  ```sh
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
   ```

### Uploading Artifacts to Nexus "Releases" Repository
+ To change uploading to "Releases" from "Snapshot" repository
+ sudo vi pom.xml
+ locate the "version" tag below the "groupID" tag
  + delete "SNAPSHOTS" and rename the version number
+ If required Select "Allow Redeploy" in Nexus Server UI repository settings


### Configure Nexus Server as Proxy for Maven Central Repository
+ It's not secure to have maven download plugins and dependencies from the Maven Central Repository
+ To remove the potential vulnerability, Nexus server can be used as a proxy for the Maven Central Repository
+ Create repository in Nexus UI
  + enter name of proxy repository
  + select "Mixed" for version policy
  + enter the URL of the Maven Central Repository being proxied as remote storage 
  ```sh
  https://repo.maven.apache.org/maven2/
  
  ```
+ Set-up Pull configuration in Maven
1. access the nexus proxy repo created
2. copy the Nexus proxy repo URL
3. within maven project folder CLI, locate the pom.xml file
4. sudo vi pom.xml
  + create a new tag named "repository" just before "dependencies" tag
  + include/paste the URL of the created Nexus proxy repo
  ```sh
  <repository>
    <id>nexus</id>
      <name>Proxy Repo</name>
        <url>http://54.89.142.136:8000/repository/td-remote-repo/</url>
  </repository>
  ```
5. within maven home directory CLI, locate the settings.xml file (conf/settings.xml)
6. sudo vi /opt/maven/conf/settings.xml
  + locate the tag named "Mirror"
  + paste the URL of the created Nexus proxy repo
     ```sh
     <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <name>Proxy Repo</name>
      <url>http://54.89.142.136:8000/repository/td-group-repo/</url>
     </mirror>
     ```
  + If other proxies (Mirrors) exist within the the "Mirrors" tag, comment all lines
  
