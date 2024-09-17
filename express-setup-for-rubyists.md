# Setting Up Your Express.js Environment

As a Rubyist, you're probably familiar with setting up development environments for Ruby projects. Setting up an Express.js environment is quite similar, but with some JavaScript-specific tools. Let's walk through the process step by step.

## Prerequisites

Before we start, make sure you have the following installed:

1. **Node.js**: This is the JavaScript runtime that Express.js runs on. It's similar to having Ruby installed for your Ruby projects.
2. **npm** (Node Package Manager): This comes with Node.js and is similar to RubyGems in the Ruby ecosystem.

## Step 1: Create a New Project

1. Open your terminal and navigate to where you want to create your project.
2. Create a new directory and navigate into it:

   ```bash
   mkdir my-express-app
   cd my-express-app
   ```

3. Initialize a new Node.js project:

   ```bash
   npm init -y
   ```

   This is similar to creating a new Ruby project, but instead of a Gemfile, it creates a `package.json` file.

## Step 2: Install Express

Now, let's install Express. This is similar to adding a gem to your Ruby project:

```bash
npm install express
```

This command does two things:
1. Adds Express to your project dependencies in `package.json`
2. Downloads Express and its dependencies into a `node_modules` folder

## Step 3: Create Your First Express App

Create a new file called `app.js` (or `index.js`) and add the following code:

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

This is similar to creating a basic Sinatra app in Ruby.

## Step 4: Run Your App

To run your app, use the following command:

```bash
node app.js
```

Visit `http://localhost:3000` in your browser, and you should see "Hello World!".

## Step 5: Install Nodemon (Optional)

In Ruby, you might use tools like Guard or Rerun to automatically restart your server when files change. In the Node.js ecosystem, we often use Nodemon for this purpose.

Install Nodemon as a development dependency:

```bash
npm install nodemon --save-dev
```

Then, add a script to your `package.json`:

```json
"scripts": {
  "start": "node app.js",
  "dev": "nodemon app.js"
}
```

Now you can run your app with auto-restart using:

```bash
npm run dev
```

## Step 6: Set Up a Git Repository

Just like with Ruby projects, it's a good idea to use version control:

```bash
git init
echo "node_modules/" >> .gitignore
git add .
git commit -m "Initial commit"
```

## Conclusion

You now have a basic Express.js environment set up! This environment is analogous to a basic Sinatra or Rails setup in the Ruby world. In the next guide, we'll dive into routing in Express.js.
