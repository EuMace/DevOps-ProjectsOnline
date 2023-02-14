## PROJECT 3: MERN STACK IMPLEMENTATION

# STEP 1 – BACKEND CONFIGURATION
1. Update ubuntu after launching EC2 instance: sudo apt update
2. Upgrade ubuntu: sudo apt upgrade
3. Retrieve the location of Node.js software from Ubuntu repositories: curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

 ![image](https://user-images.githubusercontent.com/120044190/217988637-cf6e77c0-b776-449f-9314-642a87232e16.png)





4. Install Node.js on the server with command: sudo apt-get install -y nodejs

The command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.

 ![image](https://user-images.githubusercontent.com/120044190/217988704-be060dec-18eb-4197-93ec-1383968f831b.png)



5. To verify the node installation, use node -v

 ![image](https://user-images.githubusercontent.com/120044190/217988744-6be1ba72-b547-4712-ad60-e4461547429f.png)



6. To verify the node installation for npm use: npm -v

 ![image](https://user-images.githubusercontent.com/120044190/217988770-9b5b5f3c-9b25-4999-a86a-b4c4e43712b0.png)



7. Application Code Setup—> Create a new directory for your To-Do project: mkdir Todo 
8. Run command to verify Todo directory has been created: ls

 ![image](https://user-images.githubusercontent.com/120044190/217988806-81652ce1-a86a-4796-8695-3ac275c04404.png)



9. Change current directory to Todo: cd Todo
10. Use command: npm init to initialize project, so new file named package.json is created. Follow the prompts after running the command. You can press Enter several times to accept default values, then accept to write out the package.json file by typing yes.
 
![image](https://user-images.githubusercontent.com/120044190/217988837-facafffc-099b-4509-902b-cb645981f794.png)

 ![image](https://user-images.githubusercontent.com/120044190/217988868-153d9459-d87e-4227-9625-19e3bc084bc6.png)



11. Run the command ls to confirm that you have package.json file created.

# INSTALL EXPRESSJS
1. To use express, install using npm: npm install express
2. Create a file index.js using command: touch index.js  and run ls to confirm

 ![image](https://user-images.githubusercontent.com/120044190/217988906-07c57eef-e3eb-4229-9cd4-7c719688c027.png)



3. Install the dotenv module: npm install dotenv
4. Open index.js file using command: vim index.js and type code below and save

const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});


5. Port 5000 was used in the code to use on browser.
6. Start server to see if it works. Open terminal in same directory as the index.js file and use command: node index.js

 ![image](https://user-images.githubusercontent.com/120044190/217988965-d109cda8-0fbc-4cfd-bc8b-392c0f2d634b.png)



7. Open port in EC2 Security Groups–create inbound rule to open TCP port 5000
8. Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000: http://54.157.133.238:5000

 ![image](https://user-images.githubusercontent.com/120044190/217988999-76a4ccb8-73c6-4ff0-abf5-a9e97c5a25bb.png)


# Routes
There are three actions that our To-Do application needs to be able to do:
1. Create a new task
2. Display list of all tasks
3. Delete a completed task

Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.

1. For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes:
        mkdir routes

2. Change directory to routes folder: cd routes
3. Create file apt.js: touch api.js
4. Open file with command: vim api.js and copy code below:

![image](https://user-images.githubusercontent.com/120044190/217989123-4d27fc23-af96-45b5-8d5d-d37d9a6c09b3.png)

![image](https://user-images.githubusercontent.com/120044190/217989150-24ab7fcc-c5e4-41eb-9e9e-0a58e49900ed.png)

	 
## MODELS
To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.
1. Change the directory back to Todo folder with cd .. and install Mongoose: npm install mongoose 
 
 ![image](https://user-images.githubusercontent.com/120044190/217989219-9e167dc2-d204-4c3b-890f-ad91f33ee405.png)

 
2. Create a new folder models: mkdir models
3. Create file todo.js in models folder: touch todo.js
Can also run command for all three: mkdir models && cd models && touch todo.js

4. Open file todo.js with vim and paste code in file:

![image](https://user-images.githubusercontent.com/120044190/217989278-0a21c407-22b2-42af-a0ab-7be11f36201c.png)

	 
5. Update our routes from the file api.js in ‘routes’ directory to make use of the new model.
6. In Routes directory, open api.js with vim api.js, delete the code inside with :%d command and paste there code below into it then save and exit

![image](https://user-images.githubusercontent.com/120044190/217989334-979f7635-2e37-4e18-9409-840c65251854.png)


## MONGODB DATABASE

1. Create MongoDB account.
2. Created file in Todo directory and named it .env: touch .env and vi .env
3. Add this connection string: DB = ‘mongodb+srv://Eunnie:<password>@cluster0.gwifb3m.mongodb.net/?retryWrites=true&w=majority’

 ![image](https://user-images.githubusercontent.com/120044190/217989399-2303f655-6bcb-4405-a744-f72bd9fe139a.png)



4. Then update the index.js file to reflect use of .env so Node.js can connect to the database. Delete existing content int the file and update with code below
5. Go in index.js with: vim index.js, then press esc and type : %d, then enter.
6. Type i and paste code below in file:

const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();
 
const app = express();
 
const port = process.env.PORT || 5000;
 
//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));
 
//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;
 
app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});
 
app.use(bodyParser.json());
 
app.use('/api', routes);
 
app.use((err, req, res, next) => {
console.log(err);
next();
});
 
app.listen(port, () => {
console.log(`Server running on port ${port}`)
});


7.	Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.
Start your server using the command:
node index.js

You will see a message –Database connected successfully— backend is configured now to test it.

 ![image](https://user-images.githubusercontent.com/120044190/217989483-8196cc7f-b308-466a-b14b-7167458a70b5.png)


## Testing Backend Code without Frontend using RESTful API


• The backend part of our To-do application has been written, but we also need a frontend UI. To do this we need ReactJC code. During development we need a way to test our code using RESTful API. So we will need to make use of API development client to test our code.
• To do this we use Postman to test our API
o Install Postman on your machine
1. In Postman, create a POST request to the API. This request sends a new task to our T0-Do list so the application can store in the database http://54.237.119.172:5000/api/todos
2. Make sure to set header key Content-Type as application/json
 
![image](https://user-images.githubusercontent.com/120044190/217989580-cafa0d02-e899-4af9-8220-e41d1b6045e4.png)


3. Create a GET request to your API on http://54.237.119.172:5000/api/todos 

This request retrieves all existing records out from To-Do application (the backend requests these records from the database and sends it back as a response to GET request.
	
	![image](https://user-images.githubusercontent.com/120044190/217989629-52c2ae55-6c1a-4350-8b22-661bc248a318.png)

 
## STEP 2 – FRONTEND CREATION
	
• The functionality of the backend and API has been completed, now we need to create a user interface for a Web client (browser) to interact with the application via API
- In creating frontend of To-Do app, use npx create-react-app client in same root directory as backend code, which is the Todo directory. This will creat a new folder in your Todo directory called client. The react code will go in there.

# Running a React App
Before testing the react app, there are some dependencies that need to be installed.
	

1. Install concurrently. It is used to run more than one command simultaneously from the same terminal window.   npm install concurrently --save-dev
	
	![image](https://user-images.githubusercontent.com/120044190/217989832-6e4bcffd-0e45-41c8-bbbf-7fee2f16f89e.png)

 
2. Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.   npm install nodemon --save-dev
	
	![image](https://user-images.githubusercontent.com/120044190/217989893-e11bdeef-7d75-40f0-9d3e-6feabaaccf47.png)

 
3. In Todo folder open the package.json file. Replace “scripts”: { “test”: “echo \”Error: no test specified\” && exit 1”}  with the code below.
	
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
 
	![image](https://user-images.githubusercontent.com/120044190/217989990-4ac1944d-c6a5-4252-8aa8-3718e7dfa82c.png)

	

## Configure Proxy in package.json
	
1. Change directory to “client’---cd client
2. Open the package.json file—vi package.json
3. Add the key value pair in the package.json file "proxy": "http://localhost:5000"

The whole purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos

	![image](https://user-images.githubusercontent.com/120044190/217990149-997b2ad2-f16d-4a0b-9f94-8a31b7e1920a.png)

 
4. Making sure you are in inside the Todo directory, run npm run dev
	
	![image](https://user-images.githubusercontent.com/120044190/217990207-9d127985-7a8b-49fb-9717-58aee4986757.png)

 
Your app should open and start running on Localhost:3000

To access the application from the Internet you have to open TCP port 3000 on EC2 by adding a new Security Group rule.
	
![image](https://user-images.githubusercontent.com/120044190/217995316-8e897101-8683-4b31-9a61-0244c96b295d.png)

 
## Creating your React Components
	
One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For our Todo app, there will be two stateful components and one stateless component.
1. From your Todo directory run  cd client
2. Move to the src directory    cd src
3. Inside your src folder create another folder called components
  mkdir components
	
	
4. Move into the components directory with  cd components
5. Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js:
touch Input.js ListTodo.js Todo.js

6. Open Input.js file—-vi Input.js
Copy and paste the following:
import React, { Component } from 'react';
import axios from 'axios';
 
class Input extends Component {
 
state = {
action: ""
}
 
addTodo = () => {
const task = {action: this.state.action}
 
	if(task.action && task.action.length > 0){
  	axios.post('/api/todos', task)
    	.then(res => {
      	if(res.data){
        	this.props.getTodos();
        	this.setState({action: ""})
    	  }
    	})
    	.catch(err => console.log(err))
	}else {
  	console.log('input field required')
	}
 
}
 
handleChange = (e) => {
this.setState({
action: e.target.value
})
}
 
render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}
 
export default Input



7. To use Axios, cd into your client directory and run npm install axios
	
	![image](https://user-images.githubusercontent.com/120044190/217990480-f9043e01-2358-4d28-b888-c1274dc1fdae.png)

 
## FRONTEND CREATION (CONTINUED)
	
1. Go to ‘components’ directory  cd src/components
2. Open ListTodo.js—-vi ListTodo.js
3. In this file copy and paste the following code:
	
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo

4. Then in your Todo.js file write the following code:
	
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;

5. Move to the src folder cd ..; and vi App.js to make an adjustment with the logo by deleting and paste code into App.js
6. Copy and paste in App.js:

import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;


7. In the src directory open the App.css—-vi App.css
	
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}

8. Exit 
9. In the src directory open the index.css—vim index.css and paste:
	
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}


10. Go to the Todo directory—cd ../..  Then run npm run dev
	
If there are no errors when saving all these files, our To-Do app should be ready and fully functional with the functionality discussed earlier: creating a task, deleting a task and viewing all your tasks.
 
![image](https://user-images.githubusercontent.com/120044190/217992111-8a1773c0-ac84-4415-aca9-b899a3bd3f39.png)

