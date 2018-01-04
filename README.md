# FSND-P5_Linux-Server-Configuration

This project is linked to the Configuring Linux Web Servers course. The project objective is to  have one web applications running live on a Amazon Web Service. To complete this project, a Linux server instance is needed.

- IP address: 35.167.27.204

- Application URL: http://52.77.212.59

- Accessible SSH port: 2200

## Instructions

The guide below briefly detail steps to deploy a web application solution in a Linux environment.

## 1. Create Amazon Web Server Ubuntu instance

- Sign up for an account at https://aws.amazon.com .
- Select platform '$ Linux/Unix', OS only '$ Ubuntu', name the instance and click Create.
- Connect instance using WebGUI SSH

## 2. Update Software Package and install Finger

- Update package lists with '$ sudo apt-get update'
- Fetch new versions of packages with '$ sudo apt-get upgrade'
- Install finger '$ sudo apt-get install finger'

## 3. Create new user 

- Run '$ sudo adduser grader' to create a new user named grader.
- Run 'finger grader' to verify that user grader has been created.
- Give grader sudo access by running '$ sudo visudo'.
- Bring cursor below "# User privilege specification".
- Add 'grader  ALL=(ALL:ALL) ALL'.
- Run 'sudo nano /etc/hosts'.
- Avoid error sudo: unable to resolve host by adding this line '127.0.1.1 ip-172-26-1-172'.

## 4. Configure key-based authentication for grader user

- Run '$ ssh-keygen'
- Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): /home/vagrant/.
ssh/linuxCourse
- Leave password blank
- Run '$ cat ~/.ssh/linuxCourse.pub' and copy public key
- Run '$ sudo su - grader' to login as grader
- Create a .ssh directory in the newuser home directory and change its file permissions to 700 (only the owner can read, write, or open the directory).
'$ mkdir .ssh'
'$ chmod 700 .ssh'
Without these exact file permissions, the user will not be able to log in.
-Create a file named authorized_keys in the .ssh directory and change its file permissions to 600 (only the owner can read or write to the file).
'$ touch .ssh/authorized_keys'
'$ chmod 600 .ssh/authorized_keys'
- Run '$ nano .ssh/authorized_keys'
- Paste public key, press 'ctrl + O', press 'enter' to save and press 'ctrl + x' to exit
- Logout to return as ubuntu user
- Copy content of 'cat ~/.ssh/linuxProject' and save as grader.pem
- Change the owner from root to grader: '$ sudo chown -R grader:grader /home/grader/.ssh'.

## 5. Enforce key-based authentication

$ sudo nano /etc/ssh/sshd_config. Find the PasswordAuthentication line and edit it to no.
$ sudo service ssh restart.

## 6. Setup ssh access via PuTTY

- Start PuTTYgen
- Select conversion > Import Key, select grader.pem and click open
- Click Save private key as grader.ppk
- Open PuTTY.
From Lightsail, grab the public IP address.
- Type (or paste) the public IP address into the Host Name (or IP address) field.
- Under Category, expand SSH, and then choose Auth.
- Choose Browse to navigate to the .ppk file that you created in the previous step, and then choose Open.
- Choose Open again and type 'grader' to login

## 7. Configure local timezone to UTC

- Open time configuration dialog and set timezone to Brunei with: '$ sudo dpkg-reconfigure tzdata'.
- Install ntp daemon ntpd for synchronization of server's time with NTP server: '$ sudo apt-get install ntp'.

## 8. Disable ssh login for root user

- '$ sudo nano /etc/ssh/sshd_config'. Find the PermitRootLogin line and edit it to no.
- Enable grader & ubuntu user remote ssh login '$ sudo nano /etc/ssh/sshd_config' and add '$ AllowUsers grader ubuntu',
- '$ sudo service ssh restart'.

## 9. Change default SSH port from 22 to 2200

