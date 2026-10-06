# 🧪 GameVault — Automated Testing, Postman & Newman, React Testing and ESLint

In this section, you will add automated testing and code-quality checks to the GameVault application.

Up to this point, you have mainly tested GameVault manually by running the backend/frontend and interacting with the application yourself. Manual testing is still useful, but as an application becomes larger, repeatedly checking every feature manually becomes inefficient.

Automated testing allows us to create tests that can be run whenever the application changes.

Our overall development process will begin to look like this:

```text
write code
-> run automated tests
-> run api tests
-> run frontend tests
-> run eslint
-> fix any problems
-> commit the changes
```

For GameVault, we will introduce four areas:

```text
Step 01 -> Backend Automated Testing
Step 02 -> Postman & Newman
Step 03 -> React Testing
Step 04 -> ESLint
```

---

# 🧪 Step 01 — Backend Automated Testing

## 1. What is automated testing?

Automated testing means writing code that checks whether other parts of your application behave correctly.

Instead of manually starting GameVault and checking every endpoint in Postman after every change, we can create tests such as:

```text
send GET /games
-> expect status 200
-> expect response to contain games
```

or:

```text
send POST /auth/login with invalid credentials
-> expect status 401
-> expect login to be rejected
```

If a later change accidentally breaks one of these features, the test should fail.

This is particularly useful when working in groups because one developer may change code that affects functionality created by another developer.

---

## 📦 2. Install the backend testing packages

Open a terminal inside:

```text
GameVault/backend
```

Install Jest and Supertest:

```bash
npm install --save-dev jest supertest
```

We are using:

Jest -> runs and manages our automated tests.

Supertest -> allows Jest to send HTTP requests directly to our Express application.

You can confirm the packages were installed by checking `package.json`.

You should see them under `devDependencies`.

---

## 📁 3. Create the test folder

Inside the backend, create:

```text
backend
├── tests
│   ├── system.test.js
│   ├── auth.test.js
│   └── games.test.js
```

You do not have to create every test immediately.

Start small and add tests as you work through the application.

The purpose of each file will be:

```text
system.test.js
-> basic api tests

auth.test.js
-> registration, login and authentication

games.test.js
-> game endpoint tests
```

---

## ⚙️ 4. Make sure Express can be tested separately

Remember that GameVault separates the Express application from the HTTPS server.

You should have something similar to:

```text
app.js
-> configures express

server.js
-> connects infrastructure and starts the server
```

This separation is important for testing.

Our tests should be able to import the Express `app` without starting the normal HTTPS server.

At the bottom of `app.js`, make sure the application is exported:

```javascript
module.exports = app;
```

This allows a test to do:

```javascript
const app = require("../app");
```

The test can now interact directly with the Express application.

---

# 🩺 5. Create your first API test

Open:

```text
tests/system.test.js
```

Add:

```javascript
const request = require("supertest");
const app = require("../app");

describe("system routes", () => {

    test("GET /health should return 200", async () => {

        const response = await request(app)
            .get("/health");

        expect(response.statusCode).toBe(200);

    });

});
```

Your route may be slightly different depending on your existing GameVault implementation. Use the route that actually exists in your project.

### What is happening here?

First:

```javascript
const request = require("supertest");
```

imports Supertest.

Then:

```javascript
const app = require("../app");
```

imports GameVault's Express application.

This:

```javascript
describe("system routes", () => {
```

creates a group of related tests.

Then:

```javascript
test("GET /health should return 200", async () => {
```

defines an individual test.

Finally:

```javascript
expect(response.statusCode).toBe(200);
```

is the actual assertion.

We are saying:

```text
send request
-> receive response
-> inspect response
-> expect status code 200
```

If the API returns something else, the test fails.

---

# ⚙️ 6. Configure the test command

Open:

```text
backend/package.json
```

Find:

```json
"scripts"
```

Add a test command.

For example:

```json
"scripts": {
    "start": "node server.js",
    "test": "jest"
}
```

Do not remove your existing scripts.

You are simply adding:

```json
"test": "jest"
```

Now run:

```bash
npm test
```

Jest should find files ending in:

```text
.test.js
```

and execute them.

A successful test should display something similar to:

```text
PASS tests/system.test.js

Tests: 1 passed, 1 total
```

---

# 🔐 7. Test authentication

Authentication is one of the most important areas of GameVault to test.

Create:

```text
tests/auth.test.js
```

A registration test could look like:

```javascript
const request = require("supertest");
const app = require("../app");

describe("authentication", () => {

    test("registers a new user", async () => {

        const response = await request(app)
            .post("/auth/register")
            .send({
                name: "test user",
                email: "testuser@example.com",
                password: "Password123!"
            });

        expect(response.statusCode).toBe(201);
        expect(response.body).toHaveProperty("user");

    });

});
```

Important: adjust the request body and route to match the registration endpoint you already created in GameVault.

Do not change your API just so that it matches this example.

---

## 8. Test invalid authentication

Testing successful requests is not enough.

Security-related applications should also test what happens when users do something incorrectly.

For example:

```javascript
test("rejects login when the password is wrong", async () => {

    const response = await request(app)
        .post("/auth/login")
        .send({
            email: "testuser@example.com",
            password: "thisiswrong"
        });

    expect(response.statusCode).toBe(401);

});
```

Your authentication tests should eventually cover cases such as:

```text
valid registration
-> success

duplicate email
-> rejected

missing registration field
-> rejected

invalid email
-> rejected

weak password
-> rejected

valid login
-> token returned

incorrect password
-> rejected

unknown user
-> rejected

protected route without token
-> rejected

protected route with valid token
-> allowed
```

These tests are particularly useful because authentication is something you do not want to accidentally break later.

---

# 🎮 9. Test the Game routes

Create:

```text
tests/games.test.js
```

Start with the public endpoints.

For example:

```javascript
const request = require("supertest");
const app = require("../app");

describe("game routes", () => {

    test("GET /games returns successfully", async () => {

        const response = await request(app)
            .get("/games");

        expect(response.statusCode).toBe(200);

    });

});
```

Again, use your actual GameVault route.

You can then test things such as:

```text
GET /games
-> returns games

GET /games/:id
-> returns the requested game

GET /games/invalid-id
-> rejected safely

POST /games without authentication
-> rejected

POST /games as normal user
-> forbidden

POST /games as admin
-> game created

PATCH /games/:id as admin
-> game updated

DELETE /games/:id as admin
-> game deleted
```

You don't need to create dozens of tests immediately. Start with the important behaviour and expand your test suite as GameVault grows.

---

# 🗄️ 10. Be careful when testing MongoDB

By this stage, GameVault uses MongoDB.

You do not want automated tests randomly changing your real application data.

For example, repeatedly running:

```text
register test user
-> create test game
-> delete game
-> register another user
```

against your normal database can leave test data everywhere.

A better long-term structure is:

```text
development environment
-> development database

testing environment
-> testing database

production environment
-> production database
```

For this learning unit, at minimum, make sure you understand that automated tests should be isolated from important application data.

---
