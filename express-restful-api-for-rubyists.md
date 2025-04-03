# RESTful API Development with Express.js

## RESTful API Development with Express.js

RESTful APIs provide a standardized approach to build web services that are stateless, scalable, and follow consistent conventions. This guide covers building robust REST APIs with Express.js.

### REST API Fundamentals

REST (Representational State Transfer) is an architectural style that uses HTTP methods to operate on resources. A RESTful API follows these principles:

* **Stateless**: Each request contains all information needed to complete it
* **Resource-based**: Resources are identified by URIs
* **HTTP methods**: Uses standard HTTP methods (GET, POST, PUT, DELETE, etc.)
* **Representation**: Resources can have multiple representations (JSON, XML, etc.)

### Setting Up a Basic REST API

#### Project Structure

```
my-rest-api/
├── controllers/
│   └── userController.js
├── models/
│   └── User.js
├── routes/
│   ├── index.js
│   └── users.js
├── middleware/
│   └── auth.js
├── config/
│   └── db.js
├── server.js
└── package.json
```

#### Basic Server Setup

```javascript
// server.js
const express = require('express');
const cors = require('cors');
const morgan = require('morgan');

// Import routes
const indexRoutes = require('./routes/index');
const userRoutes = require('./routes/users');

// Initialize app
const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(morgan('dev'));

// Routes
app.use('/', indexRoutes);
app.use('/api/users', userRoutes);

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    error: {
      message: err.message,
      status: err.status || 500
    }
  });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Implementing RESTful Resources

#### Resource Routes

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const auth = require('../middleware/auth');

// GET /api/users - Get all users
router.get('/', userController.getAllUsers);

// GET /api/users/:id - Get user by ID
router.get('/:id', userController.getUserById);

// POST /api/users - Create new user
router.post('/', userController.createUser);

// PUT /api/users/:id - Update user (requires auth)
router.put('/:id', auth.requireAuth, userController.updateUser);

// PATCH /api/users/:id - Partially update user (requires auth)
router.patch('/:id', auth.requireAuth, userController.patchUser);

// DELETE /api/users/:id - Delete user (requires auth)
router.delete('/:id', auth.requireAuth, userController.deleteUser);

module.exports = router;
```

#### Controller Logic

```javascript
// controllers/userController.js
const User = require('../models/User');

// Get all users
exports.getAllUsers = async (req, res, next) => {
  try {
    const users = await User.find().select('-password');
    res.json(users);
  } catch (err) {
    next(err);
  }
};

// Get user by ID
exports.getUserById = async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id).select('-password');
    
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }
    
    res.json(user);
  } catch (err) {
    next(err);
  }
};

// Create new user
exports.createUser = async (req, res, next) => {
  try {
    const newUser = await User.create(req.body);
    
    // Remove password from response
    const userResponse = newUser.toObject();
    delete userResponse.password;
    
    res.status(201).json(userResponse);
  } catch (err) {
    next(err);
  }
};

// Update entire user record
exports.updateUser = async (req, res, next) => {
  try {
    // Make sure the password is hashed if included
    const userToUpdate = req.body;
    
    const updatedUser = await User.findByIdAndUpdate(
      req.params.id,
      userToUpdate,
      { new: true, runValidators: true }
    ).select('-password');
    
    if (!updatedUser) {
      return res.status(404).json({ message: 'User not found' });
    }
    
    res.json(updatedUser);
  } catch (err) {
    next(err);
  }
};

// Partially update user
exports.patchUser = async (req, res, next) => {
  try {
    const updates = req.body;
    
    const updatedUser = await User.findByIdAndUpdate(
      req.params.id,
      { $set: updates },
      { new: true, runValidators: true }
    ).select('-password');
    
    if (!updatedUser) {
      return res.status(404).json({ message: 'User not found' });
    }
    
    res.json(updatedUser);
  } catch (err) {
    next(err);
  }
};

// Delete user
exports.deleteUser = async (req, res, next) => {
  try {
    const deletedUser = await User.findByIdAndDelete(req.params.id);
    
    if (!deletedUser) {
      return res.status(404).json({ message: 'User not found' });
    }
    
    res.json({ message: 'User deleted successfully' });
  } catch (err) {
    next(err);
  }
};
```

### HTTP Status Codes

Use appropriate HTTP status codes to communicate results:

```javascript
// 200 - OK: Standard response for successful requests
res.status(200).json(data);

// 201 - Created: Resource successfully created
res.status(201).json(newResource);

// 204 - No Content: Successful operation, no content to return
res.status(204).send();

// 400 - Bad Request: Invalid request parameters
res.status(400).json({ error: 'Invalid request parameters' });

// 401 - Unauthorized: Authentication required
res.status(401).json({ error: 'Authentication required' });

// 403 - Forbidden: Authentication succeeded but user lacks permissions
res.status(403).json({ error: 'Access denied' });

// 404 - Not Found: Resource not found
res.status(404).json({ error: 'Resource not found' });

// 409 - Conflict: Request conflicts with current state of the server
res.status(409).json({ error: 'Resource already exists' });

// 500 - Internal Server Error: Unexpected server error
res.status(500).json({ error: 'Internal server error' });
```

