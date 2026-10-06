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

# 📮 Step 02 — Postman & Newman

You have already used Postman throughout GameVault.

So far, you have probably been doing something like:

```text
open Postman
-> choose request
-> click Send
-> inspect response
```

Now we are going to make those tests reusable and executable automatically.

---

# 📁 11. Organise your Postman collection

Create or update a collection called:

```text
GameVault API
```

Organise the requests into folders.

For example:

```text
GameVault API

├── System
│   └── Health Check
│
├── Authentication
│   ├── Register - Valid
│   ├── Register - Duplicate Email
│   ├── Login - Valid
│   └── Login - Invalid Password
│
├── Games
│   ├── Get Games
│   ├── Get Game
│   ├── Create Game
│   ├── Update Game
│   └── Delete Game
│
└── Protected Routes
    ├── Profile - Valid Token
    └── Profile - No Token
```

Avoid keeping all your test scenarios inside one request and manually changing the body every time.

Each important scenario should be a saved request.

---

# 🌐 12. Create a Postman environment

Instead of writing this repeatedly:

```text
https://localhost:4000
```

create a Postman environment variable:

```text
baseUrl
```

Set its value to:

```text
https://localhost:4000
```

Then requests can use:

```text
{{baseUrl}}/auth/login
```

or:

```text
{{baseUrl}}/games
```

This makes the collection easier to move between environments.

---

# 🔑 13. Automatically save the JWT

We don't want to manually copy the JWT every time we log in.

Open your Login - Valid request.

Go to:

```text
Scripts
-> Post-response
```

Depending on your Postman version, this may also appear as the Tests area.

Add:

```javascript
const response = pm.response.json();

pm.environment.set("token", response.token);
```

If your GameVault response looks like:

```json
{
    "data": {
        "token": "..."
    }
}
```

then use:

```javascript
const response = pm.response.json();

pm.environment.set("token", response.data.token);
```

Use whichever matches your API response.

Now the flow becomes:

```text
login
-> api returns jwt
-> postman saves jwt
-> later requests use saved jwt
```

For a protected request, add:

```text
Authorization: Bearer {{token}}
```

You no longer need to copy and paste tokens manually.

---

# ✅ 14. Add Postman tests

Postman can automatically inspect responses.

For example, your successful login request could contain:

```javascript
pm.test("status should be 200", function () {
    pm.response.to.have.status(200);
});
```

You could also verify that a token exists:

```javascript
pm.test("login should return a token", function () {

    const body = pm.response.json();

    pm.expect(body).to.have.property("token");

});
```

Again, change this if your token is inside `data`.

For an invalid login:

```javascript
pm.test("invalid login should be rejected", function () {
    pm.response.to.have.status(401);
});
```

This changes Postman from:

```text
send request
-> student visually checks response
```

into:

```text
send request
-> postman checks response
-> test passes or fails
```

---

# 🧪 15. Add tests to the important requests

At minimum, your collection should automatically check:

```text
registration
-> correct success status
-> user returned
-> password/password hash not returned

login
-> correct success status
-> jwt returned

invalid login
-> 401

games
-> correct response status
-> expected data returned

protected route without jwt
-> 401

protected route with jwt
-> 200

admin-only action as normal user
-> 403

admin-only action as admin
-> succeeds
```

You can add more tests depending on your implementation.

---

# 📤 16. Export the collection

In Postman, export your collection as:

```text
Collection v2.1
```

Place it somewhere sensible in GameVault:

```text
GameVault
├── backend
├── frontend
└── postman
    └── GameVault.postman_collection.json
```

If you use a Postman environment, export that as well:

```text
postman
├── GameVault.postman_collection.json
└── GameVault.postman_environment.json
```

Do not export real secrets or real authentication tokens into GitHub.

Check the exported files before committing them.

---

# 🤖 17. Install Newman

Postman is mainly a graphical application.

Newman allows us to run a Postman collection from the terminal.

Install it:

```bash
npm install --save-dev newman
```

You can install it in the backend project so everyone receives the same version through `npm install`.

---

# ▶️ 18. Run the collection with Newman

First make sure the GameVault backend is running.

For example:

```bash
npm start
```

Then open another terminal.

From the project directory, run:

```bash
npx newman run postman/GameVault.postman_collection.json
```

If you exported an environment:

```bash
npx newman run postman/GameVault.postman_collection.json -e postman/GameVault.postman_environment.json
```

Newman will execute the collection from top to bottom.

You should see a report showing:

```text
requests
assertions
passed tests
failed tests
```

This means we can now test the API without manually clicking through every Postman request.

The process becomes:

```text
start backend
-> run newman
-> collection executes
-> assertions execute
-> terminal reports failures
```

---

# 🔒 19. HTTPS certificates and Newman

GameVault uses a locally generated self-signed HTTPS certificate.

Because the certificate isn't issued by a public certificate authority, Newman may reject it during local development.

For local development only, you can run:

```bash
npx newman run postman/GameVault.postman_collection.json --insecure
```

