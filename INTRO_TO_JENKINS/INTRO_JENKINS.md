# Tooling Website deployment automation with Continuous Integration using Jenkins


### Step 1: Install jenkins server
1. Create an aws EC2 instance based on Ubuntu Server 24.04 LTS and name it ```Jenkins```
![Jenkins server](images/launch_instance.jpg)

2. Access the instance and install JDK since Jenkins is a Java-based application
```
sudo apt-get update
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt install fontconfig openjdk-17-jre
```
![Install JDK](images/install_jdk.jpg)

3. Install jenkins
```
sudo apt-get update
sudo apt-get install jenkins -y
```
![Install Jenkins](images/install_jenkins.jpg)

4. Ensure jenkins is up and running
```
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```
![Jenkins status](images/jenkins_status.jpg)

5. By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound rule in the EC2 Security Group
![Security group](images/security_group.jpg)

6. From a browser access http://<Jenkins-Server-Public-IP-Address>:8080 You will be prompted to provide a default admin password. Retrieve it from the server.
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
![Jenkins login](images/jenkins_login.jpg)    
![Jenkins install](images/jenkins_page.jpg)
![Install plugins](images/jenkins_plugin.jpg)
![Complete setup](images/config.jpg)
![Complete setup](images/setup_complete.jpg)


### Step 2: Configure Jenkins to retrieve source codes from GitHub using  Webhooks
1. Enable webhooks in your GitHub repository settings.
*  On your GitHub repository, Select Settings > Webhooks > Add webhook
![Add webhook](images/settings.jpg)

* Go to Jenkins web console, click ```New Item``` and create a ```Freestyle project```
![Create Project](images/jenkins_project.jpg)

* To connect our GitHub repository, we will need to provide its URL, we can copy from the repository itself
```
https://github.com/Oluwafolahanmi/tooling.git
```
* In configuration of our Jenkins freestyle project choose Git repository, provide there the link to our Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.
![Configuration](images/scm.jpg)

* Save the configuration and try to run the build. For now we can only do it manually. Click Build Now button. After all was configured correctly, the build was successfull and was seen under #3 You can open the build and check in Console Output if it has run successfully.
![Console output](images/console_output.jpg)

2. Configure the project.
* Configure triggering the job from GitHub webhook and also Configure Post-build Actions to archive all the files - files resulted from a build are called artifacts:
![Build triggers](images/artifact.jpg)

* Now, go ahead and make some change in any file in our GitHub repository (e.g. README.MD file) and push the changes to the main branch.
![Test build](images/test_build.jpg)

* A new build has been launched automatically by webhook and its results - artifacts, saved on Jenkins server.
![Build update](images/builds_update.jpg)
![Build update](images/console.jpg)

* By default, the artifacts are stored on Jenkins server locally
```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```
![Build update](images/builds.jpg)    

### Step 3 - Configure Jenkins to copy files to NFS server via SSH
1. Install Publish Over SSH plugin.
* On main dashboard, Select Manage Jenkins > Manage Plugins > Available > Search for Publish over SSH and Install without restart.
![Publish over ssh](images/publish_ssh.jpg)
![Publish over ssh](images/download.jpg)

2. Configure the job/project to copy artifacts over to NFS server
* On main dashboard select Manage Jenkins > Configure System menu item.
* Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:
* Provide a private key (content of .pem file that we use to connect to NFS server via SSH/Putty)
* Arbitrary name
* Hostname - can be private IP address of our NFS server
* Username - ec2-user (since NFS server is based on EC2 with RHEL 9)
* Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server.
![Configure project](images/nfs.jpg)
![Configure project](images/test-config.png)

* Save the configuration, open your Jenkins job/project configuration page and add another one Post-build Action (Send build artifact over ssh).  Also, Configure it to send all files produced by the build into our previouslys define remote directory In our case we want to copy all files and directories, so we use ```**```
![Configure project](images/pba_config.jpg)

* Save this configuration and go ahead, change something in README.MD file in our GitHub Tooling repository. The line created previously in the README.md file have been removed
![Test configuration](images/testing.jpg)

* Webhook will trigger a new job
![Test configuration](images/build5.jpg)

* To make sure that the files in /mnt/apps have been updated - connect via SSH to our NFS Server and verify README.MD file
```
ls /mnt/apps
ls -l /mnt/apps
```
![Test configuration](images/confirm.png)   

```
