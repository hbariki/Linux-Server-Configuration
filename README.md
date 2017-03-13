# Linux-Server-Configuration
Linux server configuration will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications(Itemcatalog).


This project includes installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

1. Local IP address: http://54.237.230.200/

2. The EC2 URL is http://ec2-54-237-230-200.compute-1.amazonaws.com.

Configuration steps:
1. Create an Instance with Amazon Light sail

 - In this project we are using Amazon Lightsail. Create an AWS Account and Login using your account.
 - Create an instance. Choose an instance image:Ubuntu, give your instance a hostname.It take few minutes to startup.
 - When you ssh you will be logged as the ubuntu user.
 - Add a custom port to 2200.
 -Download the default private key and save it in your comupter in form of .pem.
 
2. Follow the instructions provided to SSH into your server.

 - Login with this command from your computer to connect to the server. ```ssh@ubuntu54.237.230.200 -p -i path to your pem keys ```.
 - Add other user grader using command 'sudo adduser grader'.
 - To check user grader information. Use this commands
  ``` sudo apt-get install finger
   finger grader ```
- Use ssh-keygen to generate public and private keys in your machine.
- Use the commands to paste public key into your server.
    1.mkdir .ssh
    2.touch .ssh/authorized_keys
    3.copy the contents of public key in your local machine and paste it nano authorized_keys and save it.
    4.give permissions chmod 700 .ssh and chmod 644 .ssh/authorized_keys
    5.nano /etc/ssh/ssh_config, change password to no.
    6.sudo service ssh restart.
    7.```ssh grader@54.237.230.200 -p 2200 -i ~/.ssh/project7 ``` in new terminal.A pop-up willopen for authentication, just fill the password that you have fill during ssh-keygen creation.
    8.Private key cant be shared.
    
3.Give the grader the permission to sudo
 - Use ```sudo visudo ``` (edit the sudoers file. it is save to use sudo visudo to edit the sudoers file otherwise file will not be saved)
 - add the below line of code after root ALL=(ALL:ALL) ALL and save it (ctrl-X, then Y and Enter)
 - you can check the grader entry by below command: ``` sudo cat/etc/sudoers ```

4.Configure the Uncomplicated Firewall(UFW) to only allow incoming connections for SSH(port 2200), HTTP(port80), and NTP(port 123)
- check the firewall status using ```sudo ufw status```
- block all incoming connections on all ports using ```sudo ufw default deny incoming```
- allow outgoing connections on all ports using ```sudo ufw default allow outgoing```
- allow incoming connection for (SSH port 2200) using ```sudo ufw allow 2200/tcp```
- allow incoming connection for HTTP(port80) using ```sudo ufw allow 80/tcp```
- allow incoming connection for NTP(port 123) using ```sudo ufw allow 123/udp```
- check the added rules using ```sudo ufw show added```.
- enable the firewall using ```sudo ufw enable```
- check whether firewall is enable or not using ```sudo ufw status```

Resources : https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4331066009/concepts/48010894990923#

5.Configure the local timezone to UTC
-configure timezone using sudo dpkg-reconfigure tzdata(select none of the above and then set timezone to UTC)
Resources - timezone to UTC

6. Install Git
- Install git using ``` sudo apt-get install git-all ```
- set up git using:
   git config --global user.name "username"
   git config --global user.name "email@domain.com"
-check the configurations items using git config --list

Resources:
https://git-scm.com/book/en/v2/Getting-Started-Installing-Git.

7.Install and configure Apache to serve a python mod_wsgi application:
 - Install apache using sudo apt-get install apache2.
 -Type 54.237.230.200. You will see the apache ubuntu default page.
 -Install mod_wsgi using sudo apt-get install libapache2-mod-wsgi.
 -You then need to configure Apache to handle requests using the WSGI module. You’ll do this by editing the /etc/apache2/sites-  enabled/000-catalog.conf file. This file tells Apache how to respond to requests, where to find the files for a particular site and much more.
 -Add the following line at the end of the <VirtualHost *:80>block, right before the closing </VirtualHost>   line:WSGIScriprAlias/var/www/html/myapp.wsgi.
 -restart Apache with the sudo apache2ctl restart command.
 -WSGI is a specification that describes how a web server communicates with web applications. Most if not all Python web frameworks are WSGI compliant, including Flask and Django; but to quickly test if you have your Apache configuration correct you’ll write a very basic WSGI application.
 -create /var/www/html/myapp.wsgi using command sudo nano /var/www/html/myapp.wsgi. Write the following in this file.
  ```
  def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]
  ```  
 
 Resources :  https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4340119836/concepts/48159388430923#.


8.Deploy flask application

 Step-1:
 WSGI (Web Server Gateway Interface) is an interface between web servers and web apps for python. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications.
 sudo apt-get install libapache2-mod-wsgi python-dev.
 
 To enable mod_wsgi, run the following command:
 sudo a2emod wsgi.
 
 Step-2:
 -move to the ```/var/www directory ```
 -create an application directory structure using mkdir ```sudo mkdir catalog```
 -move inside this directory: ```cd catalog```
 -create another directory: ``` sudo mkdir catalog ```
 -move inside the directory and create two subdirectories named static and templates. cd catalog sudo mkdir static templates.
 -create the init.py file that will contain the flask application logic. ```sudo nano __init__.py```
 -Add the following logic to the file.
 
 ```
 from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, everyone!"
if __name__ == "__main__":
    app.run()
 
 ```    
