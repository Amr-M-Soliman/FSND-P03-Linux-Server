# Linux Server Configuration

### Description
A baseline installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, securing the system from a number of attack vectors and installing/configuring web and database servers.

## Useful Info
- IP address: **3.8.140.233**

- Accessible SSH port: **2200**

- Application URL: 
    * http://3.8.140.233
    * http://ec2-3-8-140-233.eu-west-2.compute.amazonaws.com
                


## Security 
1. To update available package lists run ```sudo apt-get update``` 
2. To upgrade insatalled packages run ```sudo apt-get upgrade```
3. To remove packages no longer required run ```sudo apt-get remove ```
4. To check everything can be done run  ```man apt-get```
5. To install **finger**(a utility software to check users' status) run ```sudo apt-get install finger``` 

#### Source ####
* [Official Ubuntu Documentation](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)
* [Debian](https://www.debian.org/doc/manuals/apt-howto/ch-apt-get.en.html)
 

### Change SSH port from 22 to 2200
 1. Download Private Key below
 2. Move the private key file into the folder `~/.ssh`
 3. Run `sudo nano /etc/ssh/sshd_config`
 4. Change the port from 22 to 2200
 5. Restart ssh service ```sudo service ssh restart```
 6. Confirm by running ```ssh -i ~/.ssh/<private key> -p 2200 root@3.8.140.233``` (replace `<private key>` with the downloaded private key)
 
#### Source ####
* [Godaddy](https://www.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306)
  
### Create a new user (`grader`) and give it permission to sudo

1. Create new user ```sudo adduser grader```
2. Create a new file under the suoders directory: `sudo nano /etc/sudoers.d/grader` 
3. Fill it with the following line of text:
"grader ALL=(ALL:ALL) ALL", then save it.
4. Check giving(user/access) for grader ```sudo cat /etc/passwd```
5. Confirm by running ```ssh -i ~/.ssh/<private key> -p 2200 grader@3.8.140.233``` (replace `<private key>` with the downloaded private key)

#### source ####
* [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

### Set ssh login using keys
1. Generate keys on local machine using`ssh-keygen` 
2. Save the private key in `~/.ssh` on local machine
3. deploy public key on developement enviroment

	#### On the virtual machine
	1. Switch to grader `su - grader`
	2. Create directory called **.ssh** by `mkdir .ssh` in home directory
	3. Create a new file in .ssh called **authorized_keys** by `touch .ssh/authorized_keys`
	
    4. Copy the public key generated on **local machine** to this file and save it
    
	5. Setting up some specific permissions on this **file** and the **ssh directory**
    ```
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
    ```
	6. Disable the password base logings to force all users to only login by a key pair. 
    ```
    sudo nano /etc/ssh/sshd_config
    ```
	then go for **Password Authentication** and change **yes** to **no**.
    7. Restart SSH service using 
    ```
    sudo service ssh restart
    ```
4. now you can use ssh to login with the new user (**grader**) you created
	```
    ssh -i <privateKeyFilename> grader@3.8.140.233 -p 2200 -i ~/.ssh/<sshKeyFilename>
    ```
    
### Source ###
* [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)

### Disable ssh login for *root* user
1. `sudo nano /etc/ssh/sshd_config`
2. Find the *PermitRootLogin* line and edit it to *no*.
3. `sudo service ssh restart`.

### Configure the Uncomplicated Firewall (UFW)

Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

1. Blocking all incoming requests 
```
sudo ufw default deny incoming
``` 
2. Allowing all outgoing
```
sudo ufw default allow outgoing
```
3. Supporting *ssh* 
```
sudo ufw allow ssh
```
4. Allowing all *tcp* connections through (port *2200*) for *ssh*
```
sudo ufw allow 2200/tcp
```
5. Supporting *HTTP Server* 
```
sudo ufw allow www
```
6. Allowing all *tcp* connections through (port *80*) fo *HTTP* 
```
sudo ufw allow 80/tcp
```
7. Allowing all *udp* connections through (port *123*) fo *NTP*
```
sudo ufw allow 123/udp
```
8. Disabling all *tcp* connections through (port *22*) for *ssh* 
```
sudo ufw delete allow 80
```
9. Enabling firewall
```
sudo ufw enable 
```

#### Source ####
 
* [Official Ubuntu Documentation](https://help.ubuntu.com/community/UFW)

* [DigitalOcean ](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04)

## Configure the local timezone to UTC
* Configure the time zone `sudo dpkg-reconfigure tzdata`

#### Source ####
 
* [Official Ubuntu Documentation](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)

## Web Applications Server
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`
4. #### Install needed modules & packages
    1. Install pip:
    `$ sudo apt-get install python-pip`
    2. Install httplib2:  
    `$ sudo pip install httplib2`
    3. Install requests:  
    `$ sudo pip install requests`
    4. Install Flask:
    `$ sudo pip install flask`
    5. Install oauth2client.client:  
    `$ sudo pip install oauth2client`
    6. Install SQLAlchemy:  
    `$ sudo pip install sqlalchemy`
    7. Install the psycopg:  
    `$ sudo apt-get install python-psycopg2`
    8. Install restful: 
        `sudo pip install flask-restful`
4. #### Install and configure PostgreSQL
    1. Install PostgreSQL `sudo apt-get install postgresql`
    2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
    3. Login as user "postgres" `sudo su - postgres`
    4. Get into postgreSQL shell using `psql`
    5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	   ```
	   postgres=# CREATE DATABASE catalog;
	   postgres=# CREATE USER catalog;
	   ```
    6. Set a password for user catalog
	
	   ```
	   postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	   ```
    7. Give user "catalog" permission to "catalog" application database
	
	   ```
	   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	   ```
    8. Quit postgreSQL `postgres=# \q`
    9. Exit from user "postgres" 
	
	   ```
	   exit
	   ```
5. #### Install git, clone and setup CatalogApp project.
    1. Install Git using `sudo apt-get install git`
    2. Use `cd /var/www` to move to the /var/www directory 
    3. Clone the Catalog App to the virtual machine `git clone https://github.com/Amr-M-Soliman/FSND-P03-Linux.git`
    4. Rename the project's name `sudo mv ./FSND-P03-Linux-master ./catalogApp`
    5. Edit `module.py` and `__init__.py`(in *myproject* dir) and change `engine = create_engine('sqlite:///itemcatalog.db')`
      to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
6. #### Create a .wsgi file
    1. Create the .wsgi File under /var/www/FlaskApp: 
	
	   ```
	   cd /var/www/catalogApp
	   sudo nano catalogapp.wsgi 
	   ```
     2. Add the following lines of code to the catalogapp.wsgi file:
	
       ```
       #!/usr/bin/python
       import sys
       import logging
       logging.basicConfig(stream=sys.stderr)
       sys.path.insert(0,"/var/www/catalogApp/")

       from myproject import app as application
       application.secret_key = [your_secret_key]

       ```

#### Source ####
* Setting up githup [GitHub](https://help.github.com/en/articles/set-up-git#platform-linux)
* Deploying a flask app [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* Securing Postgresql [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
    
### Configure and Enable Virtual Host
1. Use `sudo nano /etc/apache2/sites-enabled/000-default.conf`
2. Edit the file with the following lines of code to configure the virtual host. 
	
```
<VirtualHost *:80>

   WSGIDaemonProcess catalogApp user=grader group=grader threads=5

   WSGIScriptAlias / /var/www/catalogApp/catalogapp.wsgi

   <Directory /var/www/catalogApp>

      WSGIProcessGroup catalogApp

      WSGIApplicationGroup %{GLOBAL}

      Order deny,allow

      Allow from all

   </Directory>

  <Files catalogapp.wsgi>

    Require all granted

  </Files>

  ErrorLog ${APACHE_LOG_DIR}/error.log

  LogLevel warn

  CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```

### Restart Apache
 Restart Apache `sudo service apache2 restart `
 #### Source ####
  * [Official Ubuntu Documentation](https://help.ubuntu.com/lts/serverguide/httpd.html)
  * [Official Ubuntu Documentation](https://tutorials.ubuntu.com/tutorial/install-and-configure-apache#0)
