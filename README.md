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

