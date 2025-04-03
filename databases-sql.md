# SQL Databases (PostgreSQL, MySQL) with Express.js

## SQL Databases (PostgreSQL, MySQL) with Express.js

SQL databases provide robust, structured data storage for Express applications. This guide covers integrating PostgreSQL and MySQL with Express.js applications.

### Setting Up Database Connections

#### PostgreSQL with node-postgres

First, install the required dependencies:

```javascript
// Terminal
npm install pg
```

Create a database connection pool:

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  user: 'database_user',
  host: 'localhost',
  database: 'my_database',
  password: 'password',
  port: 5432,
});

// Test connection
pool.query('SELECT NOW()', (err, res) => {
  if (err) {
    console.error('Database connection error', err.stack);
  } else {
    console.log('Database connected:', res.rows[0]);
  }
});

module.exports = pool;
```

#### MySQL with mysql2

Install MySQL dependencies:

```javascript
// Terminal
npm install mysql2
```

Set up connection pool:

```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: 'localhost',
  user: 'database_user',
  password: 'password',
  database: 'my_database',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

module.exports = pool;
```

### Structuring Database Access

It's best to organize database operations into modular components:

#### Repository Pattern

```javascript
// db/repositories/userRepository.js
const pool = require('../pool');

class UserRepository {
  async findAll() {
    const { rows } = await pool.query('SELECT * FROM users');
    return rows;
  }
  
  async findById(id) {
    const { rows } = await pool.query('SELECT * FROM users WHERE id = $1', [id]);
    return rows[0];
  }
  
  async create(user) {
    const { rows } = await pool.query(
      'INSERT INTO users(name, email) VALUES($1, $2) RETURNING *',
      [user.name, user.email]
    );
    return rows[0];
  }
  
  async update(id, user) {
    const { rows } = await pool.query(
      'UPDATE users SET name = $1, email = $2 WHERE id = $3 RETURNING *',
      [user.name, user.email, id]
    );
    return rows[0];
  }
  
  async delete(id) {
    await pool.query('DELETE FROM users WHERE id = $1', [id]);
  }
}

module.exports = new UserRepository();
```

### Using the Repository in Routes

```javascript
const express = require('express');
const router = express.Router();
const userRepository = require('../db/repositories/userRepository');

// Get all users
router.get('/', async (req, res, next) => {
  try {
    const users = await userRepository.findAll();
    res.json(users);
  } catch (err) {
    next(err);
  }
});

// Get user by ID
router.get('/:id', async (req, res, next) => {
  try {
    const user = await userRepository.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }
    res.json(user);
  } catch (err) {
    next(err);
  }
});

// Create new user
router.post('/', async (req, res, next) => {
  try {
    const newUser = await userRepository.create(req.body);
    res.status(201).json(newUser);
  } catch (err) {
    next(err);
  }
});

module.exports = router;
```

### Prepared Statements and Query Parameters

Always use parameterized queries to prevent SQL injection:

#### PostgreSQL Parameterized Queries

```javascript
// GOOD: Using parameters
const { rows } = await pool.query(
  'SELECT * FROM products WHERE category = $1 AND price < $2',
  [category, maxPrice]
);

// BAD: String concatenation (vulnerable to SQL injection)
const query = `SELECT * FROM products WHERE category = '${category}'`; // Don't do this!
```

#### MySQL Parameterized Queries

```javascript
// Using mysql2/promise
const [rows] = await pool.execute(
  'SELECT * FROM products WHERE category = ? AND price < ?',
  [category, maxPrice]
);
```

### Transactions

Transactions ensure database operations are executed as a single unit:

#### PostgreSQL Transactions

```javascript
async function transferFunds(fromAccountId, toAccountId, amount) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [amount, fromAccountId]
    );
    
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [amount, toAccountId]
    );
    
    await client.query('COMMIT');
    return true;
  } catch (e) {
    await client.query('ROLLBACK');
    throw e;
  } finally {
    client.release();
  }
}
```

#### MySQL Transactions

```javascript
async function transferFunds(fromAccountId, toAccountId, amount) {
  const connection = await pool.getConnection();
  
  try {
    await connection.beginTransaction();
    
    await connection.execute(
      'UPDATE accounts SET balance = balance - ? WHERE id = ?',
      [amount, fromAccountId]
    );
    
    await connection.execute(
      'UPDATE accounts SET balance = balance + ? WHERE id = ?',
      [amount, toAccountId]
    );
    
    await connection.commit();
    return true;
  } catch (e) {
    await connection.rollback();
    throw e;
  } finally {
    connection.release();
  }
}
```

### Migrations and Schema Management

Managing database schema changes is crucial for application evolution:

#### Using Knex.js for Migrations

```javascript
// Terminal
npm install knex

// Create a migration
npx knex migrate:make create_users_table
```

Example migration file:

```javascript
// migrations/20230125_create_users_table.js
exports.up = function(knex) {
  return knex.schema.createTable('users', table => {
    table.increments('id').primary();
    table.string('name').notNullable();
    table.string('email').notNullable().unique();
    table.string('password_hash').notNullable();
    table.timestamps(true, true);
  });
};

exports.down = function(knex) {
  return knex.schema.dropTable('users');
};
```

Run migrations:

```javascript
// Terminal
npx knex migrate:latest
```

### Query Builders vs. Raw SQL

Query builders provide a structured way to build complex SQL queries:

#### Using Knex.js as a Query Builder

```javascript
const knex = require('knex')({
  client: 'pg',
  connection: {
    host: 'localhost',
    user: 'database_user',
    password: 'password',
    database: 'my_database'
  }
});

async function getProductsWithFilters(filters) {
  const query = knex('products')
    .select('*');
  
  if (filters.category) {
    query.where('category', filters.category);
  }
  
  if (filters.minPrice) {
    query.where('price', '>=', filters.minPrice);
  }
  
  if (filters.maxPrice) {
    query.where('price', '<=', filters.maxPrice);
  }
  
  if (filters.sortBy) {
    query.orderBy(filters.sortBy, filters.sortDir || 'asc');
  }
  
  return await query;
}
```

### Summary

* Connection pooling is essential for efficient database operations in Express
* Repository pattern helps organize database access logic
* Always use parameterized queries to prevent SQL injection
* Transactions maintain data integrity for related operations
* Migration tools like Knex.js help manage database schema changes
* Query builders provide a structured way to create complex SQL queries
* Choose PostgreSQL for feature-rich applications or MySQL for simpler use cases
