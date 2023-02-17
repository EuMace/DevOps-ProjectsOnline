# IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL 
DATABASE MANAGEMENT SYSTEM (DBMS).

1.	Create and configure two Linux-based virtual servers (EC2 instances in AWS)

 ![image](https://user-images.githubusercontent.com/120044190/219702361-77d0d7fe-3983-4f6f-b306-a78ffbd86688.png)


2.	On mysql server Linux Server install MySQL Server software.

`sudo apt update -y`
`sudo apt install mysql-server -y`
`sudo systemctl enable mysql`
 
 ![image](https://user-images.githubusercontent.com/120044190/219702441-17316dc8-d3e9-43ff-8929-94676a17a8eb.png)


Install MySQL Server software

![image](https://user-images.githubusercontent.com/120044190/219702494-4d619849-4b76-4915-8f1a-68e37a82cf3b.png)

 
3.	On mysql client Linux Server install MySQL Client software.

`sudo apt update -y`
`sudo apt install mysql-client -y`

 ![image](https://user-images.githubusercontent.com/120044190/219702566-7d5dd554-7a5f-4425-9f08-d6c622c33462.png)


4.	By default, both of the EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. 

MySQL server uses TCP port 3306 by default, so I have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’.

![image](https://user-images.githubusercontent.com/120044190/219702612-e17bedbd-49aa-4565-85ff-7554a841f0ea.png)

![image](https://user-images.githubusercontent.com/120044190/219702649-14d64a65-a726-40fd-a27b-be6c9bee1824.png)

 
5.	When installation is finished, log in to the MySql console with command: $ `sudo mysql`
6.	Run security script that comes pre-installed with MySql: `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`
7.	Then exit MySQL shell with: mysql> exit
8.	Start the interactive script with command $ `sudo mysql_secure_installation` and continue with instructions with changed password.
9.	After finishing, test if able to log in to the MySQL console with command : `sudo mysql -p`
10.	MySQL server is now installed.

![image](https://user-images.githubusercontent.com/120044190/219702736-bbcc4f66-bf3d-45e7-b1f2-75301bd8e9b4.png)

 
11.	Next, create the remote user with this following command: `CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`
12.	Create database with: `CREATE DATABASE test_db;`
13.	Then grant privieges to remote_user: `GRANT ALL ON test_db.* TO 'remote_user'@'%' WITH GRANT OPTION;`
14.	. Finally, flush privileges and exit mysql : `FLUSH PRIVILEGES;` 

![image](https://user-images.githubusercontent.com/120044190/219702780-026d0fda-edd7-4ec6-81ca-14720b860465.png)
 

15.	Having created the user and database, configure MySQL server to allow connections from remote hosts. Use the following command: `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
16.	In the text editor, replace the old Bind-address from ‘127.0.0.1’ to ‘0.0.0.0’ then save and exit.
17.	Next, I restart mysql with: `sudo systemctl restart mysql`

 ![image](https://user-images.githubusercontent.com/120044190/219702852-4af3bb7a-b961-40e1-ba8a-6795e8790333.png)


18.	From mysql client connect remotely to mysql server Database Engine without using SSH. 
Using the mysql utility to perform this action type: `sudo mysql -u remote_user -h 172.31.95.205 -p` and enter PassWord.1 for user password
 
![image](https://user-images.githubusercontent.com/120044190/219702915-2dcab193-50cd-4502-929c-3cdde3ecff3a.png)


19.	This gives us access into the mysql server database engine.
20.	Finally type: `Show databases;` to show the test_db database that was created.

![image](https://user-images.githubusercontent.com/120044190/219702980-df128b54-3418-446f-9a89-c2162cfbc9d9.png)



