# project_6
LINUX SERVER CONFIGURATION-24 About: This is the Udacity project 6 about the Configuring the Linux the server.

Server Details: Server IP Address 13.127.121.235.xip.io

Hosted site Url http://13.127.121.235.xip.io/

Grader Password: unix How to connect as grader: save private key provided in your local machine and run the following command

ssh -i path/to/privatekey -p grader@13.127.121.235

Configuring Linux Server: Updating all packages:

sudo apt-get update

sudo apt-get upgrade Creating grader User: sudo adduser grader

This will add new user

sudo nano /etc/sudoers Below the Root user append the following line

grader ALL=(ALL:ALL) ALL This will grant sudo permission to grader

Creating a ssh key pair for grader: On your local machine in terminal/command prompt

ssh-keygen This will generate public and private ssh keys which is saved to .ssh folder

Then in your virtual machine change to newly created user

sudo su - grader Create a new directory .ssh and new file authorized_keys in that directory

mkdir .ssh

sudo nano .ssh/authorized_keys Copy the public key with .pub extension to authorized_keys and save the file

chmod 700 .ssh

chmod 644 .ssh/authorized_keys

700 will give read write and execute permission to user.

644 prevent other user from writting in to file. Then restart ssh server

sudo service ssh restart

Now from your log in to grader with private key generated

ssh -i .ssh/id_rsa grader@ipaddress

Changing the ssh port to 2200: sudo nano /etc/ssh/sshd_config

Change port 22 to port 2200

Restart the ssh server

service ssh restart

Note: Before Logging using ssh add custom TCP port 2200 under lightsaail firewall in networking tab in lightsail instance console

Now Login using command like this

ssh -i .ssh/id_rsa -p 2200 grader@ipaddress

Disabling ssh login as root: sudo nano /etc/ssh/sshd_config

make change PermitRootLogin no

Configurating Ufw firewall: sudo ufw allow 2200/tcp

sudo ufw allow 80/tcp

sudo ufw allow 123/udp

sudo ufw enable This will allow all required ports and enables the ufw

After that

sudo ufw status

It will display all allowed ports

Changing time Zone: sudo dpkg-reconfigure tzdata

select none from list and then select utc.

Installing Apache2: In terminal

sudo apt-get install apache2

Now mod_wsgi

sudo apt-get install python-setuptools libapache2-mod-wsgi

Enable mod_wsgi

sudo a2enmod wsgi

Setting up your flask application to work with apache2: Creating a flask app

In /var/www directory create a new folder sudo mkdir FlaskApp

Install git

sudo apt-get install git

move to the FlaskApp cd FlaskApp

In that direcory clone your github repository

sudo git clone https://github.com/username/catalog.git

Rename your repository to FlaskApp

Then rename your project file to init.py

Error : While accesssing the client_secrets.json file

PROJECT_ROOT = os.path.realpath(os.path.dirname(file)) json_url = os.path.join(PROJECT_ROOT, 'client_secrets.json') CLIENT_ID = json.load(open(json_url))['web']['client_id'] Use json_url instead client_secrets.json in script

Install and configuring postgresql for project: Install Postgres sudo apt-get install postgresql

login to postgres sudo su - postgres

postgres shell psql

create user CREATE USER catalog WITH PASSWORD 'password';

permit user to createdb ALTER USER catalog CREATEDB;

Create a db name catalog with user catalog CREATE DATABASE catalog WITH OWNER catalog;

connect to db \c catalog

revoke all permission to public REVOKE ALL ON SCHEMA public FROM public;

Give schema permission to user catalog GRANT ALL ON SCHEMA public TO catalog;

exit from db \q exit from postgres exit

Change the database connection in both db_setup.py and init.py as engine = create_engine('postgresql://catalog:password@localhost/catalog')

Now you are ready with your applicatiom

Configure and Enable a New Virtual Host: sudo nano /etc/apache2/sites-available/FlaskApp.conf

In this add the following code

<VirtualHost *:80> ServerName mywebsite.com ServerAdmin admin@mywebsite.com WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi <Directory /var/www/FlaskApp/FlaskApp/> Order allow,deny Allow from all Alias /static /var/www/FlaskApp/FlaskApp/static <Directory /var/www/FlaskApp/FlaskApp/static/> Order allow,deny Allow from all ErrorLog ${APACHE_LOG_DIR}/error.log LogLevel warn CustomLog ${APACHE_LOG_DIR}/access.log combined Enable the virtual host sudo a2ensite FlaskApp

Disabling the default apache2 page sudo a2dissite 000-default.conf

Create the .wsgi File: cd /var/www/FlaskApp sudo nano flaskapp.wsgi Add the following code

#!/usr/bin/python import sys import logging logging.basicConfig(stream=sys.stderr) sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application application.secret_key = 'Add your secret key' save and exit

Deploying flask app with apache2 is referred from Digital ocean

Installing require modules: You can either install all modules on your machine or create a virtual environment for the project and install the modules To Create virtual environment: sudo virtualenv venv To activate virtual environment: source venv/bin/activate pip install flask sqlalchemy psycopg2 requests oauth2client

To deactivate virtual environment: deactivate

Setting up your Google Oauth2: Login to your developer console and select your project and edit OAuth details as following

Javascript origin http://ip.xip.io

redirect URI

http://ip.xip.io\callback

http://ip.xip.io\gconnect

http://ip.xip.io\login

xip.io is a free DNS which will be the same as using IP address

Final Step: Restart your apache2 server

sudo service apache2 restart
