Perform Basic Configuration
=============================
Step 1:  Launch your Virtual Machine with your Udacity account and log in. Refer to: https://www.udacity.com/account#!/development_environment
  
  Performed the following steps:
    - Downloaded the Private Key attached to the page
    - Moved the private key file into the folder ~/.ssh. 
          mv ~/Downloads/udacity_key.rsa ~/.ssh/
    - In the terminal, typed the following command
          chmod 600 ~/.ssh/udacity_key.rsa 
          ssh -i ~/.ssh/udacity_key.rsa root@52.10.70.16
  
Step 2: Create a new user named grader and grant this user sudo permissions.
    
  Performed the following steps:
    - adduser grader
    - In the folder /etc/sudoers.d
         created a file called grader with the following content:
              grader ALL=(ALL:ALL) ALL
    - confirmed grader was created by using the following command:
         cat /etc/passwd

Step 3:  Update all currently installed packages.

  Performed the following steps:
   - Updated the list of available packages 
      apt-get update
   - upgraded the versions for packages
      apt-get upgrade
   - autoremoved unused packages
      apt-get autoremove
      
  
Step 4:  Configure the local timezone to UTC.
  
  Performed the following steps:
   - Opened the timezone selection dialog:
     dpkg-reconfigure tzdata
     Selected "None of the above" in the first selection window, followed  by "UTC" in the second window.

Step 5: Create ssh keys for grader
 
  Performed the following steps:
  - using ssh-keygen generated an encryption key. This should be done on a local machine. 
    ssh-keygen  
        created grader_key and grader_key.pub
  - Logged into server, in the grader account home directory /home/grader.
  - created a directory
       mkdir .ssh
  - created a file authorized_keys
  - chmod 700 .ssh
  - chmod 644 .ssh/authroized_keys
  - In that file, copied the public key (from grader_key.pub)
  - Changed the owner of the /home/grader/.ssh folder to grader
       chown -R grader:grader /home/grader/.ssh
  - Then logged out
  - Then sshed using grader credentials
      ssh -i ~/.ssh/grader_key grader@52.10.70.16

Secure your server
=================

Step 6: Change the SSH port from 22 to 2200
  
  Performed the following steps:
  - create a backup of file /etc/ssh/sshd_config 
    cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
  - Opened the sshd config file 
    sudo vi /etc/ssh/sshd_config
    Enter password for grader 
  - Made the following changes in the file
    Port 2200 (from Port 22)
    PasswordAuthentication no
    PermitRootLogin no
  - Save and exit the file
  - Restarted SSH service
      sudo service ssh restart
  - Now ssh back using the following command
    ssh -i ~/.ssh/grader_key grader@52.10.70.16 -p 2200

Step 7: Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

   Performed the following steps:
   - Set incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
     sudo ufw allow 2200/tcp
     Enter password for grader.
     sudo ufw allow 80/tcp
     sudo ufw allow 123/udp
   - Set UFW to enable
     sudo ufw enable
     type y when prompted for Command may disrupt existing ssh connections. Proceed with operation (y|n)?
   - check status
     sudo ufw status
     logout of the service and log back in 
     ssh -i ~/.ssh/grader_key grader@52.10.70.16 -p 2200

Install your application
========================

Step 8: Install and configure Apache to serve a Python mod_wsgi application
 
  Performed the following steps:
  - Installed the apache2 package
    sudo apt-get install apache2
  - To configure Apache to hand-off certain requests to an application handler - mod_wsgi. Installed mod_wsgi: 
    sudo apt-get install libapache2-mod-wsgi
  - Started apache2 service
    sudo service apache2 start
  - Enabled mod_wsgi
    sudo a2enmod wsgi
   

Step 9: Install Git and clone the catalog app from Github
   
  Performed the following steps to install git and to clone the catalog app
     sudo apt-get install git
    - Then created a catalog directory under /var/www and changed the owner to grader
       cd /var/www
       sudo mkdir catalog
    - For security purposes, change the owner and group to grader for the catalog 
       sudo chown -R grader:grader catalog
    - In the catalog directory, git clone the catalog app project from Github
       cd /catalog
       git clone https://github.com/harinikanand/CatalogApp.git catalog
    - In the same directory, create a catalog.wsgi
     
            #!/usr/bin/python
            import sys
            import logging
            logging.basicConfig(stream=sys.stderr)
            sys.path.insert(0,"/var/www/catalog/")

            from catalog import app as application
            application.secret_key = 'Add your secret key'


Step 10: Install Dependencies

  Performed the following steps to install dependencies (Flask, oauth, sqlalchemy, psycopg2)
    Installed Pip 
     - sudo apt-get install python-pip
    In the /var/www/catalog, using pip and apt-get, install virtualenv, Flash, outh, sqlalchemy and python adapter for psycopg2
     - cd /var/www/catalog
     - sudo pip install virtualenv
     - sudo virtualenv venv
     - source venv/bin/activate
     - sudo chmod -R 777 venv
     - pip install Flask
     - pip install bleach httplib2 request oauth2client sqlalchemy 
     - sudo apt-get install python-psycopg2 (sudo apt-get build-dep python-psycopg2;
                                             pip install psycopg2)
  
   
