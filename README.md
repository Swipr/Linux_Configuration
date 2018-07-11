
# Description

&nbsp;

### Linux Server Configuration.
*This is project 6 of the [Udacity Full Stack Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004) course.*



#  Server details

&nbsp;



* **IP address:** `18.191.51.64`

* **SSH port:** `2200`

* **URL:** `http://ec2-18-191-51-64.us-east-2.compute.amazonaws.com`



						* Working Google Sign-in. *



&nbsp;

# How-to

&nbsp;

### Step A: The Lightsail instance  
#### 1. Create a [Lightsail](https://aws.amazon.com/lightsail/) instance in AWS
* Inside the Lightsail page, click on “Create instance” page
* Under “Select Blueprint” section click on “OS Only”
* Select “Ubuntu”
* Select the first basic plan (free for the first month then $5 afterwards)
* Click the “Create” button. The instance will take few moments to appear on the instances page  
#### 2. Download the Lightsail default private key to local device
* On the top right click “Account” link, then go to “SSH Keys”
* Download the default SSH Key
*  SSH into your Lightsail server from your terminal : `sudo ssh -i “the_downloaded_key.pem” ubuntu@ 18.191.51.64`  
#### 3. Add ports 2200/tcp and 123/udp
* On your instance page click on the “Networking” tab
* Add ports as follow: Custom:TCP:2200 and Custom:UDP:123

#### 4. Update your Linux server
* Update server packages using the command: `sudo apt-get update`
* Upgrade the packages: `sudo apt-get upgrade`
* Remove un-used packages: `sudo apt-get autoremove`
* enable unattended updates: `sudo apt install unattended-upgrades`
* You can also install the user identifier package “finger” : `sudo apt-get install finger`

#### 5. Change ssh port from 22 to 2200
* Open the sshd_config file: `sudo nano /etc/ssh/ssdh_config`
* Change the SSH port from 22 to 2200
* Save the change
* Restart the ssh service: `sudo sevice ssh restart`

#### 6. Update ufw ports and status
* Allow the new port 2200: `sudo ufw allow 2200/tcp`
* Allow port 80: `sudo ufw allow www`
* Allow port 123: `sudo ufw allow ntp`
* Active the ufw: `sudo ufw enable`
* Restart the ssh service: `sudo service ssh restart`

#### 7. Configure the local timezone to UTC
* Configure the time zone: `sudo dpkg-reconfigure tzdata`

============================================================

### Step B: The “grader” user
#### 1. Create the grader user
* Create the grader user: `sudo adduser -m grader`
* Assign sudo status to grader: `usermod -aG sudo grader`


#### 2. Create a keypair for grader user
* ssh into the grader user account: `ssh grader@18.191.51.64 -p 2200`
* Create a directory .ssh: `sudo mkdir .ssh`
* Create a file with the name `authorized_keys` inside the `.ssh` directory: `sudo touch /.ssh/authorized_keys`
* Change the permission for the `.ssh` folder: `sudo chmod 700 .ssh`
* Change the permissions for the  `authorized_keys` file: `sudo chmod 644 /.ssh/authorized_keys`
* Open a new terminal window on your computer and type the command: `ssh-keygen`
* Create the file in which to save the key `~/.ssh`
* The command will create two files in the specified directory. Open the one with the extension PUB using the command: `cat ~/.ssh/KEY_NAME.pub`
* Copy the content
* Open the `authorized_keys` file created for the grader user: `sudo nano /.ssh/authorized_keys`
* Paste the content
* Reopen the `sshd_config` file:  `sudo nano /etc/ssh/sshd_config`
* Change `password authentication` from “yes” to “no”
* Also change the `PermitRootLogin` property to "no"
* Restart the SSH service: `sudo service ssh restart`

============================================================

### Step C: Apache2, mod-wsgi, and Git
#### 1. Install Apache2
*  `sudo apt-get install apache2`

#### 2. Add mod-wsgi for python environment
* `sudo apt-get install libapache2-mod-wsgi-py3`

#### 3. Install git
* `sudo apt-get install git`

============================================================

### Step D: Clone the app into Apache2
#### 1. Clone the Dog Breed app
* Create a new folder in the `/var/www` directory: `sudo mkdir FlaskApp`
* cd into the new FlaskApp directory: `cd /var/www/FlaskApp`
* Clone you dog breed app with the new name "FlaskApp": `sudo git clone https://github.com/Swipr/Dog_Catalog.git FlaskApp`
* The path to the catalog app should be:`/var/www/FlaskApp/FlaskApp`

#### 2. Add the packages used in your app to enable them inside the new environment
* Install pip: `sudo apt-get install python3-pip`
* Install `Flask` and the rest of the packages:
* `sudo pip3 install Flask`
* `sudo pip3 install httplib2`
* `sudo pip3 install sqlalchemy`
* `sudo pip3 install oauth2client`
* `sudo pip3 install --upgrade oauth2client`
* `sudo pip3 install sqlalchemy`
* `sudo pip3 install sqlalchemy_utils`
* `sudo pip3 install requests`
* `sudo pip3 install bleach`

#### 3. Rename and edit the application python file
* Rename the cloned application python file from its current name `dog_catalog.py`  to ` __init__.py`
* In the ` __init__.py` edit the client_secrects.json file path to : `/var/www/FlaskApp/FlaskApp/client_secrets.json`

#### 4. Set up the database  
* Install PostgreSQL : `sudo apt-get install postgresql`
* Check if no remote connections are allowed:  `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
* Login as user "postgres": `sudo su - postgres`
* Get into postgreSQL shell: `psql`
* Create a new database named "catalog" and create a new user named "catalog" in postgreSQL shell:
* `CREATE DATABASE catalog;`
* `CREATE USER catalog;`
* `ALTER ROLE catalog WITH PASSWORD 'password';`
* Give user "catalog" permission to "catalog" application database:
* `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
* Quit postgreSQL: `\q`
* Exit from user "postgres": `exit`
* Change the path to the database in the `__init__.py` and the `database_setup.py` files to : `create_engine('postgresql://catalog:password@localhost/catalog')`
* Install psycopg2: `sudo apt-get -qqy install postgresql python-psycopg2`
* Create database schema: `sudo python database_setup.py`

============================================================

### Final step: Fire the app to the web
#### 1. Create a new Virtual Host
* Create the `FlaskApp.conf` file: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
* Paste the text below inside the `FlaskApp.conf` file:  

	```  
	<VirtualHost *:80>  
		ServerName http://ec2-18-191-51-64.us-east-2.compute.amazonaws.com/dogtypes
		ServerAdmin mail@example.com  
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

* Disable the default virtual host:  `sudo a2dissite 000-default.conf`
* Enable the new virtual host: `sudo a2ensite FlaskApp.conf`

#### 2. Create a wsgi file for the app.
* The wsgi file sits inside the parent FlaskApp directory: `sudo nano /var/www/FlaskApp/flaskapp.wsgi`
* Paste the text below inside the flaskapp.wsgi file:

	```  
	#!/usr/bin/python3  
	import sys  
	import logging  
	logging.basicConfig(stream=sys.stderr)  
	sys.path.insert(0,"/var/www/FlaskApp/")  

	from FlaskApp import app as application  
	application.secret_key = 'super_secret_key'  
	```

#### 3. Restart the Apache server
* Start Apache2 service with the command: `sudo service apache2 restart`


# References:
* [https://github.com/kongling893/Linux-Server-Configuration-UDACITY](https://github.com)
