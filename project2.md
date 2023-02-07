WEB STACK IMPLEMENTATION (LEMP STACK)

1.	Created new EC2 instance on AWS
2.	Launched instance.
3.	Opened terminal and used command: cd Downloads to SSH
ssh -i "eunnie_key_ec2.pem" ubuntu@ec2-3-85-1-39.compute-1.amazonaws.com

STEP 1 – INSTALLING THE NGINX WEB SERVER
1.	Updated server package by using command : sudo apt update
2.	Next, installed Nginx by using command: sudo apt install nginx once prompted enter y to confirm that you are installing Nginx 
3.	To verify that nginx was successfully installed and running, run: sudo systemctl status nginx

 ![image](https://user-images.githubusercontent.com/120044190/217149669-125df955-9042-4e68-bfab-59605e61319b.png)


4.	To receive traffic by web server, open TCP port 80. TCP port open by default to access via SSH, so rule needed for EC2 configuration to open inbound connection through port 80
5.	To access server locally in Ubuntu shell, run: curl http://localhost:80 or curl http://127.0.0.1:80

 ![image](https://user-images.githubusercontent.com/120044190/217149709-b07904a1-e6ed-4302-9936-40070d2c24bf.png)


6.	 Test that Nginx server can respond to requests from the internet, by opening a web browser and try to access following url: http://3.85.1.39:80

To obtain Public IP address on CLI, use command: curl -s http://169.254.169.254/latest/meta-data/public-ipv4

Now the web server is correctly installed and accessible through my firewall.

![image](https://user-images.githubusercontent.com/120044190/217149785-8b2332a3-e76a-47ff-a176-3b48b6fbf8a9.png)

 
STEP 2 — INSTALLING MYSQL
After installing web server and it is up and running, install a Database Management System to store and manage data in a  relational database, such as MySQL

1.	Use ‘apt’ to acquire and install the software: sudo apt install mysql-server , then confirm by typing y and enter.
2.	After installation was finished, I loggged in to the mysql console  using: sudo mysql 

This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command. You should see output like this:
 
![image](https://user-images.githubusercontent.com/120044190/217149848-a462b376-e65f-4db3-9a67-3017901ce829.png)
 
 

3.	 Run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Before running the script you will set a password for the root user, using mysql_native_password as default authentication method. We’re defining this user’s password as 
PassWord.1

ALTER USER 'root'@'localhost'IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

4.	Next exit the MySQL shell using: mysql> exit 
5.	Started interactive script by running: sudo mysql_secure_installation Note: will be asked if want to configure the VALIDATE PASSWORD PLUGIN

If enabled, passwords which don’t match the specified criteria will be rejected by MySQL with an error. It is safe to leave validation disabled, but you should always use strong, unique passwords for database credentials.
Answer Y or yes, or anything else to continue without enabling.
For the rest of the questions, press Y and hit the ENTER key at each prompt. This will prompt you to change the root password, remove some anonymous users and the test database, disable remote root logins, and load these new rules so that MySQL immediately respects the changes you have made.
 
 ![image](https://user-images.githubusercontent.com/120044190/217149921-1644a8d5-a025-465c-959f-ce7fd566028d.png)


6.	When finished, test if  able to log in to the MySQL console by typing: sudo mysql -p
 
 ![image](https://user-images.githubusercontent.com/120044190/217149972-debec0d0-998c-4528-bbd4-a181f9056160.png)

 
7.	To exit the MySQL console, run: mysql> exit
For increased security, it’s best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server.
MySQL server is now installed and secured
STEP 3 – INSTALLING PHP
I have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.
While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. You’ll need to install 
php-fpm
, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need 
php-mysql
, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.
1.	 Install these 2 packages at once, run: sudo apt install php-fpm php-mysql and when prompted, type Y and press ENTER to confirm installation
PHP components have now been installed.
 
 ![image](https://user-images.githubusercontent.com/120044190/217150104-fd6a05e3-54dd-4c95-bfe0-7e4ddbab90a7.png)

 

STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR
1.	Create the root web directory for your_domain as follows: sudo mkdir /var/www/projectLEMP 
2.	Assign ownership of the directory with the $USER environment variable, which will reference current system user: sudo chown -R $USER:$USER /var/www/projectLEMP 
3.	Open a new configuration file in Nginx’s sites-available directory using preferred command-line editor. 
sudo vi /etc/nginx/sites-available/projectLEMP and paste the following configuration: (I had to correct the index.htm to index.html so it would work properly.)
 
 ![image](https://user-images.githubusercontent.com/120044190/217150256-9977bbf5-def2-4842-b7ae-3fd6a0ffdf89.png)

 
 
This is what each of the directives and location blocks do:
- listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.
- root — Defines the document root where the files served by this website are stored.
- index — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.
- server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.
- location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.
- location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the 
- php7.4-fpm.sock file, which declares what socket is associated with php-fpm.
- location ~ /\.ht — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.
4.	When done editing, save and close the file.
5.	Activated configuration by linking to the config file from Mginx’s sites-enabled directory: sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/  This will tell Nginx to use the configuration next time it is reloaded.
6.	Tested configuration for syntax errors by typing: sudo nginx -t
 
 ![image](https://user-images.githubusercontent.com/120044190/217150335-7e9d2cb5-9d4c-42f6-b3e6-5b5fc964d9e1.png)

 
 
7.	Disabled default Nginx host that was currently configured to listen on port 80: sudo unlink /etc/nginx/sites-enabled/default
8.	When ready, reloaded Nginx to apply the changes: sudo systemctl reload nginx
9.	The website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:

sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

10.	Go to browser and open website URL using IP address: http://<Public-IP-Address>:80

![image](https://user-images.githubusercontent.com/120044190/217150409-769dc33a-defc-428b-bc7a-5e0a04d04b5b.png)

 
Should be able to see the text from ‘echo’ command you wrote to index.html file, then it means your Nginx site is working as expected.
In the output we see the server’s public hostname (DNS name) and public IP address. You can also access your website in your browser by public DNS name, not only by IP – try it out, the result must be the same (port is optional)
http://<Public-DNS-Name>:80

STEP 5 – TESTING PHP WITH NGINX
LEMP stack is completely set up

1.	To test it to validate that Nginx can correctly hand .php files off to your PHP processor, create a test PHP file in your document root. Open a new file called info.php within your document root in text editor:
	
	sudo vi /var/www/projectLEMP/info.php
2.	Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:

	![image](https://user-images.githubusercontent.com/120044190/217152361-413ff469-fad8-4840-9117-fde2b72a97c1.png)


3.	Now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:
	
	
http://3.85.1.39/info.php

I see a web page containing detailed information about my server below:
 
![image](https://user-images.githubusercontent.com/120044190/217151134-18bffbaf-dc95-4745-8167-58061d0a1c93.png)


 ![image](https://user-images.githubusercontent.com/120044190/217151192-5481d4de-e2db-48e3-a1dd-08b653f16534.png)


 

4.	After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use rm to remove that file: sudo rm /var/www/projectLEMP/info.php

STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)
In this step I will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.

1.	Connected to the MySQL console using the root account: sudo mysql -p
2.	Created a new database, ran the following command from  MySQL console: mysql> CREATE DATABASE ‘college_database’ ;
3.	Created new user and granted full privileges on the database

The following command creates a new user named example_user, using mysql_native_password as default authentication method. We’re defining this user’s password as password, but you should replace this value with a secure password of your own choosing.

mysql> CREATE USER 'eunice_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
	
4.	Now give this user permission over the college_database database

mysql> GRANT ALL ON college_database.* TO 'eunice_user'@'%';  This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.

 ![image](https://user-images.githubusercontent.com/120044190/217151318-3d4d241c-3e69-4fe3-81cf-8f2ba45d8e97.png)



5.	Exit the MySQL shell: mysql> exit 
6.	Tested if new user has proper permissions by logging in to MySQL console again, using the custom user credentials:  mysql -u eunice_user -p

The -p flag in this command, will prompt you for the password used when creating the eunice_user user.

7.	After logging in to the MySQL console, confirmed that I have access to the example_database database: mysql> SHOW DATABASES;
		
![image](https://user-images.githubusercontent.com/120044190/217151353-853c66c5-233e-437d-869f-fc8858dc81a5.png)

 

8.	 Select database being used; run: USE college_database, then create table.

9.	Create a test table named todo_list. From the MySQL console, run the following statement:

CREATE TABLE college_database.todo_list (
    ->   item_id INT AUTO_INCREMENT,
    ->   content VARCHAR(255),
    ->   PRIMARY KEY(item_id));
I had to research the correct syntax for the table to be created.

 ![image](https://user-images.githubusercontent.com/120044190/217151440-40d01597-766b-4f9d-956d-fbc0d226b6df.png)


10.	Insert a few rows of content in the table

mysql> INSERT INTO college_database.todo_list (content) VALUES ("My first important item");

INSERT INTO college_database.todo_list (content) VALUES ("My second important item");

INSERT INTO college_database.todo_list (content) VALUES ("My third important item");

11.	To test that the data was successfully saved to  table, run: mysql> SELECT * FROM college_database.todo_list;

![image](https://user-images.githubusercontent.com/120044190/217151533-e1fb430e-7665-4c02-8fed-5ee581aae4d5.png)



12.	After confirming that there is valid data, exit MySQL console: mysql> exit
13.	Now you can create a PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor. We’ll use vi for that: sudo vi /var/www/projectLEMP/todo_list.php

14.	 The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.

Copy this content into your todo_list.php script:

<?php
$user = "eunice_user";
$password = "password";
$database = "college_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database",$user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
            }

![image](https://user-images.githubusercontent.com/120044190/217151648-62caa41e-99f4-4c59-a84f-8095a66d8708.png)

 

15.	Save and close the file once done editing.
16.	Now access this page in web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:

			http://3.85.1.39/todo_list.php

![image](https://user-images.githubusercontent.com/120044190/217151729-4b7ba217-f2c0-46a4-9d86-2da9d2adcd5a.png)


 
PHP environment is ready to connect and interact with MySQL server. I have built a a flexible foundation for serving PHP websites and applications to your visitors, using Nginx as web server and MySQL as database management system.