Step 11: Create, configure and enable a Virtual Host

   Performed the following steps to create a virtual host
     sudo \vi /etc/apache2/sites-available/catalog.conf
         <VirtualHost *:80>
    	ServerName 52.10.70.16
    	ServerAlias ec2-52-10-70-16.us-west-2.compute.amazonaws.com
    	ServerAdmin admin@52.10.70.16
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
    Enabled the virtual Host
        sudo a2ensite catalog


Step 12: Install and configure PostgreSQL:
         Do not allow remote connections
         Create a new user named catalog that has limited permissions to your catalog application database

    Performed the following steps to install PostgreSQL:
    Install PostgreSQL
      - sudo apt-get install postgresql postgresql-contrib
    Since PostgresQL is created during installation with user 'postgres'.
    Change the user to postgres and connect to the database.
      - sudo su - postgres
      - psql
    At the psql prompt, created a user called catalog with LOGIN role and password 'CATALOG'. The user catalog is able to create database tables
         - CREATE USER catalog WITH PASSWORD 'CATALOG';
         - ALTER USER catalog CREATEDB;
         - CREATE DATABASE catalog WITH OWNER catalog;
         - \c catalog
         - REVOKE ALL ON SCHEMA public FROM public;
         - GRANT ALL ON SCHEMA public TO catalog;
         - \du
         - \q, then exit

    Ensure no remote connections are allowed (That is keep the DEFAULT configuration)
      - sudo \vi /etc/postgresql/9.3/main/pg_hba.conf
      - cd to /var/www/catalog/catalog
    Open the database_setup.py and finalproject.py
        and change the sqlite to postgresql ('postgresql://catalog:CATALOG@localhost/catalog')
    change finalproject.py to __init__.py
    python database_setup.py
    Populate the genres and books in the database by running 
    python lotsofbooks.py

Step 13: Setup New OAuth2 keys
 
   By logging in https://console.developers.google.com/project,
   Generate a new client_secrets.json by correcting the credentials parameters (redirect and origin)
   New client_secrets file:
     {"web":{"client_id":"951826189120-c8c535d50qeca1hm3dgf6kc5ulpg9oee.apps.googleusercontent.com","project_id":"catalog-122815","auth_uri":"https://accounts.google.com/o/oauth2/auth","token_uri":"https://accounts.google.com/o/oauth2/token","auth_provider_x509_cert_url":"https://www.googleapis.com/oauth2/v1/certs","client_secret":"nrER3GN1EsiwxJ7_08PHBf_D","redirect_uris":["http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/login","http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/gconnect","http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/oauth2callback"],"javascript_origins":["http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com","http://52.10.70.16","http://52.10.70.16:80"]}}
   I had to make several changes to __init__.py to be able to run the catalog app.
   These are related to getting the google login work. This is particularly related to opening the client_secrets.json file.

References:
1. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
2. http://stackoverflow.com/questions/28184419/pydrive-invalid-client-secrets-file
3. http://codegur.com/30610894/oauth-attributeerror-str-object-has-no-attribute-access-token
4. http://stackoverflow.com/questions/29565392/error-storing-oauth-credentials-in-session-when-authenticating-with-google
5. https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files
6. https://help.ubuntu.com/
7. http://serverfault.com/questions/609947/database-connection-to-postgresql-refused-for-flask-app-under-mod-wsgi-when-star
8. https://blog.openshift.com/build-your-app-on-openshift-using-flask-sqlalchemy-and-postgresql-92/
9. http://docs.evergreen-ils.org/2.2/_configure_the_apache_web_server.html
10. https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts
11. http://dba.stackexchange.com/questions/35316/why-is-a-new-user-allowed-to-create-a-table
12. http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html
13. http://killtheyak.com/use-postgresql-with-django-flask/


Public IP Address: 52.10.70.16

PORT: 2200

Server Name: http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/
Full command: ssh -i ~/.ssh/grader_key grader@52.10.70.16 -p 2200

Password for grader account is grader.

14. The following public URLs are supported;
        1. http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/login - provide a mechanism to login
        2. http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/ - same as the URL 14.3
        3. http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres - shows all the genres
        4. http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/<int:genre_id>/ - same as 14.5
        5. http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/<int:genre_id>/list - shows all the books in the genre with genre_id
        6. http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/<int:genre_id>/list/JSON - Shows the JSON output for all the books in a genre with id genre_id
        7. http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/<int:genre_id>/list/<int:book_id>/JSON - shows the JSON putput for all book with book_id in the genre genre_id.
         http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/JSON - shows the JSON output for all the genres
15.  The following Private URLs are supported:
         http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/new - URL to add a new genre
         http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/<int:genre_id>/edit - URL for editing the genre with genre_id
         http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/<int:genre_id>/delete - URL for deleting the genre with genre_id
         http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/<int:genre_id>/list/new - URL to add new book in genre with id
         http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/<int:genre_id>/list/<int:book_id>/edit - URL to edit a book (with book_id) in the genre with genre_id
         http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/genres/<int:genre_id>/list/<int:book_id>/delete - URL to delete a book (with book_id) in the genre with genre_id
         http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/gconnect - URL for connect to google plus oauth2.0 system to obtain googleplus credentials
         http://ec2-52-10-70-16.us-west-2.compute.amazonaws.com/gdisconnect - URL for disconnect from google plus oauth2.0 system 