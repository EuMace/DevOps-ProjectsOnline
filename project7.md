# DEVOPS TOOLING WEBSITE SOLUTION

Introducing a set of DevOps tools that will help the team in day to day activities in managing, developing, testing, deploying and monitoring different projects.
The tools we want our team to be able to use are well known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of:
1.	Jenkins – free and open source automation server used to build CI/CD pipelines.
2.	Kubernetes – an open-source container-orchestration system for automating computer application deployment, scaling, and management.
3.	Jfrog Artifactory – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
4.	Rancher – an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.
5.	Grafana – a multi-platform open source analytics and interactive visualization web application.
6.	Prometheus – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting 	 approach.
7.	Kibana – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.

## What we need for this project: 
•	Create 4 EC2 RHEL 8 instances for - 1 NFS, 3 webservers
	- Named--NFS, webserver1, webserver2, webserver3

•	Create 1 EC2 Ubuntu 20.04 for DB--name DB

## Step 1 – Prepare NFS Server
1. Spin up a new EC2 instance with RHEL Linux 8 OS.
2. Configure LVM on the server.
•	Format the disks as xfs
•	Ensure there are 3 Logical Volumes: lv-opt,  lv-apps, and lv-logs


Connect to NFS Server  using ssh connect info on terminal
Run `lsblk`
Run gdisk to create partitions  `sudo gdisk /dev/xvdf` (do for each volume) 
	---creating new GPT entries
	---command: n (new partition)
	---partition number: 1
	---first sector: press enter
	---last sector: press enter
	---hex code: 8300
	---command: p (to view partition)
	---command: w (to save)
----Repeat above steps for other remaining block devices--xvdg and xvdh

