# MEAN STACK Deployment to Ubuntu in AWS


In this project I implemented a simple Book Register web form using MEAN stack with a new EC2 Instance of t2.nano family with an Ubuntu server 20.04 LTS (HVM) image.

## Step 1: Install NodeJs

1.	Update ubuntu---  `sudo apt update`
2.	Upgrade ubuntu---`sudo apt upgrade`
3.	Add certificates--- `sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates` 

`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash –`


![image](https://user-images.githubusercontent.com/120044190/218839089-00d91693-10f3-4016-af02-c76621397e29.png)


 
4.	Install NodeJS--- `sudo apt-get install -y nodejs` 

![image](https://user-images.githubusercontent.com/120044190/218839219-b449e3db-dd32-4cad-a848-49ceea74851d.png)



5.	Verify installation with--- `node -v`

6.	Verify node installation for npm using : `npm -v`


## Step 2: Install MongoDB

1.	Add MongoDB key server using : `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`
2.	The add repository: `echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-    3.4.list`


![image](https://user-images.githubusercontent.com/120044190/218839393-e3326d01-876d-45ad-aee1-a87ada0e624f.png)

 
3.	Install MongoDB--- `sudo apt install -y mongodb`
4.	Start the server----- `sudo service mongodb start`
5.	Verify that the service is up and running by using this command---- `sudo systemctl status mongodb`

![image](https://user-images.githubusercontent.com/120044190/218839470-9c4e4dd9-5c0d-4982-aeaa-59a355a50ae7.png)

 
6.	Install npm –Node package manager--- `sudo apt install -y npm` 
7.	Install body-parser package -- `sudo npm install body-parser`

![image](https://user-images.githubusercontent.com/120044190/218839538-018291a9-95ae-4bc6-a9bb-74795e17a5c1.png)

 
8.	Once completed, create a folder called ‘Books’ and go into it ---- `mkdir Books && cd Books`
9.	In the Books directory, initialize npm project--- `npm init`  and add a file to it named server.js—vi server.js  Use command: `npm init` to initialize project, so new file named package.json is created. Follow the prompts after running the command. You can press Enter several times to accept default values, then accept to write out the package.json file by typing yes.
 
 ![image](https://user-images.githubusercontent.com/120044190/218839611-43d5d954-b43a-4ccc-be8d-93b20b8b4c14.png)


# INSTALL EXPRESS AND SET UP ROUTES TO THE SERVER

## Step 3: Install Express and set up routes to the server

1. -- Express will used to pass book information to and from our MongoDB database.
-- Mongoose will be used to establish a schema for the database to store data of our book register
`sudo npm install express mongoose`

![image](https://user-images.githubusercontent.com/120044190/218839662-16061746-f5bf-4fb1-b1d1-0146ffe68d72.png)


 
2. In ‘Books’ folder, create a folder named apps and go into it using command: `mkdir apps && cd apps` 
3. Then create a file names routes.js---- `vi routes.js`
4. Copy and paste the following code below into routes.js

```
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
```

![image](https://user-images.githubusercontent.com/120044190/218839730-4c61b5b7-bdd2-4869-aea3-3ce91ed5d45e.png)

 
8.	Within the ‘apps’ folder, crate a folder names models – `mkdir models && cd models` 
9.	Create a file named book.js in folder models--- `vi book.js`
10.	Copy and paste the following code into book.js

```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```

![image](https://user-images.githubusercontent.com/120044190/218840624-f3d5a3f6-3eae-4eec-aac3-bc5c3906fe05.png)

 

## Step 4 – Access the routes with AngularJS

AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.

1.	Change the directory back to ‘Books’  `cd ../..`
2.	Create a folder named public ---- `mkdir public && cd public`
3.	Then add a file named script.js --- `vi script.js`
4.	Add the following code to the file:
 
 ```
 var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```

![image](https://user-images.githubusercontent.com/120044190/218841272-ef512d57-07e5-46a6-b243-688fe6b4e7a9.png)


5.	Also in public folder, create a file named index.html --- `vi index.html`
6.	Copy and paste following code below into index.html file:

```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```
  
![image](https://user-images.githubusercontent.com/120044190/218845904-5572742f-5083-4f78-ae42-714844fa40d1.png)

  

7.	Change back to ‘Books’ directory  `cd ..`
8.	Start the server by running command: `node server.js`

![image](https://user-images.githubusercontent.com/120044190/218842161-a724f9eb-b42b-4cab-bd5b-65c31bf01df1.png)

 
Received error and had to troubleshoot the issue, and found that the version for nodejs was not latest version. I updated to latest version 18 and ran the command again for node server.js
 
 ![image](https://user-images.githubusercontent.com/120044190/218842405-2bcdc548-0471-4651-8361-8d480b2a77e5.png)


9.	The server is now up and running. To connect to it within a separate SSH console I used : `curl -s http://localhost:3300`

And I received an html page in CLI:

![image](https://user-images.githubusercontent.com/120044190/218842599-d1d5a274-f87b-4b00-be2b-f35fa6c8555a.png)

 

10.	To access from the internet, I opened port 3300 in AWS EC2 instance.
11.	After opening port, I used the Public IP address and port to use on internet and received the following image http://3.86.86.61:3300:
 
 ![image](https://user-images.githubusercontent.com/120044190/218842792-64c61630-d5fb-49ad-8f00-d2ae435a80b8.png)

 
I also tried to enter information in the Book Register and received following results:
 
![image](https://user-images.githubusercontent.com/120044190/218842899-4b2582d9-a38c-42ec-8eb5-d542d1003fd8.png)

