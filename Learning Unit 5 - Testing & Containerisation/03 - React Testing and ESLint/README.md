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
