# Udacity project: Linux Server Configuration

## Project Overview
Installation of an Ubuntu server instance to host a Flask application.
The application hosted on this intance is from the Udacity project: [Item Catalog](https://github.com/j-sho/fullstack-nanodegree-vm/tree/master/vagrant/catalog).

* Public IP: [http://35.158.50.123/](http://35.158.50.123/)
* DNS: [http://ec2-35-158-50-123.eu-central-1.compute.amazonaws.com](http://ec2-35-158-50-123.eu-central-1.compute.amazonaws.com)
* SSH Port: 2200

## Configuration summary
## Get your server
- Start a new Ubuntu Linux server instance on Amazon Lightsail:
    * Log in [Amazon Lightsail](https://www.amazon.com/ap/signin?openid.assoc_handle=aws&openid.return_to=https%3A%2F%2Fsignin.aws.amazon.com%2Foauth%3Fresponse_type%3Dcode%26client_id%3Darn%253Aaws%253Aiam%253A%253A015428540659%253Auser%252Fparksidewebapp%26redirect_uri%3Dhttps%253A%252F%252Flightsail.aws.amazon.com%252Fls%252Fwebapp%253Fstate%253DhashArgs%252523%2526isauthcode%253Dtrue%26noAuthCookie%3Dtrue&openid.mode=checkid_setup&openid.ns=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0&openid.identity=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0%2Fidentifier_select&openid.claimed_id=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0%2Fidentifier_select&action=&disableCorpSignUp=&clientContext=&marketPlaceId=&poolName=&authCookies=&pageId=aws.ssop&siteState=registered%2Cen_US&accountStatusPolicy=P1&sso=&openid.pape.preferred_auth_policies=MultifactorPhysical&openid.pape.max_auth_age=120&openid.ns.pape=http%3A%2F%2Fspecs.openid.net%2Fextensions%2Fpape%2F1.0&server=%2Fap%2Fsignin%3Fie%3DUTF8&accountPoolAlias=&forceMobileApp=0&language=en_US&forceMobileLayout=0)
    * Create an instance. Choose an instance image: Ubuntu (choose "OS Only" and Ubuntu as the operating system.)
    * Choose your instance plan.
    * Give your instance a hostname.
    * Wait for it to start up.
## Give grader access
- Create a new user account named grader
```
$ sudo adduser grader
```
- Give grader the permission to sudo
```
$ sudo visudo
```
Add the following line below 'root ALL=(ALL:ALL) ALL':
```
grader ALL=(ALL:ALL) ALL
```
## Secure your server
- Update all currently installed packages:
```
$ sudo apt-get update
$ sudo apt-get upgrade
```
- Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it
```
$ sudo nano /etc/ssh/sshd_config
```
Made following changes:
- Port from '22' to '2200'
```
# What ports, IPs and protocols we listen for
Port 2200
```
- Permit Root Login from 'without-password' to 'no'
```
# Authentication:
PermitRootLogin no
```
- Temporary Password Authentication from 'no' to 'yes'
```
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication yes
```
- Add the following to the end of the file
```
UsePAM yes
UseDNS no
AllowUsers grader
```
## Configure Firewall in AWS Lightsail
-In your AWS Lightsail instance, click on the Netwroking tab
- Change Port configurations:
```
Application	 Protocol	 Port range	
HTTP	       TCP	     80	
Custom	     UDP	     123	
Custom	     TCP	     2200
```
## Create an SSH key pair for grader using the ssh-keygen tool
- On Lightsail console enter:
```
$ sudo su - grader
grader@PUBLIC_IP:~$ sudo mkdir /home/grader/.ssh
grader@PUBLIC_IP:~$ sudo chown grader:grader /home/grader/.ssh
grader@PUBLIC_IP:~$ sudo chmod 700 /home/grader/.ssh
grader@PUBLIC_IP:~$ sudo cp /root/.ssh/authorized_keys /home/grader/.ssh/
grader@PUBLIC_IP:~$ sudo chown grader:grader /home/grader/.ssh/authorized_keys
grader@PUBLIC_IP:~$ sudo chmod 644 /home/grader/.ssh/authorized_keys
```
- On your local machine enter:
```
vagrant@vagrant-ubuntu-trusty-64:~$ ssh-keygen
vagrant@vagrant-ubuntu-trusty-64:~$ ssh-copy-id grader@PUBLIC_IP
```
- Connect from your local computer:
```
vagrant@vagrant-ubuntu-trusty-64:~$ ssh -i .ssh/id_rsa grader@PUBLIC_IP -p 2200
```
- Reconfigure settings in SSHD config file:
```
$ sudo nano /etc/ssh/sshd_config

# Change from 'yes' to 'no'
PasswordAuthentication no
```

## Configure UFW, Uncomplicated Firewall
```
 grader@PUBLIC_IP:~$ sudo ufw default deny incoming
 grader@PUBLIC_IP:~$ sudo ufw default allow outgoing
 grader@PUBLIC_IP:~$ sudo ufw allow 80/tcp
 grader@PUBLIC_IP:~$ sudo ufw allow 2200/tcp
 grader@PUBLIC_IP:~$ sudo ufw allow 123/udp
 grader@PUBLIC_IP:~$ sudo ufw enable
 sudo ufw status
```
Status should be:
```
Status: active
To                         Action      From
--                         ------      ----
80/tcp                     ALLOW       Anywhere                  
2200/tcp                   ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
80/tcp (v6)                ALLOW       Anywhere (v6)             
2200/tcp (v6)              ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)    
```
## Configure the local timezone to UTC
```
grader@PUBLIC_IP:~$ sudo timedatectl set-timezone UTC
grader@PUBLIC_IP:~$ sudo apt-get install ntp
```
## Install and configure Apache to serve a Python mod_wsgi application
```
grader@PUBLIC_IP:~$ sudo apt-get install apache2
grader@PUBLIC_IP:~$ sudo apt-get install python-setuptools libapache2-mod-wsgi
grader@PUBLIC_IP:~$ sudo service apache2 restart
```
## Install git
```
grader@PUBLIC_IP:~$ sudo apt-get install git
grader@PUBLIC_IP:~$ sudo apt-get install libapache2-mod-wsgi python-dev
grader@PUBLIC_IP:~$ cd /var/www
grader@PUBLIC_IP:/var/www$ sudo mkdir catalog  
grader@PUBLIC_IP:/var/www$ cd catalog
grader@PUBLIC_IP:/var/catalog$ sudo mkdir catalog
grader@PUBLIC_IP:/var/catalog$ cd catalog
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo mkdir static templates models db
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo mv application.py __init__.py
```
## Install Pip, Flask, and virtualenv
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo apt-get install python-pip
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install virtualenv
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo virtualenv venv
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo chmod -R 777 venv
grader@PUBLIC_IP:/var/www/catalog/catalog$ source venv/bin/activate
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ pip install Flask
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ deactivate
```
## Configure Apache mod-wsgi module:
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo nano /etc/apache2/sites-available/catalog.conf

<VirtualHost *:80>

    ServerName PUBLIC_IP
    ServerAdmin admin@mywebsite.com
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
```
## Create the mod-wsgi file:
```
grader@PUBLIC_IP:/var/www/catalog/$ sudo nano /var/www/catalog/catalog.wsgi

#/user/bin/python
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/catalog/")

from __init__ import app as application
application.secret_key == 'Your application secret key'
```
## Cloning the Item Catalog project
```
sudo git clone https://github.com/j-sho/fullstack-nanodegree-vm.git
```
## Create a .htaccess file from the catalog/ directory
```
grader@PUBLIC_IP:/var/www/catalog$ sudo nano .htaccess

#Add following
RedirectMatch 404 /\.git
```
## Install necessary dependencies in the virtual environment
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ source venv/bin/activate
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo apt-get install python-setuptools
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install Flask
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install requests
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install Flask-SQLAlchemy
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install httplib2
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install oauth2client
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo service apache2 restart
```
## Install Postrgresql and set up database
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo apt-get install postrgresql postgresql-contrib
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo su - postgres
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ psql
postgres=# CREATE USER catalog WITH PASSWORD 'c@t@l0g';
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
postgres@PUBLIC_IP~$ exit
```
- Change in application files from  ```engine = create_engine('sqlite:///recipesbycategory.db')``` to ```engine = create_engine('postgresql+psycopg2://catalog:c@t@l0g@localhost/catalog/')```
- Run database_setup file and restart Apache
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ python db/database_setup.py
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo service apache2 restart
```
## Update FB and Google Plus oauth credentials
- Go to [Facebooc developers](https://developers.facebook.com) and change the site url for your app and Valid OAuth redirect URIs
- Go [Google Plus](https://console.developers.google.com) and change the oauth credentials in you app console
- After changes download new JSON files and update your fb_client secrets.json and client_secrets.json 
- Change in application files from ```oauth_flow = flow_from_clientsecrets('/client_secrets.json', scope='')``` to ```oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')```
- Restart Apache server
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo service apache2 restart
```
## the not JSON serializable
- If you will face the same error as I, please type the following three commands into your vagrant terminal:
```
sudo pip install werkzeug==0.8.3
sudo pip install flask==0.9
sudo pip install Flask-Login==0.1.3
```
## Resources
- [DigitalOcean](https://www.digitalocean.com/community/tutorials/)
- [StackOverflow](https://stackoverflow.com)
- [Udacity Forum](https://discussions.udacity.com)
- [Reverse DNS Lookup](https://remote.12dt.com/lookup.php)
