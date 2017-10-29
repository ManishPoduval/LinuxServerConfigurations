# PROJECT - LINUX SERVER CONFIGURATION

# A baseline installation of a linux server and preparing it to host our web applications, securing it from a number of attack vectors, installing and configuring a database server and deploying the Item Catalog application onto it.

PUBLIC IP ADDRESS: http://13.126.9.55 

URL: Can't create one as Amazon gives this error (Directory Service is not available in Asia Pacific (Mumbai))

PORT: 2200


-----SUMMARY OF CONFIGURATIONS MADE-----

# Update the server software
	$ sudo apt-get update
	$ sudo apt-get upgrade
	
# Create a new user	
	$ sudo adduser grader
	
# Give new user grader sudo permission	
	$ touch /etc/sudoers.d/grader
	$ nano /etc/sudoers.d/grader
		
	#add the following line and save	
		grader ALL=(ALL:ALL) ALL

# log in to grader user account
	$ su - grader
	$ sudo mkdir .ssh
	$ sudo touch .ssh/authorized_keys
	$ sudo nano .ssh/authorized_keys
	#copy the contents of the public key generated here

# change permissions 
	$ sudo chown -R grader:grader /home/grader/.ssh
	$ sudo chmod 0700 /home/grader/.ssh
	$ sudo chmod 0600 /home/grader/.ssh/authorized_keys
	$ sudo nano /etc/ssh/sshd_config
	#Find PermitRootLogin line and edit it to no
	#Find the PasswordAuthentication line and edit it to no
	#Find the Port line and change 22 to 2200
	#add the following line and save
		AllowUsers grader
	$ sudo service ssh restart

# change UFW
	$ sudo ufw default deny incoming
	$ sudo ufw allow 2200/tcp
	$ sudo ufw allow 80/tcp
	$ sudo ufw allow 123/udp
	$ sudo ufw deny 22/tcp
	$ sudo ufw enable
	
# configure timezone
	$ sudo dpkg-reconfigure tzdata
	#select none and then UTC
	
# install apache and mod-wsgi
	$ sudo apt-get install apache2 libapache2-mod-wsgi
	#Enable mod-wshi
	$ sudo ae2mod wsgi

# install git
	$ sudo apt-get install git

# Install python app dependencies
	$ sudo apt-get install python-pip
	$ sudo pip install Flask
	$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests	

# install postgresql
	$ sudo apt-get install postgresql postgresql-contrib libpq-dev python-dev
	
# log in to postgresql, create new user, database and revoke all rights
	$ sudo su - postgres
	$ psql
	>CREATE USER catalog WITH PASSWORD 'password';
	>CREATE DATABASE catalogdb WITH OWNER catalog;
	>REVOKE ALL ON SCHEMA public FROM public;
	>GRANT ALL ON SCHEMA public TO catalog;
	
# Clone Item-catalog app repo
	$ cd /var/www
	$ sudo mkdir CatalogApp
	$ cd CatalogApp
	$ git clone https://github.com/ManishPoduval/Item-Catalog.git
	$ cd Item-Catalog
	$ mv application.py __init__.py
	# edit database_setup.py items.py 
	#change line 'engine = create_engine('sqlite:///itemcatalog.db')' to
		engine = create_engine('postgresql://catalog:password@localhost/catalogdb')


# create database schema
	$ sudo python database_setup.py
	$ sudo python items.py
	
# create .wsgi file
	$ cd /var/www/CatalogApp
	$ sudo touch catalogapp.wsgi
	$ sudo nano catalogapp.wsgi
	#Add the following code and save
		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/CatalogApp/")

		from ItemCatalog import app as application
		application.secret_key = 'super_secret_key'

# edit virtual file
	$ sudo nano /etc/apache2/sites-available/000-default.conf
	#add the following code
		<VirtualHost *:80>
			ServerAdmin manishpoduval@hotmail.com
			WSGIScriptAlias / /var/www/CatalogApp/catalogapp.wsgi
			<Directory /var/www/CatalogApp/ItemCatalog/>
					Order allow,deny
					Allow from all
			</Directory>
			Alias /static /var/www/CatalogApp/ItemCatalog/static
			<Directory /var/www/CatalogApp/ItemCatalog/static/>
					Order allow,deny
					Allow from all
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			CustomLog ${APACHE_LOG_DIR}/access.log combined
		</VirtualHost>
	$ sudo sercvice apache2 restart	


-----SUMMARY OF SOFTWARES INSTALLED-----

	apache2
	libapache2-mod-wsgi
	git
	python-pip
	postgresql
	
-----REFERENCES-----

	Udacity forums
	StackOverflow
	postgresql.com





    
    
