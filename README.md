# Udacity Full Stack NanoDegree - Linux Server configuration

A baseline installation of Ubuntu Linux on a virtual machine to host a Flask web application.

## Description
This is Project 5 for the Udacity Full Stack Nanodegree. I was required to take a bare-bones Linux Server and turn it into a secure and efficient Web Application Host to serve an application built in a previous project. It deploys my [UdacityCatalogProject][1] with the appropriate changes.

## TL:DR
- **Hostname:** http://ec2-52-51-36-36.eu-west-1.compute.amazonaws.com/
- **Public IP:** http://52.51.36.36
- **SSH Port:** 2200

## Primary Features
- [x] Linux Instance created via [Amazon LightSail][2]
- [x] Remote SSH login capability via **Port 2200**
- [x] Configured UFW ([Uncomplicated Fire Wall][3] to only allow for **SSH, HTTP** and **NTP**
- [x] Created new user **grader** with `sudo` access
- [x] Remote login of `root` user disabled
- [x] Configured the local timezone to **UTC**
- [x] Installed and configured **Apache** to serve a Python `mod_wsgi` application
- [x] Created a **PostgreSQL** Database for use by my application
- [x] `.git` directory is not publicly accessible via a browser
- [x] Application is deployed an running error free

## Extras
- [x] Automatic Installation of updates
- [x] Monitoring of unsuccessful login attempts using **fail2ban**
- [x] Addition of monitoring tools: **Glances** & **PySensors**

## Step by Step Walkthrough
In the guide below I detail all the steps I executed in order to complete the project.  


### 1 - Setup Amazon LightSail Server

1. Visit https://aws.amazon.com/lightsail/, create a new account and log in.
  #### Create Instance
  2. Click **Create Instance**
  3. Select the following:
    - Platform: **Linux/Unix**
    - Blueprint: **OS Only** & **Ubuntu**
  4. Name your instance and click **Create** (this can take up to 5-10 mins so make a cup of coffee)
  5. When complete it should show **Running** in the Instance Status Card.
  6. Make note of your Static IP address as you will need this going forward.

  #### Configure LightSail SSH
  1. Now click on the Instance Status Card.
  2. Scroll to the bottom and click the **Account Page** link.
  3. _Default_ should be chosen so just click **Download** (this downloads a `.pem` file)
  4. Return to your Instance main page and go to the **Networking** Tab
  5. Click **Add Another** under Firewall and add the following:
    - `Custom TCP 123`
    - `Custom TCP 2200`
  6. Ports `22` and `80` should already be listed

  #### SSH into Server
  1. You will now use the `.pem` downloaded earlier. Open your **Terminal**
  2. Move the file in the **ssh** folder on your computer:
    `$ mv ~/Downloads/[linux_config].pem ~/.ssh/`
  3. Set file rights (only owner can write and read.):  
    `$ chmod 600 ~/.ssh/[linux_config].pem`
  4. SSH into the instance:
    `$ ssh -i ~/.ssh/[linux_config].pem root@STATIC-IP-ADDRESS`

### 2 - User Management: Create a new user and give user the permission to sudo
Source: [DigitalOcean][4]  

1. Create a new user:  
  - `$ sudo adduser grader`
  - Set password as `grader`
2. Give new user the permission to sudo
  1. Open the sudo configuration:  
    `$ sudo nano /etc/sudoers.d/grader`
  2. Add the following line:  
    `grader ALL=(ALL:ALL) ALL`
3. Install **finger** which displays information about system users:
  `$ sudo apt-get install finger`

### 3  - Update and upgrade all currently installed packages
Source: [Ask Ubuntu][6]  

1. Update the list of available packages and their versions:  
  `$ sudo apt-get update`
2. Install newer vesions of packages you have:  
  `$ sudo sudo apt-get upgrade`
  #### Include cron scripts to automatically manage package updates
  Source: [Ubuntu documentation][7]  

  1. Install the unattended-upgrades package:  
    `$ sudo apt-get install unattended-upgrades`
  2. Enable the unattended-upgrades package:  
    `$ sudo dpkg-reconfigure -plow unattended-upgrades`

### 4 - Configure SSH Access and KeyGen
Source: [Ask Ubuntu][8]  

1. Change ssh config file:
  1. Open the config file:  
    `$ sudo nano /etc/ssh/sshd_config`
  2. Change to Port 2200.
  3. Change `PermitRootLogin` from `without-password` to `no`.
    * To get more detailed logging messasges, open `/var/log/auth.log` and change LogLevel from `INFO` to `VERBOSE`.
  5. Temporalily change `PasswordAuthentication` from `no` to `yes`.
  6. Append `UseDNS no`.
  7. Append `AllowUsers grader`.  
**Note:** All options on [ssh.com][9]
2. Restart SSH Service:  
  `$ /etc/init.d/ssh restart` or `# service sshd restart`
3. Create SSH Keys:  

  1. Generate a SSH key pair on the local machine:  
    `$ ssh-keygen`
  2. Copy the public id to the server:  
    `$ ssh-copy-id username@remote_host -p**_PORTNUMBER_**`
  3. Login with the new user:  
    `$ ssh -v grader@PUBLIC-IP-ADDRESS -p2200`
  4. Open SSHD config:  
    `$ sudo nano /etc/ssh/sshd_config`
  5. Change `PasswordAuthentication` back from `yes` to `no`.

4. Get rid of the warning message `sudo: unable to resolve host ...` when sudo is executed:  
Source: [Ask Ubuntu][11]  

  1. Open `$ nano /etc/hostname`.
  2. Copy the hostname.
  3. Append the hostname to the first line:  
    `$ sudo nano /etc/hosts`
5. Simplify SSH login:  
  1. Logout of the SSH instance:  
    `$ exit`
  2. Open the SSH config file on your local machine:  
    `$ sudo nano .ssh/config`
  3. Add the following lines:  
    ```
    Host NEWHOSTNAME
    HostName PUPLIC-IP-ADDRESS
    Port 2200
    User NEWUSER
    ```
  4. Now, you can login into the server more quickly:  
    `$ ssh NEWHOSTNAME`


### 5 - Configure the Uncomplicated Firewall (UFW)
Source: [Ubuntu documentation][14]  

1. Turn UFW on with the default set of rules:  
  `$ sudo ufw enable`
2. Check the status of UFW:  
  `$ sudo ufw status verbose`
3. Allow incoming TCP packets on port 2200 (SSH):  
  `$ sudo ufw allow 2200/tcp`
4. Allow incoming TCP packets on port 80 (HTTP):  
  `$ sudo ufw allow 80/tcp`
5. Allow incoming UDP packets on port 123 (NTP):  
  `$ sudo ufw allow 123/udp`  
  #### Monitor for repeated unsuccessful login attempts and ban attackers
  Source: [DigitalOcean][15]  

  1. Install Fail2ban:  
    `$ sudo apt-get install fail2ban`
  2. Copy the default config file:  
    `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
  3. Check and change the default parameters:  
      1. Open the local config file:  
        `$ sudo nano /etc/fail2ban/jail.local`
      2. Set the following Parameters:  
      ```python
        bantime  = 1800  
        destemail = YOURNAME@DOMAIN  
        action = %(action_mwl)s  
      ```    
  6. Check the current firewall rules:  
    `$ sudo iptables -S`
  7. Stop the service:  
    `$ sudo service fail2ban stop`
  8. Start it again:  
    `$ sudo service fail2ban start`

### 6 - Configure the local timezone to UTC
Source: [Ubuntu documentation][16]

1. Open the timezone selection dialog:  
  `$ sudo dpkg-reconfigure tzdata`
2. Then chose **None of the above**, then **UTC**.
3. Setup the ntp daemon ntpd for regular and improving time sync:  
  `$ sudo apt-get install ntp`
4. Choose closer NTP time servers:  
  1. Open the NTP configuration file:  
    `$ sudo nano /etc/ntp.conf`
  2. Open http://www.pool.ntp.org/en/ and choose the pool zone closest to you and replace the given servers with the new server list.  

### 7 - Install and configure Apache to serve a Python mod_wsgi application
Source: [DevOps][17]

1. Install Apache web server:  
  `$ sudo apt-get install apache2`
3. Install **mod_wsgi** for serving Python apps from Apache and the helper package **python-setuptools**:  
  `$ sudo apt-get install python-dev build-essential libapache2-mod-wsgi`
4. Enable mod_wsgi:
  `$ sudo a2enmod wsgi`
4. Restart the Apache server for mod_wsgi to load:  
  `$ sudo service apache2 restart`
5. Now input your Public IP Address in a browser and you should see an `Apache2 Ubuntu Default Page` declaring  **It Works**
**Note:** If you do not see the page, check the error message and Google a solution

### 8 - Install Git and Clone Project
Source: [GitHub][19]

  1. Install Git:  
    `$ sudo apt-get install git`
  2. Set your name, e.g. for the commits:  
    `$ git config --global user.name "YOUR NAME"`
  3. Set up your email address to connect your commits to your account:  
    `$ git config --global user.email "YOUR EMAIL ADDRESS"`
  4. Set up folder structure for project:

    ```bash
    $ cd /var/www
    $ sudo mkdir cheatsheet
    $ sudo chown -R grader:grader cheatsheet
    ```
  5. Make the GitHub repository inaccessible:  
    Source: [Stackoverflow][22]
    1. Create and open .htaccess file:  
      `$ cd /var/www/cheatsheet/` and `$ sudo nano .htaccess`
    2. Paste in the following:  
      `RedirectMatch 404 /\.git`

### 9 - Clone Project and set up WSGI
    1. Move to the folder:
      `$ cd cheatsheet`
    2. Now clone the CheatSheet application from GitHub:
      `git clone https://github.com/grocer-of-despair/CheatSheetApp.git CheatSheetApp`
    3. Create WSGI file:
      `$ sudo nano catalog.wsgi`
    4. Paste the following into it:
      ```python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0, "/var/www/cheatsheet/")

      from cheatsheet import app as application
      application.secret_key = 'supersecretkey'
      ```
      **Note:** This will allow the `__init__.py` file to be autorun as the application



### 10 - Setup Virtual Environment & Flask
Source: [Flask][20]

1. Install pip installer:  
  `$ sudo apt-get install python-pip`
2. Install virtualenv:  
  `$ sudo pip install virtualenv`
3. Set virtual environment to name 'venv':  
  `$ sudo virtualenv venv`
4. Enable all permissions for the new virtual environment (no sudo should be used within):  
  Source: [Stackoverflow][21]              
  `$ sudo chmod -R 777 venv`
5. Activate the virtual environment:  
  `$ source venv/bin/activate`
6. Install Flask inside the virtual environment:  
  `$ pip install Flask`
7. Run the app:  
  `$ python __init__.py`
8. Deactivate the environment:  
  `$ deactivate`

### 11 - Configure and Enable a New Virtual Host#
  1. Create a virtual host config file  
    `$ sudo nano /etc/apache2/sites-available/cheatsheet.conf`

  2. Visit http://www.hcidata.info/host2ip.cgi and find your Host name using your public IP-address, e.g. for `52.51.36.436`, it would be `ec2-52-25-0-41.us-west-2.compute.amazonaws.com`

  3. Paste in the following lines of code and change names and addresses regarding your application:  
  ```xml
    <VirtualHost *:80>
        ServerName [* PUBLIC-IP-ADDRESS *]
        ServerAdmin [* admin@PUBLIC-IP-ADDRESS *]
        ServerAlias [* HOSTNAME *]
        WSGIScriptAlias / /var/www/cheatsheet/cheatsheet.wsgi
        <Directory /var/www/cheatsheet/cheatsheet/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/cheatsheet/cheatsheet/static
        <Directory /var/www/cheatsheet/cheatsheet/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
  ```
  4. Enable the virtual host:  
    `$ sudo a2ensite cheatsheet`


### 12 - Install needed modules & packages
1. Activate virtual environment:  
  `$ source venv/bin/activate`
2. Install the following inside the venv:
  ```bash
  sudo pip install httplib2
  sudo pip install requests
  sudo pip install --upgrade oauth2client
  sudo pip install sqlalchemy
  sudo apt-get install python-psycopg2
  sudo pip install itsdangerous
  ```
  #### Setup Redis
  1. Install Redis:
    `$ sudo apt-get install redis-server`
  2. Start the redis server:
    `$ sudo service redis-server start`

  #### Install Monitor application Glances
  Sources: [Glances][31]

  2. `$ sudo pip install Glances`
  3. `$ sudo pip install PySensors`


### 13 - Install and configure PostgreSQL
Source: [DigitalOcean][23]

1. Install PostgreSQL:  
```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
```
3. Change project files to postgreSQL:
  1. Open the database setup file:  
    `$ sudo nano models.py`
  2. Change the line starting with "engine" to (fill in a password):  
    ```python
    engine = create_engine('postgresql://grader:grader@localhost/cheatsheet')
    ```
  3. Open the project file and repeat above step:
    `$ sudo nano __init__.py`

### 14 - Setup PSQL Database
8. Change to default user postgres:  
  `$ sudo su - postgres`
9. Connect to the system:  
  `$ psql`
10. Add postgre user with password:  
  Sources: [postgresql][25]
  1. Create user with LOGIN role and set a password:  
    `CREATE USER grader WITH PASSWORD grader;`
  2. Allow the user to create database tables:  
    `ALTER USER grader CREATEDB;`
  3. List current roles and their attributes:
    `\du`
11. Create database:  
  `CREATE DATABASE cheatsheet WITH OWNER grader;`
12. Connect to the database catalog
  `\c cheatsheet`
13. Revoke all rights:  
  `REVOKE ALL ON SCHEMA public FROM public;`
14. Grant only access to the catalog role:  
  `GRANT ALL ON SCHEMA public TO grader;`
15. Exit out of PostgreSQl and the postgres user:  
  `\q`, then `$ exit`
16. Create postgreSQL database schema:  
  `$ python models.py`



### 15 - Run application
1. Restart Apache:  
  `$ sudo service apache2 restart`
2. Open a browser and put in your Public IP or Host Name as URL. If everything works, the application load :D
3. If getting an internal server error, check the Apache error files:  
  Source: [A2 Hosting][27]  
  - View the last 40 lines in the error log:
    `$ sudo tail -40 /var/log/apache2/error.log`

### 16 - Get OAuth-Logins Working

  1. It is best to use absolute pathnames for both `client_secrets` files. To do so add the following to the `__init__.py` file:
  ```python
  THIS_FOLDER = os.path.dirname(os.path.abspath(__file__))
  fkey = os.path.join(THIS_FOLDER, 'fb_clients_secrets.json')
  gkey = os.path.join(THIS_FOLDER, 'client_secrets.json')
  ```

  2. To get the Google+ authorization working:  
    1. Go to the project on the Developer Console: https://console.developers.google.com/project
    2. Navigate to APIs & Services > Credentials > Edit Settings
    3. Add your Host name and public IP-address to your Authorized JavaScript origins prefaced by `http://`
    4. Add `http://HOSTNAME/oauth2callback` to Authorized redirect URIs, plus any relevant redirects.
    4. Download the JSON and paste the contents into the `client_secrets.json`, deleting the previous contents

  3. To get the Facebook authorization working:
    1. Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
    2. Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
    3. To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'

## Reference
I would sincerely like to the thank the following Alumni whose instructions were thoughtful and detailed:
  - [callforsky](https://github.com/callforsky/udacity-linux-configuration)
  - [rrjoson](https://github.com/rrjoson/udacity-linux-server-configuration/blob/master/README.md)
  - [stueken](https://github.com/stueken/FSND-P5_Linux-Server-Configuration/blob/master/README.md)
  - [ryanwaite28](https://github.com/ryanwaite28/udacity-nanodegree-fsnd/tree/master/linux-server-configuration)

## License
The contents of this repository are covered under the [GNU GENERAL PUBLIC LICENSE](LICENSE.txt)


[1]: https://github.com/grocer-of-despair/UdacityCatalogProject "UdacityCatalogProject"
[2]: https://aws.amazon.com/lightsail/ "Amazon LightSail"
[3]: https://help.ubuntu.com/community/UFW "Uncomplicated FireWall"
[4]: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps "How To Add and Delete Users on an Ubuntu 14.04 VPS"
[6]: http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade "What is the difference between apt-get update and upgrade?"
[7]: https://help.ubuntu.com/community/AutomaticSecurityUpdates "AutomaticSecurityUpdates"
[8]: http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server "Create a new SSH user on Ubuntu Server"
[9]: https://www.ssh.com/ssh/sshd_config/ "SSH.COM: SSHD_CONFIG"

[11]: http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none "Error message when I run sudo: unable to resolve host (none)"
[14]: https://help.ubuntu.com/community/UFW "UFW - Uncomplicated Firewall"
[15]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04 "How To Install and Use Fail2ban on Ubuntu 14.04"
[16]: https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29 "Ubuntu Time Management"
[17]: https://devops.profitbricks.com/tutorials/install-and-configure-mod_wsgi-on-ubuntu-1604-1/ "Install and configure Apache to serve a Python mod_wsgi application"

[19]: https://help.github.com/articles/set-up-git/#platform-linux "Set Up Git for Linux"

[23]: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps "How To Secure PostgreSQL on an Ubuntu VPS"
[25]: https://wiki.postgresql.org/wiki/First_steps "PostgreSQL Documentation"

[27]: https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files "How to view Apache log files"
[28]: http://stackoverflow.com/questions/12201928/python-open-method-ioerror-errno-2-no-such-file-or-directory "Python: No such file or directory"
[29]: http://discussions.udacity.com/t/oauth-provider-callback-uris/20460 "OAuth Provider callback uris"
[31]: http://glances.readthedocs.io/en/latest/ "Glances Documentation"
