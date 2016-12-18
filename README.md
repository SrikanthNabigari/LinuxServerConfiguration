## Linux Server Configuration Udacity FSND Project 7

## AWS Server Deployed on Below Addresses

- [35.160.72.118](http://35.160.72.118/)
- [ec2-35-160-72-118.us-west-2.compute.amazonaws.com](http://ec2-35-160-72-118.us-west-2.compute.amazonaws.com/)

## Configuration

### SSH access to the instance

1. Download Private Key
2. Move the private key file into the folder ~/.ssh

    ```
    mv ~/Downloads/udacity_key.rsa ~/.ssh/
    ```
3. Secure the key with

    ```
    chmod 600 ~/.ssh/udacity_key.rsa
    ```

4. Log into root

    ```
    ssh -i ~/.ssh/udacity_key.rsa root@35.160.72.118
    ```

### Add User Grader and Provide Permissions and SSH login

1. Create User Grader

    ```
    sudo adduser Grader
    ```

    provide the neccessities of the new User

2. Copy the root authorized key into grader

    ```
    sudo cp ~/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
    ```
3. Change the Permissions to the .ssh folder and authorized_keys

    As the .ssh folder and key are added by the root we need to change the owner and group
    of the them to user so that user can access the key when he log's into the server 

    ```
    sudo chown grader /home/grader/.ssh
    ```

    ```
    sudo chgrp grader /home/grader/.ssh
    ```

    ```
    sudo chown grader /home/grader/.ssh/authorized_keys
    ```

    ```
    sudo chgrp grader /home/grader/.ssh/authorized_keys
    ```

    ```

    File Permissions adds more secure to files

    sudo chmod 700 /home/grader/.ssh
    ```

    ```
    sudo chmod 644 /home/grader/.ssh/authorized_keys
    ```

### Change the SSH port from 22 to 2200 and AllowsUsers to login
    
```
sudo nano /etc/ssh/sshd_config
```

Change the port `22 to 2200` and add `AllowsUsers grader` in the sshd config file to allow only grader user to log in to the server.

Now restart the ssh

```
sudo service ssh restart
```

Reference: [Knownhost](https://www.knownhost.com/wiki/security/misc/how-can-i-change-my-ssh-port)


### Configure Uncomplicated Firewall to only allow incoming  connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

TO add more security to the server allow only the ports the server needed and deny all.

Deny's all incoming port connections

```
sudo ufw default deny incoming
```

Allows All the outgoing ports

```
sudo ufw default allow outgoing
```

Allows the reconfigured SSH port 2200

```
sudo ufw allow 2200
```

Allows HTTP port 80 for web communications from site

```
sudo ufw allow 80
```

Allows NTP 123 port

```
sudo ufw allow 123
```

```
sudo ufw enable
```

### Configure the local timezone to UTC

```
dpkg-reconfigure tzdata
```

In the options after the command entered click on 'none of the above' option and 'UTC' to set the Universal timezone.

Reference: [AskUbuntu](http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt).


### Sudo access to User grader

Type the following commands to give access sudo Permissions to user:

```
sudo nano /etc/sudoers.d/grader
```

Add the following line to that file

```
grader ALL=(ALL:ALL) ALL
```

save and exit.

### Update all currently installed packages

Enter the following command to install updates

```
sudo apt-get update 
```

```
sudo apt-get upgrade
```

### Login as Grader User

exit from root login and

```
sudo ssh -i ~/.ssh/udacity_key.rsa grader@35.160.72.118
```

### Configure Apache Flask server application

1. Install apache2

    ```
    sudo apt-get install apache2
    ```
2. Install mod-wsgi

    ```
    sudo apt-get install libapache2-mod-wsgi
    ```
3. Install git for cloning repository 

    ```
    sudo apt-get install git
    ```

4. Create a directory catalog in /var/www/

    ```
    sudo mkdir /var/www/catalog
    ```
5. Clone the repository in the catalog folder

    ```
    sudo git clone https://github.com/SrikanthNabigari/restaurant_menu.git catalog
    ```
6. Install pip.

    ```
    sudo apt-get install python-pip 
    ```
7. Set up virtual environment

    ```
    sudo pip install virtualenv 
    ```

    ```
    sudo virtualenv venv
    ```
8. Acitivate environment

    ```
    source venv/bin/activate 
    ```
9. Install Flask and oauth in this environment

    ```
    sudo pip install Flask
    ```

    ```
    sudo pip install oauth2client
    ```
10. To deactivate from environment enter

    ```
    deactivate
    ```

### Configure and Enable a New Virtual Host

Issue the following command in your terminal:

```
sudo nano /etc/apache2/sites-available/catalog.conf
```

Add the following lines of code to the file to configure the virtual host.
Be sure to change the ServerName to your domain or cloud server's IP address:

```
<VirtualHost *:80>
                ServerName 35.160.72.118
                ServerAlias ec2-35-160-72-118.us-west-2.compute.amazonaws.com
                ServerAdmin grader@35.160.72.118
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

Save and close the file.

Enable the virtual host with the following command:

### Create the .wsgi File

Apache uses the .wsgi file to serve the Flask app. 
Move to the /var/www/FlaskApp directory and create a file 
named flaskapp.wsgi with following commands:

```
cd /var/www/catalog
sudo nano catalog.wsgi
```

Add the following lines of code to the catalog.wsgi file:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 's!e@c#r$e%t'

```

### Install and configure Postgresql Database and python ORM sqlalchemy

1. Install Posgresql and python-psycopg2

    ```
    sudo apt-get install postgresql python-psycopg2
    ```
2. Install python ORM sqlalchemy.

    ```
    sudo apt-get install python-sqlalchemy
    ```
3. Create user catalog for Posgresql db

    ```
    sudo su - postgresql
    ```

    ```
    psql
    CREATE USER catalog WITH PASSWORD 'xxx';
    CREATE DATABASE restaurantmenuwithusers;
    \q 
    ```

    exit from postgres 

4. Run the following python files to enter some application data into psql db
   to run the cloned application.

   ```
   python /var/www/catalog/catalog/database_setup.py
   ```

   ```
   python /var/www/catalog/catalog/lotsofmenus.py
   ```

Note: While running you application you may encounter some errors you can
      look into the errors in `/var/log/apache2/error.log`.
      In the `__init__.py` file change clients_secret.json and fb_secret_clients.json file locations to 
      actual path of that files like /var/www/catalog/catalog/clients_secret.json to error free.


### Restart Apache

Restart Apache with the following command to apply the changes:

```
sudo service apache2 restart 
```

You may see a message similar to the following:

```
Could not reliably determine the VPS's fully qualified domain name, using 127.0.0.1 for ServerName 
```

Ignore that message its just a warning:



Reference:

[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps).

[Udacity Discussions](https://discussions.udacity.com/c/nd004-p7-linux-based-server-configuration).

