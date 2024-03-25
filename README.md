# jenkins-docker-ansible-deployment

![image](https://github.com/vijay2181/jenkins-docker-ansible-deployment/assets/66196388/f8ea231c-8002-402d-b166-4b9b51a361a7)


```
- i have 1 jenkins server and installed ansible in it  -> Jenkins and Ansible on the same machine
- dev, qa, prod servers where these servers are nodes to ansible master

- jenkins pipeline has a stage where jenkins will run a shell script command(ansible playbook command)
- based on env and image tag choosen at jenkins parameters, then ansible use that variables and runs playbook which do docker/ecr login and pulls choosen docker image tag and runs container in that env

```
```
Jenkins and Ansible on the same machine:
=========================================

In smaller setups or for simpler deployments, Jenkins and Ansible might be installed on the same machine.
This setup is easier to manage and requires fewer resources.
It's suitable for small to medium-sized projects where the workload is not very high.
```

```
Separate machines for Jenkins and Ansible:
===========================================
In larger or more complex setups, Jenkins and Ansible might be deployed on separate machines.
This allows for better resource utilization and scalability, as each tool can be allocated dedicated resources.
It also provides better isolation and security since Jenkins and Ansible may have different security requirements.
Additionally, separating Jenkins and Ansible allows for better management of access controls and permissions.
```

```
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

```
Ansible Installation:
=====================
2) Login As a root user and create ansible user and provide sudo access in all 3 Servers.
    2.1) Create the user ansible and set the password on all hosts:
           sudo useradd ansible
           sudo passwd ansible
     
     2.2) Make the necessary entry in sudoers file /etc/sudoers for ansible user for password-less sudo access in all 3 servers:      
           sudo visudo
           ansible ALL=(ALL) NOPASSWD: ALL  

     2.3) Make the necessary changes  in sshd_config file /etc/ssh/sshd_config to enable password based authentication:
         Un comment PasswordAuthentication yes
         and comment  PasswordAuthentication no.
         And save the file .
            vi /etc/ssh/sshd_config

      2.4) Then restart sshd service.
           sudo systemctl restart sshd
           sudo systemctl status sshd


for passwordless authenctication
Generate SSH Key, Copy SSH Key(Ansible Server)
==============================================

1) Now generate SSH key in Ansible Server(jenkins server):
  sudo su - ansible
  pwd -> /home/ansible
  ssh-keygen -t rsa -N '' -f ~/.ssh/id_rsa

2) Copy public key to all agents/Host  servers as ansible user: 
   execute below below command on ansible master by updating HOST IP for all the HOST Servers.
          cd /home/ansible/.ssh
          ssh-copy-id ansible@172.31.25.107    ->  host 1 ip
          ssh-copy-id ansible@172.31.30.87    ->  host 2 ip
          ssh-copy-id ansible@172.31.25.145    ->  host 2 ip
- if all 3 servers are within the same network, then private ip address will work
- test the connection
- ssh ansible@ip


Install Ansible in Master (Jenkins Server)
==============================================
- switch to ansible user
sudo yum install python3 -y
sudo yum install python3-pip –y 
pip3 install ansible
ansible --version


Update Host Inventory in Ansible Server(Jenkins) to add agent/host servers’ details.
-------------------------------------------------------------------------------------
1) Add Host Server details
  if you dont get /etc/ansible directory by default, then create it -> sudo mkdir /etc/ansible

  sudo vi  /etc/ansible/hosts
  172.31.56.78   #agent1
  172.34.57.89   #agent2
  172.31.25.107  #agent3

 2) Use ping module to test Ansible and after successful run you can see the below output.
          cd /etc/ansible
          ansible all -m ping

            172.31.35.23 | SUCCESS => {
            "changed": false,
             "ping": "pong"
             } 

3) Install sshpass in Ansible server if you get below error "to use the 'ssh' connection type with passwords, you must install the sshpass program
$ sudo yum install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/sshpass-1.06-2.el7.x86_64.rpm

```

- now segregate hosts file according to environments by grouping 

```
sudo vi /etc/ansible/hosts
---------------------------
[dev]
172.31.25.107

[qa]
172.31.30.87

[prod]
172.31.25.145
-----------------------------

ansible dev -m ping

172.31.25.107 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.9"
    },
    "changed": false,
    "ping": "pong"
}
```

- now we need to create a playbook, where we need to pass env variable into playbook from another variable file or pass dynamically at runtime

```
/etc/ansible/playbooks

vi var.yaml
------------
env: "dev"

vi sample-playbook.yaml
-----------------------
---
- name: Echo Password
  hosts: "{{ env }}"
  vars_files:
    - var.yaml
  tasks:
    - name: Echo Environment
      ansible.builtin.debug:
        msg: "Selected Environment is {{ env }}"


ansible-playbook sample-playbook.yaml --check
----------------------------------------------
ok: [172.31.25.107] => {
    "msg": "Selected Environment is dev"
}

ansible-playbook sample-playbook.yaml --extra-vars env=qa --check
----------------------------------------------------------------
ok: [172.31.30.87] => {
    "msg": "Selected Environment is qa"
}
```

- lets try to execute this playbook by using jenkins pipeline
- we need to configure ansible user password inside jenkins credential manager
- im executing ansible playbooks without ansible plugin on jenkis

![image](https://github.com/vijay2181/jenkins-docker-ansible-deployment/assets/66196388/bbb4d5e3-e54e-422d-aff0-bcea4e7cb579)

![image](https://github.com/vijay2181/jenkins-docker-ansible-deployment/assets/66196388/cb911c0a-622a-409e-8e04-cbd2db48fe48)

![image](https://github.com/vijay2181/jenkins-docker-ansible-deployment/assets/66196388/fb3f083c-51c3-45b5-ac09-e436ab074b4f)


```
pipeline {
    agent any
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'], description: 'Select environment to deploy')
    }
    
    stages {
        stage('Run Ansible Playbook') {
            steps {
                script {
                    // Debugging output
                    echo "Attempting to switch to ansible user"
                    
                    // Execute commands as ansible user
                    withCredentials([usernamePassword(credentialsId: 'ansible-pw', usernameVariable: 'ANSIBLE_USERNAME', passwordVariable: 'ANSIBLE_PASSWORD')]) {
                        sh """
                            echo "${ANSIBLE_PASSWORD}" | su - ansible -c 'cd /etc/ansible/playbooks && ansible-playbook sample-playbook.yaml --extra-vars env=${params.ENVIRONMENT} --check'
                        """
                    }
                }
            }
        }
    }
}

```









