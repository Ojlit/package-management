#  **<span style="color:green">Landmark Technologies, Ontario, Canada.</span>**
### **<span style="color:green">Contacts: +1437 215 2483<br> WebSite : <http://mylandmarktech.com/></span>**
### **Email: mylandmarktech@gmail.com**



## .#Step0 Apache Maven Installation And Setup In AWS EC2 Redhat Instance.
##### Prerequisite
+ AWS Acccount.
+ Create Redhat EC2 T2.medium Instnace with 4GB of RAM.
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
sudo wget https://dlcdn.apache.org/maven/maven-3/3.8.3/binaries/apache-maven-3.8.3-bin.zip
sudo unzip apache-maven-3.8.3-bin.zip
sudo rm -rf apache-maven-3.8.3-bin.zip
sudo mv apache-maven-3.8.3/ maven
```
## .#Step3) Set Environmental Variable  - For Specific User eg ec2-user
``` sh
vi ~/.bash_profile  # and add the lines below
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
+ We support java based projects and a few .NET projects
   
+ MAVEN is an open source Java BASED Build  tool. It is open source implying that both software and the source code are freely available. You can download the source code and develop on the existing features

+ maven: creates jar, war and ear packages
  jar: Standalone Applications
    ebay.jar
    paypal.jar
    rbc.jar 
  war: 
    boa.war
    rbc.war
  ear:
    aa.ear 
    boa.ear 
   
   
