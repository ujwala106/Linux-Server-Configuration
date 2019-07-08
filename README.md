# Linux-Server-Configuration
This is the third project in Udacity's full stack developer course.This project deals with setup of secure linux server. By following the steps mentioned below, one can set up an web applications running on a secure web server.

# Server Configuration

- **Public IP:** 13.235.0.153
- **Port:** 2200

### The application can accesible in the following links
- http://13.235.0.153/
- http://13.235.0.153.xip.io (This DNS name is required to add oauth to our application since google cannot accept IP address only for authentication. logging in to the application requires googles authntication of client secrets. Visit [this link](http://xip.io) for more info about xip.io)

## Get Started
To complete this project, you'll need a Linux server instance. We recommend using Amazon Lightsail for this. If you don't already have an Amazon Web Services account, you'll need to set one up and get the account verified only after which the following steps can be done. Once you've done that, here are the steps to complete this project.

## Step 1: Setup Amazon LightSail instance.
- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources)
- Once you are login into the site, click `Create instance`. 
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Click the `Create` button to create the instance.
- Wait for the instance to start up.

## Step 2: SSH into your Server
- Download private key from the `SSH keys` section in the `Account` section on Amazon Lightsail.
- Create a new file named **lightsail_key.rsa** under ~/.ssh folder on your local machine.
- Copy and paste content from downloaded private key file to **lightsail_key.rsa**
- Set file permission as owner only : `$ chmod 600 ~/.ssh/lightsail_key.rsa`
- SSH into the instance: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.235.0.153`. In this case, default port for ssh is `22`.

## Step 3: Secure your server
### Update all currently installed packages
- Run `sudo apt-get update` to update packages.
- Run `sudo apt-get upgrade` to install new versions of packages, If available.
- check for future updates: `sudo apt-get dist-upgrade`.

### Change the SSH port from 22 to 2200
- In project requirements, it is metioned to change the ssh port from `22` to `2200`.
- Run `sudo nano /etc/ssh/sshd_config` to edit the mentioned file
- Change the port number from `22` to `2200`.
- Restart SSH: `sudo service ssh restart`.
  
>NOTE: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used.

### Configure the Uncomplicated Firewall
```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny tcp and udp packets on port 22.
```
- Turn UFW on: `sudo ufw enable`. The output should be like this:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Check the status of UFW to list current roles: `sudo ufw status`. The output should be like this:

  ```
  Status: active
  
  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```

- Exit the SSH connection: `exit`.
- Go to the console of amazon lightSail, click on the `Manage` option.Navigate to `Networking` Tab and make the following changes.
    1. remove the default SSH rule: port 22.(Since it is changed to 2200).
    2. allow Application `HTTP`, protocol `TCP`, allow 80.
    3. Under application `Custom`, protocol `TCP`, allow 2200.
    4. Under Application `Custom`, protocol `UDP`, allow 123.
  - Open a new terminal and you can now ssh in via the new port 2200: <br>
`$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.235.0.153 -p 2200`

## Step 4: Give grader access

### Create a new user account named
- login as `ubuntu`, add user: `sudo adduser grader`.
 
### Give grader the permission to sudo
- Edit the sudoers file: `sudo visudo`.
- add below line after 'root    ALL=(ALL:ALL) ALL'
  ```bash
  grader  ALL=(ALL:ALL) ALL
  ```
- save the file and exit: ctrl+X and press `y`.

### Create an SSH key pair for grader using the ssh-keygen tool on local Machine:
- Run `ssh-keygen` on the local machine:
- Enter filename as `garder_key` in which to save the key in the local directory `~/.ssh`.Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
- Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file
- Log in to the grader's virtual machine.
```bash
 ssh -i ~/.ssh/grader_key grader@13.235.0.153 -p 2200
```
- Create a new directory called `~/.ssh` (`mkdir .ssh`) on the grader's virtual machine
- Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
- Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
- Check in `/etc/ssh/sshd_config` file if `PasswordAuthentication` is set to `no`
- Restart SSH: `sudo service ssh restart`
- On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@13.235.0.153`.

## Step 5: Prepare to deploy your project

### Configure the local timezone to UTC

- Run `$ sudo dpkg-reconfigure tzdata` select the continent and local time zone according to your region.

