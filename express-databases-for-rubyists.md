# Working with Databases in Express.js

For Rubyists accustomed to ActiveRecord in Rails, working with databases in Express.js might feel a bit different. Express doesn't come with a built-in ORM, but there are several popular options available. We'll focus on Sequelize, which is similar to ActiveRecord.

## Setting Up Sequelize

1. Install Sequelize and your database driver:
   ```bash
   npm install sequelize pg pg-hstore # for PostgreSQL
   ```

2. Initialize Sequelize:
   ```javascript
   const { Sequelize } = require('sequelize');
   const sequelize = new Sequelize('database', 'username', 'password', {
     host: 'localhost',
     dialect: 'postgres'
   });
   ```

## Defining Models

In Sequelize:
```javascript
const { Model, DataTypes } = require('sequelize');

class User extends Model {}
User.init({
  username: DataTypes.STRING,
  email: DataTypes.STRING
}, { sequelize, modelName: 'user' });
```

Compared to ActiveRecord:
```ruby
class User < ApplicationRecord
  # Columns are automatically mapped
end
```

## CRUD Operations

Create:
```javascript
const jane = await User.create({ username: 'janedoe', email: 'jane@example.com' });
```

Read:
```javascript
const users = await User.findAll();
const jane = await User.findOne({ where: { username: 'janedoe' } });
```

Update:
```javascript
await jane.update({ email: 'newemail@example.com' });
```

Delete:
```javascript
await jane.destroy();
```

## Associations

In Sequelize:
```javascript
class User extends Model {}
class Post extends Model {}

User.hasMany(Post);
Post.belongsTo(User);
```

Compared to ActiveRecord:
```ruby
class User < ApplicationRecord
  has_many :posts
end

class Post < ApplicationRecord
  belongs_to :user
end
```

## Migrations

Sequelize provides a CLI for migrations, similar to Rails:

```bash
npx sequelize-cli migration:generate --name create_users
```

This generates a migration file where you define `up` and `down` methods.

## Query Interface

Sequelize:
```javascript
const { Op } = require('sequelize');
const users = await User.findAll({
  where: {
    username: {
      [Op.like]: 'john%'
    }
  }
});
```

ActiveRecord:
```ruby
users = User.where("username LIKE ?", "john%")
```

## Conclusion

While the syntax and setup differ, the concepts of working with databases in Express.js (using Sequelize) should be familiar to Rubyists. The main differences are in the explicit nature of model definition and the use of JavaScript's async/await for database operations.
