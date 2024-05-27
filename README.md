# Deploying-A-MERN-Stack-Todo-Application

A **MERN** stack consists of a collection of technologies used for the development of web applications. It comprises of MongoDB, Express, React and Node which are all JavaScript technologies used for creating full-stack applications and dynamic websites.

**MongoDB**: is a document-oriented NoSQL database technology used in storing data in the form of documents.

**React**: a javascript library used for building interactive user interface based on components.

**Express**: its a web application framework of Node js which is used for building server-side part of an appication and also creating Restful APIs.

**Node**: Node.js is an open-source, cross-platform runtime environment for building fast and scalable server-side and networking applications.

We will be building a simple todo list application and deploying on AWS cloud EC2 machine.

### Creating EC2 Instances

We log on to AWS Cloud Services and create an EC2 Ubuntu VM instance. When creating an instance, choose keypair authentication and download private key(*.pem) on your local computer.

![alt text](images/3.1.png)

On windows terminal, cd into the directory containing the downloaded private key.Run the below command to log into the instance via ssh:

ssh -i <private_keyfile.pem> username@ip-address

### Configuring Backend

Run `sudo apt` update and `sudo apt upgrade` to update all default ubuntu dependencies to ensure compatibility during package installation.

![alt text](images/3.2.png)

Next up will be to install nodejs, first we get the location of nodejs form the ubuntu repository using the following command. 
```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
Then we run a node install
```
sudo apt-get install -y nodejs
```
![alt text](images/3.3.jpg)

### Setting up the application

We then create a directory that will house our codes and packages and all subdirectories to represent components of our application.
```
mkdir todo
```
Inside this directory we will instantiate our project using `npm init`. This enables javascript to install packages useful for spinning up our application.

![alt text](images/3.4.jpg)

### Express installation

We will be installing express which is nodejs framework and will be helpful when creating routes for our application via HTTP requests.
```
npm install express
```
![alt text](images/3.5.jpg)

Create an `index.js` file which will contain code useful for spinning up our express server

Install the `dotenv` module which is a module that loads environment variables from a `.env` file into `process.env`. The `.env` files are useful for hiding important credentials which shouldnt be exposed.
```
npm install dotenv.
```
open the  `index.js` file with vi 
```
 vi index.js
 ```

 We copy the folowing code into it and "save" using ":wq"
```
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
```

We start our server to see if it works. Before we start the server, we need to be sure we are in the Todo directory.
Run `node index.js` to spin up our server.

This code is useful for spinning up our application via the port specified in the code.

Allow our port as part of the inbound security rules in our EC2 instance to ensure that our server is visible via the internet.

![alt text](images/3.6.jpg)

Paste our public ip address on the browser with the port to see if the server is properly configured.

![alt text](images/3.7.png)

### Defining Routes For our Application

We will create a `routes` folder which will contain code pointing to the three main endpoints used in a todo application. This will contain the post,get and delete requests which will be helpful in interacting with our client_side and database via restful apis.
```
mkdir routes
cd routes
touch api.js
```
Write the The below code in api.js. It is an example of a simple route that fires various endpoints.
```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

### Creating models

We then go ahead to create **"Models"** since the application will be using **MongoDB** which is a noSQL database. A **Model** makes the javascript application interactive. We also use Models to define the **database schema** . This is important so that we will be able to define the fields stored in each Mongodb document.

The **Schema** shows how the database will be setup, including other data fields that may not be required to be stored in the database.

To create a `Schema` and a `model`, we will need to install **mongoose** which is a **Node.js** package that makes working with mongodb easier.

To install Mongoose, we make sure we are in the Todo directory then run the command:
```
npm install mongoose
```

after that we go ahead to create a folder models by running the command:
```
mkdir models
```
Then we chaneg into the model directory:
```
cd models
```
Inside the models folder, create a file named todo.js
```
touch todo.js
```
We then open the todo.js file and paste the following codes:
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```
![alt text](images/3.8.png)

Since we have defined a schema for how our database should be structured, we then update the code in our api.js to fire specific actions when an endpoint is called.
```
cd routes
```
Open the api.js file
```
vim api.js
```

We copy and paste the following codes into the file:
```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```

### Creating a MongoDB Database

We use MongoDB Database to store our data using mLab which provides Database as a service (DBaas) solution.

To continue, we sign up [here](https://account.mongodb.com/account/register?signedOut=true).

Follow the sign up process, select AWS as the cloud provider, and choose a region near you.

![alt text](images/3.9.png)

Go to **"Network access"**, select **"Allow access from anywhere"**. This is not secure but good for testing purposes.

![alt text](images/3.10.png)

Then change the time of deleting the entry from 6 Hours to 1 Week.

![alt text](images/3.11.png)

Create a `MongoDB database` and collection inside mLab by clicking on `database`, click on `cluster0` and then open `collections`.

![alt text](images/3.12.png)


To get the connection string, we click on `cluster0` then click on `connect`

![alt text](images/3.13.png)

Then we click on `Mongodb drivers` ...connect your application..."

![alt text](images/3.14.png)

We then copy then connection string displayed into the .env file.

![alt text](images/3.15.png)

In the `index.js` file, we specified `process.env` to access environment variables, but we are yet to create this file.

To create the process.env file, we creta a file in Todo directory and name it .env.
```
touch .env
```
Open the file
```
vim .env
```
Add the connection string to access the database in it, just as below:
```
DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'
```
Make sure to update "<username, <password, <network-address" and <database according to your setup.