- "Manage" instance, go to networking and add custom firewall rules for tcp 2200.
- '$ sudo nano /etc/ssh/sshd_config'.
- Under "What ports, IPs and protocols we listen for" change 22 to 2200,
- press 'ctrl + o', press 'enter' to save and press 'ctrl + x' to exit.
- '$ sudo service ssh restart'.

## 10. Configure the Uncomplicated Firewall (UFW)
  ```
   $ sudo ufw default deny incoming
   $ sudo ufw default allow outgoing
   $ sudo ufw allow 2200/tcp
   $ sudo ufw allow ssh
   $ sudo ufw allow www
   $ sudo ufw allow ntp
   $ sudo ufw enable
  ```
- '$ sudo ufw status verbose' to confirm

## 11. Install Apache

- 'sudo apt-get install apache2'
- Confirm installation by visiting http://52.77.212.59

## 12. Install mod_wsgi & Python (start here)

-Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools:
'$ sudo apt-get install python-setuptools libapache2-mod-wsgi'
-Restart the Apache server for mod_wsgi to load:
'$ sudo service apache2 restart'
- Run 'sudo apt-get install libapache2-mod-wsgi python-dev'
- Enable mod_wsgi with 'sudo a2enmod wsgi'
- Start the web server with 'sudo service apache2 start'

## 13. Clone the Catalog app from Github

- Install git using: 'sudo apt-get install git'
- 'sudo mkdir /var/www/catalog'
- Change owner of the catalog folder 'sudo chown -R grader:grader /var/www/catalog'
- 'cd /var/www/catalog'
- Clone your project from github 'sudo git clone https://github.com/geraldgsh/item-catalog.git catalog'
- Return to root directory 'cd ..'
- 'sudo ls -al' verify files have been clone from github.
- Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'super_secret_key'
  ```
- Rename application.py to __init__.py `mv application.py __init__.py`

## 14. Install virtual environment

- 'sudo easy_install pip'
- Install the virtual environment 'sudo apt-get install virtualenv'
- Create a new virtual environment with 'sudo virtualenv venv'
- Activate the virutal environment source and access it 'venv/bin/activate'
- Change permissions 'sudo chmod -R 777 venv'

## 15.  Install Flask and other dependencies

- Install pip with 'sudo apt-get install python-pip'
- Install Flask 'pip install Flask'
-Install other project dependencies 'sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests'


## 16. Update path of client_secrets.json file
- 'nano __init__.py'
- Change client_secrets.json path to /var/www/catalog/catalog/client_secrets.json

## 17. Configure and enable a new virtual host
- Run this: sudo nano /etc/apache2/sites-available/catalog.conf
- Paste this code:

'''
<VirtualHost *:80>
    ServerName 52.77.212.59
    ServerAdmin admin@52.77.212.59
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
'''
- Enable the virtual host sudo a2ensite catalog

## 18. Install and configure PostgreSQL

- 'sudo apt-get install libpq-dev python-dev'
- 'sudo apt-get install postgresql postgresql-contrib'
- 'sudo su - postgres'
- 'CREATE USER catalog WITH PASSWORD 'p@ssw0rd';'
- 'ALTER USER catalog CREATEDB;'
- 'CREATE DATABASE catalog WITH OWNER catalog;'
- '\c catalog'
- 'REVOKE ALL ON SCHEMA public FROM public;'
- 'GRANT ALL ON SCHEMA public TO catalog;'
- '\q'
- 'exit'
- Change create engine line in your __init__.py, loaddatabase.py and database_setup.py to: engine = create_engine('postgresql://catalog:p@ssw0rd@localhost/catalog')
- 'sudo python /var/www/catalog/catalog/database_setup.py'
- 'sudo cat /var/log/apache2/error.log' to diagnose issue if any.

** Note: Go to https://developers.facebook.com. Proceed to My Apps > (Project app) > Settings and add http://52.77.212.59/login to Site URL. Authorised redirect URIs (i.e. http://52.77.212.59/login) cannot be added in OUTH settings of Google Developer since "Invalid Redirect: http://52.77.212.59/login must end with a public top-level domain (such as .com or .org)."
