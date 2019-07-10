# Linux_Server_Configuration
This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity.
In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website. 
You can visit (http://34.232.50.176/) for the website deployed.

*Public IP address: 34.232.50.176
*SSH port: 2200

## New Ubuntu Linux Server instance on Amazon Lightsail
Create an AWS account
Click **Create instance** button on the home page
Select **Linux/Unix** platform
Select **OS Only** and Ubuntu as blueprint
Select an instance plan
Name your instance
Click **Create** button
## SSH into Server
1. Download private key from the SSH keys section in the **Account** section on Amazon Lightsail. The file name should be like **LightsailDefaultPrivateKey-us-east-2.pem**
2. Create a new file named **lightsail_key.rsa** under ~/.ssh folder on your local machine
3. Copy and paste content from downloaded private key file to lightsail_key.rsa
4.Set file permission as owner only : `$ chmod 600 ~/.ssh/lightsail_key.rsa`
5. SSH into the instance: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@34.232.50.176`
## Update all currently installed packages
1. Run `sudo apt-get update` to update packages
2. Run `sudo apt-get upgrade` to install newest versions of packages
3. Set for future updates: `sudo apt-get dist-upgrade`
## Change the SSH port from 22 to 2200
1. Run `$ sudo nano /etc/ssh/sshd_config` to open up the configuration file
2. Change the port number from **22** to **2200** in this file
3. Save and exit the file
4. Restart SSH: `$ sudo service ssh restart`
## Configure the firewall
1. Check firewall status: `$ sudo ufw status`
2. Set default firewall to deny all incomings: `$ sudo ufw default deny incoming`
3. Set default firewall to allow all outgoings: `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 to allow SSH: `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 to allow www: `$ sudo ufw allow www`
6. Allow incoming UDP packets on port 123 to allow NTP: `$ sudo ufw allow 123/udp`
7. Enable firewall: `$ sudo ufw enable`
8. Check out current firewall status: `$ sudo ufw status`
9. Update the firewall configuration on Amazon Lightsail website under Networking. Delete default SSH port 22 and add port 80, 123, 2200
10. Open up a new terminal and you can now ssh in via the new port 2200: $ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@34.232.50.176 -p 2200
## Create a new user account grader and give grader sudo access
1. Create a new user account grader:`$ sudo adduser grader`
2.`$ sudo nano /etc/sudoers`
3. Create a file named grader under this path: `$ sudo touch /etc/sudoers.d/grader`
4. Edit this file: `$ sudo nano /etc/sudoers.d/grader`, add code `grader ALL=(ALL:ALL) ALL`. Save and exit
## Set SSH login using keys
1. generate keys on local machine using `ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. deploy public key on developement enviroment
3. On you virtual machine:
```
$ su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ vim .ssh/authorized_keys
```
4. Copy the public key generated on your **local machine** to this file and save
```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
5. reload SSH using service ssh restart

6. now you can use ssh to login with the new user you created
`ssh -i .ssh/linuxcourse grader@34.232.50.176 -p 2200`
7. open configuration file again: `$ sudo nano /etc/ssh/sshd_config`
8. Change `PasswordAuthentication yes` to **no**
9. Restart SSH: `$ sudo service ssh restart`
## Configure the local timezone to UTC
Run `$ sudo dpkg-reconfigure tzdata`
Choose **None of the above** to set timezone to UTC
## Install and configure Apache
1. Install Apache: `$ sudo apt-get install apache2`
## Install and configure Python mod_wsgi
1. Install the mod_wsgi package: `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable mod_wsgi: `$ sudo a2enmod wsgi`
3. Restart Apache: `$ sudo service apache2 restart`
4. Check if Python is installed: `$ python`
## Install PostgreSQL
1. Run `$ sudo apt-get install postgresql`
2. Make sure PostgreSQL does not allow remote connections
3. Open file: `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
4. Check to make sure it looks like this:
```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```
## Create new PostgreSQL user called catalog
1. Switch to PostgreSQL defualt user postgres: `$ sudo su - postgres`
2. Connect to PostgreSQL: `$ psql`
3. Create user catalog with LOGIN role: `# CREATE ROLE catalog WITH PASSWORD 'password';`
4. Allow user to create database tables: `# ALTER USER catalog CREATEDB;`
5. Create database: `# CREATE DATABASE catalog WITH OWNER catalog;`
6. Connect to database catalog: `# \c catalog`
7. Revoke all the rights: `# REVOKE ALL ON SCHEMA public FROM public;`
8. Grant access to catalog: `# GRANT ALL ON SCHEMA public TO catalog;`
9. Exit psql: `\q` 
10.Exit user postgres: `exit`
## Create new Linux user called catalog and new database
1. Create a new Linux user: `$ sudo adduser catalog`
2. Give catalog user sudo access:
*$ sudo visudo
*Add $ catalog ALL=(ALL:ALL) ALL under line $ root ALL=(ALL:ALL) ALL
*Save and exit the file
*Log in as catalog: $ sudo su - catalog
*Create database catalog: createdb catalog
*Exit user catalog: exit
## Install git and clone catalog application from github
1. Run `$ sudo apt-get install git`
2. Use `$ cd /var/www` to move to the `/var/www` directory
3. Create the application directory sudo `mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/tulikaghosh/Item-Catalog.git FlaskApp`
6. Move to the inner FlaskApp directory using `cd FlaskApp`
7. Rename **website.py** to** __init__.py** using sudo `mv website.py __init__.py`
8. Edit `database_setup.py, website.py and functions_helper.py` and change `engine = create_engine('sqlite:///toyshop.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
9. Install pip `sudo apt-get install python-pip`
10. Use pip to install dependencies 1sudo pip install -r requirements.txt`
11. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
12. Create database schema `sudo python database_setup.py`
13. Change line `app.run(host='0.0.0.0', port=8000)` to `app.run()` in init.py file
14. Edit client_secrets.json file
15. Add full path(client_secretes) in init.py file.
16.Create a new project on Google API Console and download client_scretes.json file
```
oauth_flow = flow_from_clientsecrets(
                '/var/www/FlaskApp/FlaskApp/client_secrets.json', scope='')

CLIENT_ID = json.loads(
      open('/var/www/FlaskApp/FlaskApp/client_secrets.json', 'r').read())['web'    ]['client_id']
```      
## Setup for deploying a Flask App on Ubuntu VPS     
1. Install pip: `$ sudo apt-get install python-pip`
2. Install packages:
```
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
```
## Setup and enble a virtual host
1. Create file: `$ sudo touch /etc/apache2/sites-available/catalog.conf`
2. Add the following to the file:
```
   <VirtualHost *:80>
		ServerName 34.232.50.176
		ServerAdmin admin@xx.xx.xx.xx
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
 ```  
3. Run `$ sudo a2ensite catalog to enable the virtual host`
4. Restart Apache: `$ sudo service apache2 reload`
## Configure .wsgi file
1. Create file: `$ sudo touch /var/www/catalog/catalog.wsgi`
2. Add content below to this file and save:
```
   #!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```   
3. Restart Apache: `$ sudo service apache2 reload`
## Edit the database path
1. Replace lines in` __init__.py`, `database_setup.py`, and `lotsofitems.py` with `engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')`
2. Disable defualt Apache page
`$ sudo a2dissite 000-defualt.conf`
4. Restart Apache: `$ sudo service apache2 reload`
## Set up database schema
1. Run `$ sudo python database_setup.py`
2. Run `$ sudo python lotsofitems.py`
3. Restart Apache: `$ sudo service apache2 reload`
4. Now follow the link to 9http://34.232.50.176/) the application should be runing online
## Sources
Amazon Lightsail Website
Google API Concole
Udacity
Apache
