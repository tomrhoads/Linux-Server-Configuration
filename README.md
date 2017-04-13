# Linux-Server-Configuration

This project is a linux server configuration designed to host project 3 of the full stack web development program for Udacity.  The public IP address is http://52.206.38.191 and the EC2 URL is http://ec2-52-206-38-191.compute-1amazonaws.com.  An amazon lightsail instance was used to configure the ubuntu linux server.

# Create an instance

Use the tutorial given in the project's final lessons to complete the simple setup of a lightsail instance

# Add user

Add a user by typing sudo adduser grader, then type sudo cat /etc/passwd to confirm the grader has been added

Give grader permission to sudo with this: sudo nano /etc/sudoers.d/grader
Then edit contents to this: grader ALL=(ALL) NOPASSWD:ALL

#  Set up key-based authentication

On your local machine, use ssh-keygen to create a key.  There will be two keys, public and private.  The private stays on your local machine and you add the public key to /home/grader/.ssh/authorized_keys.  After that, set up permissions:

chmod 700 .ssh
chmod 664 .ssh/authorized_keys

log in with ssh grader@52.206.38.191 -i ~/.ssh/'private key file name' -p 2200

#  Force login with key pair and disable remote login of root user

edit the sshd_config file:  sudo nano /etc/ssh/sshd_config
Change Password Authentication to no
Change PermitRootLogin to no
also add AllowUsers grader
Restart ssh with sudo service ssh restart

#  Update packages

sudo apt-get update
sudo apt-get upgrade

#  Change ssh port to 2200

sudo nano /etc/ssh/sshd_config
Add port 2200

#  Configure Firewall

Check firewall status with sudo ufw status
Use these commands:
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw enable

#  Install apache, python, postgresql, git, and mod_wsgi

sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install postgresql
sudo apt-get install git

#  Clone project from GitHub

cd /var/wwww/itemcatalog
sudo git clone https://github.com/tomrhoads/GuitarCatalogProject

#  Install PIP and python packages

sudo apt-get install pip
sudo pip install sqlalchemy
sudo pip install flask
sudo pip install oauth2client
sudo pip install httplib2
sudo pip install python-psycopg2

#  Create new database user

Connect with sudo su - postgres
Type psql
Create user:  CREATE USER catalog WITH PASSWORD 'password';
Confirm creation: /du
Update permissions: ALTER ROLE catalog WITH LOGIN;
                    ALTER USER catalog CREATEDB;
Create database: CREATE DATABASE catalog WITH OWNER catalog;
Login: \c catalog
Revoke all rights: REVOKE ALL ON SCHEMA public FROM public;
Grant access to catalog user:  GRANT ALL ON SCHEMA public TO catalog;
Exit and restart:  \q then exit the sudo service postgresql restart

#  Configure Application

Configure WSGI file:
cd /var/www/itemcatalog
sudo nano itemcatalog.wsgi then add this:
#!/usr/bin/python 
import sys 
import logging 

logging.basicConfig(stream=sys.stderr) 
sys.path.insert(0,"/var/www/itemcatalog/")  

from catalog import app as application 
application.secret_key = 'super_secret_key'

restart apache:  sudo service apache2 restart

In python files change your create_engine() to engine = create_engine('postgresql://catalog:'password'@localhost/catalog')
Use full path to client_secrets.json file in __init__.py

Disable unused .conf files from /etc/apache2/sites-enabled/ with sudo a2dissite 000-default.conf

Restart Apache:  sudo service apache2 restart
Restart app:  sudo python __init__.py

#  Resources

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04
https://discussions.udacity.com/t/application-shows-with-my-public-ip-address-but-not-with-my-amazon-ec2-instance-public-url/37117/3