close and save the file.

Step-3:
- Now , we will create a virtual environment for our flask application. use pip to install virtualenv and Flask. Install pip :
  ``` sudo apt-get install python-pip ```.
- Install virtualenv: ``` sudo pip install virtualenv ```
-set the environment name using: ``` sudo virtualenv venv```
-Install Flask in that environment by activating the virtual environment with the following command.
 ``` source venv/bin/activate```.
-Give this command to install Flask inside.
 ``` sudo pip install Flask ```
-Run the following command to test if the installation is successful and the app is running.
 ```sudo python __init__.py```
-It should display “Running on http://localhost:5000/” or "Running on http://127.0.0.1:5000/". If you see this message, you have successfully configured the app. 
-To deactivate the environment, give the following command:
 ```deactivate```

Step-4:

-Issue the following command in your terminal:

```sudo nano /etc/apache2/sites-available/catalog.conf```

- Add the following lines of code to the file to configure the virtual host. Be sure to change the ServerName to your domain or cloud servers Ip address.

```
 <VirtualHost *:80>
		ServerName 54.237.230.200
		ServerAdmin admin@mywebsite.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
save and close the file.

-Enable the virtual host with the following command:
 ```sudo a2ensite catalog```.
 
Step-5: 
- Apache uses the .wsgi file to serve the Flask app. Move to the /var/www/FlaskApp directory and create a file named flaskapp.wsgi with following commands:

```
cd /var/www/catalog
sudo nano flaskapp.wsgi 
```
- Add the following lines of code to the catalog.wsgi file:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'
 
```
Step-6:

-Restart Apache with the following command to apply the changes:

```
- sudo service apache2 restart

```
Resources: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

9.Clone the Github Repository
 - sudo mv UdacityItemcatalog ``` /var/www/catalog/catalog/ ```
 - The above command helps you to move the itemcatalog project to the directory /var/www/catalog/catalog.
 - To make github repository inaccessible make a .htaccess file in ```/var/www/catalog ```
 - paste the content - ``` RedirectMatch 404/\.git``` in this file and save it.

10.Install other packages.
 - ``` sudo apt-get install python-pip ```
 - ``` pip install httplib2 ```
 - ``` pip install requests ```
 - ``` sudo pip install --upgrade oauth2client ```.
 - ``` sudo pip install sqlalchemy ```
 - ``` pip install Flask-SQLAlchemy ```
 - ``` sudo pip install flask-seasurf ```
 - ``` pip freeze ``command to check installed packages.
 
11.Install Postgresql
- Install the Python Postgresql ``` sudo apt-get install postgresql ```.
- To check, no remote connections are allowed : ``` sudo nano /etc/postgresql/9.5/main/pg_hba.conf ```
- open database_setup.py using : sudo nano database_setup.py
- Update the create_engine line: python engine = create_engine('postgresql://catalog:catalog-pwd@localhost/catalog')
- Update the create_engine line in project.py and lotsofmenus.py too.
- move the project.py to init.py file.
-change to default user postgres ``` sudo -u postgres createuser catalog ```
-``` sudo su -postgre ```
-``` psql ```
-``` \password ```
-check the list of roles using \du
-Allow the user to create database : ALTER USER catalog CREATEDB; and check the roles and attributes using \du.
-Create database using : ``` CREATE DATABASE catalog WITH OWNER catalog ```
-Connect to database using : ``` \c catalog ```
-Revoke all the rights : ``` REVOKE ALL ON SCHEMA public FROM public ```;
-Grant the access to catalog: ``` GRANT ALL ON SCHEMA public TO catalog ```;
-After executing database_setup.py, again you can login as psql and check all the tables with the following commands.
 1. connect to database using: ``` \c catalog ```
 2.To see the tables in schema : ``` \dt ```
 3.to see particular table: ``` \d [tablename]
 4.Select * from [TableName] to see the entries in the table
 5.\q and then exit the postgresql user
 6.Restart postgresql using ``` sudo service postgresql restart ```

Resources: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps, Udacity discussion forms


12.Run the application: 
- create the database schema : ``` python database_setup.py python lotsofmeus.py ```.
--After executing lotsofmenus.py You can get an error saying

  ``` sqlalchemyerror : value too long for type character varying(250) ```.
-Use this command

 ``` ALTER TABLE menu_item ALTER COLUMN description TYPE text ```
- Restart Apache: ``` sudo service apache2 restart ```
-cd /var/www/catalog/catalog: execute - ``` Python __init__.py```
-type public IP address (http://54.237.230.200/) and you will find the Itemcatalog project in the webpage.
-change the paths in your __init__.py into
 ``` CLIENT_ID = json.loads(open('/var/www/catalog/catalog', r).read())['web']['client_id'] ```
 
 Resources : Udacity discussions forum.
 
 13.OAuth Login
  - Google Authorization steps
    1.Go to console.developer
    2.click on Credentials
    3.Add your host name(http://ec2-54-237-230-200.compute-1.amazonaws.com)and public IP address (http://54.237.230.200)to Javascript origins.
    4.update the client_secret.json file too (adding hostname and public IP address)
   
Resources -  Udacity Discussion Forums. 
 
 
 
References for whole project : Lessons from Linux Server Configuarion, Discussionsforums.

Thanks to Trish and Steve Udacity coaches for suggesting me the relevant material to complete the project  and clarifying my questions.
 

 
 
  

 
  