### Install and configure Apache to serve a Python mod_wsgi application
- Install **Apache**: `$ sudo apt-get install apache2`
- Go to http://13.235.0.153/, if Apache is working correctly, a **Apache2 Ubuntu Default Page** will show up
- Install the **mod_wsgi** package: `$ sudo apt-get install libapache2-mod-wsgi python-dev`
- Enable **mod_wsgi**: `$ sudo a2enmod wsgi`
- Restart **Apache**: `$ sudo service apache2 restart`

###  Install and configure PostgreSQL
- login as `grader`, Run `sudo apt-get install postgresql` to install postgresql
- PostgreSQL should not allow remote connections. In the  `/etc/postgresql/9.5/main/pg_hba.conf` file, you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```
- run `sudo su - postgres` to login as postgres user.
- Open PostgreSQL interactive terminal with `psql`
- Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```
- Exit psql using `\q`.
- Switch back to the `grader` user: `exit`.
- login as grader and create a new Linux user called `catalog`: `sudo adduser catalog`
- Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
- add below line under `root    ALL=(ALL:ALL) ALL grader  ALL=(ALL:ALL) ALL` to give sudo previliges to catalog user
  ```
  catalog  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.
- While logged in as `catalog`, create a database: `createdb catalog`.
- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.

### Install git
- Run `$ sudo apt-get install git`
- Create dictionary: `$ mkdir /var/www/catalog`
- CD to this directory: `$ cd /var/www/catalog`
- Clone the catalog app: `$ sudo git clone 'URL OF YOUR REPO' catalog`
- Change the ownership: `$ sudo chown -R ubuntu:ubuntu catalog/`
- CD to `/var/www/catalog/catalog`
- Change file **project.py** to **__init__.py**: `$ mv project.py __init__.py`
- Change line `app.run(host='0.0.0.0', port=8000)` to `app.run()` in **__init__.py** file.Since, the final deployed web application will not be serving on localhost, This change has to be made.
- Create a new project on Google API Console and download `client_scretes.json` file.
- Add authorized uris, and javascript _origins depending on where the application is expected to be running.
>Note : http://<ip_address>.xip.io can be to javascript_origins and http://<ip_address>.xip.io/login can be added to redirect_uri.
- Copy and paste contents of downloaded `client_scretes.json` to the file with same name under directory `/var/www/catalog/catalog/client_secrets.json`

## Step 6: Final setup

### Installing the required packages
- Install pip: `$ sudo apt-get install python-pip`
- Install the following packages:
```
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
   ```
### Setup and enble a virtual host

- Create file: `$ sudo touch /etc/apache2/sites-available/catalog.conf`
- Add the following to the file:
```
   <VirtualHost *:80>
		ServerName 13.235.0.153
		ServerAdmin admin@13.235.0.153
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
 ```
- Run `$ sudo a2ensite catalog` to enable the virtual host
- Restart **Apache**: `$ sudo service apache2 reload`

### Configure .wsgi file
- Create file: `$ sudo touch /var/www/catalog/catalog.wsgi`
- Add content below to this file and save:

```python
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/catalog/")
   sys.path.insert(1,"/var/www/catalog/catalog")

   from catalog import app as application
   application.secret_key = 'super_secret_key'

```
- Restart **Apache**: `$ sudo service apache2 reload`

### Edit the database path

- Replace lines in `__init__.py`, `database_setup.py`, and `data.py` with `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

### Disable defualt Apache page

- `$ sudo a2dissite 000-default.conf`
- Restart **Apache**: `$ sudo service apache2 reload`

### Set up database schema

- Run `$ sudo python database_setup.py` to set up database.
- Run `$ sudo python data.py` to add data to the database
- Restart **Apache**: `$ sudo service apache2 reload`
- Now follow the link to http://13.235.0.153.xip.io/  the application should be runing online

#### Use xip.io for setting DNS name to your IP Address in order to run your application smoothly. 

## Resources used

- [Udacity](https://www.udacity.com)
- [Amazon Lightsail](https://lightsail.aws.amazon.com) for creating ubuntu instance
- [Google API Console](https://console.developers.google.com/)
- [Apache](https://httpd.apache.org/docs/2.2/configuring.html)
- [Github](https://github.com/)
- [Postgresql](https://tecadmin.net/install-postgresql-server-on-ubuntu/) can be used for reference.
- [xip.io for DNS](http://xip.io)