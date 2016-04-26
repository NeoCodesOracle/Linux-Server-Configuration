#LINUX SERVER CONFIGURATION
This project involves the configuration of a  Linux server to host the catalog application we wrote for project 3. This is broken down in two steps, the setup and security of the server and the deployment of the application.

#####INSTRUCTIONS
1. The server can be accessed via SSH through **port 2200**  
1. The Public IP Address is **52.38.112.238**  
1. The complete URL to the hosted web application is [http://ec2-52-38-112-238.us-west-2.compute.amazonaws.com/](http://ec2-52-38-112-238.us-west-2.compute.amazonaws.com/)    
1. A detailed list of software installed is included at the bottom of this document  under Installed Packages  
1. A detailed list of third party resources is listed at the bottom of this document under Additional References

##SERVER SETUP & SECURITY
####1. [ACCESS AMAZON INSTANCE](https://www.udacity.com/account#!/developmentenvironment)  
1.  Download Private Key 
1.  Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal  

    `mv ~/Downloads/udacity_key.rsa ~/.ssh/`

3.  Open your terminal and type in  

    `chmod 600 ~/.ssh/udacity_key.rsa`  

4. Followed by the command  

    `ssh -i ~/.ssh/udacity_key.rsa root@52.38.248.138`  

5. You are now logged in to your Amazon remote instance

####2. [MAKE SURE ALL PACKAGES ARE UP TO DATE](https://classroom.udacity.com/nanodegrees/nd004/parts/00413454010/modules/357367901175461/lessons/4331066009/concepts/48010894520923)

`sudo apt-get update`  
`sudo apt-get upgrade`

    
####3. [CREATE A NEW USER NAMED GRADER AND GRANT SUDO ACCESS](http://askubuntu.com/questions/168280/how-do-i-grant-sudo-privileges-to-an-existing-user)      
Before adding a user, make sure to change password for root user by typing `passwd` in the command line. This step is somewhat redundant as we will later disable remote root login.

Having changed passwords, enter the following two commands for user creation and granting sudo privileges to our new user.

    sudo adduser grader
    sudo adduser grader sudo

Optionally, we may install the finger application by typing `sudo apt-get install finger` and then ensuring our user was created successfully and granted the correct privileges by typing `finger grader`  
  
######RESOLVING UNRESOLVED HOST ISSUE  

When running this command, we receive a warning message alerting us that the `host ip-52-38-248-138 cannot be resolved`. We can fix this issue by adding this host to the hosts file.  

1. Enter the following command on the prompt and copy its output.  

    `hostname`  

1. Enter the following command  

    `sudo nano /etc/hosts`  

1. Append the hostname you copied when you ran the first command to the last line so you end up with something that looks like this  

    `127.0.0.1 localhost ip-10-20-51-76`  

    `# The following lines are desirable for IPv6 capable hosts`  
    `::1 ip6-localhost ip6-loopback`  
    `fe00::0 ip6-localnet`  
    `ff00::0 ip6-mcastprefix`  
    `ff02::1 ip6-allnodes`  
    `ff02::2 ip6-allrouters`  
    `ff02::3 ip6-allhosts`   

1. You should no longer see the warning message from now on 

####4. [GENERATE KEYS LOCALLY FOR GRADER KEY-BASED IDENTIFICATION](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
1. Locally, run the following command  

    `LocalUser ~ $ ssh-keygen -t rs`

1.  Enter a file name for the key we are about to generate (grader_key) and optional passphrase
1.  Copy the grader_key.pub file to the remote server by typing the following command  

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

    sudo reboot

This will shutdown and disconnect all users from server. Wait about 10 seconds before you attempt to log back in, otherwise, you may receive a timed out error because you tried logging in when the server was in the middle of rebooting.

####5. [CONFIGURE THE LOCAL TIMEZONE TO UTC](http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)
1. Run the following command   

    `sudo dpkg-reconfigure tzdata`  

2. From the pop-up menu, select your geographic area (US if you live in the US) and press Enter
3. From the pop-up menu, select your time zone and press Enter
4. You should now see the following output on your console  

    Current default time zone: 'US/Pacific'  
    Local time is now:      Fri Apr 15 17:29:17 PDT 2016.  
    Universal Time is now:  Sat Apr 16 00:29:17 UTC 2016.  

5. Configure NTP Synchronization to stay in sync if we ever need to deploy along with other servers  

    `sudo apt-get install ntp`

####6. [CHANGE PORT NUMBER FROM 22 TO 2200](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04)
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

####7. [CONFIGURE UNCOMPLICATED FIREWALL (UFW)](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers#tutorial_series_32)
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
  
###7. [PROTECTING SERVER AGAINST BRUTE FORCE ATTACKS](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-12-04)
Now that our server is out in the open, we need to take measures to make it more difficult to hijack. We can do this by installing fail2ban.  

1. Install fail2ban  

    `sudo apt-get install fail2ban`  

1. Copy configuration file  

    `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`  

1. Configure the details in jail.local file  
  
    `sudo nano /etc/fail2ban/jail.local`  
  
    Then, look for the section below and make the appropriate changes
    
    `[DEFAULT]`
    
    `# "ignoreip" can be an IP address, a CIDR mask or a DNS host`  
    ` ignoreip = 127.0.0.1/8`  
    `bantime  = 600`  
    `maxretry = 3`

    `# "backend" specifies the backend used to get files modification. Available`  
    `# options are "gamin", "polling" and "auto".`  
    `# yoh: For some reason Debian shipped python-gamin didn't work as expected`  
    `#      This issue left ToDo, so polling is default backend for now`  
    `backend = auto`
 
  
    `# Destination email address used solely for the interpolations in`  
    `# jail.{conf,local} configuration files.`   
    `destemail = root@localhost`  
  
    Add your IP address(es) that you'd be loggin in from to the `ignoreip` list. Separate multiple addresses with spaces.  This will prevent you from accidentally banning yourself out of the system. Make sure to change the `destmail` email address to where ever you want notifications to be sent. Finally, make sure to update the port number under `[SSH]` to 2200 or whatever your login port is. 

1. Restart fail2ban  
    
      `sudo service fail2ban restart`

    You can check if the system is running by typing `pgrep fail2ban -fl` and if everything installed correctly you will see an output like this  
  
    *`15915 fail2ban-server`*  

####How to Test Banning Policies
[To test whether our banning policies are working, you can follow the rules outlined here.](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04) under the section "Testing the Banning Policies"  

#####Warning: make sure you to add your IP address to the whitelist, or at least test this at the very end since you are at risk of locking yourself out.

##APPLICATION INSTALLATION AND DEPLOYMENT
####1. [INSTALL AND CONFIGURE APACHE2 TO SERVE A PYTHON MOD_WSGI APPLICATION](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)  
1. To install our application server, run  

    `sudo apt-get install apache2`  

1. Next, install application dependencies by running  

    `sudo apt-get install libapache2-mod-wsgi python-dev`  

1. Restart the server to activate the changes  

    `sudo service apache2 restart`

####2. [INSTALL AND CONFIGURE POSTGRE](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
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
  

####3. [INSTALL GIT AND CLONE REPOSITORY](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
1. Start by installing git  

    `apt-get install git`  

1. Navigate to the www directory so we can copy files directly into target directory  

    `cd ../../../var/www`  

1.  Create a directory called catalog and navigate to it
  
    `sudo mkdir catalog`
    `cd catalog`

1. Clone the catalog application from git repository  

    `sudo git clone https://github.com/yourgitusername/repotoclone.git`  

1. Follow the next instructions to make Github repository inaccessible. Change directory into newly-cloned directory  

    `cd CatalogApp`  

1. Create a file named .htaccess  

    `sudo nano .htaccess`  

1. Paste the following line and save file  

    `RedirectMatch 404 /\.git`  
  
Note: In order to ensure our application works as expected, we nee to make the following changes to the following files:  

1. Change the name of `catalogApp.py` to `__init__.py`  
1. Within `__init__.py` and `catalog_database.py`, update the the line   
`engine = create_engine('sqlite:///catalog.db')`  with `engine = create_engine('postgresql://catalog:mypasswordgoeshere@localhost/catalog')`.   
1. Within `__init__.py`, update all references to `client_secrets.json` to reflect the absolute path

###RESOLVING OAUTH  
####Google Signin
Now that Apache is serving our application from the amazon instance we can see that our login functionality is not working. Fixing it is just a matter of updating the Authorized JavaScript origins and Authorized redirect URIs.  

1. Go to [https://console.developers.google.com/project](https://console.developers.google.com/project)  
1. Select the project from the list of projects  
1. Then go to *__Credentials__* and select your project from the list of OAuth 2.0 client IDs  
1. Add the public IP address where your project is hosted (i.e. 52.38.112.238) to the Authorized JavaScript origins section  
1. Add the FQDN (fully qualified domain name) to the Authorized JavaScript origins section  
1. Add the FQDN (fully qualified domain name) to the Authorized redirect URIs section and append _/oauth2callback_ (i.e. _http://ec2-52-38-112-238.us-west-2.compute.amazonaws.com/oauth2callback_)

####Facebook Signin
1. Go to [https://developers.facebook.com/](https://developers.facebook.com/)
1. Select your application
1. Go to Settings
1. On the right pane, select Advanced and scroll down to the section Valid OAuth redirect URIs
1. Add the FQDN
1. Lastly, make sure the OAuth Logins for Client, Web and Embedded Browser are all set to *Yes*

---
###Installed Packages
1. ntp  
1. apache2  
1. libapache2-mod-wsgi
1. python-dev
1. git
1. python-pip
1. virtualenv
1. Flask
1. httplib2
1. requests
1. sqlalchemy
1. python-psycopg2
1. sqlalchemy
1. oauth2client
1. postgresql
1. postgresql-contrib
1. fail2ban
1. sendmail  

###Additional References  
1. [https://help.ubuntu.com/lts/serverguide/postgresql.html](https://help.ubuntu.com/lts/serverguide/postgresql.html)    
1. [http://askubuntu.com/questions/25374/how-do-you-install-mod-wsgi](http://askubuntu.com/questions/25374/how-do-you-install-mod-wsgi)
1. [https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)  
1. [https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)  
1. [http://unix.stackexchange.com/questions/119623/check-that-fail2ban-is-running](http://unix.stackexchange.com/questions/119623/check-that-fail2ban-is-running)  
1. [http://www.linuxquestions.org/questions/linux-security-4/fail2ban-is-it-working-705244/](http://www.linuxquestions.org/questions/linux-security-4/fail2ban-is-it-working-705244/)  
1. [https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)
1. [https://www.youtube.com/watch?v=kDRRtPO0YPA](https://www.youtube.com/watch?v=kDRRtPO0YPA)  
1. [https://www.youtube.com/watch?v=x6SvecADw2M](https://www.youtube.com/watch?v=x6SvecADw2M)  
1. [https://discussions.udacity.com/c/nd004-p5-linux-based-server-configuration](https://discussions.udacity.com/c/nd004-p5-linux-based-server-configuration)  
  