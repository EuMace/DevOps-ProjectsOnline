# WEB SOLUTION WITH WORDPRESS 

In this project I was tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS)

## Step 1 — Prepare a Web Server
1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes   in the same AZ as your Web Server EC2, each of 10 GiB.

![image](https://user-images.githubusercontent.com/120044190/220681293-c2974662-875e-43f1-b897-32f6b323bd1c.png)


 

2.	Attach all three volumes one by one to your Web Server EC2 instance.

 ![image](https://user-images.githubusercontent.com/120044190/220681412-9eb256f8-c7a1-4615-be62-f7d08f384673.png)


3.	Open up the Linux terminal to begin configuration.
4.	Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/         directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

![image](https://user-images.githubusercontent.com/120044190/220681832-7c8c3497-7a85-4ec7-8005-4bf5591b8c86.png)

 
![image](https://user-images.githubusercontent.com/120044190/220681881-fb813152-3339-4565-9fc0-388628fed952.png)

 

5.	Use `df -h` command to see all mounts and free space on your server
6.	Use gdisk utility to create a single partition on each of the 3 disks
        `sudo gdisk /dev/xvdf`

 
![image](https://user-images.githubusercontent.com/120044190/220682093-af079d7b-6f89-49d8-9f3b-0eb604ab9823.png)



7.	Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

![image](https://user-images.githubusercontent.com/120044190/220682558-45a4bb53-3822-42f6-83a9-4a17dd9020ea.png)

 
8.	Install lvm2 package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.

 
![image](https://user-images.githubusercontent.com/120044190/220682921-2e518e3e-bd06-424b-9055-94ba220a2c75.png)

 ![image](https://user-images.githubusercontent.com/120044190/220683234-c13f71f5-b9e0-4287-b6fb-79ce24bb4371.png)


9.	Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
        `sudo pvcreate /dev/xvdf1`
        `sudo pvcreate /dev/xvdg1`
        `sudo pvcreate /dev/xvdh1`

 
![image](https://user-images.githubusercontent.com/120044190/220683492-f7b0b17b-66e1-410a-926e-6195bca5e7a9.png)





10.	Verify that your Physical volume has been created successfully by running `sudo pvs`

 ![image](https://user-images.githubusercontent.com/120044190/220683671-381eb7c8-759b-48b0-8ef7-07a90f24573b.png)


11.	Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
        `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
	
12.	Verify that your VG has been created successfully by running `sudo vgs`

![image](https://user-images.githubusercontent.com/120044190/220683948-81c70e9e-f822-490a-9cde-69026ef8ff0c.png)


 
13.	Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will   	      be used to store data for the Website while, logs-lv will be used to store data for logs.

       ` sudo lvcreate -n apps-lv -L 14G webdata-vg`
       
       `sudo lvcreate -n logs-lv -L 14G webdata-vg`

14.	Verify that your Logical Volume has been created successfully by running `sudo lvs`

 ![image](https://user-images.githubusercontent.com/120044190/220684500-9dd36642-4e85-4e22-a849-ffcb099f35d3.png)


15.	Verify the entire setup using command:

   `sudo vgdisplay -v #view complete setup - VG, PV, and LV`
   `sudo lsblk`

16.	Use mkfs.ext4 to format the logical volumes with ext4 filesystem.

    `sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`
    `sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![image](https://user-images.githubusercontent.com/120044190/220684583-754feeec-0576-446e-be03-50016acc6342.png)

 
17.	Create /var/www/html directory to store website files
       `sudo mkdir -p /var/www/html`

18.	Create /home/recovery/logs to store backup of log data
      `sudo mkdir -p /home/recovery/logs`

19.	Mount /var/www/html on apps-lv logical volume

       `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

 ![image](https://user-images.githubusercontent.com/120044190/220684951-f6d79dc2-d9b8-441c-bd0b-ecfdca7be4fe.png)


![image](https://user-images.githubusercontent.com/120044190/220685049-3467798e-9107-4d3c-9dff-29be239fbb60.png)


 
20.	Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`

21.	Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

22.	Restore log files back into /var/log directory

`sudo rsync -av /home/recovery/logs/. /var/log`

23.	Update /etc/fstab file so that the mount configuration will persist after restart of the server.

UPDATE THE `/ETC/FSTAB` FILE

The UUID of the device will be used to update the /etc/fstab file;
     `sudo blkid`
     
     `sudo vi /etc/fstab`

Update /etc/fstab in this format using your own UUID and remember to remove the leading and ending quotes.

 ![image](https://user-images.githubusercontent.com/120044190/220685403-979a94fb-f765-4c39-9177-bd0add8fa170.png)


1.	Test the configuration and reload the daemon
2.	 `sudo mount -a`
3.	 
         `sudo systemctl daemon-reload`
	 
3.	Verify your setup by running `df -h`, output must look like this:

 ![image](https://user-images.githubusercontent.com/120044190/220685569-3f1c05b7-7aa0-4624-9ee1-eaec9619b2f7.png)


## Step 2 - Prepare the Database server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.
 
![image](https://user-images.githubusercontent.com/120044190/220685660-e9b3b701-7ef3-4d25-b1ab-9694061a31df.png)

 
![image](https://user-images.githubusercontent.com/120044190/220685703-faef92a1-6322-4a3f-8642-686633b1eafb.png)


## Step 3 - Install Wordpress on Web Server EC2
1.	Update the repository `sudo yum -y update`
2.	Install wget, Apache and its dependencies
3.	
        `Sudo yum -y install wget heetpd php php-mysqlnd php-fpm php-json`
	
3.	Start Apache
	`sudo systemctl enable httpd`
	
        `sudo systemctl start httpd`

![image](https://user-images.githubusercontent.com/120044190/220685898-79e619b3-f94d-473b-83bb-5fe37dec47e3.png)

 
4.	To install PHP and it’s dependencies—I had to troubleshoot here for RHEL9 updates https://techviewleo.com/enable-epel-remi-repos-rocky-linux/ 

       `sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm`
       
       `sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`
       
       `sudo yum module list php`
       
       `sudo yum module reset php`
       
       `sudo yum module enable php:remi-8.1`
       
       `sudo yum install php php-opcache php-gd php-curl php-mysqlnd`
       
       `sudo systemctl start php-fpm`
       
       `sudo systemctl enable php-fpm`
       
       `setsebool -P httpd_execmem 1`

5.	Restart Apache

       `sudo systemctl restart httpd`

6.	Download wordpress and copy wordpress to var/www/html

       `mkdir wordpress`
       
       `cd   wordpress`
       
       ` sudo wget http://wordpress.org/latest.tar.gz`
       
       ` sudo tar xzvf latest.tar.gz`
       
       ` sudo rm -rf latest.tar.gz`
       
       ` cp wordpress/wp-config-sample.php wordpress/wp-config.php`
       
       `cp -R wordpress /var/www/html/`

7.	Configure SELinux Policies
	
       ` sudo chown -R apache:apache /var/www/html/wordpress`
       
       ` sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`
       
       `sudo setsebool -P httpd_can_network_connect=1`
       
       `sudo setsebool -P httpd_can_network_connect db 1`

 
![image](https://user-images.githubusercontent.com/120044190/220687707-ad371c74-04c1-4913-bcb6-23a519773e07.png)

 ![image](https://user-images.githubusercontent.com/120044190/220687883-d339528b-7f5e-495a-bafc-b541861fbfeb.png)

 ![image](https://user-images.githubusercontent.com/120044190/220688069-eb61b349-6600-48f4-8efe-ea5f713ca685.png)

![image](https://user-images.githubusercontent.com/120044190/220688122-0df27546-1c62-4f66-ae6f-28d23f711644.png)


 
## Step 4 — Install MySQL on your DB Server EC2

      `sudo yum update`
      
      `sudo yum install mysql-server`
      
Verify that the service is up and running by using` sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:

     `sudo systemctl restart mysqld`
     
     `sudo systemctl enable mysqld`

![image](https://user-images.githubusercontent.com/120044190/220688325-5843fac6-6beb-45cb-991e-112acbf23c7d.png)

 
## Step 5 — Configure DB to work with WordPress

    `Sudo mysql_secure_installation on DB server  sudo mysql -u root -p`
    
    `sudo mysql`
    
    `CREATE DATABASE wordpress1;`
    
    `CREATE USER `myuser1`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass1';`
    
    `GRANT ALL ON wordpress.* TO 'myuser1'@'<Web-Server-Private-IP-Address>';`
    
    `FLUSH PRIVILEGES;`
    
    `SHOW DATABASES;`
    
    `exit`

![image](https://user-images.githubusercontent.com/120044190/220688704-7d9ee662-36ed-4b94-9890-e5d2e5bdd650.png)

![image](https://user-images.githubusercontent.com/120044190/220688780-582336c0-13e3-48ca-bceb-6c8118f024fc.png)

 ![image](https://user-images.githubusercontent.com/120044190/220688843-227c77f7-8117-4009-b40d-edeb5d73d068.png)

 
 

## Step 6 — Configure WordPress to connect to remote database.

Open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

 ![image](https://user-images.githubusercontent.com/120044190/220688962-d13f0b57-116a-4ae3-97fa-a276bd4415ba.png)


1.	Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
2.	
        `sudo yum install mysql
      
       `sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`
	
3.	Verify if you can successfully execute `SHOW DATABASES;` command and see a list of existing databases.
	
4.	Change permissions and configuration so Apache could use WordPress:

5.	Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)
	
6.	Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/
 
	![image](https://user-images.githubusercontent.com/120044190/220689555-62bf57ac-d7c3-40b6-b990-18126696a314.png)
	

Fill out your DB credentials:
 
![image](https://user-images.githubusercontent.com/120044190/220689667-50104a1c-e6f3-4a5a-894d-68a166eccca7.png)

	

If you see this message – it means your WordPress has successfully connected to your remote MySQL database
 
![image](https://user-images.githubusercontent.com/120044190/220689720-252b73bc-4119-40a8-ab07-963bda397359.png)

 ![image](https://user-images.githubusercontent.com/120044190/220689769-70914436-6c88-4667-b49b-c9ef8e18de0d.png)

