# Templating Engine

## Templating Engines

Templating engines enable server-side rendering of dynamic HTML pages in Express.js applications. They allow you to generate HTML with variable data and include reusable components such as headers, footers, and navigation elements.

### Introduction to Templating

In Express applications, a templating engine replaces variables in a template file with actual values and transforms the template into an HTML file that is sent to the client. This approach offers several advantages:

* Server-side rendering for faster initial page loads
* Better SEO compared to client-side rendering
* Consistent UI structure across pages
* Reduced client-side JavaScript dependency

### Popular Templating Engines

Express works with many templating engines. The most popular ones include:

* EJS (Embedded JavaScript)
* Pug (formerly Jade)
* Handlebars
* Nunjucks

### Setting Up a Templating Engine

#### Installing the Engine

First, install your chosen templating engine. For this guide, we'll use EJS:

```
npm install ejs
```

#### Configuring Express

In your Express application, set up the templating engine:

```javascript
const express = require('express');
const app = express();
const path = require('path');

// Set the view engine
app.set('view engine', 'ejs');

// Set the directory for views
app.set('views', path.join(__dirname, 'views'));

// Routes
app.get('/', (req, res) => {
  res.render('index', { title: 'Home Page', message: 'Welcome to Express!' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Working with EJS Templates

EJS is a templating engine that lets you generate HTML markup with plain JavaScript. It uses `<% %>` tags to execute JavaScript and `<%= %>` tags to output values.

#### Basic EJS Template

Create a file `views/index.ejs`:

```html
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
  <link rel="stylesheet" href="/css/style.css">
</head>
<body>
  <h1><%= message %></h1>
  
  <% if (users && users.length > 0) { %>
    <ul>
      <% users.forEach(function(user) { %>
        <li><%= user.name %></li>
      <% }); %>
    </ul>
  <% } else { %>
    <p>No users found</p>
  <% } %>
</body>
</html>
```

#### Rendering with Data

```javascript
app.get('/users', (req, res) => {
  const users = [
    { name: 'John Doe', email: 'john@example.com' },
    { name: 'Jane Smith', email: 'jane@example.com' },
    { name: 'Bob Johnson', email: 'bob@example.com' }
  ];
  
  res.render('index', { 
    title: 'User List', 
    message: 'Our User Directory', 
    users: users 
  });
});
```

### Template Partials and Layouts

Most templating engines support partials (reusable template fragments) to maintain DRY (Don't Repeat Yourself) code.

#### Creating Partials with EJS

Create partial templates:

```
views/
  ├── partials/
  │   ├── header.ejs
  │   └── footer.ejs
  └── index.ejs
```

`views/partials/header.ejs`:

```html
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
  <link rel="stylesheet" href="/css/style.css">
</head>
<body>
  <header>
    <nav>
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
  </header>
```

`views/partials/footer.ejs`:

```html
  <footer>
    <p>&copy; <%= new Date().getFullYear() %> My Express App</p>
  </footer>
</body>
</html>
```

`views/index.ejs`:

```html
<%- include('partials/header') %>

<main>
  <h1><%= message %></h1>
  
  <% if (users && users.length > 0) { %>
    <ul>
      <% users.forEach(function(user) { %>
        <li><%= user.name %></li>
      <% }); %>
    </ul>
  <% } else { %>
    <p>No users found</p>
  <% } %>
</main>

<%- include('partials/footer') %>
```

### Working with Pug Templates

Pug (formerly Jade) uses indentation instead of closing tags, making it more concise.

#### Installing Pug

```
npm install pug
```

#### Configuring Pug

```javascript
app.set('view engine', 'pug');
app.set('views', path.join(__dirname, 'views'));
```

#### Basic Pug Template

Create a file `views/index.pug`:

```pug
doctype html
html
  head
    title= title
    link(rel="stylesheet", href="/css/style.css")
  body
    h1= message
    
    if users && users.length > 0
      ul
        each user in users
          li= user.name
    else
      p No users found
```

### Working with Handlebars

Handlebars focuses on simplicity and minimal logic in templates.

#### Installing Handlebars for Express

```
npm install express-handlebars
```

#### Configuring Handlebars

```javascript
const exphbs = require('express-handlebars');

// Configure handlebars
app.engine('handlebars', exphbs({ defaultLayout: 'main' }));
app.set('view engine', 'handlebars');
```

#### Basic Handlebars Template Structure

```
views/
  ├── layouts/
  │   └── main.handlebars
  └── home.handlebars
```

`views/layouts/main.handlebars`:

```html
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
  <link rel="stylesheet" href="/css/style.css">
</head>
<body>
  {{{body}}}
  
  <footer>
    <p>&copy; {{currentYear}} My Express App</p>
  </footer>
</body>
</html>
```

`views/home.handlebars`:

```html
<h1>{{message}}</h1>

{{#if users.length}}
  <ul>
    {{#each users}}
      <li>{{this.name}}</li>
    {{/each}}
  </ul>
{{else}}
  <p>No users found</p>
{{/if}}
```

Rendering:

```javascript
app.get('/', (req, res) => {
  res.render('home', {
    title: 'Home Page',
    message: 'Welcome to Express!',
    users: users,
    currentYear: new Date().getFullYear()
  });
});
```

### Passing Custom Functions to Templates

Templates can also use custom functions passed in the render object:

```javascript
app.get('/posts', (req, res) => {
  const posts = [
    { title: 'First Post', body: 'This is the first post.', date: new Date('2023-01-15') },
    { title: 'Second Post', body: 'This is the second post.', date: new Date('2023-02-20') }
  ];
  
  res.render('posts', {
    title: 'Blog Posts',
    posts: posts,
    // Helper function to format dates
    formatDate: (date) => {
      return date.toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    }
  });
});
```

In EJS template:

```html
<% posts.forEach(function(post) { %>
  <article>
    <h2><%= post.title %></h2>
    <time><%= formatDate(post.date) %></time>
    <p><%= post.body %></p>
  </article>
<% }); %>
```

### Caching Templates

For production applications, enable template caching to improve performance:

```javascript
// Enable view caching in production
if (process.env.NODE_ENV === 'production') {
  app.enable('view cache');
} else {
  app.disable('view cache');
}
```

### Error Handling in Templates

Create an error template (`views/error.ejs`):

```html
<!DOCTYPE html>
<html>
<head>
  <title>Error</title>
</head>
<body>
  <h1>Error: <%= status %></h1>
  <p><%= message %></p>
  <% if (stack && process.env.NODE_ENV !== 'production') { %>
    <pre><%= stack %></pre>
  <% } %>
</body>
</html>
```

Error handler:

```javascript
app.use((err, req, res, next) => {
  res.status(err.status || 500);
  res.render('error', {
    status: err.status || 500,
    message: err.message,
    stack: err.stack
  });
});
```

### Summary

* Templating engines provide server-side rendering of dynamic content
* Popular engines include EJS, Pug, Handlebars, and Nunjucks
* Each engine has its own syntax but similar functionality
* Templates can access data passed from Express routes
* Partials and layouts help maintain DRY code principles
* Custom functions can be passed to templates for formatting or other operations
* Template caching improves performance in production environments
* Error handling can be implemented using specialized templates
