# Routing in Express.js

For Rubyists familiar with Sinatra or Rails, routing in Express.js will feel quite familiar. Let's explore how to define routes in Express.js and compare it with Ruby frameworks.

## Basic Routing

In Express.js, a route is a combination of a URL path and a specific HTTP request method (GET, POST, PUT, DELETE, etc.).

### Basic GET Route

```javascript
app.get('/', (req, res) => {
  res.send('Hello World!');
});
```

This is similar to Sinatra:

```ruby
get '/' do
  'Hello World!'
end
```

### Routes with Parameters

Express.js:

```javascript
app.get('/users/:id', (req, res) => {
  res.send(`User ID: ${req.params.id}`);
});
```

Sinatra equivalent:

```ruby
get '/users/:id' do
  "User ID: #{params[:id]}"
end
```

## HTTP Methods

Express.js provides methods for all HTTP verbs:

```javascript
app.post('/users', (req, res) => {
  // Create a new user
});

app.put('/users/:id', (req, res) => {
  // Update a user
});

app.delete('/users/:id', (req, res) => {
  // Delete a user
});
```

## Route Handlers

You can provide multiple callback functions that behave like middleware to handle a request:

```javascript
app.get('/example', (req, res, next) => {
  // do something
  next();
}, (req, res) => {
  res.send('Hello from Example!');
});
```

## Route Modules

For larger applications, you can organize routes using Express.Router:

```javascript
// users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.send('Users Index');
});

router.get('/:id', (req, res) => {
  res.send(`User ID: ${req.params.id}`);
});

module.exports = router;

// app.js
const usersRouter = require('./users');
app.use('/users', usersRouter);
```

This is somewhat similar to how you might organize controllers in Rails.

## Conclusion

Routing in Express.js should feel familiar to Rubyists. The main difference is the syntax and the way middleware can be integrated directly into routes. In the next section, we'll dive deeper into middleware in Express.js.
