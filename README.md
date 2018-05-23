# Linux Server Configuration - Udacity project

## Summary
Linux Server configuration to host catalog project as part of the Full Stack nanodegree program.
This tutorial will describe application hosting on AWS lightsail Ubuntu instance.

#### Resources
- [Amazon AWS Lightsail Ubuntu Linux Instance](https://lightsail.aws.amazon.com/ls/docs/getting-started/article/getting-started-with-amazon-lightsail)
-  [Git](https://git-scm.com/)
- [VirtualBox](https://www.virtualbox.org)
- [OS Linux Ubuntu](https://www.ubuntu.com)

#### Installation instructions:

- Amazon AWS 

    - Create Amazon AWS account: [Amazon AWS Lightsail account](https://portal.aws.amazon.com/) with [Ubuntu linux based instance](https://lightsail.aws.amazon.com/ls/docs/en/articles/getting-started-with-amazon-lightsail). 
    - Create new instance
    - Select instance image (Platform: Linux/Unix, Blueprint: OS only, Ubuntu)
    - Name your instance > **Create**
    - Create Static IP for your instance (in Networking Menu)
    - Name your static IP > **Create** (note your static IP, example: 18.184.238.72)
    - Configure port:

        | Application   | Protocol      | Port Range  |
        | ------------- |---------------| ------------|
        | SSH           | TCP           | 22          |
        | HTTP          | TCP           | 80          |
        | CUSTOM        | TCP           | 123         |
        | CUSTOM        | TCP           | 2200        |

     - Find your DNS address via [reverse IP lookup](https://mxtoolbox.com), Example: ec2-18-184-238-72.eu-central-1.compute.amazonaws.com


- Update and configure Amazon AWS 
     - Connect to your AWS instance via browser in AWS instance **CONNECT** menu (Connect using SSH).
     - type and execute in terminal: 
         - Update the update list: `$ sudo apt-get update` _Fetches the list of available updates_
         - Upgrade packages: `$ sudo apt-get upgrade` _Strictly upgrades the current packages_
         - Upgrade distribution `$ sudo apt-get dist-upgrade` _Installs updates (new ones)_
         - Remove no longer required packages: `$ sudo apt-get autoremove`
         - Reboot the instance with: `$ sudo reboot`
         - Reconnect via Connect using SSH from AWS instance menu.
    - Create new user **`grader`**
        - `$ sudo adduser grader` When prompted enter Full name and Unix password.
             ``` 
            ubuntu@ip-172-26-10-33:~$ sudo adduser grader
            Adding user `grader' ...
            Adding new group `grader' (1001) ...
            Adding new user `grader' (1001) with group `grader' ...
            Creating home directory `/home/grader' ...
            Copying files from `/etc/skel' ...
            Enter new UNIX password: 
            Retype new UNIX password: 
            passwd: password updated successfully
            Changing the user information for grader
            Enter the new value, or press ENTER for the default
            Full Name []: Udacity Student
            Room Number []: 
            Work Phone []: 
            Home Phone []: 
            Other []: 
            Is the information correct? [Y/n] y
            ubuntu@ip-172-26-10-33:~$ 
            ```
        - Use the usermod command to add the user **`grader`** to the sudo group. [How to create a sudo user on ubuntu docs](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart)
            - `$ sudo usermod -aG sudo grader`
        - Enter **`grader`** account: `$ su grader` and confirm with grader password
        
    - Configure sshd_conf file on your AWS instance:
        - Add the following entry to sshd_config to disable root to login to the server directly:
        `$ vi /etc/ssh/sshd_config` 
        Edit line `PermitRootLogin prohibit-password` to `PermitRootLogin no`
        Edit line `PasswordAuthentication no` to: `PasswordAuthentication yes`
        Restart ssh service `$ sudo service ssh restart`
        You can try to connect to AWS instance via terminal on you linux PC.
        `$ ssh grader@18.184.238.72` 
    - Configure firewall on your AWS instance. [How to set up a Firewall with UFW on ubunut](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-16-04):
        - To set the defaults used by UFW:
        `$ sudo ufw default deny incoming`
        `$ sudo ufw default allow outgoing`
        - Allowing SSH Connections:
        `$ sudo ufw allow ssh`
        `$ sudo ufw allow 2200`
        - Allowing Other Connections:
        `sudo ufw allow 80` _HTTP on port 80_
        `sudo ufw allow 123` _NTP on port 123_
        - Enable Firewall:
        `$ sudo ufw enable` _Enabling UFW_
        `$ sudo ufw status verbose` _Check Status_
    - Set Up SSH Keys: [How To Set Up SSH Keys on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604)
        - Create the RSA Key Pair (usually your computer): 
        `$ ssh-keygen`
        `$ ssh-copy-id grader@18.184.238.72`
        Try logging into the AWS instance:
        `$ ssh grader@18.184.238.72`
        If all set correctly and were able to login, we can change the line `PasswordAuthentication yes` in `/etc/ssh/sshd_config` to `PasswordAuthentication no`
    - Set Timezone and Install NTP
        - `$ sudo timedatectl set-timezone UTC` _Set Timezone to UTC_
        - `$ sudo apt-get install ntp` _Install NTP_
        - `$ sudo reboot` _Reboot instance_

- Install packages for application hosting:
    - Install [APACHE2](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04) packages:
        - `$ sudo apt-get update`
        - `$ sudo apt-get install apache2` _Install Apache 2_
        - `$ sudo systemctl status apache2` _Check if web server is running_
        - Enter your ip addres or domain name into browser address bar (we should see default Ubuntu Apache 2 web page): `http://server_domain_or_IP`
    - Install mod wsgi to run wsgi files
        - `$ sudo apt-get install libapache2-mod-wsgi`
    - [Install PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04):
        - `$ sudo apt-get update`
        - `$ sudo apt-get install postgresql postgresql-contrib`
    - Install sqlite3 (used by the catalog app)
        - `$ sudo apt-get install sqlite3`  
    - clone catalog app to /var/www/ [link to app repository and documentation](https://github.com/micond/catalog)
        - `$ cd /var/www/`      
        - `$ sudo git clone https://github.com/micond/catalog.git`
    - Create Virtual Environment
        - `$ sudo pip install virtualenv`
        - `$ sudo virtualenv venv`
        - `$ sudo chmod -R 777 venv`
        - activate virtual environment: `$ source venv/bin/activate`
        - Install python modules: `$ pip install -r requirements.txt`
        - run project.py in /var/www/catalog/ and check if app will run on localhost with no missing python modules.
        - `$ deactivate` _deactivate venv_
    - Configure wsgi / Apache
        - create file `catalog.wsgi` in /var/www/ with below content
         ``` 
        #! /usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,'/var/www/')
        from catalog import app as application
        application.secret_key = 'super_secret_key'
        ```
        - create file `catalog.conf` in /var/www/ with below content:
        ```
            WSGIPythonPath /var/www/catalog/venv/lib/python2.7/site-packages/
        <VirtualHost *:80>
            ServerName 18-184-238-72
            ServerAlias ec2-18-184-238-72.eu-central-1.compute.amazonaws.com
            ServerAdmin your@email.com
            WSGIDaemonProcess catalog user=www-data group=www-data threads=5 python-path=/var/www/catalog/venv/lib/python2.7/site-packages/
            WSGIScriptAlias / /var/www/catalog.wsgi
            <Directory /var/www/catalog/>
                Require all granted
            </Directory>
            <Directory /var/www/catalog>
            <Files __init__.py>
                Require all granted
            </Files>
            </Directory>
            Alias /static /var/www/catalog/static
            <Directory /var/www/catalog/static/>
                Require all granted
            </Directory>
            Alias /templates /var/www/catalog/templates
            <Directory var/www/catalog/templates/>
                Require all granted
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
        ```
        - run `$ sudo a2ensite catalog` _Enable the virtual host_
        - rename `project.py` to `__init__.py` in /var/www/catalog/
        - restart apache server: `$ sudo service apache2 restart`
### Login Authorization Configuration:
You need to have a [facebook developer account](https://developers.facebook.com/) and a [google developer account](https://console.developers.google.com/)

Google:
Authorized JavaScript origins and Authorized redirect URIs
http://ec2-18-184-238-72.eu-central-1.compute.amazonaws.com

Facebook:      
Valid OAuth Redirect URIs:
http://ec2-18-184-238-72.eu-central-1.compute.amazonaws.com
        
### Usefull links:
- [Amazon AWS lightsail account](https://portal.aws.amazon.com/)
- [mxtoolbox](https://mxtoolbox.com/SuperTool.aspx) _Reverse IP Lookup_
- [How to install PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
- [mod_wsgi(Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
- [How To Set Up a Firewall with UFW on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-16-04)
- [How To Set Up SSH Keys on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604)
- [How To Configure SSH Key-Based Authentication on a Linux Server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
- [How To Create a Sudo User on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart)
- [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
        
