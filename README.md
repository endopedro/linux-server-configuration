# Linux Server Configuration
This project is a Linux server to host a bookstore python application (Udacity's item catalog project).
##### About the server
- Amazon Lightsail Server
- OS Ubuntu 18.04
- Public IP Address: __34.200.250.175__
- SSH port: __2200__

## Tasks
##### Create grader user and give it sudo permition
- Run `$ sudo adduser grader`
- Create sudo file at `$ sudo nano /etc/sudoers.d/grader`
- Type `grader ALL=(ALL) NOPASSWD: ALL` on the file and save

##### Configure RSA keys to grader user
- Copy the public key and paste in the file `/home/grader/.ssh/authorized_keys`
- Run `$ chmod 700 /home/grader/.ssh` and `$ chmod 644 /home/grader/.ssh/authorized_keys`
- Restart ssh service
- Then you're able to login using `ssh grader@34.200.250.175 -p 2200 -i ~/.ssh/id_rsa`

##### Update packages
- `$ sudo apt-get update`
- `$ sudo apt-get upgrade`

##### Change SSH port and disable root login
- Open file `$ sudo nano /etc/ssh/sshd_config`
- Change the ssh port 22 to 2200
- Save the file
- Change `PermitRootLogin without-password` line to `PermitRootLogin no`
- Restart ssh `$ sudo service ssh restart`

##### Use Uncomplicated Firewall to allow only SSH, HTTP, and NTP connections
- `$ sudo ufw allow 2200/tcp`
- `$ sudo ufw allow 80/tcp`
- `$ sudo ufw allow 123/udp`
- `$ sudo ufw enable`

##### Configure the local timezone to UTC
- Run `$ sudo dpkg-reconfigure tzdata`
- Choose UTC

##### Install Apache to serve a Python WSGI application
- Install Apache `$ sudo sudo apt-get install apache2`
- Install wsgi `$ sudo apt-get install python-setuptools libapache2-mod-wsgi`
- Restart Apache `$ sudo service apache2 restart`

##### Install PostreSQL
- `$ sudo apt-get install libpq-dev python-dev`
- `$ sudo apt-get install postgresql postgresql-contrib`
- `$ sudo su - postgres`

##### Create the catalog database
- `$ psql`
- `$ CREATE USER catalog WITH PASSWORD 'password';`
- `$ ALTER USER catalog CREATEDB;`
- `$ CREATE DATABASE catalog WITH OWNER catalog;`

##### Clone the app from Github
- Go to directory ` /var/www`
- Create catalog dir `$ sudo mkdir catalog`
- Give owner to grader`$ sudo chown -R grader:grader catalog`
- Go to ` /var/www/catalog`
- Clone the project `$ git clone https://github.com/endopedro/book-store catalog`

##### Install application dependencies
- Install pip `$ sudo apt-get install python-pip`
- Then install dependencies:
-- `$ sudo pip install flask`
-- `$ sudo pip install requests`
-- `$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

##### Configure application
- Create a catalog.wsgi file at ` /var/www/catalog`
- Type this inside :
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecret'
```
- Rename `book_store.py` to `__init__.py`
- In `__init__.py` file, change __CLIENT_ID__ variable content to `json.loads(open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']`
- Change the __engine__ content in files `database_setup.py`, `crud.py` and `stock.py` to `create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`
- Create tables `$ python database_setup.py`
- Pupulate tables `$ python stock.py`

##### Configure Apache
- Create file `/etc/apache2/sites-available/catalog.conf` and add the content below:
```
<VirtualHost *:80>
  ServerName 34.200.250.175
  ServerAdmin admin@34.200.250.175
  WSGIScriptAlias / /var/www/catalog/catalog.wsgi
  <Directory /var/www/catalog/catalog>
      Order allow,deny
      Allow from all
  </Directory>
  Alias /static /var/www/catalog/catalog/static
  <Directory /var/www/catalog/catalog/static/>
      Order allow,deny
      Allow from all
  </Directory>
</VirtualHost>
```
- Restart Apache `$ sudo service apache2 restart`

## Application
Visible at http://34.200.250.175

## References
- [Digital Ocean: How To Set Up Apache Virtual Hosts on Ubuntu 14.04 LTS](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
- [Udacity: Configuring Linux Web Servers](https://br.udacity.com/course/configuring-linux-web-servers--ud299)
- [Ubuntu: HTTPD - Servidor Web Apache2](https://help.ubuntu.com/lts/serverguide/httpd.html.pt-BR)
