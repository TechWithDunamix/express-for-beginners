# RESTful API Development with Express.js

For Rubyists familiar with building APIs in Rails, Express.js offers a flexible way to create RESTful APIs. While Express doesn't have built-in conventions like Rails does, it's straightforward to implement RESTful principles.

## Basic RESTful Route Structure

Here's a basic structure for a RESTful API for a `users` resource:

```javascript
const express = require('express');
const router = express.Router();

// GET /users
router.get('/', (req, res) => {
  // List all users
});

// POST /users
router.post('/', (req, res) => {
  // Create a new user
});

// GET /users/:id
router.get('/:id', (req, res) => {
  // Get a specific user
});

// PUT /users/:id
router.put('/:id', (req, res) => {
  // Update a user
});

// DELETE /users/:id
router.delete('/:id', (req, res) => {
  // Delete a user
});

module.exports = router;
```

In your main app file:

```javascript
const usersRouter = require('./routes/users');
app.use('/users', usersRouter);
```

This is similar to Rails' `resources :users` in `routes.rb`, but more explicit.

## Handling JSON

Express can parse incoming JSON and send JSON responses:

```javascript
app.use(express.json());

router.post('/', (req, res) => {
  const newUser = req.body;
  // Create user in database
  res.status(201).json(newUser);
});
```

## Status Codes

Use appropriate status codes for your responses:

```javascript
router.get('/:id', (req, res) => {
  const user = // ... fetch user from database
  if (user) {
    res.json(user);
  } else {
    res.status(404).json({ message: 'User not found' });
  }
});
```

## Error Handling

Create a middleware for handling errors:

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Something went wrong!' });
});
```

## Versioning

You can version your API by prefixing routes:

```javascript
app.use('/api/v1/users', usersRouterV1);
app.use('/api/v2/users', usersRouterV2);
```

## Authentication

For API authentication, you might use JWT:

```javascript
const jwt = require('jsonwebtoken');

function authenticateToken(req, res, next) {
  const token = req.headers['authorization'];
  if (token == null) return res.sendStatus(401);

  jwt.verify(token, process.env.TOKEN_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
}

router.get('/protected', authenticateToken, (req, res) => {
  res.json({ data: 'This is protected data.' });
});
```

## Testing

You can use libraries like Mocha and Chai for testing your API:

```javascript
const chai = require('chai');
const chaiHttp = require('chai-http');
const app = require('../app');

chai.use(chaiHttp);
const expect = chai.expect;

describe('Users API', () => {
  it('should GET all users', (done) => {
    chai.request(app)
      .get('/users')
      .end((err, res) => {
        expect(res).to.have.status(200);
        expect(res.body).to.be.an('array');
        done();
      });
  });
});
```

## Conclusion

While Express.js doesn't provide as many conventions out of the box as Rails does for API development, it offers great flexibility in how you structure your RESTful APIs. The core principles remain the same, and with a bit of setup, you can create robust and scalable APIs in Express.js.
