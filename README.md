# UdacityProject 6
Linux Server Configuration -- secure and set up a Linux server and have one of my web applications running live on the secure web server.

## Get your server.
1. Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://aws.amazon.com/lightsail/). There are full details on setting up Lightsail instance on the next page.
2. Follow the instructions provided to SSH(port:`22` by default) into your server.

## Check and Update all currently installed packages
1. Get info of all installed packaged: `sudo apt-get update`
2. Update  all packages to latest: `sudo apt-get upgrade`

## Change the SSH port and configure the firewall
1. Open the config file: `sudo nano /etc/ssh/sshd_config`
2. Change Port `22` to Port `2200`
3. Reload SSH: `sudo service ssh restart`
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):
```
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow ntp
sudo ufw allow 123/udp
sudo ufw enable 
```
5. Verify the rules are active: `sudo ufw status`

## Add a new user named grader
1. Create a new user account named grader: `sudo adduser grader`
2. Give grader the permission to sudo: 
   a.Open and copy the content for ubuntu user in sudoers -- `nano /etc/sudoers`
   b.Create grader file -- `nano touch /etc/sudoers.d/grader`
   c. Paste the and save the content in grader file
3. Create an SSH key pair for grader using the ssh-keygen tool on your local machine: `sudo ssh-keygen ~/.ssh/LinuxCourse`
4. Copy the content from the public key file generated: `sudo nano ~/.ssh/LinuxCourse.pub`
5. The privatee key will be saved on your local machine as LinuxCourse in ~/.ssh
6. Switch to user grader: `sudo su - grader`
7. Create a .ssh directory: `mkdir .ssh`
8. Create a file to store the public key: `sudo touch .ssh/authorized_keys`
9. Paste the content you ust copied from public key file to authorized_keys file: `sudo nano .ssh/authorized_keys`
10. Change the permission of the directory and file:
```
sudo chmod 700 /home/grader/.ssh
sudo chmod 644 /home/grader/.ssh/authorized_keys
```
11. Change the owner from root to grader: `sudo chown -R grader:grader /home/grader/.ssh`
12. Restart ssh service: `sudo service ssh restart`

## Disable root login
1. Edit the 'sshd_config' file: `sudo nano /etc/ssh/sshd_config`
2. Update 'PermitRootLogin without-password' to 'PermitRootLogin no'
3. Make sure there is no password authentication since we are going to only use private key authentication: 'PasswordAuthentication no'
4. Restart ssh service: `sudo service ssh restart`

## Prepare to deploy your project
1. Configure the local timezone to UTC: `sudo timedatectl set-timezone UTC`
2. Install Apache to serve a Python mod_wsgi application:
    a. Install Apache: `sudo apt-get install apache2`
    b. Install the libapache2-mod-wsgi package for Python3: `sudo apt-get install libapache2-mod-wsgi-py3`
3. Install required packages for Item Catalog project:
```
sudo pip install httplib2
sudo apt-get install python-pip
sudo pip install Flask
sudo pip install httplib2
sudo pip install oauth2client
sudo pip install sqlalchemy
sudo pip install psycopg2
sudo pip install sqlalchemy_utils
sudo pip install requests
sudo pip install render_template
sudo pip install redirect
```
## Install and configure PostgreSQL
1. Install PostgreSQL: `sudo apt-get install postgresql`
2. Switch to user "postgres": sudo su - postgres
3. Get into postgreSQL shell: `psql`
4. Create a new database named catalog: `postgres=# CREATE DATABASE catalog;`
5. Create a new user named catalog: `postgres=# CREATE USER catalog;`
6. Set a password for user catalog `postgres=# ALTER ROLE catalog WITH PASSWORD 'catalog';`
7. Give user catalog permission to catalog database: `postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
8. Quit postgreSQL shell: `postgres=# \q`
9. Make sure no remote connections are allowed from 'pa_hba.conf' file: `sudo nano /etc/postgresql/9.6/main/pg_hba.conf`

## Install Git
1. Install Git package: `sudo apt-get install git`

## Deploy the Item Catalog project
1. Switch to user ubuntu: `sudo su - ubuntu`
2. Move to '/var/www' directory: `cd /var/www`
3. Create a directory for Flask apps: `sudo mkdir FlaskApp`
4. Move inside 'FlaskApp' directory: `cd FlaskApp`
5. Clone the Catalog Project from git: `git clone https://github.com/mengluo/Catalog_Project.git`
6. Move into 'Catalog_Project' directory: `cd Catalog_Project`
7. Rename 'CatalogApp.py' to '\__init__.py': `sudo mv CatalogApp.py __init__.py`
8. Edit 'database_setup.py', 'CatalogDataLoad.py' and '\__init__.py': `sudo nano [filename]`
9. Update all "create_engine('sqlite:///categories.db') to "engine = create_engine('postgresql://catalog:catalog@localhost/catalog')"
10. Create database schema: `sudo python3 databse_setup.py`
11. Load catalog data: `sudo python3 CatalogDataLoad.py`
12. Create 'flaskapp.wsgi' File under /var/www/FlaskApp: `sudo touch /var/www/FlaskApp/flaskapp.wsgi`
13. Add the following lines of code to the flaskapp.wsgi file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp")
sys.path.insert(1,"/var/www/FlaskApp/Catalog_Project")

from Catalog_Project import app as application
application.secret_key = 'super_secret_key'
```
14. Create 'FlaskApp.conf' under '/etc/apache2/sites-available': `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
15. Add the following lines of code to FlaskApp.conf file to configure the virtual host:
```
<VirtualHost *:80>
                ServerName http://ec2-18-188-39-140.us-east-2.compute.amazonaws$
                ServerAdmin Raymond2012.L@gmail.com
                WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
                <Directory /var/www/FlaskApp/Catalog_Project/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/FlaskApp/Catalog_Project/static
                <Directory /var/www/FlaskApp/Catalog_Project/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
16. Reload Apache sudo service apache2 reload
17. Restart Apache sudo service apache2 restart
18. Go to Broswer and type 'http://ec2-18-188-39-140.us-east-2.compute.amazonaws.com/', you will see the deployed Flask app runing!

## Server information
IP address: `18.188.39.140`
URL: `http://ec2-18-188-39-140.us-east-2.compute.amazonaws.com`
SSH port: `2200`
HTTP port: `80`
NTP port: `123`

## How to ssh login as the grader:
1. Download the private key file 'LinuxCourse' to put under '~/.ssh' on local machine
2. Open terminal and do SSH login on local machine: `sudo ssh -i ~/.ssh/LinuxCourse grader@18.188.39.140 -p 2200`
3. Enter your machine credentials: ******
4. Enter passphrase for key 'LinuxCourse': 1234567890

## References
1. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
