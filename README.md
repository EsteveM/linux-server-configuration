# Linux Server Configuration Project

This project is intended to work on a baseline installation of a publicly accessible Linux server, so that it eventually hosts one of my web applications. As part of this work, the server is accessed and secured against a variety of attacks, a database server is set up, and one of my web applications is successfully deployed.

## Table of Contents

* [Some key information](#some-key-information)
* [Description of the Project](#description-of-the-project)
* [Getting Started](#getting-started)
* [Acknowledgements](#acknowledgements)
* [Contributing](#contributing)

## Some key information

This section provides you with some key information about this project:

* Server IP address: 18.185.87.128.
* Server SSH port: 2200.
* Complete URL to the hosted web application: http://18.185.87.128.xip.io/.
* Summary of software I installed and configuration changes I made: See section [Description of the Project](#description-of-the-project).
* List of third-party resources consulted to complete this project: See section [Acknowledgements](#acknowledgements).

## Description of the Project

This project progressively transforms a barebones Ubuntu server on a secure server where one of my web applications runs live. The work that has been made is best described by explaining the steps followed:

### Get the server

#### Step 1 - Start an Ubuntu Linux server instance

The first step in this project is to start a publicly accessible Ubuntu Linux server instance in [Amazon Lightsail](https://lightsail.aws.amazon.com). Firstly, an Amazon Web Services account must be previously set up. Then, we create a Lightsail instance, and indicate we want a plain Ubuntu Linux instance. After that, as instance plan, we choose the lower tier one, which is good enough for our purposes. Once that has been done, a name must be given to the instance. I have named mine *_linux-server-configuration_*. After these steps, the instance starts up. In this case, its IP address is 18.185.87.128. As DNS name, 18.185.87.128.xip.io has been chosen because it is a simple and inexpensive way to derive a DNS name from an IP address provided by [Basecamp](http://xip.io).

#### Step 2 - SSH into the server instance

At this point, it is possible to log into the instance with SSH as the *_ubuntu_* user, from the main page for the instance at [Amazon Lightsail](https://lightsail.aws.amazon.com).

### Secure the server

#### Step 3 - Update currently installed packages

This is the first step to secure our server, and consists of a number of commands on the Linux console:

* Firstly, we update the package source list: `sudo apt-get update`.
* Secondly, we upgrade the software: `sudo apt-get upgrade`.
* Thirdly, and optionally, we remove packages no longer needed: `sudo apt-get autoremove`.
* Finally, we install the finger application, which we will use when we work with users: `sudo apt-get install finger`.


#### Step 4 - Change the SSH port from 22 to 2200, and configure the Lightsail firewall to allow it

This is the second step to secure our server, and consist of the following actions:

* Firstly, we download the default private key for the *_ubuntu_* user from [Amazon Lightsail](https://lightsail.aws.amazon.com). It is downloaded into the machine we want to access the server instance from. This private key is stored within the *_LightsailDefaultKey-eu-central-1.pem_* file.
* Secondly, we change the permissions of the private key file so that only the owner can read and write: `chmod 600 ~/.ssh/LightsailDefaultKey-eu-central-1.pem`.
* Thirdly, we successfully check we can ssh into the server machine on port 22 using our private key: `ssh ubuntu@18.185.87.128 -p 22 -i ~/.ssh/LightsailDefaultKey-eu-central-1.pem`.
* Then, we check on the *_sshd_config_* file that password authentication is already disabled: `sudo nano /etc/ssh/sshd_config`.
* After that, we configure the Lightsail firewall to allow SSH on port 2200. This is accomplished on the main page for the instance at [Amazon Lightsail](https://lightsail.aws.amazon.com).
* At this point, we allow port 2200 on the *_sshd_config_* file through the `Port 2200` directive.
* Finally, we restart sshd: `sudo sshd restart`.

It is noteworthy that we do not need to remove the rule that allows ssh on port 22 on the Lightsail firewall because this will not be allowed by the Uncomplicated Firewall (UFW), which takes precedence.

#### Step 5 - Configure UFW to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

This is the third and last step to secure our server, and consists of a series of commands on the Linux console:

* Deny all incoming traffic: `sudo ufw default deny incoming`.
* Allow all outgoing traffic: `sudo ufw default allow outgoing`.
* Allow traffic on port 2200, so that we can have SSH on this port: `sudo ufw allow 2200/tcp`.
* Allow HTTP (port 80): `sudo ufw allow www`.
* Allow NTP (port 123): `sudo allow ntp`.
* Finally, enable UFW: `sudo ufw enable`.

### Give grader access

#### Step 6 - Create new user account grader

In this first step, we create a new user account named grader issuing this command: `sudo adduser grader`. We indicate that its password is *_azbycx_*, and its full name *_Udacity Project Reviewer_*.

#### Step 7 - Give grader the permission to sudo

In this second step, we give grader the permission to sudo by creating a new file *_/etc/sudoers.d/grader_*. This file will contain the statement: `grader ALL=(ALL) NOPASSWD:ALL`.

#### Step 8 - Create SSH key pair for grader using ssh-keygen tool

In this third and last step to give grader access, a number of steps are performed:

* Firstly, we generate the public-private key pair by issuing the command `ssh-keygen` on our client machine (i.e, on the machine we want to access the server from). The private key is now located on the file *_/Users/estebanmasobro/.ssh/linuxConfiguration_* on our client machine, with passphrase *_zaybxc_*. In addition, the public key is now located on the file *_/Users/estebanmasobro/.ssh/linuxConfiguration.pub_* on the client machine.
* Secondly, on the *_grader_* home directory *_/home/grader_* on the server machine, we create the directory *_ssh_* by issuing `mkdir .ssh`. This is a special directory where all key related files must be stored.
* Thirdly, we create the *_/home/grader/.ssh/authorized_keys_* file by issuing the command `touch /home/grader/.ssh/authorized_keys` on the server machine.  This is another special file that stores all of the public keys that this account is allowed to use for authentication.
* Then, we copy the contents of the *_/Users/estebanmasobro/.ssh/linuxConfiguration.pub_* file on the client machine into the *_/home/grader/.ssh/authorized_keys_* file on the server machine.
* After that, from the *_grader_* home directory *_/home/grader_* on the server machine, we issue the commands `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`. In this way, we set special permissions so that other users do not have access to the grader's account.
* Finally, we successfully check that grader can ssh into the server machine by issuing the command: `ssh grader@18.185.87.128 -p 2200 -i ~/.ssh/linuxConfiguration`.

### Prepare to deploy one of my web applications on the server

#### Step 9 - Configure the local timezone to UTC

We successfully check that the local timezone on the server machine is already UTC. This can be accomplished by running the command `timedatectl status` which tells us that *_Time zone: Etc/UTC (UTC, +0000)_*.

#### Step 10 - Install and configure Apache to serve a Python mod_wsgi application

In this second step to deploy one of my web applications on the server, the following steps have been performed:

* Firstly, we install Apache by issuing the command `sudo apt-get install apache2`. Now, the web server is up and running, and can be accessed on *_http://18.185.87.128_*.
* Secondly, we install mod_wsgi by issuing the command `sudo apt-get install libapache2-mod-wsgi`.
* Thirdly, we tell Apache to handle requests using the WSGI module. We do this by adding `WSGIScriptAlias / /var/www/html/myapp.wsgi`to the *_/etc/apache2/sites-enabled/000-default.conf_* file.
* Then, we restart Apache by issuing the command `sudo apache2ctl restart`.
* Finally, we add a simple test application on the *_/var/www/html/myapp.wsgi_* file, and successfully check that Apache serves the applicacion from this file. To do that, we access the web server on *_http://18.185.87.128_*.

#### Step 11 - Install and configure PostgreSQL

In this third step to deploy one of my web applications on the server, a number of steps have been performed:

* Firstly, we install PostgreSQL by issuing the command `sudo apt-get install postgresql`.
* Secondly, we successfuly check that remote connections are not allowed by examining the *_/etc/postgresql/9.5/main/pg_hba.conf_* file.
* Thirdly, we change to user postgres by issuing the command `sudo su - postgres`, and enter the psql command line interface issuing `psql`.
* Then, we create our database *_itemcatalog_* by issuing the command `CREATE DATABASE itemcatalog;`.
* After that, we create the user *_catalog_* by issuing the command `CREATE USER catalog WITH PASSWORD 'apbqcr' VALID UNTIL '2019-05-31';`.
* Finally, we grant permissions to our *_itemcatalog_* database to the new database user *_catalog_* by issuing the command: `GRANT ALL PRIVILEGES ON DATABASE itemcatalog to catalog;`.

#### Step 12 - Install git

In this fourth and last step to deploy one of my web applications on the server, git is installed by issuing the command `sudo apt-get install git`.

### Deploy the Item Catalog project

The web application chosen to be deployed on the server is the [*_Item Catalog_*](https://github.com/EsteveM/item-catalog) web application.

#### Step 13 - Clone and setup the Item Catalog web application from its Github repository

The first step to deploy the *_Item Catalog_* application consists of a number of steps, which are listed below:

* Firstly, we activate the mod_wsgi Apache HTTP server module, which will enable Apache to serve Flask applications. To do that, the command `sudo a2enmod wsgi` is issued.
* Secondly, we move to the *_/var/www_* directory, where our Flask application will be located. To this end, the command `cd /var/www` is issued.
* Thirdly, we create the application directory structure within /var/www by issuing the command `sudo mkdir itemcatalog`, and then moving to the newly created directory `cd itemcatalog`.
* Then, we clone the *_Item Catalog_* web application from its GitHub repository by issuing the command `git clone https://github.com/EsteveM/item-catalog.git`.
* After that, we create the *_ __init__.py _* file from *_application.py_*, which contains the application's logic.
* At this point, we change the sentences *_create_engine_* on the files *_ __init__.py _*, *_database_setup.py_*, and *_populatedb.py_*, so as to adapt them to PostgreSQL: `create_engine('postgresql://catalog:apbqcr@localhost/itemcatalog')`.
* Now, we install *_pip_* by issuing the command `sudo apt-get install python-pip`. *_Pip_* will allow us to install *_virtualenv_* and *_Flask_*. We want to create a virtual environment for our flask application.
* Then, we install *_virtualenv_* by issuing the command `sudo pip install virtualenv`.
* After that, we create the temporary environment *_catalog_* by issuing the command `sudo virtualenv catalog`.
* Once this has been done, we activate the virtual environment by issuing the command `source catalog/bin/activate`.
* Now it is time to install *_Flask_* within the virtual environment by issuing the command `sudo pip install Flask`.
* Next, we install SQLAlchemy by issuing the command `sudo pip install sqlalchemy`, and the Flask-SQLAlchemy extension by issuing the command `sudo pip install Flask-SQLAlchemy`.
* The following step is to install *_psycopg2_*, the most popular PostgreSQL database adapter for Python, by issuing the command `sudo pip install psycopg2`.
* At this point, we install the *_Oauth2 client_* for *_Oauth2_* authentication by issuing the command `sudo pip install oauth2client`.
* Now, we install the *_requests_* module, an HTTP library for Python, by issuing the command `sudo pip install requests`.
* It is now time to successfully check that the app runs by issuing the command ` sudo python __init__.py`.
* Finally, we set up our database by running the application's program *_database_setup.py_*, and populate it by running another application's program named *_populatedb.py_*.

#### Step 14 - Set the Item Catalog web application up in the server and ensure the .git directory is not publicly accessible via a browser

In this last step, we want to set up our web application in our server so that it functions correctly when it is accessed from a browser. With this aim in view, the following steps have been performed:

* Firstly, a new virtual host has been configured on the new *_/etc/apache2/sites-available/itemcatalog.conf_* file. These are the contents of the file:
```
<VirtualHost *:80>
                ServerName 18.185.87.128.xip.io
                ServerAdmin esteban.masobro@gmail.com
                WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi
                <Directory /var/www/itemcatalog/itemcatalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/itemcatalog/itemcatalog/static
                <Directory /var/www/itemcatalog/itemcatalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Secondly, the virtual host is enabled by issuing the command `sudo a2ensite itemcatalog`.
* Thirdly, we create the *_/var/www/itemcatalog/itemcatalog.wsgi_* wsgi file so that Apache serves the *_Flask_* application. The contents of this file are:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/itemcatalog/")

from itemcatalog import app as application
application.secret_key = 'super_secret_key'
```
* Then, we restart Apache to apply the changes by issuing the command `sudo service apache2 restart`.

* After that, I have created a file named *_.htaccess_* within the /www/var/ directory, and set `RedirectMatch 404 /\.git` as its contents. In this way, we ensure the .git directory is not publicly accessible via a browser.

* Finally, it is noteworthy that we have adapted the *_login.html_* template of the *_Item Catalog_* web application so that user authentication is provided. The changes made follow the sign-in flow illustrated in the Google Developers guide [Google Sign-In for server-side apps](https://developers.google.com/identity/sign-in/web/server-side-flow).

## Getting Started

There are two possible ways to interact with this project:

* The first one is to access the web server at http://18.185.87.128.xip.io/ from any client machine.
* The second one is to SSH into the server. For instance, as the grader user, you could issue the command `ssh grader@18.185.87.128 -p 2200 -i ~/.ssh/linuxConfiguration`, where *_linuxConfiguration_* could be the name of the file which contains the private SSH key for the grader user. Of course, that assumes you are in possession of the private key. Then, you must also provide the system with the passphrase for the key file. This passphrase is *_zaybxc_*.

## Acknowledgements

I want to acknowledge the resources used during the development of this project:

* The material covered on Udacity's [Configuring Linux Web Servers](https://classroom.udacity.com/courses/ud299) course, has been extremely helpful throughout.
* I checked the Digital Ocean's article ["How To Secure PostgreSQL on an Ubunty VPS"](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps) by Justin Ellingwood to know how to check that remote connections are not allowed on PostgreSQL.
* I consulted the [Creating user, database and adding access on PostgreSQL](https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e) by Arnav Gupta, to identify the sentences to create database and user, and grant priveleges on the database to the user.
* I checked the Digital Ocean's ["How To Deploy a Flask Application on an Ubuntu VPS"](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) by Kundan Singh, to identify how to deploy the *_Item Catalog_* application on the server.
* I consulted the [SQLAlchemy 1.3 Documentation - Engine Configuration](https://docs.sqlalchemy.org/en/latest/core/engines.html) to identify how to adapt the *_create_engine_* call to PostgreSQL.
* I checked the Python Central's article ["How to Install SQLAlchemy"](https://www.pythoncentral.io/how-to-install-sqlalchemy/) to know how to install SQLAlchemy.
* I reviewed the Pypi's documentation ["psycopg2 2.7.7"](https://pypi.org/project/psycopg2/) to know how to install *_psycopg2_*.
* I have followed the Google Developers guide [Google Sign-In for server-side apps](https://developers.google.com/identity/sign-in/web/server-side-flow) to adapt the sign-in process of the *_Item Catalog_* web application so that this feature is provided.
* I have read the StackOverflow's article [Make .git directory web inaccessible](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible) to identify how to prevent the .git directory from being publicly accessible via a browser.

## Contributing

This repository contains all the work that makes up the project. Individuals and I myself are encouraged to further improve this project. As a result, I will be more than happy to consider any pull requests.