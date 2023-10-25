# Continous Integration Pipeline For Tooling Website
## INSTALL AND CONFIGURE JENKINS SERVER
### Step 1 – Install Jenkins server

Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it “Jenkins”
Install JDK (since Jenkins is a Java-based application)
`sudo apt update`
`sudo apt install default-jdk-headless`

![Alt text](<Screenshot 2023-07-10 014258.png>)

![Alt text](<Screenshot 2023-07-10 014606.png>)


Install Jenkins

### Jenkins Debian Packages

### This is the Debian package repository of Jenkins to automate installation and upgrade. To use this repository, first add the key to your system:

`curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null`

### Then add a Jenkins apt repository entry

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null`

### Update your local package index, then finally install Jenkins:

`sudo apt-get update`
`sudo apt-get install fontconfig openjdk-11-jre`
`sudo apt-get install jenkins`
`sudo systemctl status jenkins`

 ![Alt text](<Screenshot 2023-07-10 021434.png>)

 ### By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group

![Alt text](<Screenshot 2023-07-10 021721.png>)

![Alt text](<Screenshot 2023-07-10 021959.png>)

## Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks

### Enable webhooks in your GitHub repository settings

### To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

### In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

### Save the configuration and let us try to run the build. For now we can only do it manually.
### Click “Build Now” button, if you have configured everything correctly, the build will be successfull and you will see it under #1

### You can open the build and check in “Console Output” 

![Alt text](<Screenshot 2023-07-10 184333.png>)

### Click “Configure” your job/project and add these two configurations
### Configure triggering the job from GitHub webhook:

![Alt text](<Screenshot 2023-07-10 201735.png>)


![Alt text](<Screenshot 2023-07-10 201552.png>)

## CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH

### Install “Publish Over SSH” plugin.

![Alt text](<Screenshot 2023-07-10 210953.png>)

### Configure the job/project to copy artifacts over to NFS server.

### Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)

![Alt text](<Screenshot 2023-07-10 211303.png>)

### Arbitrary name
### Hostname – can be private IP address of your NFS server
### Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
### Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
### Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

### Save the configuration, open your Jenkins job/project configuration page and add another one “Post-build Action”(Send build artifact over SSH)

### Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories – so we use **.

### Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

### Webhook will trigger a new job and in the “Console Output” of the job you will find something like this:

### To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to your NFS server and check README.MD file

### Change ownership from root to ec2-user 

`sudo chown -R ec2-user:ec2-user /mnt/apps/`

`cat /mnt/apps/README.md`

![Alt text](<Screenshot 2023-07-10 210704.png>)

The End