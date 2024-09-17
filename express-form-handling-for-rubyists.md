# Handling Form Data in Express.js

For Rubyists accustomed to handling form submissions in Rails or Sinatra, Express.js offers similar capabilities with some key differences. Let's explore how to handle form data in Express.js.

## Setting Up

First, you'll need to use middleware to parse form data. Express.js 4.16.0 and later have built-in middleware for this:

```javascript
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
```

This is similar to Rails' built-in support for parsing form data.

## Handling a Form Submission

HTML Form:
```html
<form action="/submit" method="POST">
  <input type="text" name="username">
  <input type="email" name="email">
  <button type="submit">Submit</button>
</form>
```

Express.js Route:
```javascript
app.post('/submit', (req, res) => {
  const { username, email } = req.body;
  console.log(`Received: ${username}, ${email}`);
  res.send('Form submitted successfully');
});
```

In Rails, this might look like:
```ruby
post '/submit' do
  username = params[:username]
  email = params[:email]
  # process the data
end
```

## File Uploads

For file uploads, you'll need additional middleware like `multer`:

```javascript
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), (req, res) => {
  console.log(req.file);
  res.send('File uploaded successfully');
});
```

This is similar to how you might use `CarrierWave` or `Active Storage` in Rails.

## Form Validation

Express.js doesn't have built-in form validation like Rails does with Active Record validations. You'll typically use a library like `express-validator` or implement your own validation:

```javascript
const { check, validationResult } = require('express-validator');

app.post('/submit', [
  check('username').notEmpty().withMessage('Username is required'),
  check('email').isEmail().withMessage('Invalid email')
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // Process the valid form data
});
```

## CSRF Protection

For CSRF protection, you can use the `csurf` middleware:

```javascript
const csrf = require('csurf');
app.use(csrf());

app.get('/form', (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});
```

Then in your form:
```html
<input type="hidden" name="_csrf" value="<%= csrfToken %>">
```

This is similar to Rails' built-in CSRF protection.

## Conclusion

While the syntax and specific libraries differ, the concepts of form handling in Express.js should be familiar to Rubyists. The main differences are in the middleware-based approach and the need for explicit setup of features that might be more automatic in Rails.
