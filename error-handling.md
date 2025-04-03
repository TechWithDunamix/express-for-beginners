# Error Handling in Express.js

## Error Handling in Express.js

Error handling is a critical aspect of developing robust Express.js applications. Proper error handling improves application reliability, provides better user experience, and simplifies debugging.

### Basic Error Handling

Express comes with a built-in error handler that handles any errors that might occur in your application. However, for production applications, you'll want to implement custom error handling.

#### Using the Default Error Handler

Express's default error handler is executed when errors are passed to it:

```javascript
app.get('/user/:id', (req, res, next) => {
  // If id is not valid, pass an error to Express
  const userId = parseInt(req.params.id);
  if (isNaN(userId)) {
    const err = new Error('Invalid user ID');
    err.status = 400;
    next(err);
  } else {
    // Continue normal processing
    getUserById(userId)
      .then(user => res.json(user))
      .catch(next); // Pass any errors from getUserById to Express
  }
});
```

### Custom Error Handlers

Custom error handlers are defined as middleware functions with four arguments instead of the usual three:

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    error: {
      message: err.message,
      status: err.status || 500
    }
  });
});
```

### Structured Error Handling

For larger applications, it's beneficial to create a structured approach to error handling:

#### Creating Custom Error Classes

```javascript
class ApplicationError extends Error {
  constructor(message, status) {
    super(message);
    this.name = this.constructor.name;
    this.status = status || 500;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ResourceNotFoundError extends ApplicationError {
  constructor(resource = 'resource') {
    super(`The requested ${resource} was not found`, 404);
  }
}

class ValidationError extends ApplicationError {
  constructor(message) {
    super(message || 'Validation failed', 400);
  }
}
```

#### Using Custom Error Classes

```javascript
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      throw new ResourceNotFoundError('user');
    }
    res.json(user);
  } catch (err) {
    next(err);
  }
});
```

#### Comprehensive Error Handler

```javascript
// Error logger middleware
app.use((err, req, res, next) => {
  console.error(`[ERROR] ${err.name}: ${err.message}`);
  console.error(err.stack);
  next(err); // Pass to the next error handler
});

// API error response middleware
app.use((err, req, res, next) => {
  const status = err.status || 500;
  const errorResponse = {
    error: {
      message: err.message || 'Internal Server Error',
      status,
      type: err.name
    }
  };
  
  // Don't expose stack traces in production
  if (process.env.NODE_ENV !== 'production') {
    errorResponse.error.stack = err.stack;
  }
  
  res.status(status).json(errorResponse);
});
```

### Handling Async Errors

Express doesn't catch errors inside async functions by default. There are several approaches to handle this:

#### Manual Try/Catch

```javascript
app.get('/users', async (req, res, next) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (err) {
    next(err);
  }
});
```

#### Using a Wrapper Function

```javascript
// Helper function to wrap async route handlers
const asyncHandler = fn => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Using the wrapper
app.get('/users', asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json(users);
}));
```

#### Using Third-Party Middleware

Express-async-errors is a popular package that patches Express to handle async errors:

```javascript
// At the top of your main file
require('express-async-errors');

// Now you can use async handlers without try/catch
app.get('/users', async (req, res) => {
  const users = await User.find(); // Any error here will be caught
  res.json(users);
});
```

### Handling 404 Errors

Add a middleware after all your routes to catch 404 errors:

```javascript
app.use((req, res, next) => {
  const err = new ResourceNotFoundError('route');
  next(err);
});
```

### Summary

* Error handling is essential for robust Express applications
* Express provides a default error handler, but custom handlers offer more flexibility
* Create structured error classes for consistent error responses
* Handle async errors with try/catch, wrapper functions, or libraries like express-async-errors
* Always include error handling middleware at the end of your middleware chain
* Use appropriate HTTP status codes to communicate the error type
* Log errors for debugging but limit exposed information in production
