# Introduction

## Introduction to Express.js

Express.js is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications. It simplifies the server creation process that would otherwise be more complex using Node.js's built-in HTTP module alone.

### What is Express.js?

Express is a lightweight framework built on top of Node.js that helps organize web application features into a Model-View-Controller (MVC) architecture. It provides mechanisms to:

* Set up middleware to respond to HTTP requests
* Define routing rules for different HTTP methods and URLs
* Render HTML pages based on passing arguments to templates
* Set common web application settings like the port to use for connecting

### Why Use Express?

```javascript
// Node.js HTTP module (without Express)
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Home Page');
  } else if (req.url === '/about') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('About Page');
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('Page Not Found');
  }
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

The same functionality with Express:

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Home Page');
});

app.get('/about', (req, res) => {
  res.send('About Page');
});

app.use((req, res) => {
  res.status(404).send('Page Not Found');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Installing Express

To start a new Express project, you need Node.js installed on your system. Then:

```javascript
// Create a new directory for your project
mkdir my-express-app
cd my-express-app

// Initialize a new Node.js project
npm init -y

// Install Express
npm install express
```

### Creating a Basic Express Server

```javascript
// Import the express module
const express = require('express');

// Create an Express application
const app = express();

// Define a route for the home page
app.get('/', (req, res) => {
  res.send('Hello World!');
});

// Start the server
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

Save this as `app.js` and run it with:

```
node app.js
```

### Express Application Structure

A simple Express application structure might look like this:

```
my-express-app/
  ├── node_modules/
  ├── public/
  │   ├── css/
  │   ├── js/
  │   └── images/
  ├── routes/
  │   ├── index.js
  │   └── users.js
  ├── views/
  │   ├── index.ejs
  │   └── error.ejs
  ├── app.js
  ├── package.json
  └── package-lock.json
```

### Express vs Other Frameworks

Express is often compared to other Node.js frameworks like Koa, Hapi, or NestJS. The key advantages of Express include:

* **Simplicity**: Minimalist approach with no strong opinions on structure
* **Flexibility**: Can be extended with middleware for any functionality
* **Widespread adoption**: Large community and ecosystem of compatible modules
* **Performance**: Lightweight core with minimal overhead

### Summary

* Express.js is a minimal and flexible Node.js web application framework
* It simplifies server-side development compared to using Node.js HTTP module alone
* Express provides routing, middleware support, and template rendering
* Setting up a basic Express application requires just a few lines of code
* The framework follows the middleware pattern, making it highly extensible
* Express has a large community and ecosystem, making it an excellent choice for both beginners and experienced developers
