# Middlewares

## Middleware in Express.js

Middleware functions are functions that have access to the request object (`req`), the response object (`res`), and the next middleware function in the application's request-response cycle. They can execute code, modify request and response objects, end the request-response cycle, or call the next middleware function.

### How Middleware Works

In Express, middleware functions are executed sequentially in the order they are added. They can perform the following tasks:

* Execute any code
* Make changes to the request and response objects
* End the request-response cycle
* Call the next middleware in the stack

### Basic Middleware Syntax

A middleware function follows this basic structure:

```javascript
function myMiddleware(req, res, next) {
  // Middleware logic goes here
  // ...
  
  // Call next() to pass control to the next middleware
  next();
}
```

The `next` parameter is a function that, when called, executes the next middleware in the stack.

### Applying Middleware

There are three ways to apply middleware in Express:

#### 1. Application-level middleware

Applied to the entire application using `app.use()` or `app.METHOD()`:

```javascript
const express = require('express');
const app = express();

// Application-level middleware
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});

// Route-specific middleware
app.use('/user/:id', (req, res, next) => {
  console.log('Request Type:', req.method);
  next();
});

// Handler for the route
app.get('/user/:id', (req, res) => {
  res.send('User Info');
});

app.listen(3000);
```

#### 2. Router-level middleware

Works the same as application-level middleware but is bound to an instance of `express.Router()`:

```javascript
const express = require('express');
const router = express.Router();

// Router-level middleware
router.use((req, res, next) => {
  console.log('Router Time:', Date.now());
  next();
});

router.get('/user/:id', (req, res) => {
  res.send('User Info');
});

// Add the router to the application
const app = express();
app.use('/api', router);

app.listen(3000);
```

#### 3. Error-handling middleware

Takes four arguments instead of three (`err`, `req`, `res`, `next`):

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

### Built-in Middleware

Express comes with a few built-in middleware functions:

#### express.static

Serves static files:

```javascript
// Serve static files from the 'public' directory
app.use(express.static('public'));

// You can also specify a virtual path prefix
app.use('/static', express.static('public'));
```

#### express.json

Parses incoming requests with JSON payloads:

```javascript
// Parse JSON requests
app.use(express.json());
```

#### express.urlencoded

Parses incoming requests with URL-encoded payloads:

```javascript
// Parse URL-encoded requests
app.use(express.urlencoded({ extended: false }));
```

### Third-party Middleware

You can use third-party middleware to add functionality to your Express application. First, install the middleware using npm:

```
npm install cookie-parser
```

Then use it in your application:

```javascript
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();

// Load the cookie-parsing middleware
app.use(cookieParser());
```

### Common Middleware Examples

#### 1. Logging Middleware

```javascript
const logger = (req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
  next();
};

app.use(logger);
```

#### 2. Authentication Middleware

```javascript
const authenticate = (req, res, next) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader) {
    return res.status(401).json({ message: 'No token provided' });
  }
  
  const token = authHeader.split(' ')[1];
  
  // Validate token (simplified example)
  if (token === 'valid-token') {
    req.user = { id: 123, username: 'user' };
    next();
  } else {
    res.status(403).json({ message: 'Invalid token' });
  }
};

// Apply to specific routes
app.get('/protected', authenticate, (req, res) => {
  res.json({ message: 'Protected data', user: req.user });
});
```

#### 3. Error Handling

```javascript
// Regular middleware
app.get('/item/:id', (req, res, next) => {
  // Simulated error
  if (req.params.id === '0') {
    // Create an error and pass it to the next middleware
    const err = new Error('Item not found');
    err.status = 404;
    next(err);
  } else {
    res.send(`Item ${req.params.id}`);
  }
});

// Error-handling middleware
app.use((err, req, res, next) => {
  res.status(err.status || 500);
  res.json({
    error: {
      message: err.message
    }
  });
});
```

#### 4. CORS Middleware

```javascript
// Custom CORS middleware
const cors = (req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept');
  if (req.method === 'OPTIONS') {
    res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    return res.status(200).json({});
  }
  next();
};

app.use(cors);
```

### Middleware Execution Order

Middleware functions are executed in the order they are defined. This order is crucial for the application's behavior:

```javascript
const express = require('express');
const app = express();

// This middleware will be executed first
app.use((req, res, next) => {
  console.log('First middleware');
  next();
});

// This middleware will be executed second
app.use((req, res, next) => {
  console.log('Second middleware');
  next();
});

// Route handler
app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(3000);
```

### Creating Configurable Middleware

You can create middleware that accepts options:

```javascript
// Configurable logging middleware
function logger(options) {
  const defaults = { format: 'short' };
  const opts = { ...defaults, ...options };
  
  return (req, res, next) => {
    if (opts.format === 'short') {
      console.log(`${req.method} ${req.url}`);
    } else if (opts.format === 'verbose') {
      console.log(`${new Date().toISOString()} - ${req.method} ${req.url} - ${req.ip}`);
    }
    next();
  };
}

// Usage
app.use(logger({ format: 'verbose' }));
```

### Summary

* Middleware functions are fundamental to Express applications
* They have access to request, response, and next function in the request-response cycle
* Middleware can modify request and response objects or end the cycle
* Express has application-level, router-level, and error-handling middleware
* Built-in middleware includes express.static, express.json, and express.urlencoded
* Third-party middleware extends functionality (like cookie-parser)
* Middleware execution order is critical and follows the sequence they're added
* Configurable middleware can be created to accept options for flexibility
