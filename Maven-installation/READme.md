
## .#Step0 Apache Maven Installation And Setup In AWS EC2 Redhat Instance.
##### Prerequisite
+ AWS Acccount.
+ Create Redhat EC2 T2.medium Instance with 4GB of RAM.
+ Create Security Group and open Required ports.
   + 22 ..etc
+ Attach Security Group to EC2 Instance.
+ Install java openJDK 1.8+

## .#Step1 Install Java JDK 1.8+  and other softares (GIT, wget and tree)

``` sh
# install Java JDK 1.8+ as a pre-requisit for maven to run.

sudo hostname maven
cd /opt
sudo yum install wget nano tree unzip git-all -y
sudo yum install java-11-openjdk-devel java-1.8.0-openjdk-devel -y
java -version
git --version
```

## .#Step2. Download, extract and Install Maven
``` sh
#Step1) Download the Maven Software
sudo wget https://dlcdn.apache.org/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.zip
sudo unzip apache-maven-3.8.4-bin.zip
sudo rm -rf apache-maven-3.8.4-bin.zip
sudo mv apache-maven-3.8.4/ maven
```
## .#Step3) Set Environmental Variable  - For Specific User eg ec2-user
``` sh
vi ~/.bashrc  # and add the lines below
export M2_HOME=/opt/maven
export PATH=$PATH:$M2_HOME/bin
#
```
## .#Step4) Refresh the profile file and Verify if maven is running
```sh
source ~/.bashrc
mvn -version
```

## .#Step5) Optional - To Create a Maven Custom Repository
+ ls /opt/maven/conf
+ cd conf/
+ sudo vi settings.xml 
+ scroll down to <localRepository> tag
+ Add <localRepository>/rootdirectoryName/directoryName</localRepository> as a new line and remove comment
+ Return to project directory (cd)
+ mvn package
+ ls -l /tmp
+ ls -l ~/.m2/repository
+ mvn clean package
   
   
## Maven Study Notes 
### What kind of projects are you supporting?
+ I support java based projects and a few .NET projects. These includes federated Enterprise micro-service applications with over 21 modules for a Banking client 

   
+ Maven is an open source Java BASED Build  tool. It is open source implying that both software and the source code are freely available. You can download the source code and develop on the existing features.
   
### IQ : Explain the maven lifecycle??
Maven has 3 lifecycles:  Clean, site and default
   + Clean          (mvn clean) - deletes old builds
   + Site/Swagger   (mvn site)  - creates java classes (byte code) in the JVM 
   + Default  
       + mvn validate: It will validate the project structure and resource files
       + mvn compile:  It will compile all java classes and test cases
       + mvn test:     It will run the unit test cases (JUNit)
       + mvn package:  It will create packages in target directory (*.jar/*.war/*ear) app.war
       + mvn install:  It will store the build artifacts in MAVEN LOCAL REPO default location: .m2/repository
       + mvn deploy:   It will upload the build artifacts into maven-remote-repo, NEXUS
   
   + Maven uses plugins/dependencies in the build Process.It gets the plugins from repositories:
       + Maven local repository   ~/.m2/repository = default 
       + Maven remote repository - Nexus
       + Maven central repository - Apache Maven
   
### IQ : What problem have you face in your project?
+ Maven taking longer than expected time to build - 
   + Solution: By skipping the test goal with 
   ```sh
     mvn package -DskipTests 
   ```
     or
   ```sh
     mvn package -Dmaven.test. skip=true
   ```

+ Maven-local-repo was accidentally deleted @ ~/.m2/repository
  + Solution : We created a custom Maven-local-repo by adding the name and locatio of the new repo in the /conf/settings.xml file

### IQ : Assuming that 699 Testcases passed and 1 fails, what can be done for  maven to still do a build? 
   + mvn package -DskipTests
   + mvn package -Dmaven.test. skip=true
   
### IQ : Explain the d/f b/w mvn package & install:
 + mvn package creates artifacts(packages) in target and will be deleted if we run mvn clean 
 + mvn install creates and install packages in target and MLR . Artifacts in MLR won't be deleted if we run mvn clean

### IQ : How can a Specific module be built in maven-enterprise-applications?
   ```sh
   mvn  package -P profilename
   ```
### IQ : How can we trouble-shoot a fail build?
 + Check the logs to understand the Errors 
 + mvn -X package (BUILD in debugging mode)
    + if error, run "sudo yum update"

   
```sh
   mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-webapp -DarchetypeVersion=1.4
```
