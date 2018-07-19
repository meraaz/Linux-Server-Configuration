## Linux Server Configuration

> This project is about how to configure and secure a Linux Server to host a Python Flask Web Application using WSGI

## Technology Stack

Server : VPS

OS : Ubuntu

Web Server : Apache2 using WSGI specification

Programming Language : Python

**The Deployed project is Catalog Items Project**

### Live Demo

Visit Udacity - Category Items Project :  http://136.243.232.209.xip.io/loging

- Connect to the vps with root ssh credentials provided from your server provider

`ssh root@136.243.232.209`

> Enter password when asked to

- Update the packages installed

`apt-get update`

`apt-get upgrade`

- Add new user named `grader`

`adduser grader`

> Enter the Passwords when required and user details

- add user to sudo group

`usermod -aG sudo grader`

> Now grader user by default will have sudo permissions

**Test the sudo access to grader**

`su - grader`

**Try to run command with sudo**

`sudo ls /root`

**will ask you to enter password that created before**

- Generate public key on /home/grader/.ssh

`sudo ssh-keygen`

```
  when ask to file path , Enter : /home/grader/.ssh/id_rsa
  Enter passphrase password to secure your key
  PS : _Will Send it to the reviewer with the file
```

- check file created successfully

`ls -la ~/.ssh/`

```
you will find 2 new files id_rsa ( The private key ), id_rsa.pub (The public key)
```

- to change ssh port open /etc/ssh/sshd_config

`nano /etc/ssh/sshd_config`

```
  - change port to ` Port 2200`
  - Now disable root user from loging remotly , change `PermitRootLogin no`
  - change RSAAuthentication yes  , PubkeyAuthentication yes
  - disable loging with password  PasswordAuthentication no
```

- Restart SSH Service

`sudo service ssh restart`

##### Allow only connections from port 2200 , 80 and 123

- check the allowed rules

`sudo ufw status numbered`

- add new rule to enable ntp

`sudo ufw allow ntp`

- to enable http

`sudo ufw allow http`

- to enable port 2200

`sudo ufw allow 2200`

**update packages**

`sudo apt-get update`

`sudo apt-get upgrade`

### Web Server Installation

- install Apache2 to be able to serve a Python web application

`sudo apt-get install apache2`

- open /etc/apache2/apache2.conf

`sudo nano /etc/apache2/apache2.conf`

- add ServerName 136.243.232.209 to set globally

- check no errors with the file

`sudo apache2ctl configtest`

**you should see syntax Ok message**

- Restart Apache2

`sudo systemctl restart apache2`


`sudo ufw app list`


`sudo ufw app info "Apache Full"`


`sudo ufw allow in "Apache Full"`

`sudo apt-get update`
`sudo apt-get install postgresql postgresql-contrib`

- Do Not Allow Remote Connections to postgresql
sudo nano /etc/postgresql/10/main/pg_hba.conf

- make sure it only accept connections from local and Linux domain sockets

- connect to postgres user

`sudo su - postgres`

`psql`

```
CREATE USER grader WITH PASSWORD 'catalog_password';
CREATE DATABASE catalog OWNER grader;
```
- to get out of postgresql our
`\q`

- then to get back to grader user

`exit`

- update server timezone to UTC

`sudo dpkg-reconfigure tzdata`

**from first list choose non of the above , second list choose UTC**

### Running the Web Appliction

- install WSGI

`sudo apt-get install libapache2-mod-wsgi-py3`

- create a new virtual host config file

`sudo nano /etc/apache2/sites-available/catalogItems.conf`

- write these Configuration

```
<VirtualHost *:80>
        ServerName 136.243.232.209
        ServerAdmin mervat.karam.m@gmail.com
        WSGIScriptAlias / /var/www/catalogItems/catalog_items.wsgi
        <Directory /var/www/catalogItems/catalogItems/>
                AllowOverride None
                Require all granted
        </Directory>
        Alias /static /var/www/catalogItems/catalogItems/static
        <Directory /var/www/catalogItems/catalogItems/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

- Create the `catalog_items.wsgi` file into `/var/www/catalogItems` directory

`sudo nano /var/www/catalogItems/catalog_items.wsgi`

```
#!/usr/bin/python3
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalogItems/")

from catalogItems import app as application
application.secret_key = 'your_secret_key'

```

- change main file from `appliction.py` to `__init__.py`

- reconfigure the DB connection in both file `__init__.py` , `config/db/createDB.py`

from :
**engine = create_engine('sqlite:///item_catalog.db')**

to :
**engine = create_engine('postgresql://grader:catalog_password@localhost/catalog')**

- install pip3

`sudo apt install python3-pip`

`sudo pip3 install -r requirements.txt`

- create DB

`python3 config/db/createDB.py`

- Restart Apache2 Service

`sudo service apache2 restart`

- use `xip.io` to update your Google API Console and Facebook Developer APP

> PS : Facebook disallow login on websites that not run with HTTPS

- update your file `config/googleCredentials.json` with the new one exported from you Google API Console

#### Third Parties we yous in the project to authenticate user

- Google API Console

> PS : find the credentials located undet the project root into config/googleCredentials

- Facebook Developer App

> PS : find the credentials located undet the project root into config/facebookCredentials


Now Go to http://136.243.232..209 and enjoy your Web Application
