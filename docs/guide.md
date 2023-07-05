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
Search for EC2 > Launch Instance > Name: Jenkins > OS: Ubuntu > Instance Type: t3.micro (Free tier) > Key pair(login) > Create new key pair > Key pair name: SSH-KEY-Jenkins > Key pair type: RSA > Private key file format: .pem > Create key pair > Launch Instance

### Create an instance for SonarQube:
SonarQube consumes alot of memory so choose an instance with minimum 2GB memory.
Search for EC2 > Launch Instance > Name: SonarQube > OS: Ubuntu > Instance Type: t3.small > Key pair(login) > Select the key created in previous step. Key pair name: SSH-KEY-Jenkins > Launch Instance

### Create an instance for Docker:
Search for EC2 > Launch Instance > Name: Docker-Server > OS: Ubuntu > Instance Type: t3.micro (Free tier) > Key pair(login) > Select the key created in previous step. Key pair name: SSH-KEY-Jenkins > Launch Instance

## SSH into the Jenkins Instance:
Click on the Jenkins Instance ID under Instance.
Copy the Public IPv4 address.
Open Terminal

```
cd Downloads
ssh -i SSH-KEY-Jenkins.pem ubuntu@16.171.143.132
```

You get a message:
```
The authenticity of host '...' can't be established.
ED25519 key fingerprint is ......
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? 
```

You get an error. Fix it by updating the permissions:
```
chmod 400 SSH-KEY-Jenkins.pem
ssh -i SSH-KEY-Jenkins.pem ubuntu@16.171.143.132
```

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
ssh ubuntu@16.171.143.132
```

Now we are in the Virtual machine. Next, we need to update and install Java Runtime Envionment to run Jenkins and then install Jenkins. Go to https://www.jenkins.io/doc/book/installing/linux/#debianubuntu  

Go to Installing Jenkins > Linux > Debian/Ubuntu and copy the code under Long Term Support Release.

```
sudo apt update
sudo apt install openjdk-11-jre
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

## Allow Port 8080 in your EC2 Instance:
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

Copy the Public IPv4 Instance. Now go to 16.171.143.132:8080 > Install suggested plugins.

## Create First Admin User in Jenkins:
username: valyndsilva
email:
Save and Continue
Copy the Jenkins URL: http://16.171.143.132:8080/ > Save and Finish > Start using Jenkins

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
ssh -i SSH-KEY-Jenkins.pem ubuntu@16.171.135.185
```

Now we are in the SonarQube Virtual Machine. We can change the hostname as below:
```
sudo hostnamectl set-hostname sonarqube
/bin/bash
```

You will notice the hostname has now changed from ubuntu@ip-172-31-43-223 to ubuntu@sonarqube.

Let's do the same for Jenkins Virtual Machine. Open the terminal priviously running with Jenkins.
```
sudo hostnamectl set-hostname jenkins
/bin/bash
```

the hostname has now changed from ubuntu@ip-172-31-34-239 to ubuntu@jenkins.