If you also need the environment:

```bash
npx newman run postman/GameVault.postman_collection.json -e postman/GameVault.postman_environment.json --insecure
```

`--insecure` is appropriate here because we deliberately created a local self-signed certificate.

It is not something you should automatically use against a deployed production application.

---

# ⚛️ Step 03 — React Testing

We also need tests for the GameVault React frontend.

Backend tests answer questions such as:

> Does `/auth/login` return the correct response?

Frontend tests answer different questions:

> Does the Login form display correctly?

> Can the user enter an email and password?

> Does the interface respond correctly when the user clicks Login?

---

# 📦 20. Install React testing packages

Open:

```text
GameVault/frontend
```

For a Vite React project, install:

```bash
npm install --save-dev vitest jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

These packages have different responsibilities:

Vitest -> test runner.

React Testing Library -> renders and interacts with React components during tests.

jest-dom -> gives us useful DOM assertions such as `toBeInTheDocument()`.

user-event -> simulates user behaviour such as typing and clicking.

jsdom -> provides a browser-like DOM while tests run in Node.

---

# ⚙️ 21. Configure Vitest

Open your Vite configuration file.

Usually:

```text
frontend/vite.config.js
```

Make sure it contains a test configuration.

For example:

```javascript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
    plugins: [react()],

    test: {
        environment: "jsdom",
        setupFiles: "./src/test/setup.js"
    }
});
```

Now create:

```text
src
└── test
    └── setup.js
```

Inside:

```javascript
import "@testing-library/jest-dom";
```

This loads the additional DOM assertions before tests execute.

---

# 📝 22. Add the frontend test script

Open the frontend `package.json`.

Add:

```json
"scripts": {
    "test": "vitest"
}
```

Keep all existing scripts such as:

```text
dev
build
lint
```

You are only adding the new test command.

Run:

```bash
npm test
```

Vitest should start.

For a single run rather than watch mode, you can use:

```bash
npm test -- --run
```

---

# 🧪 23. Create your first React test

Suppose GameVault already has:

```text
src/components/LoginForm.jsx
```

Create:

```text
src/components/LoginForm.test.jsx
```

A simple test might be:

```javascript
import { render, screen } from "@testing-library/react";
import { describe, expect, test } from "vitest";

import LoginForm from "./LoginForm";

describe("LoginForm", () => {

    test("shows the login form", () => {

        render(<LoginForm />);

        expect(
            screen.getByRole("button", { name: /login/i })
        ).toBeInTheDocument();

    });

});
```

This test isn't opening Chrome.

Instead:

```text
vitest
-> jsdom
-> render LoginForm
-> search rendered page
-> find Login button
-> assertion passes
```

---

# 👤 24. Test what the user actually sees

Avoid writing frontend tests that are too dependent on the internal implementation.

For example, instead of trying to check a component's internal variables, test what the user can see and do.

You might check:

```javascript
expect(
    screen.getByLabelText(/email/i)
).toBeInTheDocument();
```

And:

```javascript
expect(
    screen.getByLabelText(/password/i)
).toBeInTheDocument();
```

This is useful because the test describes the expected interface rather than how you happened to code it.

---

# ⌨️ 25. Test user interaction

Import `userEvent`:

```javascript
import userEvent from "@testing-library/user-event";
```

Then you can simulate actual input.

For example:

```javascript
test("allows the user to enter login details", async () => {

    const user = userEvent.setup();

    render(<LoginForm />);

    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);

    await user.type(emailInput, "student@example.com");
    await user.type(passwordInput, "Password123!");

    expect(emailInput).toHaveValue("student@example.com");
    expect(passwordInput).toHaveValue("Password123!");

});
```

The test flow is:

```text
render login form
-> locate email field
-> locate password field
-> simulate typing
-> verify values
```

---

# 🎮 26. What should you test in GameVault?

Don't try to test every line of React code.

Focus on important user behaviour.

For RegisterForm:

```text
form renders
-> required fields visible
-> user can enter details
-> validation errors can be displayed
```

For LoginForm:

```text
form renders
-> email/password accepted
-> login button works
-> failed login displays feedback
```

For GamesPage:

```text
page loads
-> games returned by api are displayed
-> loading state displayed when appropriate
-> api error displays useful feedback
```

For GameDetailsPage:

```text
game loads
-> correct details displayed
```

For Navigation:

```text
authenticated user
-> user navigation displayed

admin
-> admin dashboard option displayed