Run `lsblk` again
 
 ![image](https://user-images.githubusercontent.com/120044190/221267540-3d76a0ab-6428-4f21-b448-10653fbc7d3e.png)


Install lvm2 package by running command: `sudo yum install lvm2 -y` then run `sudo lvmdiskscan`
 
 ![image](https://user-images.githubusercontent.com/120044190/221267596-2be86290-e252-4508-b7d3-fc15747979a6.png)

 
Create physical volume to be used by LVM by using pvcreate command:
	`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

Check if the PV has been created successfully--Run: `sudo pvs`
 
![image](https://user-images.githubusercontent.com/120044190/221267655-7cf1e802-4a03-46ee-b5aa-c6585d099bdb.png)
 
 
Create the volume group and name it webdata-vg: `sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`
 
 ![image](https://user-images.githubusercontent.com/120044190/221267715-27385621-8929-4a6d-afa3-052d60fe8fa0.png)

 
View the newly created volume group: `sudo vgs`
 
 ![image](https://user-images.githubusercontent.com/120044190/221267781-5e9448d8-8e16-4aba-83e6-1a055b9eeb83.png)


Create 3 logical volumes using lvcreate utility. Name them: lv-apps for storing data for the webservers, lv-logs for storing data for webserver logs and lv-opt for Jenkins server in project 8

	`sudo lvcreate -n lv-apps -L 9G webdata-vg`
	`sudo lvcreate -n lv-logs -L 9G webdata-vg`
	`sudo lvcreate -n lv-opt -L 9G webdata-vg`
 
 ![image](https://user-images.githubusercontent.com/120044190/221267833-b62fd55e-d62b-42e2-a535-c6955fac72bb.png)


To verify the logical volume has been created successfully, run: `sudo lvs`

 ![image](https://user-images.githubusercontent.com/120044190/221267874-86cdc1ee-3c2f-49bc-a2df-5c5d6141e8a1.png)
 
Format the logical volumes with xfs:
	`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`
	`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`
	`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`
 
 ![image](https://user-images.githubusercontent.com/120044190/221267978-b0527f93-0d82-4d3c-8b66-101cc3a1678c.png)


3. Create mount points on /mnt directory for the logical volumes as follows:

Mount lv-apps on /mnt/apps – To be used by webservers
Mount lv-logs on /mnt/logs – To be used by webserver logs
Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8  

Create mount point for the logical volumes on /mnt directory:
	
	`sudo mkdir /mnt/apps`
	`sudo mkdir /mnt/logs`
	`sudo mkdir /mnt/opt`
	

Mount to /dev/webdata-vg/lv-apps /dev/webdata-vg/lv-logs and /dev/webdata-vg/lv-opt:

	`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`
	`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`
	`sudo mount /dev/webdata-vg/lv-opt /mnt/opt`
 
 ![image](https://user-images.githubusercontent.com/120044190/221268043-cc6a9d6f-d883-4ba6-8c88-72ea96ac8fff.png)


4. Install NFS server, configure it to start on reboot and make sure it is up and running

	`sudo yum -y update`

	`sudo yum install nfs-utils -y`
	
	`sudo systemctl start nfs-server.service`
	
	`sudo systemctl enable nfs-server.service`
	
	`sudo systemctl status nfs-server.service`
 
 ![image](https://user-images.githubusercontent.com/120044190/221268114-7197c821-460b-4937-8082-db97f227d88e.png)

 
5. Export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, you will install all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security.
To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:
 
![image](https://user-images.githubusercontent.com/120044190/221268180-ee8f224e-4c2c-47f6-a63f-b6dcf1c1f242.png)


6. Set up permission that will allow our Web servers to read, write and execute files on NFS:
`sudo chown -R nobody: /mnt/apps`
	`sudo chown -R nobody: /mnt/logs`
	`sudo chown -R nobody: /mnt/opt`

	`sudo chmod -R 777 /mnt/apps`
	`sudo chmod -R 777 /mnt/logs`
	`sudo chmod -R 777 /mnt/opt`

`sudo systemctl restart nfs-server.service`
 
 ![image](https://user-images.githubusercontent.com/120044190/221268299-8bb6562b-a6ea-410f-afbf-41a0baa96eb1.png)

 
7. Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.48.0/20 ):
`sudo vi /etc/exports`

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

`sudo exportfs -arv`
 
![image](https://user-images.githubusercontent.com/120044190/221268463-021e0bc3-3bb9-463a-895c-68e828783f59.png)
	

8. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

	rpcinfo -p | grep nfs

Important to note: In order for the NFS server to be accessible from your client, you must additionally open the following ports: TCP 111, UDP 111, UDP 2049, NFS 2049
	
 
 ![image](https://user-images.githubusercontent.com/120044190/221268571-19fdf7ab-37ef-4661-be2c-43ad3dd470fc.png)
	
	
	
 ![image](https://user-images.githubusercontent.com/120044190/221268813-c493e4b6-e214-4638-9b55-1309a85d9e45.png)


## Step 2— Configure Database Server  

1. Install and configure a MySQL DBMS to work with remote Web Server
2. SSH in to the provisioned DB server and run an update on the server: `sudo apt update`
3. Install mysql-server: `sudo apt install mysql-server -y`
4. Create a database and name it tooling:

	`sudo mysql`
	`create database tooling;`
	
Create a database user and name it webaccess and grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr:
-go to AWS to Webserver 1--->networking--->Subnet ID--->find IPv4 CIDR: 172.31.48.0/20

	create user 'webaccess'@'172.31.48.0/20' identified by 'password';
	grant all privileges on tooling.* to 'webaccess'@'172.31.48.0/20';
	flush privileges;
 
![image](https://user-images.githubusercontent.com/120044190/221268896-e0064443-72c6-4f02-829a-52c941313147.png)
	

![image](https://user-images.githubusercontent.com/120044190/221268919-745b95c8-a9b8-4770-a8b3-bfadfdcc1cd8.png)


## Step 3—Prepare the Webservers

1. Launch a new EC2 instance with RHEL 8 Operating System—I used RHEL 9

2. Install NFS client on the webserver1: `sudo yum install nfs-utils nfs4-acl-tools -y`
	
	
![image](https://user-images.githubusercontent.com/120044190/221269102-0710c623-a7b0-455f-a5ad-86e3747b28ac.png)

 
3. Mount /var/www/ and target the NFS server’s export for apps (Use the private IP of the NFS server)

	`sudo mkdir /var/www`
	
	`sudo mount -t nfs -o rw,nosuid 172.31.51.176:/mnt/apps /var/www`
	
4. Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot:
	
	`sudo vi /etc/fstab`
	
add following line:
	
172.31.51.176:/mnt/apps /var/www nfs defaults 0 0

5. INSTALL Remi’s repository, APACHE, and PHP---------Since I used RHEL 9, I had to update my system so I can run my commands properly. (Update for RHEL 9 https://techviewleo.com/enable-epel-remi-repos-rocky-linux/ )
	
> `sudo yum install httpd -y` (install Apache)
	
`sudo dnf update -y`
	
`sudo dnf upgrade --refresh -y`

`sudo subscription-manager repos --enable codeready-builder-for-rhel-9-$(arch)-rpms`
	
`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm`
	
`sudo dnf update`
	
`sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm`
	
`sudo yum repolist`
	
`sudo dnf --disablerepo="*" --enablerepo="remi" list available`
	
`sudo dnf --disablerepo="*" --enablerepo="remi-safe" list available` 
	
`sudo dnf module list php`
	
*****sudo dnf module install php:remi-8.1
sudo dnf module enable php:remi-8.1 -y ( you can also enable other PHP modules: 8.0 or 7.4) To disable php remi-8.1. sudo dnf module disable  php:remi-8.1 -y

To enable or disable a given remi repository permanently, we need to edit the /etc/yum.repos.d/remi.repo with the text editor. Scroll down through the file to the
	
line labeled enabled=0.
	
The ‘0’ value denotes a given repository is disabled. Changing the value to ‘1’ enables a given repository.
	
`sudo vi /etc/yum.repos.d/remi.repo`

`sudo dnf update` 
	
`sudo dnf repolist

CONTINUE with rest of install

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`
	
`sudo systemctl start php-fpm`
	
`sudo systemctl enable php-fpm`
	
`sudo setsebool -P httpd_execmem 1`

![image](https://user-images.githubusercontent.com/120044190/221270277-d1630ff2-ee32-4de6-b4c8-7f68220244d6.png)

 
6. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files – it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.  

`lls /mnt/apps` on NFS and `ls /var/www` on webservers

7. Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №3 and №4 to make sure the mount point will persist after reboot:

`sudo mkdir /var/www`

	`sudo mount -t nfs -o rw,nosuid 172.31.51.176:/mnt/logs /var/log/httpd`
	
       	`sudo vi /etc/fstab`
	
	And enter:
	
	172.31.51.176:/mnt/logs /var/log/httpd nfs defaults 0 0


 ![image](https://user-images.githubusercontent.com/120044190/221270398-caa2416f-9d87-46a1-a0a1-cf61aeadac53.png)



8. Fork the tooling source code from Darey.io Github Account to your Github account. 
	
	 `sudo yum install git -y`
	
	 `git init`
	
	 `git clone https://github.com/darey-io/tooling.git`
	
	![image](https://user-images.githubusercontent.com/120044190/221270451-b5731dd5-c603-4991-9a6d-531155f3e402.png)

 
9. Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html
	
	`cd tooling`
	
	`ls`
	
	`ls /var/www`
	
	`sudo cp -R html/. /var/www/html`
	
	then `ls /var/www/html`
	
	`ls html`
	
Note 1: Open port 80 on webserver1--go to inbound rules in security group
Note 2: If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0
To make this change permanent – open following config file `sudo vi /etc/sysconfig/selinux` and set SELINUX=disabledthen restrt httpd.
	
Check to see it Apache running: `sudo systemctl status httpd`----if inactive `cd ..` and `sudo setenforce 0`
	
THEN go to sudo vi /etc/sysconfig/selinux and set SELINUX=disabled then restart httpd---sudo systemctl start httpd
	

![image](https://user-images.githubusercontent.com/120044190/221270673-bf060e09-93f1-48e8-a2a7-38ed03530dec.png)

 
10. Update the website’s configuration to connect to the database: `sudo vi /var/www/html/functions.php`

ENTER $db = mysqli_connect('172.31.14.247', 'webaccess', 'password', 'tooling');
	
	
![image](https://user-images.githubusercontent.com/120044190/221270738-471a54dd-60e0-48e0-ad1b-285575eb9a11.png)

 
Apply tooling-db.sql script to your database using this command:
	
 `mysql -h 172.31.14.247 -u webaccess -p tooling  < tooling-db.sql` 
								   
(if it doesnt work, you need to install mysql--->`sudo yum install mysql`)  
	
----cd tooling and enter command again

REMEMBER to go to inbound rules for DB server and add MYSQL/Aurora  TCP 3306 and subnet IPv4 172.31.48.0/20

Go to database server and the bind address to 0.0.0.0 by entering: 
		
`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf` THEN restart `sudo systemctl restart mysql`

11.	Go back to DB server and run `sudo mysql` and `show databases`

	use tooling;
	
	show tables;
	
	select * from users;

To backup welcome page on Apache:  `sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.backup` THEN `sudo systemctl restart httpd`
	
 
![image](https://user-images.githubusercontent.com/120044190/221270925-8ecddfd5-42f9-4df4-adfe-a1b905058606.png)
	
	

![image](https://user-images.githubusercontent.com/120044190/221270952-fae1df85-49d8-4007-81a9-3ce5abb0dd28.png)

 
12. Create in MySQL a new admin user with username: myuser and password: password:
	
	INSERT INTO users(id, username, password, email, user_type, status)
	--> VALUES ("5", "myuser", "21232f297a57a5a743894a0e4a801fc3", "eunicedeola@gmail.com", "admin", "5");

LOGIN with myuser and password is admin

![image](https://user-images.githubusercontent.com/120044190/221271057-71a11d21-23aa-4fa5-a8d6-68e5330b2887.png)
	

![image](https://user-images.githubusercontent.com/120044190/221271092-1515de4a-06a5-4d7d-98e3-ee8865ecf537.png)
	

![image](https://user-images.githubusercontent.com/120044190/221271111-cdf84306-aaa5-488b-a3d4-385c00804578.png)

 



 

