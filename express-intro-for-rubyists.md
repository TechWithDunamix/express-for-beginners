# Introduction to Express.js for Rubyists

Welcome to this guide on Express.js for Rubyists! If you're coming from a Ruby background, particularly if you've worked with Sinatra or Ruby on Rails, you'll find many familiar concepts in Express.js. This guide will help you transition your knowledge to the world of Node.js and Express.

## What is Express.js?

Express.js is a minimal and flexible web application framework for Node.js. It's often described as the Sinatra of Node.js world, providing a lightweight yet powerful set of features for building single-page, multi-page, and hybrid web applications.

## Express.js vs. Ruby Frameworks

To help you understand Express.js in terms you're already familiar with, let's compare it to some Ruby frameworks:

1. **Express.js vs. Sinatra**: 
   - Both are minimalist and unopinionated
   - Both allow you to quickly create web applications with routes
   - Express middleware is similar to Sinatra's filters and helpers

2. **Express.js vs. Ruby on Rails**:
   - Express is much more lightweight than Rails
   - Express doesn't follow convention over configuration like Rails
   - You have more control over the structure and components in Express

## Key Concepts

1. **Routing**: Similar to Sinatra, you define routes with HTTP methods:

   ```javascript
   app.get('/', (req, res) => {
     res.send('Hello World!');
   });
   ```

2. **Middleware**: Functions that have access to the request and response objects. This is similar to Rack middleware in the Ruby world.

3. **Template Engines**: Express can use various template engines, similar to how Rails uses ERB or Haml.

4. **Static File Serving**: Express can serve static files, much like the `public` folder in Rails.

## A Simple Express.js Application

Here's a basic Express.js application, with comments relating it to Ruby concepts:

```javascript
const express = require('express');
const app = express();
const port = 3000;

// This is like defining a route in Sinatra
app.get('/', (req, res) => {
  res.send('Hello World!');
});

// This is similar to Rails' server startup
app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

In the next sections, we'll dive deeper into each of these concepts and show you how to leverage your Ruby knowledge in the Express.js ecosystem.
