# NoSQL Databases (MongoDB) with Express.js

## NoSQL Databases (MongoDB) with Express.js

NoSQL databases like MongoDB provide flexible, schema-less data storage that works well with JavaScript and Express.js. This guide covers how to integrate MongoDB with Express applications.

### Setting Up MongoDB with Express

#### Installing Dependencies

First, install the MongoDB driver for Node.js:

```javascript
// Terminal
npm install mongodb mongoose
```

#### Connecting with Native MongoDB Driver

```javascript
const { MongoClient } = require('mongodb');

const uri = 'mongodb://localhost:27017';
const client = new MongoClient(uri);
const dbName = 'my_application';

async function connectToDatabase() {
  try {
    await client.connect();
    console.log('Connected to MongoDB');
    return client.db(dbName);
  } catch (error) {
    console.error('MongoDB connection error:', error);
    process.exit(1);
  }
}

module.exports = { connectToDatabase, client };
```

#### Connecting with Mongoose ODM

Mongoose is an Object Data Modeling (ODM) library that provides schema-based solutions:

```javascript
const mongoose = require('mongoose');

const MONGODB_URI = process.env.MONGODB_URI || 'mongodb://localhost:27017/my_application';

async function connectDB() {
  try {
    await mongoose.connect(MONGODB_URI);
    console.log('MongoDB connected');
  } catch (error) {
    console.error('MongoDB connection error:', error);
    process.exit(1);
  }
}

module.exports = connectDB;
```

### Defining Data Models with Mongoose

Mongoose allows you to define schemas for your data:

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// Define a schema
const userSchema = new Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    trim: true,
    lowercase: true
  },
  password: {
    type: String,
    required: true
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  profile: {
    bio: String,
    location: String,
    website: String,
    social: {
      twitter: String,
      facebook: String,
      linkedin: String
    }
  },
  isActive: {
    type: Boolean,
    default: true
  }
});

// Create a model
const User = mongoose.model('User', userSchema);

module.exports = User;
```

### CRUD Operations with Mongoose

#### Creating Documents

```javascript
const User = require('../models/User');

async function createUser(userData) {
  try {
    const newUser = new User(userData);
    await newUser.save();
    return newUser;
  } catch (error) {
    throw error;
  }
}

// Alternative create method
async function createUserAlt(userData) {
  try {
    return await User.create(userData);
  } catch (error) {
    throw error;
  }
}
```

#### Reading Documents

```javascript
// Find all users
async function getAllUsers() {
  return await User.find();
}

// Find with specific criteria
async function findActiveAdmins() {
  return await User.find({
    isActive: true,
    role: 'admin'
  });
}

// Find one document
async function getUserById(id) {
  return await User.findById(id);
}

// Find with projection (select specific fields)
async function getUserProfiles() {
  return await User.find({}, 'name email profile.bio');
}

// Advanced querying with sort, limit, skip
async function getPaginatedUsers(page = 1, limit = 10) {
  const skip = (page - 1) * limit;
  return await User.find()
    .sort({ createdAt: -1 })
    .skip(skip)
    .limit(limit);
}
```

#### Updating Documents

```javascript
// Update by ID
async function updateUser(id, updateData) {
  return await User.findByIdAndUpdate(
    id,
    updateData,
    { new: true } // Return the updated document
  );
}

// Update multiple documents
async function deactivateInactiveUsers(threshold) {
  const cutoffDate = new Date(Date.now() - threshold);
  
  return await User.updateMany(
    { lastLogin: { $lt: cutoffDate } },
    { isActive: false }
  );
}
```

#### Deleting Documents

```javascript
// Delete by ID
async function deleteUser(id) {
  return await User.findByIdAndDelete(id);
}

// Delete multiple documents
async function deleteInactiveUsers() {
  return await User.deleteMany({ isActive: false });
}
```

### Handling Relationships

MongoDB offers different approaches for handling relationships:

#### Referenced Relationships (Normalization)

```javascript
// Post model
const postSchema = new Schema({
  title: String,
  content: String,
  author: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

const Post = mongoose.model('Post', postSchema);

// Querying with population
async function getPostsWithAuthors() {
  return await Post.find()
    .populate('author', 'name email') // Only include name and email fields
    .sort({ createdAt: -1 });
}
```

#### Embedded Documents (Denormalization)

```javascript
// Comment subdocument schema
const commentSchema = new Schema({
  content: String,
  author: {
    type: Schema.Types.ObjectId,
    ref: 'User'
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Post schema with embedded comments
const postSchema = new Schema({
  title: String,
  content: String,
  author: {
    type: Schema.Types.ObjectId,
    ref: 'User'
  },
  comments: [commentSchema],
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Adding a comment to a post
async function addCommentToPost(postId, comment) {
  return await Post.findByIdAndUpdate(
    postId,
    { $push: { comments: comment } },
    { new: true }
  );
}
```

### Validation and Middleware

Mongoose provides powerful validation and middleware capabilities:

#### Document Validation

```javascript
const userSchema = new Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    minlength: [2, 'Name must be at least 2 characters']
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    validate: {
      validator: function(v) {
        return /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(v);
      },
      message: props => `${props.value} is not a valid email address!`
    }
  },
  age: {
    type: Number,
    min: [18, 'Must be at least 18 years old'],
    max: [120, 'Age cannot exceed 120']
  }
});
```

#### Schema Middleware (Hooks)

```javascript
const bcrypt = require('bcrypt');

// Hash password before saving
userSchema.pre('save', async function(next) {
  // Only hash the password if it's modified (or new)
  if (!this.isModified('password')) return next();
  
  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Add a method to the schema
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};
```

### Express Routes with MongoDB

```javascript
const express = require('express');
const router = express.Router();
const User = require('../models/User');

// Get all users
router.get('/', async (req, res, next) => {
  try {
    const users = await User.find({}, '-password'); // Exclude password field
    res.json(users);
  } catch (err) {
    next(err);
  }
});

// Get user by ID
router.get('/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id, '-password');
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }
    res.json(user);
  } catch (err) {
    next(err);
  }
});

// Create new user
router.post('/', async (req, res, next) => {
  try {
    const newUser = await User.create(req.body);
    // Don't return the password in the response
    const userResponse = newUser.toObject();
    delete userResponse.password;
    
    res.status(201).json(userResponse);
  } catch (err) {
    next(err);
  }
});

// Update user
router.put('/:id', async (req, res, next) => {
  try {
    const updatedUser = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    ).select('-password');
    
    if (!updatedUser) {
      return res.status(404).json({ message: 'User not found' });
    }
    
    res.json(updatedUser);
  } catch (err) {
    next(err);
  }
});

// Delete user
router.delete('/:id', async (req, res, next) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }
    
    res.json({ message: 'User deleted successfully' });
  } catch (err) {
    next(err);
  }
});

module.exports = router;
```

### Summary

* MongoDB provides a flexible, schema-less NoSQL storage solution for Express applications
* Mongoose adds schema validation, middleware, and relationship management
* Use MongoDB's native driver for raw performance or Mongoose for developer productivity
* Referenced relationships work well for frequently changing data, while embedded documents suit data that's queried together
* Mongoose middleware enables powerful pre/post hooks for data processing
* MongoDB's query capabilities support complex filtering, sorting, and pagination
* Properly structure your data models based on access patterns rather than relationships
