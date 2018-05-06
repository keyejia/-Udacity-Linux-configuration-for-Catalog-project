# Udacity Linux Configuration Project

took a baseline installation of a Linux server and prepared it to host web applications. Secured the server from a number of attack vectors, installed and configured a database server, and deploy one of my existing web applications onto it.

## Server Information

	IP Address:54.88.224.29
	Access port:2200
	Application URL: http://54.88.224.29.xip.io
	    Note: Google authentication was recently updated. Their Oauth API is having issues with Virginia           DNS. xip.io is a work around for this issue. 

## Steps for Server Configuration
A summary of software you installed and configuration changes made.

#### 1. Start a new Ubuntu Linux Server instance on Lightsail and SSH into the server
- Registered an account with Amazon lightsail
- Created an instance with Ubuntu
- Chose $5/month plan
- Chose kellenjia as host name
- logged into the server through SSH in the browser

### 2. Update all current installed packages.
```
    sudo apt-get update
    sudo apt-get upgrade
```
### 3.Change  the SSH port from 22 to 2200. 
    
Change port 22 to port 2200 in the sshd_config file
    
    sudo nano /ect/ssh/sshd_config 
    Change port 20 to 2200
    save and quit
Restart SSH
    
    sudo service ssh restart
    
### 4.Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

    sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 123/udp
    sudo ufw enable

### 5. Create a new user account named grader,and give grader the permission to sudo.

    sudo adduser grader
    sudo nano /etc/sudoers
    sudo touch /etc/sudoers.d/grader
    sudo nano /etc/sudoers.d/grader
        add grader ALL=(ALL:ALL) ALL 
        save and quit

### 6.Create an SSH key pair for grader.
Generate key using ssh-keygen and save the private key in .ssh locally named udacity_key
Move public key to the server
```
    sudo su - grader
    sudo mkir .ssh
    sudo touch .ssh/authorized_keys
    sudo nano .ssh/authorized_keys
```
Paste the public in the authorized_keys file 
Save and close

    sudo service ssh restart
    
Log on using grader remotely
```
 ssh -i ~/.ssh/udacity_key.rsa grader@54.235.131.221
```

### 7.Configure the local timezone to UTC

    sudo dpkg-reconfigure tzdata
    Set the timezone to UTC
    
### 8. Install and configure Apache to serve a Python mod_wsgi application

    sudo apt-get install apache2
    sudo apt-get install python-setuptools libapache2-mod-wsgi
    sudo service apache2 restart

### 9. Install and configure PostgreSQL
Install

    sudo apt-get install postgresql

Configure.
Setup Catalog user and database, and grant catalog user access to catalog database

    sudo su -postgres
    shell psql
    postgres=# CREATE DATABASE catalog;
    postgres=# CREATE USER catalog;
    postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
    postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
    \q
    exit
    
### 10. Install Git and Clone catalog App from github
Install
    
    sudo apt-get install git
    
Clone catalog App

    cd /var/www 
    sudo mkdir FlaskApp
    cd FlaskApp
    git clone https://github.com/keyejia/udacity-catalog-project.git
    
Configure the App 
    
    sudo mv ./udacity-catalog-project ./FlaskApp
    cd FlaskApp
    sudo mv application.py __init__.py

    change 
         engine = create_engine('sqlite:///catalog.db') 
    in __init__.py, database_setup.py and lotsofalbums.py to 
         engine = create_engine('postgresql://catalog:password@localhost/catalog')
Install required libraries for the app
    
    sudo apt-get install python-pip
    sudo pip install Flask
    sudo pip install SQLAlchemy
    sudo pip install oauthlib
    sudo pip install psycopg2
    sudo apt-get -qqy install postgresql python-psycopg2

Setup the database for the app
    
    python database_setup.py
    python lotsofalbums.py
    
### 11. Setup the apache2 and wsgi script
Create a FlaskApp.conf file in sites-available folder

    sudo nano /etc/apache2/sites-available/FlaskApp.conf
Post the following in this file
    
    <VirtualHost *:80>
    	ServerName 54.88.224.29
	    ServerAdmin keyejia@gmail.com
    	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
    	<Directory /var/www/FlaskApp/FlaskApp/>
		    Order allow,deny
	    	Allow from all
    	</Directory>
    	Alias /static /var/www/FlaskApp/FlaskApp/static
    	<Directory /var/www/FlaskApp/FlaskApp/static/>
	    	Order allow,deny
	    	Allow from all
	    </Directory>
    	ErrorLog ${APACHE_LOG_DIR}/error.log
    	LogLevel warn
    	CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>

Create a .wsgi file in the App folder

    sudo nano /var/www/FlaskApp/flaskapp.wsgi
post the following in this file
    
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/")

    from FlaskApp import app as application
    application.secret_key = 'super_secret_key'

### 12.Setup Google Oauth Api
    
In Google OAuth 2.0 client IDs and client secret key:
Add http://54.88.224.29.xip.io to authorized javascript origin.
Add http://54.88.224.29.xip.io/login and http://54.88.224.29.xip.io/gconnect to authorized redirect URL.

### 13. Restart Apache
    
    sudo service apache2 restart
    
## Resources
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://discussions.udacity.com/c/nd004-full-stack-broadcast
