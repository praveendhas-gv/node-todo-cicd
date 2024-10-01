# **Deploying a containerized web application through CI pipeline and docker**

This project is to build, containerize and deploy a web application on a docker container with github remote build trigger. 

## **Setting up project environment** 

1. Initializing EC2 Instance. 


Log into AWS console using IAM User account. 
Create an EC2 instance
Select an OS image - Ubuntu
Create a new key pair with any name & download .pem file
Use the same key pair in the EC2 instance creation screen. 
Instance type - t2.micro
Connecting to the instance using ssh

Use the below command to connect to EC2 instance using SSH

```
ssh -i <keypair_name>.pem ubunutu@<IP_ADDRESS>
```

### **Configuring Ubuntu on EC2 instance and installing dependencies**

Update the outdated packages and dependencies. 

```
sudo apt update
```

### **Installing Git**

Install git using the below command. 

```
sudo apt install git
```

### **Installing Jenkins**

```
sudo apt install openjdk-17-jre
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

### **Starting Jenkins service**

```
sudo systemctl enable Jenkins
sudo systemctl start Jenkins
```

### **Installing Docker**


```
sudo apt update
sudo apt install docker.io
```

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```
The above commands add jenkins user and ubuntu user to docker group so that docker commands can be executed without root privileges.

### **Integrating git with Jenkins for CI**

**Logging into jenkins**

Log into Jenkins using the link
```
http://<ec2-public-ip>:8080
```
As per the instruction use the below command to get the admininstrator password

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Create an admin user after entering the password. 

After login, go to Manage Jenkins > Manage Plugins > Available plugins and install "Sonarqube scanner" plugin

Restart Jenkins after installation of plugin by going to the following link
```
http://<ec2-public-ip>:8080/restart
```
Go to Jenkins Dashboard and create new item. 

Choose "Freestyle Project" , enter a name and click on "OK"

Go to source code management, choose git, provide the details of the git repository and select the master branch. 

> https://github.com/praveendhas-gv/node-todo-cicd.git

Go to build trigger section and select github hook trigger for GIT scm polling

For the webhook, go to the settings for the repository "node-todo-cicd", select webhooks in the left pane and click on add webhook. 

The payload url should point to the Jenkins server
```
http://<ec2-public-ip>:8080/github-webhook/
```
Content type is Application/json

Click on add webhook to finalize setting up the webhook. 

Any change in code in git scm will now trigger the build job in Jenkins. 

From the jenkins dashboard, select the current project and click on configure build. 

Under the build steps section, select "Add build step" and choose "Execute shell command"

```
docker build . -t node-app-todo
docker run -d --name nodeapp -p 6000:8000 node-app-todo
```

This will build the docker image and run the application on the container. The application will be reachable through the below link.

```
http://<ec2-public-ip>:6000
```
