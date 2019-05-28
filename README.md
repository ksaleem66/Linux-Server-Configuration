Project Details
---------------

You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

    IP address: 54.163.15.92

    Accessible SSH port: 2200

    Application URL: http://54.163.15.92.xip.io

    Local Machine OS: Ubuntu 18.04 LTS


Project Requirements
--------------------

1. Start a new Ubuntu Linux server instance on Amazon Lightsail.
2. Update all currently installed packages.
3. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
5. Create a new user account named grader.
6. Give grader the permission to sudo.
7. Create an SSH key pair for grader using the ssh-keygen tool.
8. Configure the local timezone to UTC.
9. Install and configure Apache to serve a Python mod_wsgi application.
10. Install and configure PostgreSQL
11. Install git.
12. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
13. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser.


Lightsil instance creation
--------------------------

1. Log in to Lightsail (https://aws.amazon.com/), if you don't already have an Amazon Web Services account, you'll be prompted to create one.
2. Lightsail will give message with a robot on it, prompting you to create an instance, click on create instance.
3. Select OS Only image.
4. Select Ubuntu Linux image (Ubuntu 16.04 LTS).
5. Choose your instance plan and pick the lowest price to get free-tier access.
6. Provide the instance a unique hostname.
7. Click create.
8. Wait for the instance to start up, it may take a few minutes for the instance to start up.
9. The public IP address of the instance is displayed along with its name. 


Connecting to the instance
--------------------------

1. Download the private key and rename it to lightsail_key.rsa.

2. Copy the private key to your home directory under ~/.ssh/ on your local machine : 

   $ mv lightsail_key.rsa ~/.ssh/

3. Change the permission of the private key on your local machine : 

   $ chmod 600 ~/.ssh/lightsail_key.rsa

4. To connect from local terminal type : 

   $ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@54.163.15.92


Create a new user named grader
------------------------------

1. Create grader user id and provide password as "grader":

   $ sudo adduser grader

2. Add grader user to sudoers 

	- from shell command line check the default sudoer file name: 

          $ sudo ls /etc/sudoers.d

	- from shell command line copy default file to grader file: 

          $ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader

	- from shell command line edit the file and change default name to grader: 

          $ sudo nano /etc/sudoers.d/grader

3. Login as grader and enter the password: 

   $ su - grader

4. To test sudo permission from shell command line type: 

   $ sudo cat /etc/passwd

   (if it displays the list then grader is a sudo)

5. From shell command line go to home directory: 

   $ cd

6. Create ssh directory: 

   $ mkdir .ssh

7. Create ssh key pair on local machine (provide file name as "grader_key" and password as "grader"): 

   $ ssh-keygen

8. On the AWS remote server create "authorized_keys" file: 

   $ touch /home/grader/.ssh/authorized_keys

9. Copy the contents of grader_key.pub on local machine to authorized_keys file on the server: 

   $ sudo nano .ssh/authorized_keys

   (copy the contents of grader_key.pub and paste in authorized_keys file)

10. From local machine terminal login to remote server as grader id with password 'grader': 

    $ ssh grader@54.163.15.92 -p 22

11. On the server change permissions for .ssh directory and authorized_keys file: 

    $ chmod 700 .ssh
    $ chmod 644 .ssh/authorized_keys

12. From local machine terminal login to Ubuntu AWS remote server using private key file: 

    $ ssh grader@54.163.15.92 -p 22 -i ~/.ssh/grader_key


Update installed packages
-------------------------

Login as grader and perform below steps on the server:

1. To update local file : 

   $ sudo apt-get update

2. To upgrade packages  : 

   $ sudo apt-get upgrade


Change the SSH port from 22 to 2200
-----------------------------------

Login as grader and perform below steps on the server:

1. Edit sshd_config file and change PasswordAuthentication value to 'no': 
   
   $ sudo nano /etc/ssh/sshd_config

2. In the same file change Port from 22 to 2200: 

   $ sudo nano /etc/ssh/sshd_config

3. Restart ssh service for changes to take effect: 

   $ sudo service ssh restart

4. On your Amazon Lightsail home page select "Networking" under your instance and add the following custom ports under firewall section:

	- Application = Custom
	- Protocol = TCP
	- Port range = 2200

   repeat the same and add

	- Application = Custom
	- Protocol = UDP
	- Port range = 123

   then click save

5. From local machine terminal login with grader id using new port: 

   $ ssh grader@54.163.15.92 -p 2200 -i ~/.ssh/grader_key

6. On the remote server configure firewall for new rules as following:

	- $ sudo ufw default deny incoming
	- $ sudo ufw default allow outgoing
	- $ sudo ufw allow 2200/tcp (for ssh)
	- $ sudo ufw allow 80/tcp   (for HTTP)
	- $ sudo ufw allow 123/udp  (for NTP)
	- $ sudo ufw enable (to activate firewall and enable above settings)

7. Confirm firewall is active with new settings: 

   $ sudo ufw status

   should get below screen:

   Status: active

   To                         Action      From
   --                         ------      ----
   2200/tcp                   ALLOW       Anywhere 
   80/tcp                     ALLOW       Anywhere
   123/udp                    ALLOW       Anywhere 
   2200/tcp (v6)              ALLOW       Anywhere (v6) 
   80/tcp (v6)                ALLOW       Anywhere (v6) 
   123/udp (v6)               ALLOW       Anywhere (v6) 


Change UTC time
---------------

1. From shell command line check UTC time: 

   $ timedatectl 

   (UTC time already set on the server, no change required).


Install Apache2
---------------

Login as grader and perform the following steps:

1. From shell command line type: 

   $ sudo apt-get install apache2

2. To confirm proper apache installation visit (http://54.163.15.92:80), it should display default page.

3. From shell command line install application handler mod_wsgi : 

   $ sudo apt-get install libapache2-mod-wsgi

   **NOTE : projects built with Python 3 need to install the Python 3 mod_wsgi package on the server (sudo apt-get install libapache2-mod-wsgi-py3)**
   **For my project I have installed libapache2-mod-wsgi-py3 as being developed using python3**

4. Edit the file /etc/apache2/sites-enabled/000-default.conf: 

   $ sudo nano /etc/apache2/sites-enabled/000-default.conf

   add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost>
   
   line to add "WSGIScriptAlias / /var/www/html/myapp.wsgi"

5. Restart apache for changes to take effect: 

   $ sudo apache2ctl restart

6. After installing WSGI create myapp.wsgi file and place the contents below for testing purpose: 

   $ sudo nano /var/www/html/myapp.wsgi

   def application(environ, start_response):
       status = '200 OK'
       output = 'Hello Udacity!'

       response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
       start_response(status, response_headers)

       return [output]

7. Revisit the page (http://54.163.15.92:80) and should display 'Hello Udacity' to confirm proper functionality


Install Postgresql
------------------

Login as grader and perform following steps:

1. Install postgresql: 

   $ sudo apt-get install postgresql

2. Change user to postgres to test connectivity: 

   $ sudo su - postgres

   (or use shortcut to switch user and connect to db at once)

   $ sudo -u postgres psql postgres

3. While connected to the database type the following to create catalog id:

	- postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
	- postgres=# ALTER ROLE catalog CREATEDB;

4. Connect to catalog db:

   - postgres=#\c catalog

5. grant access as follows:

   - postgres=# REVOKE ALL ON SCHEMA public FROM public;
   - postgres=# GRANT ALL ON SCHEMA public TO catalog;

6. Type \du from postgres prompt to list users and confirm creation of catalog id:

   - postgres=#\du

7. Quit from db

   - postgres=#\q

8. Restart psql: 

   $ sudo service postgresql restart

9. Switch back to the grader id: 

   $ exit

10. Create a new Linux user called catalog and follow on screen instructions to create the id: 

    $ sudo adduser catalog 

11. Give the catalog user id the permission to sudo (follow steps 1 & 2 under Create a new user named grader)

12. Login as catalog and create a database: 

    createdb catalog

13. Run psql and list databases to confirm creation of catalog db:

    postgres=#\l

14. Exit psql:

    postgres=#\q

15. Switch back to grader user id: 

    $ exit

16. Confirm remote connections are not allowed: 

    $ sudo nano /etc/postgresql/9.5/main/pg_hba.conf

    you should see below settings :

    local   all             postgres                                peer
    local   all             all                                     peer
    host    all             all             127.0.0.1/32            md5
    host    all             all             ::1/128                 md5


   - The first two security lines specify "local" as the scope that they apply to. This means they are using Unix/Linux domain sockets.
   - The second two declarations are remote, but if we look at the hosts that they apply to (127.0.0.1/32 and ::1/128), we see that these are interfaces that specify the 
     local machine.


Install git
-----------

Login as grader and perform the following steps:

1. Install git: 

   $ sudo apt-get install git

2. From shell command line provide your name: 

   $ git config --global user.name "Your Name"

3. From shell command line provide your email: 

   $ git config --global your.email.id@hotmail.com

4. From shell command line list configurations to make sure above settings done: 

   $ git config --list

5. Change directory to: 

   $ cd /var/www

6. Clone Item catalog Project repository from github to above directory: 

   $ sudo git clone https://github.com/ksaleem66/Item-Catalog-Project.git catalog

7. Change the ownership of the catalog directory to grader using: 

   $ sudo chown -R grader:grader catalog/

8. Change directory to catalog and rename application.py to __init__.py: 

   $ sudo mv application.py __init__.py

9. Modify following lines in __init__.py: 

   $ sudo nano __init__.py

   add absolute path to client_secrets.json file (and any occurance of it in __init__.py)

      CLIENT_ID = (json.loads(open('/var/www/catalog/catalog/client_secrets.json', 'r').read())
             ['web']['client_id'])

10. In __init__.py, database_setup.py and populate_data.py modify below line:

    $ sudo nano __init.py
    $ sudo nano database_setup.py
    $ sudo nano populate_data.py


    from this line: 
 
    engine = create_engine('sqlite:///moviecatalog.db', connect_args={'check_same_thread': False})
    
    to this line:

    engine = create_engine('postgresql://catalog:catalog@localhost/catalog')



Install Virtual Environment
---------------------------

Login as grader and perform the following steps:

1. Install python pip3: 

   $ sudo apt-get install python3-pip

2. Install virtual environment: 

   $ sudo apt-get install python-virtualenv

3. Change directory to catalog: 

   $ cd /var/www/catalog/catalog/

4. Create virtual environment: 

   $ sudo virtualenv -p python3 venv3

5. Change ownership to grader: 

   $ sudo chown -R grader:grader venv3/

6. Activate virtual environment: 

   $ source venv3/bin/activate

7. Change directory to venv3: 

   $ cd /var/www/catalog/catalog/venv3

8. Install the following dependencies on virtual environment:

	- $ pip3 install flask
	- $ pip3 install sqlalchemy
	- $ pip3 install oauth2client
	- $ pip3 install requests
	- $ pip3 install httplib2
	- $ pip3 install psycopg2
	- $ sudo apt-get install libpq-dev

9. From shell command line run the application,  you should see flask server running as below: 

   $ python3 __init__.py
   
   * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

10. Deactivate the virtual environment: 

    $ deactivate


Configure and enable a new Virtual Host
---------------------------------------

1. Add the following line in /etc/apache2/mods-enabled/wsgi.conf file to use Python 3: 

   $ sudo nano wsgi.conf

   #WSGIPythonPath directory|directory-1:directory-2:...
   WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages

2. Create new file and add the following lines: 

   $ sudo nano /etc/apache2/sites-available/catalog.conf

   <VirtualHost *:80>
		ServerName 54.163.15.92.xip.io
                ServerAlias ec2-54-163-15-92.compute-1.amazonaws.com
		ServerAdmin ksaleem66@hotmail.com
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

3. Enable the virtual host: 

   $ sudo a2ensite catalog

4. Reload apache2 server: 

   $ sudo service apache2 reload



Create WSGI file
----------------

1. Change to catalog directory: 

   $ cd /var/www/catalog

2. Create and edit catalog.wsgi file: 

   $ sudo nano catalog.wsgi

3. Add the following in the file:

   activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
   with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/catalog/catalog/")
   sys.path.insert(1, "/var/www/catalog/")

   from catalog import app as application
   application.secret_key = 'my_secret_key'

4. Restart Apache server: 

   $ sudo service apache2 restart

5. Now the directory structure should look like this:

|--------catalog
|----------------catalog
|-----------------------static
|-----------------------templates
|-----------------------venv3
|-----------------------__init__.py
|----------------catalog.wsgi


Disable default site
--------------------

1. Disable the default Apache site: 

   $ sudo a2dissite 000-default.conf

2. Restart Apache server: 

   $ sudo service apache2 restart


Create tables and populate data
-------------------------------

1. Change directory to catalog: 

   $ cd /var/www/catalog/catalog

2. Activate the virtual environment, you should see below shell command line: 

   $ source venv3/bin/activate

   (venv3) grader@ip-172-26-10-207:/var/www/catalog/catalog$

3. From shell command line run python to create catalog database tables: 

   $ python3 database_setup.py 

4. From shell command line run python to populate sample data to above created tables: 

   $ python3 populate_data.py 

5. From shell command line deactivate virtual environment: 

   $ deactivate 

   **Note: Table creation and data population already done, to avoid data duplication tables should be cleared first in PSQL before attempting above steps**


Google Developer Console (for google Signin)
--------------------------------------------

Login to your Google Developer Console

1. Under Credentials > OAuth consent screen > Authorized domains add "xip.io" domain
2. Click save

Under Credentials select your Item Catalog Project and add the following:

1. For Authorized JavaScript origins add "http://54.163.15.92.xip.io"
2. For Authorized redirect URIs add "http://54.163.15.92.xip.io/login" and "http://54.163.15.92.xip.io/gconnect"
3. Click Save


Launch the Web Application
--------------------------

1. Do a final restart of apache server: 

   $ sudo service apache2 restart

2. Open the browser to http://54.163.15.92.xip.io

3. Item Catalog Application should display on the screen

Resources
---------

1. GitHub Repository boisalai/udacity-linux-server-configuration
2. https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
3. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

Reviewer Comments
-----------------

1. There are some packages to be updated.

Action taken: Ran the following to update packages

   $ sudo apt-get update
   $ sudo apt-get dist-upgared


**Below is a screen shot after upgrade (no packages pending upgrade)**

Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-1083-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.

New release '18.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Tue May 28 10:13:18 2019 from 77.69.252.196


2. Looks like your SSH is configured to only prohibit log in as root with passwords, with the option 'prohibit-password'.
The safer option would be 'no'.


Action taken: Corrected the missed value from 'prohibit-password to 'no'
due to multiple AWS instances I have created during the project to reach for a final working one I must have had missed this value change.