normal user
-> admin dashboard option hidden
```

For the Admin Dashboard:

```text
games displayed
-> admin can open add form
-> admin can edit game
-> admin can initiate delete
-> confirmation displayed
```

You do not need to make every test huge.

Several small tests are normally easier to understand and maintain.

---

# 🌐 27. Be careful with real API calls

A frontend test should normally not depend on the real GameVault backend running.

Otherwise:

```text
frontend test
-> backend must be running
-> mongodb must be available
-> test data must exist
-> network request must succeed
```

Now one small frontend test depends on several unrelated systems.

Instead, frontend tests should normally mock the API/service response.

For example, when testing `GamesPage`, you can mock the function from:

```text
src/services/api.js
```

and tell it:

```text
when getGames() is called
-> pretend the api returned these games
```

Then test whether those games appear on the screen.

This keeps the frontend test focused on the frontend.

---

# 🧹 Step 04 — ESLint

Testing asks:

> Does the application behave correctly?

ESLint asks:

> Does the JavaScript follow our expected code-quality rules?

ESLint analyses JavaScript without needing to execute every application feature.

It can detect things such as:

```text
unused variables
undefined variables
incorrect React Hook usage
problematic code patterns
```

---

# 📦 28. Check whether ESLint already exists

Because the GameVault frontend was created with Vite, you may already have ESLint configured.

Look inside:

```text
GameVault/frontend
```

You may already see:

```text
eslint.config.js
```

and your `package.json` may already contain:

```json
"lint": "eslint ."
```

If these already exist, do not create another ESLint configuration unnecessarily.

Run:

```bash
npm run lint
```

and inspect the result.

---

# 🛠️ 29. Add ESLint to the backend

The backend can also use ESLint.

Open:

```text
GameVault/backend
```

Install ESLint:

```bash
npm install --save-dev eslint @eslint/js globals
```

Create:

```text
eslint.config.js
```

Because the GameVault backend uses CommonJS, a simple configuration could be:

```javascript
const js = require("@eslint/js");
const globals = require("globals");

module.exports = [
    js.configs.recommended,

    {
        files: ["/*.js"],

        languageOptions: {
            globals: {
                ...globals.node
            }
        },

        rules: {
            "no-unused-vars": "warn"
        }
    }
];
```

The important part is that ESLint understands this is Node code.

Otherwise it may incorrectly complain about things such as:

```javascript
require
module
process
```

---

# ▶️ 30. Add the backend lint command

Inside:

```text
backend/package.json
```

add:

```json
"lint": "eslint ."
```

Your scripts may now look roughly like:

```json
"scripts": {
    "start": "node server.js",
    "test": "jest",
    "lint": "eslint ."
}
```

Run:

```bash
npm run lint
```

ESLint will scan the project and report problems.

---

# ⚠️ 31. Don't blindly disable ESLint rules

Suppose ESLint reports:

```text
'user' is assigned a value but never used
```

Don't immediately disable `no-unused-vars`.

First inspect the code.

Maybe you genuinely created a variable that isn't needed.

For example:

```javascript
const user = await User.findById(id);

return res.status(200).json({
    message: "done"
});
```

If `user` is never used, either:

```text
use it
```

or:

```text
remove it
```

The point of linting is to help you find code-quality problems.

Turning off every rule defeats the purpose.

---

# ⚛️ 32. Run ESLint on the frontend

Inside:

```text
GameVault/frontend
```

run:

```bash
npm run lint
```

React's ESLint configuration can catch particularly useful issues involving React Hooks.

For example, it can help identify incorrect use of:

```text
useState
useEffect
```

and other React patterns.

Resolve the warnings/errors rather than simply ignoring them.

---

# 🔄 33. Your final testing workflow

Once everything is configured, your normal GameVault workflow should become:

```text
make a change
-> save the code
-> run backend tests
-> run frontend tests
-> run eslint
-> start backend
-> run newman collection
-> investigate failures
-> fix problems
-> run tests again
-> commit
```

Useful commands will therefore include:

Backend:

```bash
npm test
npm run lint
```

Frontend:

```bash
npm test
npm run lint
```

API collection:

```bash
npx newman run postman/GameVault.postman_collection.json --insecure
```

If you're using an exported environment:

```bash
npx newman run postman/GameVault.postman_collection.json -e postman/GameVault.postman_environment.json --insecure
```

---

# 📁 Suggested GameVault structure after this step

Your project could now look approximately like:

```text
GameVault
│
├── backend
│   ├── certificates
│   ├── config
│   ├── controllers
│   ├── middleware
│   ├── models
│   ├── routes
│   ├── tests
│   │   ├── auth.test.js
│   │   ├── games.test.js
│   │   └── system.test.js
│   ├── utils
│   ├── app.js
│   ├── eslint.config.js
│   ├── package.json
│   └── server.js
│
├── frontend
│   ├── src
│   │   ├── components
│   │   ├── pages
│   │   ├── services
│   │   └── test
│   │       └── setup.js
│   ├── eslint.config.js
│   └── package.json
│
└── postman
    ├── GameVault.postman_collection.json
    └── GameVault.postman_environment.json
```

## ✅ What you should have working

By the end of this section, you should be able to:

```text
npm test
-> automatically test backend functionality

npm test
-> automatically test react components

npm run lint
-> analyse code quality

newman
-> automatically execute the postman api collection
```

The goal is not simply to have four new tools installed. You should be moving GameVault towards a workflow where changes can be checked automatically before they are committed, making it much easier to detect broken functionality and security regressions as the application grows.
