                      ******** Hossam sabry ********
# Udacity - Linux Server Configuration

 

Project Overview
A baseline installation of a Linux server and preparing it to host our web applications,
securing it from a number of attack vectors, installing and configuring a database server, and deploying our one of your existing web applications onto it.

Why this Project?
This Project provides deep understanding of exactly what our web applications are doing, how they are hosted, 
and interats with multiple systems. In this Project, we will turn a brand-new, bare bones, Linux server into the secure and efficient web application host needed by Our Applications.


## IP & Hostname
 & port
- Host Name: ec2-52-47-99-96.eu-west-3.compute.amazonaws.com
- IP Address: 52.47.99.96

- Accessible SSH port: 2200

## Software to install during the configuration
Apache2
mod_wsgi
PostgreSQL
git
pip
virtualenv
httplib2
Python Requests
oauth2client
SQLAlchemy
Flask
libpq-dev
Psycopg2


## Amazon Lightsail Set Up

 
1. Go to the [Amazon Lightsail](https://aws.amazon.com/lightsail/) website
 
2. Click get started for free
 
3. Create your first instance


4. Select OS Only and Ubuntu

5. Scroll down and name your instance whatever you'd like and click create.

6. It might take a minute or two for the instance to set up. Once running click on it and click "Account Page" at the bottom so you can download your private SSH key.
7. Now click download to get your private key. The file type is .pem and will be used to SSH into the server.

8. The last thing we will need to do is configure the ports Amazon Lightsail will allow. By default the firewall is set to only allow connects from port 22 and port 80. We need to set up port 2200 and 123. 

9. Click the networking tab


10. From this tab click add another under "Firewall" and choose **Custom** for application, **TCP** for protocol, and the port number under **Port Range**. Then click save.

11. That should be all that needs to be done with the Lightsail website.



# Linux Configuration
 
1. To make our key secure type `$ chmod 600 ~/.ssh/udacity.pem` into the terminal.
 
2. From here we will log into the server as the user `ubuntu` with our key. From the terminal type `$ ssh -i ~/.ssh/udacity.pem ubuntu@52.47.99.96`
 
3. Once logged in you will see the command line change to `root@[ip-your-private-ip]:$ `
 
4. Lets switch to the root user by typing `sudo su -`
 
5. As Udacity requires we need to create a user called `grader`. From the command line type `$ sudo adduser grader`. It will ask for 2 passwords and then a few other fields which you can leave blank.
 
6. We must create a file to give the user grader superuser privileges. To do this type `$ sudo nano /etc/sudoers.d/grader`. This will create a new file that will be the superuser configuration for grader. When nano opens type `grader ALL=(ALL:ALL) ALL`, to save the file hit Ctrl-X on your keyboard, type 'Y' to save, and return to save the filename.
 
7. One of the first things you should always do when configuring a Linux server is updating it's package list, upgrading the current packages, and install new updates with these three commands:
 ```
 $ sudo apt-get update
 $ sudo apt-get upgrade
 $ sudo apt-get dist-upgrade
 
 ```
 
8. We will also install a useful tool called **Finger** with the command  `$ sudo apt-get install finger`. This tool will allow us to see the users on this server.
 
9. Now we must create an SSH Key for our new user **grader**. From a new terminal run the command: `$ ssh-keygen -f ~/.ssh/udacity.rsa`
 
10. In the same terminal we need to read and copy the public key using the command: `$ cat ~/.ssh/udacity.rsa.pub`. Copy the key from the terminal.
 
11. Back in the server terminal locate the folder for the user **grader**, it should be `/home/grader`. Run the command `$ cd /home/grader` to move to the folder.
 
12. Create a directory called .ssh with the command `$ mkdir .ssh`
 
13. Create a file to store the public key with the command `$ touch .ssh/authorized_keys`
 
14. Edit that file using `$ nano .ssh/authorized_keys`
 
15. Now paste in the public key
 
16. We must change the permissions of the file and its folder by running
 ```
 $ sudo chmod 700 /home/grader/.ssh
 $ sudo chmod 644 /home/grader/.ssh/authorized_keys 
 ```
 
17. Change the owner of the `.ssh` directory from **root** to **grader** by using the command `$ sudo chown -R grader:grader /home/grader/.ssh`
 
18. The last thing we need to do for the SSH configuration is restart its service with `$ sudo service ssh restart`
 
19. **Disconnect from the server**
 
20. Now we need to login with the grader account using ssh. From your local terminal type `$ ssh -i ~/.ssh/udacity.rsa grader@52.47.99.96`
 
21. You should now be logged into your server via SSH
 
22. Lets enforce key authentication from the ssh configuration file by editing `$ sudo nano /etc/ssh/sshd_config`. Find the line that says **PasswordAuthentication** and change it to no. Also find the line that says **Port 22** and change it to **Port 2200**. Lastly change **PermitRootLogin** to no.
 
23. Restart ssh again: `$ sudo service ssh restart`
 
24. Disconnect from the server and try step "**23.**" again BUT also adding **-p 2200** at the end this time. You should be connected.
 
25. From here we need to configure the firewall using these commands:
 ```
 $ sudo ufw allow 2200/tcp
 $ sudo ufw allow 80/tcp
 $ sudo ufw allow 123/udp
 $ sudo ufw enable
 ```
 
26. Running `$ sudo ufw status` should show all of the allowed ports with the firewall configuration.
 
27. That pretty much wraps up the Linux configuration, now onto the app deployment.



# Application Deployment
 
Hosting this application will require the Python virtual environment, Apache with mod_wsgi, PostgreSQL, and Git.
 
1. Start by installing the required software

```
$ sudo apt-get install apache2

$ sudo apt-get install libapache2-mod-wsgi python-dev

$ sudo apt-get install git
```

2. Enable mod_wsgi with the command `$ sudo a2enmod wsgi` and restart Apache using `$ sudo service apache2 restart`.

3. If you input the servers IP address into a web browser you'll see the Apache2 Ubuntu Default Page

4. We now have to create a directory for our catalog application and make the user **grader** the owner.

```
$ cd /var/www
$ sudo mkdir catalog

$ sudo chown -R grader:grader catalog

$ cd catalog
```

5. In this directory we will have our **catalog.wsgi** file `var/www/catalog/catalog.wsgi`, our virtual environment directory which we will create soon and call **venv** `/var/www/catalog/venv`, and also our application which will sit inside of another directory called **catalog** `/var/www/catalog/catalog`.

6. First lets start by cloning our Catalog Application repository by `$ git clone [repository url] catalog`

7. Create the .wsgi file by `$ sudo nano catalog.wsgi` and make sure your secret key matches with your project secret key

```
import sys

import logging

logging.basicConfig(stream=sys.stderr)

sys.path.insert(0, "/var/www/catalog/")


from catalog import app as application

application.secret_key = 'super_secret_key'
```

8. Rename your `application.py`, `project.py`, or whatever you called it in your catalog application folder to `__init__.py` by `$ mv project.py __init__.py`

9. Now lets create our virtual environment, make sure you are in `/var/www/catalog`.

```
$ sudo pip install virtualenv

$ sudo virtualenv venv

$ source venv/bin/activate

$ sudo chmod -R 777 venv
``` 

 

10.  While our virtual environment is activated we need to install all packages required for our Flask application. Here are some defaults but you may have more to install.

```
$ sudo apt-get install python-pip

$ sudo pip install flask

$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 #etc...
```


11. Now for our application to properly run we must do some tweaking to the `__init__.py` file.

12. Anywhere in the file where Python tries to open `client_secrets.json` or `fb_client_secrets.json` must be changed to its complete path ex: `/var/www/catalog/catalog/client_secrets.json`



13. Time to configure and enable our virtual host to run the site

```
$ sudo nano /etc/apache2/sites-available/catalog.conf
```

Paste in the following:
 
```
<VirtualHost *:80>

    ServerName [Public IP]

    ServerAlias [Hostname]

    ServerAdmin admin@35.167.27.204

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
```

For servers hostname go [here](https://whatismyipaddress.com/ip-hostname) and paste the IP address. Save and quit nano


14. Enable to virtual host: `$ sudo a2ensite catalog.conf` and **DISABLE** the default host `$ a2dissite 000-default.conf` .
15. The final step is setting up the database

```
$ sudo apt-get install libpq-dev python-dev

$ sudo apt-get install postgresql postgresql-contrib

$ sudo su - postgres -i
$ psql
```

16. Create a database user and password

```
postgres=# CREATE USER catalog WITH PASSWORD [password];

postgres=# ALTER USER catalog CREATEDB;

postgres=# CREATE DATABASE catalog with OWNER catalog;

postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;

catalog=# GRANT ALL ON SCHEMA public TO catalog;

catalog=# \q
$ exit
```
Your command line should now be back to `grader`.


17. Now use nano again to edit your` __init__.py`, `database_setup.py`, and `createitems.py` files to change the database engine from `sqlite://catalog.db` to `postgresql://username:password@localhost/catalog`
18. Restart apache server `$ sudo service apache2 restart` and now the IP address and hostname should both load your application.

## SSH key

Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,FC5607321DBA86F08625A4E599C11C53

2DnhYIGW1Tvf2pN+gbtAKTE2cs3pZf0zJIE9JziXIUK0BKXtjl5DMKmQq69e6z99
P03DPWDGThfBcg02P+S608CECtVw+RIIiLuBySm+JegwaVsU0N72aBBK92Dl0whD
64LWyPeYiJJBhUPSQWQKAGEtJinLFruETRNAji3jPOH+Lbh3PV8Cwr18LJO1aXtW
l8oTXeJRWA+I+5Xj3T4qdxU8ExdKAZy4NyETSmti8a2cv9cmN/X4rfWoUZam7Rqb
mckeTLEZbIfs7E8KtPHugP0SWsYbsCKygjlQf/BdHnPpdI8M82HBe09a2aaLY04i
aLPISXGOAEvPSNEhVqkzCmSwFEgSK5Lzg6IMbpynImKlZB7lI2Isy3tNaMb0wCgu
DaZdQ/PkctXeIC1VTS2ruu45X45n/VUDYv81gdhH7QpOUPAgxIhgR09Wg9JdF4DO
f6kdKTZTsYXYHdBhZiygiyz8IpwcgbOfTUUNCTtu0HUhNVqrWegBcfCjyr5vp/y1
xTGRlFLYd9o+bvvTWEBYh7J+o+W0N1yffayVaVjx9breFACpQMz8xQ1MOCswuah6
M7DdaYn+FY6Ziz9pV5UNlbLAIbho2pj4MFoIepYTkG+p3FN8Hw0B0xp/cT6mrJ7A
6nBEOdfO3aQnsuym7hx81gTC9qjEsoYuN8ZkWbKT4mW7cByGKFQaV0pxcbfrn2U2
cUAfa2AIUTwZ0knpmBwHPgOcCJzE/Y69CpHL+btW7nz1wh8pVxEYxVjLCv3iG6Lm
JyZWhAsU8qRQC0QG/TtDVEnUEoke5Ix2k/RId0NIqvc0FVYzOgzKQx2OmGnIO08f
F33aRq+iQxgWVeEZR20ggbuGmd6JiuiCZEYU1E8iRU/u+NXjOcplUkYU89gHLRcB
f2hEnSDA7C/F1hpMmjdNvY+em8CorYI4QJHIl8npJjIy/1j5BfZeNU7lt5HsEbcS
fjtvzwRdfjjhwbljkMmlczWu6Z1LVgdrD6ycJl8hR5phoL9fy3tjn5It7Lg7DmS5
Yjuz+BSsT0WvwUbrmSJIGIGJIGh6r4vMdAC5OAOKQ6F+O/dQWFsN01xAHNVzXJgw
hk5hnBs4d/0Kfn7UY+JIuUbJdUcrpTvJgI3plvTLtMX0DhcaNCPJq1K7Sw3i8OPf
fy882SpAraXC061dvULncJLo1+mTXGMQqeqG7Z06MJ4dLUDitnB5XMRrWIhmWHU9
ybO8BUQYKDonwR/pu9GbZPl6UVsL2AeWY0iyIG/XQPYVtEijd0FGNnPq1m4Xuvym
aN30MEW0nRNZinujqFeqJiSllkE1Wsx+mdcXK7QsYVvL+WKKWHjbw9DF8/uFkESJ
/bENlAiia2y4IEGLhTt2G24k7EBJiN7ciXl3F8RvmPC2lruAi/VmlxuS/w+k3wCp
Z9gVDClqLNlYUcpsJkcNTz0oHFJGqSGhIKL/YkK1LIBZkFJDLhaiCOWTTuOCOZAl
1zMUe9EHN6SJQBWtOUejZ5HR2Ox0Ki1B60xrnRzqQQSeQqEGFIfkD7iyTDcqCIoB
/cY4+YvBnuyCQJDCcqVcOkQAz+kRs0W+FiPEC2xQyr64KB0NjSYDe76XgCLsnK5G
