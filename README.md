# jenkins-docker-ansible-deployment

![image](https://github.com/vijay2181/jenkins-docker-ansible-deployment/assets/66196388/830ff9b5-415a-4131-a089-dfd504bcd44a)

```
- i have 1 jenkins server
- 1 ansible master
- dev, qa, prod servers where these servers are nodes to ansible master

- jenkins pipeline has a stage where jenkins will run a shell script command(ansible playbook command) on remote ansible master
- based on env and image tag choosen at jenkins parameters, then ansible use that variables and runs playbook which do docker/ecr login and pulls choosen docker image tag and runs container in that env

Jenkins Installation:
======================
- take t2.medium amazon linux 2 instance
- in real time just tell them that we are using Memory Optimized jenkins instances -> r5a.xlarge or r5a.2xlarge

- Install java:
sudo dnf install java-17-amazon-corretto-devel -y      ---> includes jdk
sudo dnf install java-17-amazon-corretto -y            ----> includes only jre

- Donwload Jenkins
use LTS version
https://pkg.jenkins.io/redhat-stable/

- add jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install jenkins

- when we install jenkins, it will create a service called jenkins
systemctl enable jenkins
systemctl start jenkins
systemctl status jenkins

- access jenkins
http://ip:8080


Install git
------------
sudo yum install git

Install maven
--------------
- we may have to install multiple maven verions, so on jenkins global tool configurations i will install maven
Dashboard > Manage Jenkins > Tools > Maven installations
Name: MAVEN3.9.5
Install automatically
Version: 3.9.5

Java17 project:
---------------
- take java 17 project for maven build, because we have java17 installed for jenkins
https://github.com/vijay2181/java-maven-SampleWarApp.git



```



