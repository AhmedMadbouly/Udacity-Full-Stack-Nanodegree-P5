# Linux-Server-Configuration

Project 5 under `Full Stack Web Developer Nanodegree at Udacity`.


In this project we take a baseline installation of a Linux distribution on a virtual machine and prepare it to host our web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

You can visit http://52.24.125.52 for the website deployed.

### Tasks
- Launch Virtual Machine
- Create a new user named grader
- Give the grader the permission to sudo
- Give grader secure passwords
- Update all currently installed packages
- Change the SSH port from 22 to 2200
- Configure SSH authentication
- Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
- Configure the local timezone to UTC
- Install and configure Apache to serve a Python mod_wsgi application
- Install and configure `PostgreSQL`
- Do not allow remote connections
- Create a new user named `catalog` with limited permissions
- Install git, clone and setup Catalog App project (from GitHub repository)

##### Launch Virtual Machine and create a new user named grader and give him permission to sudo
`$ sudo adduser grader`
`vim /etc/sudoers`
`touch /etc/sudoers.d/grader`
`vim /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

##### Update password policy for the user
- Force user to update his password on first login (expire)
`sudo passwd -e grader`


##### Update all currently installed packages
`$ sudo apt-get update` # update the source lists
`$ sudo apt-get upgrade` # upgrade software

##### Change the SSH port from 22 to 2200
Use `sudo vim /etc/ssh/sshd_config` and then change Port `22` to Port `2200` , save & quit.
Reload SSH using `$ sudo service ssh restart`


##### Configure SSH authentication:
- Generate SSH key from local environment `$ ssh-keygen` # for example in ~/.ssh/key_pairs
- Display and copy the contents of your new public key file and Copy the public key
`cat ~/.ssh/key_pairs.pub`
- Access remote server
`ssh grader@PUBLIC_IP -p 2200`
- Create a new SSH directory and move inside this directory
`$ mkdir ~/.ssh && cd .ssh`
- Create new file `authorized_keys`
`$ touch .ssh/authorized_keys`
- Add the public key i just copied to the `authorized_keys` file
`$ vim .ssh/authorized_keys`
`$ chmod 700 .ssh`
`$ chmod 644 .ssh/authorized_keys`
reload SSH using `$ sudo service ssh restart`

Now i can use ssh to login with the new user
`ssh grader@52.24.125.52 -p 2200 -i ~/.ssh/authorized_keys`

##### Configure the Uncomplicated Firewall (UFW)
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
`$ sudo ufw allow 2200/tcp`
`$ sudo ufw allow 80/tcp`
`$ sudo ufw allow 123/udp`
`$ sudo ufw enable `

##### Configure the local timezone to UTC
Configure the time zone `sudo dpkg-reconfigure tzdata`, It is already set to UTC.

##### Install and configure Apache to serve a Python mod_wsgi application
Install Apache `$ sudo apt-get install apache2`
Install mod_wsgi `$ sudo apt-get install libapache2-mod-wsgi python-dev`
Restart Apache `$  sudo service apache2 restart`

##### Install and configure PostgreSQL
Install PostgreSQL `$ sudo apt-get install postgresql`

Check if no remote connections are allowed
`sudo vi /etc/postgresql/9.5/main/pg_hba.conf`

Login as user `postgres`
`$ sudo su - postgres`

Connect to postgreSQL shell `$ psql`

Create a new database named `catalog` and create a new user named `catalog` in postgreSQL shell

`postgres=# CREATE DATABASE catalog;`
`postgres=# CREATE USER catalog;`

Set a password for user catalog
`postgres=# ALTER ROLE catalog WITH PASSWORD 'password';`

Give user "catalog" permission to "catalog" application database
`postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

Quit postgreSQL `postgres=# \q`

Exit from user "postgres" `$ exit`


##### Install git, clone and setup your Catalog App project.
Install Git using `sudo apt-get install git`

Use `cd /var/www` to move to Apache www folder

Create the application directory `sudo mkdir FlaskApp`
Move inside directory using `cd FlaskApp`

Clone the Catalog App to the virtual machine `git clone https://github.com/kongling893/Item_Catalog_UDACITY.git`
Rename the project's name `sudo mv ./Item_Catalog_UDACITY ./FlaskApp`
Move to the inner FlaskApp directory using `cd FlaskApp`
Rename `website.py` to `__init__.py` using `sudo mv website.py __init__.py`
Edit `database_setup.py`, `website.py` and `functions_halper.py` and change engine = create_engine('sqlite:///toyshop.db') to engine = create_engine('postgresql://catalog:password@localhost/catalog')
Install pip `sudo apt-get install python-pip`
Use pip to install dependencies `sudo pip install -r requirements.txt`
Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
Create database schema `sudo python database_setup.py`
Configure and Enable a New Virtual Host
Create FlaskApp.conf to edit: 
`sudo nano /etc/apache2/sites-available/FlaskApp.conf`

Add the following lines of code to the file to configure the virtual host.

```
<VirtualHost *:80>
	ServerName 52.24.125.52
	ServerAdmin qiaowei8993@gmail.com
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
```

Enable the virtual host wif the following command: `sudo a2ensite FlaskApp`

Create the .wsgi File under /var/www/FlaskApp:
`cd /var/www/FlaskApp`
`$ sudo nano flaskapp.wsgi`

Add the following lines of code to the flaskapp.wsgi file:
`#!/usr/bin/python`
`import sys`
`import logging`
`logging.basicConfig(stream=sys.stderr)`
`sys.path.insert(0,"/var/www/FlaskApp/")`
`from FlaskApp import app as application`
`application.secret_key = 'Secret key`

Restart Apache
`Restart Apache sudo service apache2 restart`


References:
`https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps`
