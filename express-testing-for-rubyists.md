# Testing Express.js Applications

For Rubyists accustomed to testing Rails applications with RSpec or Minitest, testing Express.js applications will feel familiar in concept, though the tools and syntax differ. We'll explore how to test Express.js applications using Mocha, Chai, and Supertest.

## Setting Up the Testing Environment

1. Install the necessary packages:
   ```bash
   npm install --save-dev mocha chai supertest
   ```

2. Add a test script to your `package.json`:
   ```json
   "scripts": {
     "test": "mocha --exit"
   }
   ```

## Writing Tests

Here's a basic structure for your tests:

```javascript
const chai = require('chai');
const expect = chai.expect;
const request = require('supertest');
const app = require('../app'); // Your Express app

describe('User API', () => {
  it('should GET all users', (done) => {
    request(app)
      .get('/api/users')
      .end((err, res) => {
        expect(res.statusCode).to.equal(200);
        expect(res.body).to.be.an('array');
        done();
      });
  });

  it('should POST a new user', (done) => {
    const newUser = { name: 'John Doe', email: 'john@example.com' };
    request(app)
      .post('/api/users')
      .send(newUser)
      .end((err, res) => {
        expect(res.statusCode).to.equal(201);
        expect(res.body).to.have.property('name', newUser.name);
        done();
      });
  });
});
```

This is similar to how you might structure RSpec tests in Rails:

```ruby
RSpec.describe UsersController, type: :request do
  describe "GET /api/users" do
    it "returns all users" do
      get "/api/users"
      expect(response).to have_http_status(200)
      expect(JSON.parse(response.body)).to be_an(Array)
    end
  end

  describe "POST /api/users" do
    it "creates a new user" do
      post "/api/users", params: { user: { name: "John Doe", email: "john@example.com" } }
      expect(response).to have_http_status(201)
      expect(JSON.parse(response.body)["name"]).to eq("John Doe")
    end
  end
end
```

## Mocking and Stubbing

For mocking in JavaScript, you can use libraries like Sinon:

```javascript
const sinon = require('sinon');
const User = require('../models/user');

describe('User Controller', () => {
  it('should return all users', (done) => {
    const stubValue = [{ name: 'John Doe' }, { name: 'Jane Doe' }];
    const stub = sinon.stub(User, 'find').returns(stubValue);

    request(app)
      .get('/api/users')
      .end((err, res) => {
        expect(res.statusCode).to.equal(200);
        expect(stub.calledOnce).to.be.true;
        expect(res.body).to.eql(stubValue);
        stub.restore();
        done();
      });
  });
});
```

This is similar to how you might use RSpec mocks in Rails:

```ruby
RSpec.describe UsersController, type: :controller do
  it "returns all users" do
    users = [User.new(name: "John Doe"), User.new(name: "Jane Doe")]
    allow(User).to receive(:all).and_return(users)

    get :index
    expect(response).to have_http_status(200)
    expect(JSON.parse(response.body)).to eq(users.as_json)
  end
end
```

## Testing Middleware

You can test middleware functions separately:

```javascript
const { expect } = require('chai');
const sinon = require('sinon');

const authMiddleware = require('../middleware/auth');

describe('Auth Middleware', () => {
  it('should call next() if authentication succeeds', () => {
    const req = { headers: { authorization: 'valid-token' } };
    const res = {};
    const next = sinon.spy();

    authMiddleware(req, res, next);

    expect(next.calledOnce).to.be.true;
  });

  it('should return 401 if no token provided', () => {
    const req = { headers: {} };
    const res = {
      status: sinon.stub().returnsThis(),
      json: sinon.spy()
    };
    const next = sinon.spy();

    authMiddleware(req, res, next);

    expect(res.status.calledWith(401)).to.be.true;
    expect(res.json.calledWith({ error: 'No token provided' })).to.be.true;
    expect(next.called).to.be.false;
  });
});
```

## Conclusion

While the syntax and specific tools differ, the concepts of testing in Express.js should be familiar to Rubyists. Both environments allow for unit testing, integration testing, and mocking/stubbing. The main differences lie in the asynchronous nature of Node.js, which often requires the use of callbacks or Promises in tests.
