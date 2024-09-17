# Template Engines in Express.js

For Rubyists accustomed to ERB, Haml, or Slim in Rails, Express.js offers similar templating capabilities through various template engines. We'll focus on EJS (Embedded JavaScript), which is similar to ERB.

## Setting Up EJS

1. Install EJS:
   ```bash
   npm install ejs
   ```

2. Set EJS as the view engine in your Express app:
   ```javascript
   app.set('view engine', 'ejs');
   ```

## Using EJS

Create a file named `index.ejs` in a `views` directory:

```html
<h1><%= title %></h1>
<p>Welcome to <%= title %></p>
```

In your route:

```javascript
app.get('/', (req, res) => {
  res.render('index', { title: 'Express' });
});
```

## EJS vs ERB

EJS:
```html
<% if (user) { %>
  <h2><%= user.name %></h2>
<% } %>
```

ERB:
```erb
<% if user %>
  <h2><%= user.name %></h2>
<% end %>
```

## Layouts

EJS doesn't have built-in layout support like Rails, but you can achieve similar functionality:

```html
<!-- views/layout.ejs -->
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
  </head>
  <body>
    <%- body %>
  </body>
</html>
```

Use the `express-ejs-layouts` package to implement this.

## Conclusion

While the syntax differs slightly, the concept of template engines in Express.js should be familiar to Rubyists. EJS provides a comfortable transition for those used to ERB.
