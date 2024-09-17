# Authentication and Authorization in Express.js

For Rubyists familiar with authentication gems like Devise in Rails, Express.js offers several libraries and approaches for implementing authentication and authorization. We'll cover some common methods and compare them to Ruby equivalents.

## Passport.js

Passport is an authentication middleware for Node.js, similar in concept to Warden in Ruby.

1. Install Passport:
   ```bash
   npm install passport passport-local
   ```

2. Configure Passport:
   ```javascript
   const passport = require('passport');
   const LocalStrategy = require('passport-local').Strategy;

   passport.use(new LocalStrategy(
     function(username, password, done) {
       User.findOne({ username: username }, function (err, user) {
         if (err) { return done(err); }
         if (!user) { return done(null, false); }
         if (!user.verifyPassword(password)) { return done(null, false); }
         return done(null, user);
       });
     }
   ));
   ```

3. Use Passport middleware:
   ```javascript
   app.use(passport.initialize());
   app.use(passport.session());
   ```

4. Implement login route:
   ```javascript
   app.post('/login',
     passport.authenticate('local', { successRedirect: '/',
                                      failureRedirect: '/login' }));
   ```

## JSON Web Tokens (JWT)

JWTs are commonly used for API authentication. Here's a basic implementation:

1. Install required packages:
   ```bash
   npm install jsonwebtoken bcryptjs
   ```

2. Implement JWT generation:
   ```javascript
   const jwt = require('jsonwebtoken');
   const bcrypt = require('bcryptjs');

   app.post('/login', async (req, res) => {
     const user = await User.findOne({ email: req.body.email });
     if (user && await bcrypt.compare(req.body.password, user.password)) {
       const token = jwt.sign({ userId: user.id }, 'your-secret-key', { expiresIn: '1h' });
       res.json({ token });
     } else {
       res.status(400).json({ error: 'Invalid credentials' });
     }
   });
   ```

3. Implement JWT verification middleware:
   ```javascript
   function authenticateToken(req, res, next) {
     const token = req.headers['authorization'];
     if (token == null) return res.sendStatus(401);

     jwt.verify(token, 'your-secret-key', (err, user) => {
       if (err) return res.sendStatus(403);
       req.user = user;
       next();
     });
   }

   app.get('/protected', authenticateToken, (req, res) => {
     res.json({ data: 'This is protected data.' });
   });
   ```

## Session-based Authentication

For traditional web applications, you might use session-based authentication:

1. Install required packages:
   ```bash
   npm install express-session
   ```

2. Configure session middleware:
   ```javascript
   const session = require('express-session');

   app.use(session({
     secret: 'your-secret-key',
     resave: false,
     saveUninitialized: true,
     cookie: { secure: true }
   }));
   ```

3. Implement login and authentication check:
   ```javascript
   app.post('/login', async (req, res) => {
     const user = await User.findOne({ email: req.body.email });
     if (user && await bcrypt.compare(req.body.password, user.password)) {
       req.session.userId = user.id;
       res.redirect('/dashboard');
     } else {
       res.redirect('/login');
     }
   });

   function requireLogin(req, res, next) {
     if (req.session && req.session.userId) {
       return next();
     } else {
       res.redirect('/login');
     }
   }

   app.get('/dashboard', requireLogin, (req, res) => {
     res.render('dashboard');
   });
   ```

## Authorization

For role-based authorization, you might implement something like this:

```javascript
function requireAdmin(req, res, next) {
  if (req.user && req.user.role === 'admin') {
    next();
  } else {
    res.status(403).json({ error: 'Access denied' });
  }
}

app.get('/admin', requireLogin, requireAdmin, (req, res) => {
  res.render('admin');
});
```

## Conclusion

While Express.js doesn't provide built-in authentication like Devise in Rails, it offers flexibility in implementing various authentication strategies. The concepts remain similar – you're still managing user sessions, encrypting passwords, and controlling access to resources. The main difference is that you have more control over the implementation details in Express.js.
