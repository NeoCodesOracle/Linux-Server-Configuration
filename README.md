#SERVER SETUP & SECURITY
####[ACCESS AMAZON INSTANCE](https://www.udacity.com/account#!/developmentenvironment)  
1.  Download Private Key from link below
1.  Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal  

    `mv ~/Downloads/udacity_key.rsa ~/.ssh/`

3.  Open your terminal and type in  

    `chmod 600 ~/.ssh/udacity_key.rsa`  

4. Followed by the command  

    `ssh -i ~/.ssh/udacity_key.rsa root@52.38.248.138`  

5. You are now logged in to your Amazon remote instance

####[MAKE SURE ALL PACKAGES ARE UP TO DATE](https://classroom.udacity.com/nanodegrees/nd004/parts/00413454010/modules/357367901175461/lessons/4331066009/concepts/48010894520923)

`sudo apt-get update`  
`sudo apt-get upgrade  `

    
####[CREATE A NEW USER NAMED GRADER AND GRANT SUDO ACCESS](http://askubuntu.com/questions/168280/how-do-i-grant-sudo-privileges-to-an-existing-user)      
Before adding a user, make sure to change password for root user by typing `passwd` in the command line. This step is somewhat redundant as we will later disable remote root login.

Having changed passwords, enter the following two commands for user creation and granting sudo privileges to our new user.

    `sudo adduser grader`  
    `sudo adduser grader sudo`

Optionally, we may install the finger application by typing `sudo apt-get install finger` and then ensuring our user was created successfully and granted the correct privileges by typing `finger grader`  
  
######RESOLVING UNRESOLVED HOST ISSUE  

When running this command, we receive a warning message alerting us that the `host ip-52-38-248-138 cannot be resolved`. We can fix this issue by adding this host to the hosts file.  

1. Enter the following command on the prompt and copy its output.  

    `hostname`  

1. Enter the following command  

    `sudo nano /etc/hosts`  

1. Append the hostname you copied when you ran the first comman to the last line so you end up with something that looks like this  

    `127.0.0.1 localhost ip-10-20-51-76`  

    `# The following lines are desirable for IPv6 capable hosts`  
    `::1 ip6-localhost ip6-loopback`  
    `fe00::0 ip6-localnet`  
    `ff00::0 ip6-mcastprefix`  
    `ff02::1 ip6-allnodes`  
    `ff02::2 ip6-allrouters`  
    `ff02::3 ip6-allhosts`   

1. You should no longer see the warning message from now on 

####[GENERATE KEYS LOCALLY FOR GRADER KEY-BASED IDENTIFICATION](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
1. Locally, run the following command  

    `LocalUser ~ $ ssh-keygen -t rs`

1.  Enter a file name for the key we are about to generate (grader_key) and optional passphrase
1.  Copy the grader_key file to the remote server by typing the following command  

    `cat ~/.ssh/grader_key.pub | ssh root@52.38.248.138 -i ~/.ssh/udacity_key.rsa "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"`  

1. Log into remote server  

    `ssh -i ~/.ssh/udacity_key.rsa root@52.38.248.138`  

1. Navigate to the grader directory with the following command  

    `cd ../../home/grader`  

1. Copy the .ssh folder in the root directory with the `authorized_keys` file to the current directory  

    `cp -avr ~/.ssh ./`  

1. Make sure to change the ownership of both the .ssh directory and authorized_keys file to grade  

    `sudo chmod 644 .ssh/authorized_keys`  
    `sudo chmod 700 .ssh`  
    `sudo chown grader:grader .ssh`  
    `sudo chown grader:grader .ssh/authorized_keys`  

1. Open a new terminal on local machine and log in using new user credentials  

    ` ssh -i ~/.ssh/grader_key grader@52.38.112.238`  

1. You are now logged in as the newly created user.  
  
Upon login, you may see a message that says __*system restart required*__. To do so, just enter the command  

    `sudo reboot`    

This will shutdown and disconnect all users from server. Wait about 10 seconds before you attempt to log back in, otherwise, you may receive a timed out error because you tried logging in when the server was in the middle of rebooting.

####[CONFIGURE THE LOCAL TIMEZONE TO UTC](http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)
1. Run the following command   

    `sudo dpkg-reconfigure tzdata`  

2. From the pop-up menu, select your geographic area (US if you live in the US) and press Enter
3. From the pop-up menu, select your time zone and press Enter
4. You should now see the following output on your console  

    `Current default time zone: 'US/Pacific'`    
    `Local time is now:      Fri Apr 15 17:29:17 PDT 2016.`  
    `Universal Time is now:  Sat Apr 16 00:29:17 UTC 2016.`    
5. Configure NTP Synchronization to stay in sync if we ever need to deploy along with other servers  

    `sudo apt-get install ntp`

