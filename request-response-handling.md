# Request Response Handling

## Request and Response Handling

Express.js provides enhanced objects for handling HTTP requests and responses, making server-side development significantly more straightforward than using Node.js's native HTTP module. This document explains how to work with request (`req`) and response (`res`) objects effectively.

### The Request Object

The request object (`req`) represents the HTTP request and contains properties for the request query string, parameters, body, HTTP headers, and more.

#### Common Request Properties and Methods

**req.params**

Contains route parameters (named URL segments):

```javascript
// Route: /users/:userId/books/:bookId
app.get('/users/:userId/books/:bookId', (req, res) => {
  console.log(req.params.userId); // "34"
  console.log(req.params.bookId);  // "8989"
});
```

**req.query**

Contains the query string parameters as an object:

```javascript
// URL: /search?term=express&sort=name
app.get('/search', (req, res) => {
  console.log(req.query.term);  // "express"
  console.log(req.query.sort);  // "name"
});
```

**req.body**

Contains data submitted in the request body. This requires middleware like `express.json()` or `express.urlencoded()`:

```javascript
const express = require('express');
const app = express();

// Middleware to parse JSON bodies
app.use(express.json());
// Middleware to parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

app.post('/users', (req, res) => {
  console.log(req.body.username); // "johndoe"
  console.log(req.body.email);    // "john@example.com"
});
```

**req.headers**

Contains the request headers:

```javascript
app.get('/', (req, res) => {
  console.log(req.headers['user-agent']);
  console.log(req.headers.authorization);
});
```

**req.cookies**

Contains cookies sent by the client. Requires the `cookie-parser` middleware:

```javascript
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();

app.use(cookieParser());

app.get('/', (req, res) => {
  console.log(req.cookies.name); // "value"
});
```

**req.path**

Contains the path part of the URL:

```javascript
// URL: /users?sort=desc
app.get('/users', (req, res) => {
  console.log(req.path); // "/users"
});
```

**req.protocol**

Contains the request protocol:

```javascript
app.get('/', (req, res) => {
  console.log(req.protocol); // "http" or "https"
});
```

**req.method**

Contains the HTTP method of the request:

```javascript
app.use((req, res, next) => {
  console.log(req.method); // "GET", "POST", etc.
  next();
});
```

**req.get()**

Gets the specified HTTP request header field:

```javascript
app.get('/', (req, res) => {
  console.log(req.get('Content-Type'));
  console.log(req.get('User-Agent'));
});
```

### The Response Object

The response object (`res`) represents the HTTP response that an Express app sends when it receives an HTTP request.

#### Common Response Methods

**res.send()**

Sends the HTTP response with various types of data:

```javascript
// Send a string
res.send('Hello World!');

// Send an HTML string
res.send('<h1>Hello World!</h1>');

// Send JSON
res.send({ user: 'john', id: 1 });

// Send an array
res.send([1, 2, 3]);

// Send a Buffer
res.send(Buffer.from('whoop'));
```

**res.json()**

Sends a JSON response:

```javascript
res.json({ user: 'john', id: 1 });

// Also works with arrays and other values
res.json([1, 2, 3]);
res.json(null);
res.json({ success: true });
```

**res.status()**

Sets the HTTP status code for the response:

```javascript
res.status(404).send('Not Found');
res.status(201).json({ created: true });
```

**res.sendStatus()**

Sets the response status code and sends the status as the response body:

```javascript
res.sendStatus(200); // equivalent to res.status(200).send('OK')
res.sendStatus(404); // equivalent to res.status(404).send('Not Found')
res.sendStatus(500); // equivalent to res.status(500).send('Internal Server Error')
```

**res.render()**

Renders a view template and sends the rendered HTML:

```javascript
res.render('index', { title: 'Express', message: 'Hello' });
```

**res.redirect()**

Redirects to the specified path or URL:

```javascript
// Redirect to a path
res.redirect('/users');

// Redirect to a full URL
res.redirect('https://example.com');

// Redirect back to the referring page
res.redirect('back');

// Set a different status code (default is 302)
res.redirect(301, '/new-page');
```

**res.set()**

Sets response headers:

```javascript
res.set('Content-Type', 'text/plain');
res.set({
  'Content-Type': 'text/plain',
  'X-Custom-Header': 'CustomValue'
});
```

**res.cookie()**

Sets cookie name and value:

```javascript
// Set a cookie
res.cookie('name', 'value');

// With options
res.cookie('name', 'value', { 
  maxAge: 900000, 
  httpOnly: true,
  secure: true
});
```

**res.clearCookie()**

Clears the specified cookie:

```javascript
res.clearCookie('name');
```

**res.download()**

Prompts the user to download a file:

```javascript
res.download('/path/to/file.pdf');
res.download('/path/to/file.pdf', 'custom-filename.pdf');
```

**res.sendFile()**

Sends a file as an octet stream:

```javascript
res.sendFile('/path/to/file.pdf');
```

**res.end()**

Ends the response process without any data:

```javascript
res.end();
```

### Practical Examples

#### Creating a Basic API

```javascript
const express = require('express');
const app = express();

app.use(express.json());

const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' }
];

// Get all users
app.get('/api/users', (req, res) => {
  res.json(users);
});

// Get user by id
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) {
    return res.status(404).json({ message: 'User not found' });
  }
  res.json(user);
});

// Create a new user
app.post('/api/users', (req, res) => {
  const newUser = {
    id: users.length + 1,
    name: req.body.name
  };
  
  users.push(newUser);
  res.status(201).json(newUser);
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

#### Form Handling

```javascript
const express = require('express');
const app = express();

app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

app.get('/form', (req, res) => {
  res.send(`
    <html>
      <head><title>Form Example</title></head>
      <body>
        <form action="/submit-form" method="POST">
          <input name="name" placeholder="Name" /><br />
          <input name="email" placeholder="Email" /><br />
          <button type="submit">Submit</button>
        </form>
      </body>
    </html>
  `);
});

app.post('/submit-form', (req, res) => {
  const { name, email } = req.body;
  res.send(`Form submitted with name: ${name} and email: ${email}`);
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

#### File Upload Response

```javascript
const express = require('express');
const multer = require('multer');
const app = express();

const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), (req, res) => {
  res.json({
    filename: req.file.filename,
    originalname: req.file.originalname,
    size: req.file.size,
    mimetype: req.file.mimetype
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

#### Content Negotiation

```javascript
app.get('/data', (req, res) => {
  const data = {
    name: 'John',
    age: 30,
    city: 'New York'
  };
  
  res.format({
    // HTML response
    'text/html': () => {
      res.send(`
        <html>
          <body>
            <h1>${data.name}</h1>
            <p>Age: ${data.age}</p>
            <p>City: ${data.city}</p>
          </body>
        </html>
      `);
    },
    
    // JSON response
    'application/json': () => {
      res.json(data);
    },
    
    // Plain text response
    'text/plain': () => {
      res.send(`Name: ${data.name}\nAge: ${data.age}\nCity: ${data.city}`);
    },
    
    // Default response
    default: () => {
      res.status(406).send('Not Acceptable');
    }
  });
});
```

### Summary

* The request (`req`) object provides information about the HTTP request
* Key request properties include `req.params`, `req.query`, `req.body`, and `req.headers`
* The response (`res`) object is used to send data back to the client
* Common response methods include `res.send()`, `res.json()`, `res.status()`, and `res.redirect()`
* Middleware is required to parse request bodies and handle cookies
* Express provides methods for serving different content types, files, and templates
* Proper use of status codes and content types is important for RESTful API design
* Express enables content negotiation through the `res.format()` method
