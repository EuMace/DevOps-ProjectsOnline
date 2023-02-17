# IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL 
DATABASE MANAGEMENT SYSTEM (DBMS).

1.	Create and configure two Linux-based virtual servers (EC2 instances in AWS)

 

2.	On mysql server Linux Server install MySQL Server software.

`sudo apt update -y`
`sudo apt install mysql-server -y`
`sudo systemctl enable mysql`
 

Install MySQL Server software
 
3.	On mysql client Linux Server install MySQL Client software.

`sudo apt update -y`
`sudo apt install mysql-client -y`

 

4.	By default, both of the EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. 

MySQL server uses TCP port 3306 by default, so I have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’.



 
 
5.	When installation is finished, log in to the MySql console with command: $ `sudo mysql`
6.	Run security script that comes pre-installed with MySql: `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`
7.	Then exit MySQL shell with: mysql> exit
8.	Start the interactive script with command $ `sudo mysql_secure_installation` and continue with instructions with changed password.
9.	After finishing, test if able to log in to the MySQL console with command : `sudo mysql -p`
10.	MySQL server is now installed.

 
11.	Next, create the remote user with this following command: `CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`
12.	Create database with: `CREATE DATABASE test_db;`
13.	Then grant privieges to remote_user: `GRANT ALL ON test_db.* TO 'remote_user'@'%' WITH GRANT OPTION;`
14.	. Finally, flush privileges and exit mysql : `FLUSH PRIVILEGES;` 


 

15.	Having created the user and database, configure MySQL server to allow connections from remote hosts. Use the following command: `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
16.	In the text editor, replace the old Bind-address from ‘127.0.0.1’ to ‘0.0.0.0’ then save and exit.
17.	Next, I restart mysql with: `sudo systemctl restart mysql`

 

18.	From mysql client connect remotely to mysql server Database Engine without using SSH. 
Using the mysql utility to perform this action type: `sudo mysql -u remote_user -h 172.31.95.205 -p` and enter PassWord.1 for user password
 


19.	This gives us access into the mysql server database engine.
20.	Finally type: `Show databases;` to show the test_db database that was created.