####[CHANGE PORT NUMBER FROM 22 TO 2200](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04)
1. Open the configuration file with nano file editor using the following command   
`sudo nano /etc/ssh/sshd_config`
1. Look for, and change the following values:  

    `# What ports, IPs and protocols we listen for`  
    `Port 22`  

    Change this value from `Port 22` to `Port 2200`. Then look for the section below

        `# Authentication:`  
        `LoginGraceTime 120`
        `PermitRootLogin without-password`  
        `StrictModes yes`  

    Change `PermitRootLogin without-password` to `PermitRootLogin no`. These changes will allow access to users with SSH keys only, rendering the root user useless.
    
    From now on when logging in, we need to specify the port to connect to, since we changed the default value. Connections to our instance will now have the following form  
  
    `ssh -i ~/.ssh/grader_key grader@52.38.248.138 -p 2200`

####[CONFIGURE UNCOMPLICATED FIREWALL (UFW)]([https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers#tutorial_series_32])
1. Allow ssh connections on port 2200 only  

    `sudo ufw allow 2200/tcp`  

1. Allow http connections on port 80 only  

    `sudo ufw allow 80/tcp`  

1. Allow udp connections on port 123 only  

    `sudo ufw allow 123/udp`  

1. Now, check the rules by using the following command  

    ` sudo ufw show added`  

1. Finally, with the rules in place, enable the firewall  

    `sudo ufw enable`  

1. One last check to make sure everything is in good standing  

    `sudo ufw status verbose`  

#APPLICATION INSTALLATION AND DEPLOYMENT
####[INSTALL AND CONFIGURE APACHE2 TO SERVE A PYTHON MOD_WSGI APPLICATION](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)  
1. To install our application server, run  

    `sudo apt-get install apache2`  

1. Next, install application dependencies by running  

    `sudo apt-get install libapache2-mod-wsgi python-dev`  

1. Restart the server to activate the changes  

    `sudo service apache2 restart`

####[INSTALL AND CONFIGURE POSTGRE]([https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps])
1. To install Postgre, run  

    `sudo apt-get install postgresql postgresql-contrib`  

1. Next, we need to remove a potential attack vector by not allowing remote connections to the database. This is the current default when installing PostgreSQL from the Ubuntu repositories. We can double check that no remote connections are allowed by looking in the host based authentication file  

    `sudo nano /etc/postgresql/9.1/main/pg_hba.conf`  

1. Create a user named catalog with limited permissions to database. First, log into psql  

    `sudo su - postgres`  
    `psql`  

    Then,  

    `CREATE ROLE catalog WITH NOSUPERUSER NOCREATEROLE NOCREATEUSER CREATEDB;`  

1. Now create database with catalog as owner  

    `CREATE DATABASE catalog WITH OWNER catalog;`  

1. We can now connect to the database and lock down the permissions to only let catalog create tables  

    `\c catalog`  
    `REVOKE ALL ON SCHEMA public FROM public;`  
    `GRANT ALL ON SCHEMA public TO catalog;`  

1. Exit psql by typing the following commands  

    `\q`  
    `exit`

####[INSTALL GIT AND CLONE REPOSITORY]([https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps])
1. Start by installing git  

    `apt-get install git`  

1. Navigate to the www directory so we can copy files directly into target directory  

    `cd ../../../var/www`  

1. Clone the catalog application from git repository  

    `sudo git clone https://github.com/NeoCodesOracle/CatalogApp.git`  

1. Follow the next instructions to make Github repository inaccessible. Change directory into newly-cloned directory  

    `cd CatalogApp`  

1. Create a file named .htaccess  

    `sudo nano .htaccess`  

1. Paste the following line and save file  

    `RedirectMatch 404 /\.git`


---
Activity log  
1.sudo pip install virtualenv  
1.sudo virtualenv venv  
1.sudo chmod -R 777 venv  
1.source venv/bin/activate  
1.sudo pip install Flask  
1.sudo pip install Flask -t /var/www/catalog/catalog/venv/lib/python2.7/site-packages  
1. pip install httplib2  
1. pip show flask
1. pip install requests  
1. sudo pip install flask-seasurf  
1. sudo pip install --upgrade oauth2client  
1. sudo pip install sqlalchemy  
1. sudo apt-get install python-psycopg2  
1.Deactivate venv: $ deactivate  
1.Create configuration file for new site sudo nano /etc/apache2/sites-available/catalog.conf (see file content below)  
1.Enable site: sudo a2ensite catalog  
1.Create app file: sudo nano /var/www/catalog/app.wsgi (see file content below)  
1.Restart server: sudo service apache2 restart  

####RESOLVING OAUTH
Now that Apache is serving our application from the amazon instance we can see that our login functionality is not working. Fixing it is just a matter of updating the Authorized JavaScript origins and Authorized redirect URIs.  

1. Go to [https://console.developers.google.com/project](https://console.developers.google.com/project)  
1. Select the project from the list of projects  
1. Then go to *__Credentials__* and select your project from the list of OAuth 2.0 client IDs  
1. Add the public IP address where your project is hosted (i.e. 52.38.112.238) to the Authorized JavaScript origins section  
1. Add the FQDN (fully qualified domain name) to the Authorized JavaScript origins section  
1. Add the FQDN (fully qualified domain name) to the Authorized redirect URIs section and append *__/oauth2callback__* (i.e. *__http://ec2-52-38-112-238.us-west-2.compute.amazonaws.com/oauth2callback__*)

