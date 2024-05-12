# WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

### Introduction:  
The LAMP stack is a popular open-source web development platform used for building and hosting dynamic web applications. LAMP stands for Linux, Apache, MySQL (or MariaDB), and PHP (or Perl or Python). It provides a robust and scalable environment for building a wide range of web applications, from simple websites to complex web services. It's flexible enough to accommodate various programming languages, databases, and web servers, but the combination of Linux, Apache, MySQL/MariaDB, and PHP remains one of the most popular and widely used configurations.  

**Linux:** This is the operating system (OS) on which the entire stack runs. Linux is chosen for its stability, security, and open-source nature. Common distributions used in LAMP setups include Ubuntu, CentOS, and Debian.  
**Apache:** Apache is a widely used open-source web server software. It is responsible for serving web pages to users' browsers when they request them. Apache is highly configurable and supports various modules for extending its functionality.  
**MySQL/MariaDB:** MySQL and MariaDB are relational database management systems (RDBMS) used for storing and managing the data of web applications. MySQL was the original choice, but MariaDB, a fork of MySQL, is now often preferred due to its community-driven development and open-source nature.  
**PHP:** PHP is a server-side scripting language used for developing dynamic web pages and web applications. PHP code is executed on the server, generating HTML which is then sent to the client's browser. PHP is known for its ease of use, extensive documentation, and strong community support.

### Gettin started with LAMP stack  
##### Step 1: Creating Aws account 
* Visited the AWS website (https://aws.amazon.com/) and clicked on "Create an AWS Account".  
* Followed the instructions to create a new AWS account.  
* Details such as email address, password, and payment information were provided.
* Phone number was verified to complete registration and access services such as launching instance.
##### Step 2: Launch ec2 instance  
* Signed inot the Aws account
* Navigated to EC2 dashboard and clicked on the "Launch Instance" button to start the process of launching a new EC2 instance.
* created a name for the instance and a key pair (.pem) which are used to ssh the instance.
* Edited inbound security settings to allow traffic from port ssh 20, http 80, https 443 with source from anywhere. left others  settings at default.
* launched the instance with the "Launch" button
* connected to the instance using the ssh key copied from the connect page of the instance. The connection is done at the directory containing the downloaded key pair.
##### step 3:  Install Apache and Update the Firewall
1. Install apache using ubuntu package manager "apt" 
```
sudo apt update
sudo apt upgrade
```

2. Verified if the apache2 is running as service in the Os. 
```sudo systemctl status apache2```

It was green which means the server is successfully running.
![Apache status](C:\Users\PC\OneDrive\Pictures\apacherunning)

 4. Checked if the server can be accessed locally in the ubuntu shell. 
```curl http://127.0.0.1:80```

This returned a bunch of html tags  
![html tags](C:\Users\PC\OneDrive\Pictures\htmltags)

5. Tested if apache server responded to request from the internet by usung my ip address as url. returned this page which indicated successs.
6. 

   ![webpage](C:\Users\PC\OneDrive\Pictures\htmltags)


##### Step 4: Installing mysql

 1. Used 'apt' to acquire and install mysql
```sudo apt install mysql-server```

replied yes to the following prompts  

2. Logged into mysql after the installation

   ```sudo mysql```

3. Set a password for root user using mysql_native_password as default authentication method. <my password> contains the password used

 ```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '<my password>';``` 

 4. Exited the mysq shell

![exit](C:\Users\PC\OneDrive\Pictures\exit)

 5. Started an interactive script with

     ```sudo mysql_secure_installation```

    followed the prompt to edit some settings includig password validation and removing some insecure settings and lock down access to the database system.once done tested the setting by logging into mysql.
```sudo mysql -p```

    it prompted for password and then signed in when supplied. exited the shell 
![exit](C:\Users\PC\OneDrive\Pictures\exit)


##### Step 5: Installing Php
1. Install php Apache is installed to serve the content and MySQL is installed to store and manage data. PHP is the component of the set up that processes code to display dynamic content to the end user.

The following were installed:
* php package
* php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases.
* libapache2-mod-php, to enable Apache to handle PHP files.

  ```sudo apt install php libapache2-mod-php php-mysql```

2. After installing, the version was confirmed with

   ```php -v```
![Php version](C:\Users\PC\OneDrive\Pictures\version)

##### Step 6: Creating a virtual host for the website using apache
To test the setup with a PHP script, it's best to set up a proper Apache Virtual Host to hold the website files and folders. Virtual hosts allow you to host multiple websites on a single machine without being noticed by the website users.
1. created a directory for projectlamp

 ```sudo mkdir /var/www/projectlamp```
 
 2. Assigned the directory ownership with $USER environment variable which references the current system user.

  ```sudo chown -R $USER:$USER /var/www/projectlamp```
  
3.  Created a Virtual Host Configuration File by Navigaing to the Apache configuration directory 

```sudo vim /etc/apache2/sites-available/projectlamp.conf```

4. Configured the Virtual Host by adding the following lines to set up a basic virtual host

 ```
<VirtualHost *:80>
  ServerName projectlamp
  ServerAlias www.projectlamp
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/projectlamp
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

![configuration setting](C:\Users\PC\OneDrive\Pictures\config)

5. Enabled the Virtual Host Using the a2ensite command

```sudo a2ensite projectlamp```

![enabled virtualhost](C:\Users\PC\OneDrive\Pictures\host)

6. Disable apache default website so as not to overwrite the virtual host

```sudo a2dissite 000-default```

7. Ensure the configuration does not contain syntax error

```sudo apache2ctl configtest```

it doesnt contain error

![check syntax error](C:\Users\PC\OneDrive\Pictures\error)

8. Reload apache to apply changes

```sudo systemctl reload apache2```

9. The new website is now active but the web root /var/www/projectlamp is still empty. Created an index.html file in the location to test if the virtual host work as expected.

    ```sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html```

10. Opened the website on a browser using the public IP address.

 ![webpage](C:\Users\PC\OneDrive\Pictures\page)

##### step 7: Enable Php on the website

With Apache's default DirectoryIndex setting, an index.html file will take precedence over an index.php file. This is beneficial for setting up maintenance pages in PHP applications. To change this behavior, the file /etc/apache2/mods-enabled/dir.conf was edited by adjusting the order in which index.php is listed within the DirectoryIndex directive to prioritize PHP files over HTML files.
1. The directory was opened with

 ```sudo vim /etc/apache2/mods-enabled/dir.conf```

 The content was edited 

 ```
<IfModule mod_dir.c>
  # Change this:
  # DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
  # To this:
  DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

  ![directory](C:\Users\PC\OneDrive\Pictures\directory)

2. Apache is reloaded to effect the changes.

 ```sudo systemctl reload apache2```
 
3. A php test script was created to confirm that Apache is able to handle and process requests for PHP files.

 ```vim /var/www/projectlamp/index.php```

 the file contains 

 ```
<?php
phpinfo();
```

![Php file](C:\Users\PC\OneDrive\Pictures\php)

4. browser page was refreshed to check the changes

![Php page](C:\Users\PC\OneDrive\Pictures\php_page

5. After checking the information about the server on the page, it was deleted because it contains sensitive details about the PHP environment and the Ubuntu server. If required in the future, the file can always be recreated.

```sudo rm /var/www/projectlamp/index.php```

### Conclusion:

By adhering to the guidelines provided in this documentation, it was feasible to establish, configure, and manage a LAMP environment proficiently. The LAMP stack furnishes a resilient and adaptable platform for crafting and deploying web applications, empowering the development of potent and scalable web solutions.
   