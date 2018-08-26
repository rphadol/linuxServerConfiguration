# linuxServerConfiguration
deployment of CatalogApp project 

# Description:

Baseline installation of a Linux server and prepare it to host your web applications here its catalogApp. Secure server from a number of attack vectors, install and configure a database server, and deploy catalog applications onto it.

# Important info
IP address: 18.222.164.3

Accessible SSH port: 2200.
Application URL: http://18.222.164.3.xip.io/

# Steps to configure Linux Server
## Create a new user named grader and grant this user sudo permissions.
1. Log into the remote VM as root user through ssh:

            $ ssh root@18.222.164.3.
2. Add a new user called grader:

            $ sudo adduser grader.
3. Create a new file under the suoders directory: 

            $ sudo nano /etc/sudoers.d/grader. 
4. Fill that newly created file with the following line of text: "grader ALL=(ALL:ALL) ALL", then save it.

## Update all currently installed packages
            1.  $ sudo apt-get update.
            2.  $ sudo apt-get upgrade.
3. Install finger, a utility software to check users' status:

            $ apt-get install finger.

## Configure the key-based authentication for grader user
1. Generate an encryption key on your local machine with: 

            $ ssh-keygen  and then enter the path -f ~/.ssh/graderprv.ppk. after that enter paraphrase.
2. Log into the remote VM as root user through ssh and create the following file: 

            $ touch /home/grader/.ssh/authorized_keys.
3. Copy the content of the udacity_key.pub file from your local machine to the /home/grader/.ssh/authorized_keys file you just created on the remote VM. 
4. Then change some permissions:

       $ sudo chmod 700 /home/grader/.ssh.
       $ sudo chmod 644 /home/grader/.ssh/authorized_keys.
5. Finally change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh.
6. Now you are able to log into the remote VM through ssh with the following command: $ ssh -i ~/.ssh/graderprv.ppk grader@18.222.164.3 (ssh -i [privateKeyFilename] grader@18.222.164.3)


## Configure the Uncomplicated Firewall (UFW)
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

                  sudo ufw allow 2200/tcp
                  sudo ufw allow 80/tcp
                  sudo ufw allow 123/udp
                  sudo ufw enable 
                  
## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache, Install mod_wsgi

            sudo apt-get install apache2 
            sudo apt-get install python-setuptools libapache2-mod-wsgi
            sudo apt-get install libapache2-mod-wsgi python-dev.
            
2. Restart Apache sudo service apache2 restart   

## Install and configure PostgreSQL
1. Install PostgreSQL 

            sudo apt-get install postgresql

2. Check if no remote connections are allowed sudo vim /etc/postgresql/9.3/main/pg_hba.conf

                        local   all             postgres                                peer
                        local   all             all                                     peer
                        host    all             all             127.0.0.1/32            md5
                        host    all             all             ::1/128                 md5

3. Login as user "postgres" 

                        sudo su - postgres

4. Get into postgreSQL shell 
                        
                        psql

5. Create a new database named catalog and create a new user named catalog in postgreSQL shell

            postgres=# CREATE DATABASE catalog;
            postgres=# CREATE USER catalog;
            
6. Set a password for user catalog

            postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
   
7. Give user "catalog" permission to "catalog" application database

            postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;

8. Quit postgreSQL postgres=# \q

9. Exit from user "postgres" by typing exit

## Install git.
1. Install Git using 
      
            sudo apt-get install git
2. Use cd /var/www to move to the /var/www directory
3. Create the application directory 

            sudo mkdir catalog
4. Move inside this directory using: 

            cd catalog


## Install virtual environment, Flask and the project's dependencies
1. Install pip, the tool for installing Python packages: 

            $ sudo apt-get install python-pip.
2. If virtualenv is not installed, use pip to install it using the following command: 

            $ sudo pip install virtualenv.
3. Move to the catalog folder: 

            $ cd /var/www/catalog. 
4. Then create a new virtual environment with the following command: 

            $ sudo virtualenv venv.
5. Activate the virtual environment: 
                 
            $ source venv/bin/activate.
6. Change permissions to the virtual environment folder: 
      
            $ sudo chmod -R 777 venv.
6. Install Flask: 

            $ pip install Flask.
7. Install all the other project's dependencies: 

            $ pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2.
(You may need to use sudo for pip installation)



## clone and setup your Catalog App project
1. Clone the Catalog App to the virtual machine git clone https://github.com/rphadol/Catalog-App
2. Rename the project's name 

            sudo mv ./Catalog-App ./catalog
3. Move to the inner catalog directory using 

            cd catalog ( /var/www/catalog/catalog)
8. Rename project.py to __init__.py using 

            sudo mv project.py __init__.py
9. Edit database_setup.py,  __init__.py, lotsofdata.py and change 

            engine = create_engine('sqlite:///catalog.db') to 
            engine = create_engine('postgresql://catalog:sillypassword@localhost/catalog')
10. Install pip

            sudo apt-get install python-pip
11. Install psycopg2 

            sudo apt-get -qqy install postgresql python-psycopg2
12. Setup the database with: 

            $ cd /var/www/catalog/catalog/  python database_setup.py
13. to populate data run: 

                        sudo python lotsofdata.py
14.cd catalog directory and create catalog.wsgi file 

                  cd /var/www/catalog 
                  sudo nano catalog.wsgi 
and copy the following code and save it

            activate_this = '/var/www/catalog/venv/bin/activate_this.py'
            execfile(activate_this, dict(__file__=activate_this))

            import sys
            import logging
            logging.basicConfig(stream=sys.stderr)
            sys.path.insert(0, "/var/www/catalog/")

            from catalog import app as application
            application.secret_key = 'super_secret_key'
            
  Now in catalog directory you will see two things: catalog and catalog.wsgi        

## Configure and Enable a New Virtual Host
Create catalog.conf to edit: sudo nano /etc/apache2/sites-available/catalog.conf

Add the following lines of code to the file to configure the virtual host.

            <VirtualHost *:80>
                ServerName 18.222.164.3
                ServerAlias 18.222.164.3.xip.io
                ServerAdmin admin@18.222.164.3
                WSGIProcessGroup catalog
                WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/catalog/catalog/>
                    Order allow,deny
                    Allow from all
                </Directory>
                Alias /static /var/www/catalog/catalog/static/
                <Directory /var/www/catalog/catalog/static/>
                    Order allow,deny
                    Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
            </VirtualHost>


Enable the virtual host with the following command: sudo a2ensite catalog

## Update OAuth authorized JavaScript origins
To let users correctly log-in change the authorized URI to http://18.222.164.3.xip.io/ on both Google and Facebook developer dashboards.
2. after saving the URI download new json file copy content of the file into client_secrets.json file on VM.


## Restart Apache to launch the app
$ sudo service apache2 restart.
