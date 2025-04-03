# Routing

## Routing in Express.js

Routing refers to determining how an application responds to client requests to particular endpoints, which are URIs (or paths) and a specific HTTP request method (GET, POST, etc.). This document covers the fundamentals of routing in Express.js.

### Basic Routing

At its core, a route in Express has the following structure:

```javascript
app.METHOD(PATH, HANDLER);
```

Where:

* `app` is an instance of Express
* `METHOD` is an HTTP request method (get, post, put, delete, etc.)
* `PATH` is a path on the server
* `HANDLER` is the function executed when the route is matched

Here's a simple example demonstrating basic routing:

```javascript
const express = require('express');
const app = express();

// Respond to GET request on the root route
app.get('/', (req, res) => {
  res.send('GET request to homepage');
});

// Respond to POST request on the root route
app.post('/', (req, res) => {
  res.send('POST request to homepage');
});

// Respond to GET request on the /about route
app.get('/about', (req, res) => {
  res.send('About page');
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

### Route Parameters

Route parameters are named URL segments used to capture values at specific positions in the URL. The captured values are stored in the `req.params` object.

```javascript
// Route path: /users/:userId/books/:bookId
app.get('/users/:userId/books/:bookId', (req, res) => {
  const { userId, bookId } = req.params;
  res.send(`User ID: ${userId}, Book ID: ${bookId}`);
});
```

For example, if you access `/users/34/books/8989`, the response would be:

```
User ID: 34, Book ID: 8989
```

### Route Handlers

You can provide multiple callback functions as handlers for a route. These work like middleware and must either call `next()` to pass control to the next handler or end the request-response cycle.

```javascript
// Multiple callback functions
app.get('/example', 
  (req, res, next) => {
    console.log('First handler');
    next(); // Pass control to the next handler
  }, 
  (req, res) => {
    console.log('Second handler');
    res.send('Response from the second handler');
  }
);

// Array of callback functions
const cb1 = (req, res, next) => {
  console.log('CB1');
  next();
};

const cb2 = (req, res, next) => {
  console.log('CB2');
  next();
};

const cb3 = (req, res) => {
  res.send('Response from CB3');
};

app.get('/callback-array', [cb1, cb2, cb3]);
```

### Route Methods

Express supports all HTTP methods. Here are the most commonly used ones:

```javascript
// GET method
app.get('/users', (req, res) => {
  res.send('Get all users');
});

// POST method
app.post('/users', (req, res) => {
  res.send('Create a new user');
});

// PUT method
app.put('/users/:id', (req, res) => {
  res.send(`Update user with ID ${req.params.id}`);
});

// DELETE method
app.delete('/users/:id', (req, res) => {
  res.send(`Delete user with ID ${req.params.id}`);
});

// Special method app.all() that applies to all HTTP methods
app.all('/secret', (req, res) => {
  res.send('Accessing the secret section with any HTTP method');
});
```

### Route Paths

Route paths can be strings, string patterns, or regular expressions:

```javascript
// String-based route
app.get('/about', (req, res) => {
  res.send('About page');
});

// String pattern with route parameters
app.get('/flights/:from-:to', (req, res) => {
  res.send(`Flight from ${req.params.from} to ${req.params.to}`);
});

// Regular expression
app.get(/.*fly$/, (req, res) => {
  res.send('URL ends with "fly"');
});
```

### Express Router

For larger applications, it's best to organize routes using the Express Router. This allows you to create modular route handlers:

```javascript
// userRoutes.js
const express = require('express');
const router = express.Router();

// Define routes relative to /users
router.get('/', (req, res) => {
  res.send('Get all users');
});

router.get('/:id', (req, res) => {
  res.send(`Get user with ID ${req.params.id}`);
});

router.post('/', (req, res) => {
  res.send('Create a new user');
});

// Export the router
module.exports = router;
```

Then in your main application file:

```javascript
// app.js
const express = require('express');
const app = express();
const userRoutes = require('./userRoutes');

// Mount the router at /users
app.use('/users', userRoutes);

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

### Chaining Route Handlers

Express allows you to chain route handlers for the same path:

```javascript
app.route('/books')
  .get((req, res) => {
    res.send('Get all books');
  })
  .post((req, res) => {
    res.send('Add a new book');
  })
  .put((req, res) => {
    res.send('Update all books');
  });
```

### Summary

* Routing in Express determines how the application responds to client requests
* Basic routes follow the pattern `app.METHOD(PATH, HANDLER)`
* Route parameters allow for dynamic values in URLs, accessible via `req.params`
* Express supports multiple handlers for a single route
* The Express Router provides a way to create modular, mountable route handlers
* Route paths can be defined using strings, patterns, or regular expressions
* Route handlers can be chained for the same path using `app.route()`
