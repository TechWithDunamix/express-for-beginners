# Middleware in Express.js

Middleware functions are a core concept in Express.js. If you're familiar with Rack middleware in Ruby, you'll find some similarities, but Express middleware is more integrated into the framework.

## What is Middleware?

Middleware functions are functions that have access to the request object (req), the response object (res), and the next middleware function in the application's request-response cycle, commonly denoted by a variable named `next`.

## Basic Middleware Example

```javascript
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});
```

This middleware will execute for every request to the app.

## Middleware Types

1. **Application-level middleware**: Bound to the app object using `app.use()` and `app.METHOD()`.

2. **Router-level middleware**: Bound to an instance of `express.Router()`.

3. **Error-handling middleware**: Defined with four arguments `(err, req, res, next)`.

4. **Built-in middleware**: Like `express.static`, `express.json`, `express.urlencoded`.

5. **Third-party middleware**: Install via npm and load in your app to add functionality.

## Middleware vs. Ruby Rack Middleware

While similar in concept to Rack middleware, Express middleware is more flexible:

- It can end the request-response cycle.
- It can call the next middleware function in the stack.
- It has direct access to the request and response objects.

## Common Use Cases

1. **Logging**

```javascript
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

2. **Body Parsing**

```javascript
app.use(express.json()); // for parsing application/json
app.use(express.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded
```

3. **Error Handling**

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

## Conclusion

Understanding middleware is crucial for effective Express.js development. It allows you to write pluggable, modular code that can be easily reused and composed. In the next section, we'll explore the request and response objects in more detail.
