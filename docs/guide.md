# Jenkins CI/CD Pipeline - SonarQube, Docker, Github Webhooks on AWS

## Create a new repo on GitHub

## Create project and push the code to GitHub Repo:
```
npx create-next-app@latest
git init
git add .
git commit -m "initial commit"
git remote add origin https://github.com/valyndsilva/ci-cd-pipeline.git
git branch -M main
git push -u origin main
```

## Login to AWS:
We have to create 3 EC2 instances (Virtual Servers in the Cloud) on AWS.

### Create an instance for Jenkins:
Search for EC2 > Launch Instance > Name: Jenkins-Server > OS: Ubuntu > Instance Type: t3.medium > Key pair(login) > Create new key pair > Key pair name: SSH-KEY-Jenkins > Key pair type: RSA > Private key file format: .pem > Create key pair > Launch Instance

### Create an instance for SonarQube:
SonarQube consumes alot of memory so choose an instance with minimum 2GB memory.
Search for EC2 > Launch Instance > Name: SonarQube > OS: Ubuntu > Instance Type: t3.medium > Key pair(login) > Select the key created in previous step. Key pair name: SSH-KEY-Jenkins > Launch Instance

### Create an instance for Docker:
Search for EC2 > Launch Instance > Name: Docker-Server > OS: Ubuntu > Instance Type: t3.medium > Key pair(login) > Select the key created in previous step. Key pair name: SSH-KEY-Jenkins > Launch Instance

## SSH into the Jenkins Instance:
Click on the Jenkins Instance ID under Instance. Copy the Public IPv4 address. Open Terminal

```
cd Downloads
chmod 400 SSH-KEY-Jenkins.pem (update permissions to access the pem file)
ssh -i SSH-KEY-Jenkins.pem ubuntu@16.171.132.48
```

Rename the Jenkins Virtual Machine Hostname
```
sudo hostnamectl set-hostname jenkins
/bin/bash
```

The hostname has now changed from ubuntu@ip-172-31-35-144 to ubuntu@jenkins.

Now we are in the Virtual machine. Next, we need to update and install Java Runtime Envionment to run Jenkins and then install Jenkins. Go to https://www.jenkins.io/doc/book/installing/linux/#debianubuntu  

Go to Installing Jenkins > Linux > Debian/Ubuntu and copy the code under Long Term Support Release.

```
sudo apt update
sudo apt install openjdk-11-jre (require version 11 of the jre for jenkins)
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

## Setup ubuntu administrator password:
we need to switch to superuser by running this command:
```
sudo su
```
Edit the configuration to modify the ssh login system for password. So, let’s open the configuration file with a nano editor. This will let us edit the default configuration.
```
nano /etc/ssh/sshd_config
```

Wwe need to tell ssh we want password authentication. For this, we are going to change PasswordAuthentication no to PasswordAuthentication yes. After that, press crtl+ X and save this file.
```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no
```

to this:
```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes
#PermitEmptyPasswords no
```
Ctrl + X > Yes >Save


Next, we need to restart the sshd by running this command:
```
service sshd restart
```

Now, we can set a password for our authentication by running this command:
```
passwd ubuntu
```

And set your preferred password. Now, exit from the server and login again with your password that you’ve set just now. Run this command, and then give the password.
```
ssh ubuntu@16.171.132.48
```

## Allow Port 8080 in your Jenkins EC2 Instance:
Go to the Jenkins EC2 Instance > Security > Security groups link > Edit Inbound Rules > Add Rule > Type: Custom TCP >  Port Range: 8080 > Source: 0.0.0.0/0 > Save Rules

## Check if Jenkins is Installed:
Copy the secret token key on the 1st line after inputting this command in terminal:
```
systemctl status jenkins
```

If you don't know the administartor password or didn't copy the secret token key earlier just type:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword 
```

Copy the Public IPv4 Instance. Now go to http://16.171.132.48:8080/ > Install suggested plugins.

## Create First Admin User in Jenkins:
username: valyndsilva
email:
Save and Continue
Copy the Jenkins URL: http://16.171.132.48:8080/ > Save and Finish > Start using Jenkins

## Create a Pipeline in Jenkins:
+ New Item > Item Name: automated-pipeline > Freestyle Project > OK

### Go to Source Code Management:
Git > Enter the Repo URL (Use the HTTPS url): https://github.com/valyndsilva/ci-cd-pipeline.git > Branch Specifier: */main > Save

### Go to Build Triggers
Select the check box - GitHub hook trigger for GITScm pooling

### Go to the Git Repo to add a Webhook  
Settings > Webhooks > Add webhook > Authenticate > Copy the Jenkins Url (IP:Port/github-webhook)

