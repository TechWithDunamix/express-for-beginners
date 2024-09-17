# Request and Response Objects in Express.js

In Express.js, the request (req) and response (res) objects are crucial for handling HTTP communications. If you're coming from Ruby, these objects are similar to the request and response objects in Rack or the controller context in Rails.

## The Request Object (req)

The req object represents the HTTP request and has properties for the request query string, parameters, body, HTTP headers, and so on.

### Common Properties and Methods

1. **req.params**: Contains route parameters (in the path portion of the URL).
   ```javascript
   // Route: /user/:id
   app.get('/user/:id', (req, res) => {
     console.log(req.params.id);
   });
   ```

2. **req.query**: An object containing a property for each query string parameter.
   ```javascript
   // URL: /search?q=express
   app.get('/search', (req, res) => {
     console.log(req.query.q); // 'express'
   });
   ```

3. **req.body**: Contains key-value pairs of data submitted in the request body. Requires body-parsing middleware.
   ```javascript
   app.use(express.json());
   app.post('/users', (req, res) => {
     console.log(req.body.username);
   });
   ```

4. **req.headers**: Contains the request headers.
   ```javascript
   app.get('/', (req, res) => {
     console.log(req.headers['user-agent']);
   });
   ```

## The Response Object (res)

The res object represents the HTTP response that an Express app sends when it gets an HTTP request.

### Common Methods

1. **res.send()**: Sends the HTTP response.
   ```javascript
   res.send('Hello World!');
   res.send({ some: 'json' });
   res.send('<p>some html</p>');
   ```

2. **res.json()**: Sends a JSON response.
   ```javascript
   res.json({ user: 'tobi' });
   ```

3. **res.status()**: Sets the HTTP status for the response.
   ```javascript
   res.status(404).send('File not found');
   ```

4. **res.redirect()**: Redirects to the specified path.
   ```javascript
   res.redirect('/home');
   ```

5. **res.render()**: Renders a view template.
   ```javascript
   res.render('index', { title: 'Express' });
   ```

## Comparison with Ruby

In Ruby's Sinatra, you might write:

```ruby
get '/hello/:name' do
  "Hello #{params[:name]}!"
end
```

The equivalent in Express would be:

```javascript
app.get('/hello/:name', (req, res) => {
  res.send(`Hello ${req.params.name}!`);
});
```

## Conclusion

Understanding the request and response objects is fundamental to working with Express.js. They provide all the information you need about incoming requests and allow you to construct appropriate responses. In the next section, we'll look at how to use template engines with Express.js.
