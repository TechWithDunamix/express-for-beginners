# Serving Static Files in Express.js

In Ruby on Rails, you have the `public` folder for serving static assets. Express.js provides similar functionality with its built-in `express.static` middleware.

## Basic Usage

1. Create a `public` directory in your project root.

2. Use the `express.static` middleware:

   ```javascript
   app.use(express.static('public'));
   ```

Now, files in the `public` directory will be served directly. For example, `public/images/logo.png` will be accessible at `http://localhost:3000/images/logo.png`.

## Multiple Static Directories

You can use multiple static directories:

```javascript
app.use(express.static('public'));
app.use(express.static('files'));
```

Express.js will look for files in the order directories are set.

## Virtual Path Prefix

You can create a virtual path prefix:

```javascript
app.use('/static', express.static('public'));
```

Now, you can access files at `/static/images/logo.png`.

## Best Practices

1. Use a reverse proxy like Nginx for better performance in production.
2. Set cache headers for static files to improve performance.

## Comparison with Rails

In Rails, you might use:

```ruby
config.public_file_server.enabled = true
```

Express.js's approach is more explicit but offers more flexibility.

## Conclusion

Serving static files in Express.js is straightforward and flexible, allowing you to easily manage and serve your application's assets.