Add it to Payload URL: http://16.171.143.132:8080/github-webhook/
Which events would you like to trigger this webhook? Let me select individual events. > Enable "Pull requests" and "Pushes" > Add webhook

### Test Pipeline without and with webhook:
Go to http://16.171.143.132:8080 > Select item "automated-pipeline" > Build Now > Status shows green check success

Do a few updates in your project code and commit the changes to your repo and push.
Go to Workspace > You will see the updates

Any changes done to the Github Repo will trigger Jenkins automatically and pull the code from GitHub.

## Create a Server for SonarQube.
Go to AWS EC2 SonarQube Instance > Instance ID > Copy Public IPv4 address
Open a new Terminal > new tab:
```
cd Downloads
ssh -i SSH-KEY-Jenkins.pem ubuntu@13.51.194.9
```

Now we are in the SonarQube Virtual Machine. We can change the hostname as below:
```
sudo hostnamectl set-hostname sonarqube
/bin/bash
```

The hostname has now changed from ubuntu@ip-172-31-37-196 to ubuntu@sonarqube.

Let's do the same for Jenkins Virtual Machine. Open the terminal previously running with Jenkins.
```
sudo hostnamectl set-hostname jenkins
/bin/bash
```

The hostname has now changed from ubuntu@ip-172-31-37-196 to ubuntu@jenkins.

## Update the system repository and download SonarQube on VM:
```
sudo apt update
sudo apt install openjdk-17-jre (require version 17 of the jre for sonarqube)
```

Go to https://www.sonarsource.com/products/sonarqube/downloads/success-download-community-edition/
Right click on "Download Community Edition" > Copy link address

```
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.1.0.73491.zip
ls (copy the zip name)
sudo apt install unzip
unzip sonarqube-10.1.0.73491.zip
ls
cd sonarqube-10.1.0.73491
ls
cd bin
ls
cd macosx-universal-64
./sonar.sh console
```

## Allow Port 9000 in your SonarQube EC2 Instance:
Go to the SonarQube EC2 Instance > Security > Security groups link > Edit Inbound Rules > Add Rule > Type: Custom TCP >  Port Range: 9000 > Source: 0.0.0.0/0 > Description: SonarQube > Save Rules

## Open SonarQube:
Go to the SonarQube EC2 Instance > Copy the Public IPv4 address
Next in the web browser add this URL: http://13.51.194.9:9000/
The default username and pass for Sonarqube is admin > Login
Update password.

Select project Type: Manually
Project display name*: nextjs-website
Project key*: nextjs-website
main branch name: main
Next
What should be the baseline for new code for this project? Use the global setting
How do you want to analyze your repository? With Jenkins
Select your DevOps platform > GitHub
Prerequisites > Configure Analysis 
Create a Pipeline Job > Continue 
Create a GitHub Webhook > Continue 
Create a Jenkinsfile > Other > Create a sonar-project.properties file in your repository and paste in the sonar.projectKey=nextjs-website > Finish this tutorial

### Generate a Token in SonarQube:
Go to the Admin user logo on the top right corner > My Account > Security
Name: SonarQube-Token > Type: Global Analysis Token > Expires in: 30days > Generate
Copy the token and save it somewhere for later use 

## Install Plugins in Jenkins:
Go to http://16.171.132.48:8080/ > Manage Jenkins > Plugins > Available Plugins 
Search: SonarQube Scanner > Download now and install after restart > Check box "Restart Jenkins when installation is complete and no jobs are running"
Search: SSH2 Easy > Install without restart  > Download now and install after restart > Check box "Restart Jenkins when installation is complete and no jobs are running"

 ## Setting up SonarQube
 Go to the Jenkins URL http://16.171.132.48:8080/
 Go to Manage Jenkins > Tools > SonnarQube Scanner > Add SonarQube Scanner > Name: SonarQube Scanner > Check "Install automatically" > Save
 Go to Manage Jenkins > System > SonarQube servers > Add SonarQube > Name: SonarQube-Server > Server URL: http://13.51.194.9:9000 > Server authentication token - Add: Jenkins > Domain:Global credentials > Kind: Secret Text > Secret: (text copied earlier from SonarQube - My Account) >  ID: Sonar-Token > Add > Select Server authentication token: Sonar-Token > Save
 Go to the pipeline created at the begining "automated-pipeline" > Configure > Build Steps > Add build step > Execute SonarQube Scanner > Analysis Properties: sonar.projectKey=nextjs-website > Save
 Go to "automated-pipeline" > Build Now > All Tests Should Pass

## Create a Server for Docker.
Go to AWS EC2 Docker Instance > Instance ID > Copy Public IPv4 address
Open a new Terminal > new tab:
```
cd Downloads
ssh -i SSH-KEY-Jenkins.pem ubuntu@16.171.154.7
```

