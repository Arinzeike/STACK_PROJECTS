# MEAN STACK DEPLOYMENT TO UBUNTU IN AWS
 
**MEAN Stack** is a combination of the following components:
* MongoDB (Document database) – Stores and allows retrieval of data.
* Express (Back-end application framework) – Makes requests to Database for Reads and Writes.
* Angular (Front-end application framework) – Handles Client and Server Requests
* Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user

## Step 0 – Preparing prerequisites
- A virtual server with Ubuntu Server OS.
- SSH Client.
- Knowledge of Security Groups.
![Alt text](<images/ec2 instance.png>)
![Alt text](<images/putty working.png>)
In this project, I implemented a simple Book Register web form using MEAN stack.

## Step 1: Install NodeJs <br>
Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this project to set up the Express routes and AngularJS controllers.

- Update Ubuntu
`sudo apt update`
![Alt text](<images/sudo update.png>)

- Upgrade ubuntu
`sudo apt upgrade`
![Alt text](<images/sudo upgrade.png>)


- Add certificates
`sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`
![Alt text](images/certificate.png)


- Get NodeSource from Ubuntu repoeitories
`curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -`

- Install NodeJS <br>
`sudo apt install -y nodejs`
![Alt text](<images/install node js.png>)

- To be able to compile native addons from npm you’ll need to install the development tools: <br>
`sudo apt install build-essential`

- Install npm – Node package manager.
`sudo apt install -y npm`
![Alt text](<images/install npm.png>)
## Step 2: Install MongoDB
MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. For this application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

- Install MongoDB <br>
`sudo apt install -y mongodb`
![Alt text](<images/install mongodb.png>)
- Start The server
`sudo service mongodb start` <br>

- Verify that the service is up and running <br>
`sudo systemctl status mongodb`


- Install body-parser package
We need ‘body-parser’ package to help us process JSON files passed in requests to the server <br>
`sudo npm install body-parser`

- Create a folder named ‘Books’
`mkdir Books && cd Books`
- In the Books directory, Initialize npm project <br>
`npm init`


 
- Add a file to it named server.js `vi server.js`

Copy and paste the web server code below into the server.js file.
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```

## STEP 3 -INSTALL EXPRESS AND SET UP ROUTES TO THE SERVER
**Install Express and set up routes to the server** <br>
Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express to pass book information to and from our MongoDB database.

We also used Mongoose package which provides a straightforward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.

- Install Express and Mongoose:
`sudo npm install express mongoose`
![Alt text](<images/install express mongooses.png>)
- In ‘Books’ folder, create a folder named apps
`mkdir apps && cd apps`
Create a file named routes.js
`vi routes.js`
 
Copy and paste the code below into routes.js
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



- In the ‘apps’ folder, create a folder named models
`mkdir models && cd models`

- Create a file named book.js
`vi book.js`
Copy and paste the code below into ‘book.js’
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


## Step 4 – Access the routes with AngularJS <br>
AngularJS provides a web framework for creating dynamic views in your web applications. In this project, we used AngularJS to connect our web page with Express and perform actions on our book register.

- Change the directory back to ‘Books’
`cd ../..`

- Create a folder named public
`mkdir public && cd public`
- Add a file named script.js
`vi script.js`

Copy and paste the Code below (controller configuration defined) into the script.js file.
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



- In the public folder, create a file named index.html;
`vi index.html`
 
Copy and paste the code below into index.html file.
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



- Change the directory back up to Books
`cd ..`

- Start the server by running this command: <br>
`node server.js` <br>

- The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what the curl command returns locally. <br>
`curl -s http://localhost:3300` It shall return an HTML page.

- To access it from the Internet, open TCP port 3300 in your AWS EC2 Instance.
![Alt text](<images/port 3300.png>)

You can access our Book Register web application from the Internet with a browser using a Public IP address or Public DNS name.
[http://<PublicIP>:3300]
This is how the WebBook Register Application will look in the browser:
![Alt text](images/6_success_from_web_add_delete.png)