### Request Validation

Input validation is critical for API security and reliability:

```javascript
// Install validator package
// npm install express-validator

// routes/users.js
const { body, param, validationResult } = require('express-validator');

// Validation middleware
const validateUser = [
  body('name').trim().isLength({ min: 2 }).withMessage('Name must be at least 2 characters'),
  body('email').isEmail().normalizeEmail().withMessage('Must provide valid email'),
  body('password')
    .isLength({ min: 6 })
    .withMessage('Password must be at least 6 characters'),
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    next();
  }
];

// Use validation middleware in routes
router.post('/', validateUser, userController.createUser);
```

### Authentication and Authorization

Secure your API with authentication:

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const config = require('../config');

exports.requireAuth = (req, res, next) => {
  // Get token from header
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Authorization token required' });
  }
  
  const token = authHeader.split(' ')[1];
  
  try {
    // Verify token
    const decoded = jwt.verify(token, config.jwtSecret);
    
    // Add user from payload to request
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

exports.checkRole = (role) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'User not authenticated' });
    }
    
    if (req.user.role !== role) {
      return res.status(403).json({ error: 'Access denied' });
    }
    
    next();
  };
};
```

### API Versioning

Version your API to maintain backward compatibility:

```javascript
// server.js
const v1Routes = require('./routes/v1');
const v2Routes = require('./routes/v2');

// API versioning using URL path
app.use('/api/v1', v1Routes);
app.use('/api/v2', v2Routes);

// Or using request headers
app.use('/api', (req, res, next) => {
  const apiVersion = req.headers['accept-version'] || '1';
  
  if (apiVersion === '1') {
    return v1Routes(req, res, next);
  } else if (apiVersion === '2') {
    return v2Routes(req, res, next);
  } else {
    return res.status(400).json({ error: 'API version not supported' });
  }
});
```

### Query Parameters and Filtering

Implement search, pagination, and filtering:

```javascript
// controllers/productController.js
exports.getProducts = async (req, res, next) => {
  try {
    let query = {};
    
    // Filtering
    if (req.query.category) {
      query.category = req.query.category;
    }
    
    if (req.query.minPrice && req.query.maxPrice) {
      query.price = { 
        $gte: parseFloat(req.query.minPrice), 
        $lte: parseFloat(req.query.maxPrice) 
      };
    }
    
    // Search
    if (req.query.search) {
      query.name = { $regex: req.query.search, $options: 'i' };
    }
    
    // Pagination
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const skip = (page - 1) * limit;
    
    // Sorting
    const sortField = req.query.sortBy || 'createdAt';
    const sortOrder = req.query.sortOrder === 'asc' ? 1 : -1;
    const sortOptions = {};
    sortOptions[sortField] = sortOrder;
    
    // Execute query
    const products = await Product.find(query)
      .sort(sortOptions)
      .skip(skip)
      .limit(limit);
    
    // Get total count for pagination
    const total = await Product.countDocuments(query);
    
    res.json({
      products,
      pagination: {
        total,
        page,
        limit,
        pages: Math.ceil(total / limit)
      }
    });
  } catch (err) {
    next(err);
  }
};
```

### API Documentation

Document your API with tools like Swagger/OpenAPI:

```javascript
// Install Swagger packages
// npm install swagger-jsdoc swagger-ui: express

// server.js
const swaggerJsDoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

// Swagger configuration
const swaggerOptions = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My REST API',
      version: '1.0.0',
      description: 'A sample Express REST API'
    },
    servers: [
      {
        url: 'http://localhost:3000/api'
      }
    ]
  },
  apis: ['./routes/*.js']
};

const swaggerDocs = swaggerJsDoc(swaggerOptions);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocs));
```

Add documentation to routes:

```javascript
/**
 * @swagger
 * /users:
 *   get:
 *     summary: Returns a list of users
 *     tags: [Users]
 *     responses:
 *       200:
 *         description: A list of users
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/User'
 */
router.get('/', userController.getAllUsers);
```

### Rate Limiting

Protect your API from abuse with rate limiting:

```javascript
// npm install express-rate-limit

const rateLimit = require('express-rate-limit');

// Create limiter middleware
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again after 15 minutes'
});

// Apply to all API requests
app.use('/api', apiLimiter);

// Or apply to specific routes
app.use('/api/auth', authLimiter);
```

### Summary

* RESTful APIs follow conventions that make them predictable and easy to use
* Organize your code with routes, controllers, and models for maintainability
* Use appropriate HTTP methods and status codes for clear communication
* Validate input data to keep your API secure and reliable
* Implement authentication and authorization to protect sensitive resources
* Version your API to maintain compatibility as your API evolves
* Support pagination, filtering, and sorting for better data access
* Document your API thoroughly for developers who will consume it
* Add rate limiting to prevent abuse and ensure fair resource usage