Now we are in the Docker Virtual Machine. We can change the hostname as below:
```
sudo hostnamectl set-hostname docker
/bin/bash
```

The hostname has now changed from ubuntu@ip-172-31-33-54 to ubuntu@docker.


## Update the system repository and download Docker on VM:
Go to https://docs.docker.com/engine/install/ubuntu/ and follow the steps.

### Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```
sudo apt update
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

### Add Docker’s official GPG key:
```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### Use the following command to set up the repository:
```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker Engine
#### Update the apt package index:
```
sudo apt-get update
```

#### Install Docker Engine, containerd, and Docker Compose:
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Swith to Jenkins User:
On the terminal running Jenkins:
```
sudo su jenkins
ssh ubuntu@16.171.154.7 (check if you can access the docker server from here)
```

You get an error: "Permission denied (publickey)"

Got to the terminal running Docker:
```
sudo su
nano /etc/ssh/sshd_config
```

In the file search this line:
```
#PubkeyAuthentication yes 
```
Uncomment this line by removing the  # at the beginning

```
PubkeyAuthentication yes 
```

Also update this line:
```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
```

to 

```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes
```

## Restart the SSHD Service:
In the docker terminal window:
```
systemctl restart sshd
```

Now go back to the Jenkins Terminal window and verify again:
```
ssh ubuntu@16.171.154.7
```

It asks for the ubuntu user password. We can set a password by running this command:
First go to the Docker Terminal
```
passwd ubuntu
```
Set your preferred password. Now, go back to the Jenkins Server Terminal:
```
ssh ubuntu@16.171.132.48
password: (Enter the ubuntu user password when prompted)
```

This should log you into the Docker Ubuntu VM.
```
exit
```

## Generate a public and private key:

In the Jenkins Terminal window:
```
ssh-keygen
Enter file in which to save the key (/var/lib/jenkins/.ssh/id_rsa): Enter
Enter passphrase (empty for no passphrase): Enter
Enter same passphrase again: Enter
ssh-copy-id ubuntu@16.171.154.7
ssh ubuntu@16.171.154.7 (Verify access to docker after adding the keys)
exit
```

Now go to the Jenkins Dashboard (http://16.171.132.48:8080/) > Manage Jenkins > System > Scroll to "Server Group Center" > Server Group List : Add
Group Name: Docker-Server, SSH Port: 22, User Name: ubuntu, Password: ubuntu user password created in terminal earlier. > Save
Go back to Manage Plugins > System > Scroll to "Server List" : Add
Server Group: Socker-Server, Server Name: Docker-1, Server IP: 16.171.154.7 > Save
Go to Jenkins Dashboard > automated-pipeline > Configure > Build steps > Add build step > Remote Shell > Shell: touch test.txt > Save > Build Now

Go to Docker Terminal:
```
ls
```
You should now see the touch test.txt file here.

## Create a folder in the docker VM:
In the ubuntu@docker terminal:
```
mkdir website
cd website
pwd (get the path and copy it to add in the Execute Shell command)
```

## Create Dockerfile in root
Go to https://hub.docker.com/_/nginx 

Create Dockerfile without any extension in the root directory.
```
FROM nginx
COPY ./usr/share/nginx/html/
```

Commit and push the changes to the git repo.

Now, go to http://16.171.132.48:8080/ and you should see the automated-pipeline is automaticlly triggered and executed.
Go to Configure > Scroll to Build Steps > Remove the "Remote Shell" > Add build step > Execute Shell

We copy the current contents to the remote docker server:
Copy the public IPv4 address from the Docker instance
```
scp -r ./* ubuntu@16.171.154.7:~/website/
```
Save > Build Now

In the docker terminal:
```
ls
```

Now, go to http://16.171.132.48:8080/. Go to Configure > Scroll to Build Steps > Add build step > Add "Remote Shell"

```
cd /home/ubuntu/website
```
Save

First, we verify to check if we can run the docker command in the docker terminal.
```
docker ps
```
We receive a permission denied error.

If we run it with a sudo it works:
```
sudo docker ps
```

But it doesn't work if you run it as the current user. To fix this issue we need to add the current user to the docker group using this command in the docker terminal:
```
sudo usermod -aG docker ubuntu
newgrp docker
docker ps
```

Now, go to http://16.171.132.48:8080/. Go to Configure > Scroll to Build Steps > Update "Remote Shell" commands:

```
cd /home/ubuntu/website
docker build -t mywebsite . (here mywebsite is the docker image name we want to name it)
docker run -d -p 8085:80 --name=nextjs-website mywebsite (run container from this image, 8085 port of the system , 80 port of the container)
```
Save > Build Now

Next, verify if the container is working fine by running this comand in the docker terminal:
```
docker ps
```