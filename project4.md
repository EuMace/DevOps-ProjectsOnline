## H2 MEAN STACK Deployment to Ubuntu in AWS
MEAN STACK Deployment to Ubuntu in AWS
In this project I implemented a simple Book Register web form using MEAN stack.
Step 1: Install NodeJs
1.	Update ubuntu---sudo apt update
2.	Upgrade ubuntu---sudo apt upgrade
3.	Add certificates--- sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates 

curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash –

 
4.	Install NodeJS--- sudo apt-get install -y nodejs 
5.	 

6.	Verify installation with--- node -v

7.	Verify node installation for npm using : npm -v
Step 2: Install MongoDB
1.	Add MongoDB key server using : sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
2.	The add repository: echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
 
3.	Install MongoDB--- sudo apt install -y mongodb
4.	Start the server----- sudo service mongodb start
5.	Verify that the service is up and running by using this command----sudo systemctl status mongodb
 
6.	Install npm –Node package manager---sudo apt install -y npm 
7.	Install body-parser package -- sudo npm install body-parser
 
8.	Once completed, create a folder called ‘Books’ and go into it ----mkdir Books && cd Books
9.	In the Books directory, initialize npm project--- npm init  and add a file to it named server.js—vi server.js  Use command: npm init to initialize project, so new file named package.json is created. Follow the prompts after running the command. You can press Enter several times to accept default values, then accept to write out the package.json file by typing yes.
 

INSTALL EXPRESS AND SET UP ROUTES TO THE SERVER
Step 3: Install Express and set up routes to the server
1. -- Express will used to pass book information to and from our MongoDB database.
-- Mongoose will be used to establish a schema for the database to store data of our book register
sudo npm install express mongoose

 
2. In ‘Books’ folder, create a folder named apps and go into it using command: mkdir apps && cd apps 
3. Then create a file names routes.js----vi routes.js
4. Copy and paste the following code below into routes.js
 
8.	Within the ‘apps’ folder, crate a folder names models – mkdir models && cd models 
9.	Create a file named book.js in folder models--- vi book.js
10.	Copy and paste the following code into book.js
 

Step 4 – Access the routes with AngularJS
AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.
1.	Change the directory back to ‘Books’  cd ../..
2.	Create a folder named public ---- mkdir public && cd public
3.	Then add a file named script.js ---vi script.js
4.	Add the following code to the file:
 

5.	Also in public folder, create a file named index.html ---vi index.html
6.	Copy and paste following code below into index.html file:
  

7.	Change back to ‘Books’ directory  cd ..
8.	Start the server by running command: node server.js
 
Received error and had to troubleshoot the issue, and found that the version for nodejs was not latest version. I updated to latest version 18 and ran the command again for node server.js
 

9.	The server is now up and running. To connect to it within a separate SSH console I used : curl -s http://localhost:3300

And I received an html page in CLI:

 

10.	To access from the internet, I opened port 3300 in AWS EC2 instance.
11.	After opening port, I used the Public IP address and port to use on internet and received the following image http://3.86.86.61:3300:
 
I also tried to enter information in the Book Register and received following results:
 

