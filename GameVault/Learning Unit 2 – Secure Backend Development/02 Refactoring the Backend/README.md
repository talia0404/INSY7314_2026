# 🧱 GameVault Learning Unit 2 Part A: Refactoring the Backend

## Overview

In Learning Unit 1, the entire GameVault backend was created inside one file:

```text
backend/server.js
```

This file currently contains:

* Environment configuration.
* HTTPS certificate configuration.
* Express application setup.
* The temporary games collection.
* The root route.
* The about route.
* The health-check route.
* The game routes.
* Input validation.
* The invalid-route handler.
* The HTTPS server startup logic.

This approach is acceptable for a small introductory application. However, as GameVault grows, keeping everything inside one file will make the project difficult to:

* Read.
* Maintain.
* Test.
* Debug.
* Extend.
* Secure.
* Divide between team members.

In Learning Unit 2 Part A, you will reorganise the GameVault backend into a more professional structure.

You will not add MongoDB yet.

You will not add authentication yet.

You will not create the frontend yet.

The main purpose of this section is to separate the existing GameVault responsibilities into appropriate files and folders without changing the existing behaviour of the application.

After refactoring, all existing endpoints should continue to work over HTTPS.

---

## 📚 Learning Outcomes

By the end of this section, you should be able to:

* Explain why large applications should not place all logic inside one file.
* Explain separation of concerns.
* Organise an Express application into routes, controllers, middleware, configuration files, and data files.
* Distinguish between route definitions and controller functions.
* Move reusable validation into middleware.
* Separate application creation from server startup.
* Create a central not-found handler.
* Create a central error-handling middleware file.
* Maintain environment and HTTPS configuration after refactoring.
* Test that refactoring has not changed existing application behaviour.
* Use meaningful Git commits while restructuring a project.

---

# 🔍 Part 1: Understand Why the Backend Must Be Refactored

Your current `server.js` file performs too many different responsibilities.

It:

* Loads environment variables.
* Creates the Express application.
* Reads HTTPS certificate files.
* Stores application data.
* Validates incoming game data.
* Handles requests.
* Creates responses.
* Starts the server.

This makes `server.js` responsible for nearly every part of the backend.

As GameVault grows, additional features will be introduced, including:

* More game operations.
* Updating games.
* Deleting games.
* MongoDB.
* Mongoose models.
* User accounts.
* Authentication.
* Authorisation.
* Reviews.
* Security middleware.
* Logging.
* Testing.
* Deployment.

If every new feature is added directly to `server.js`, the file will become very large and difficult to manage.

The refactored application will separate responsibilities into smaller files.

Each file should have one clear purpose.

This principle is called:

```text
Separation of concerns
```

It means that unrelated responsibilities should be placed in separate parts of the application.

---

# 🧭 Part 2: Understand the New Backend Structure

By the end of Part A, the backend should resemble:

```text
GameVault
├── .gitignore
└── backend
    ├── certificates
    │   ├── certificate.pem
    │   └── privatekey.pem
    ├── config
    │   └── httpsConfig.js
    ├── controllers
    │   ├── gameController.js
    │   └── systemController.js
    ├── data
    │   └── games.js
    ├── middleware
    │   ├── errorHandler.js
    │   ├── notFound.js
    │   └── validateGame.js
    ├── routes
    │   ├── gameRoutes.js
    │   └── systemRoutes.js
    ├── node_modules
    ├── .env
    ├── .env.example
    ├── app.js
    ├── package-lock.json
    ├── package.json
    └── server.js
```

Your project may contain additional files later, but this will be the expected structure for the first refactoring stage.

---

# 🛑 Part 3: Stop the Current Server

Before restructuring the project, stop the running backend.

Click inside the terminal and press:

```text
Ctrl + C
```

Do not move or rename files while the server is still running.

Confirm that the terminal has returned to the command prompt.

---

# 📍 Part 4: Confirm the Terminal Location

All commands in this guide must be run from:

```text
GameVault/backend
```

Check the current directory.

In Command Prompt, run:

```cmd
cd
```

In PowerShell, run:

```powershell
Get-Location
```

The path should end in:

```text
GameVault\backend
```

If the terminal is in the main project folder, move into the backend folder:

```bash
cd backend
```

If you are elsewhere, navigate to the correct location before continuing.

---

# 📁 Part 5: Create the New Folders

Inside the `backend` folder, create the following folders:

```text
config
controllers
data
middleware
routes
```

You may create them manually in Visual Studio Code or File Explorer.

You may also use terminal commands.

Using Command Prompt or PowerShell:

```bash
mkdir config
mkdir controllers
mkdir data
mkdir middleware
mkdir routes
```

After creating the folders, confirm that the structure resembles:

```text
backend
├── certificates
├── config
├── controllers
├── data
├── middleware
├── routes
├── node_modules
├── .env
├── .env.example
├── package-lock.json
├── package.json
└── server.js
```

Do not create these folders inside `node_modules`.

Do not create them in the main `GameVault` folder.

They must be inside:

```text
GameVault/backend
```

---

# ⚙️ Part 6: Create app.js

Inside the `backend` folder, create:

```text
app.js
```

The purpose of `app.js` is to create and configure the Express application.

After refactoring, `app.js` should be responsible for:

* Importing Express.
* Creating the Express application.
* Enabling JSON request-body processing.
* Registering the system routes.
* Registering the game routes.
* Registering the not-found middleware.
* Registering the error-handling middleware.
* Exporting the Express application.

It should not:

* Start the HTTPS server.
* Read the certificate files.
* Store the games array.
* Contain controller logic.
* Contain detailed validation logic.
* Call the server’s listening function.

Separating `app.js` from `server.js` makes the application easier to test and maintain.

The Express application can later be imported into automated tests without automatically starting the server.

---

# 🚀 Part 7: Reduce the Responsibility of server.js

The current `server.js` contains the entire application.

After refactoring, `server.js` should become much smaller.

Its responsibilities should be limited to:

* Loading environment variables.
* Importing the Express application from `app.js`.
* Importing the HTTPS configuration.
* Reading the application name and port.
* Creating the HTTPS server.
* Starting the server.
* Displaying the startup message.

The following logic should be removed from `server.js` and moved elsewhere:

* Express route definitions.
* The games array.
* Game validation.
* Root endpoint logic.
* About endpoint logic.
* Health endpoint logic.
* Invalid-route response logic.

Do not delete the original code until it has been moved to the correct new files.

Move one responsibility at a time and test frequently.

---

# 🔐 Part 8: Create the HTTPS Configuration File

Inside the `config` folder, create:

```text
httpsConfig.js
```

This file will be responsible for preparing the HTTPS configuration.

Move the following responsibilities from `server.js` into this file:

* Importing the file-system module.
* Importing the path module.
* Reading the SSL key path from the environment variables.
* Reading the SSL certificate path from the environment variables.
* Resolving both paths.
* Reading the private key file.
* Reading the certificate file.
* Preparing the HTTPS options object.
* Exporting the HTTPS options.

The expected file location is:

```text
backend/config/httpsConfig.js
```

The environment variables should remain in:

```text
backend/.env
```

Your current environment values should continue to use:

```text
HTTPS_PORT=4000
APP_NAME=GameVault API
NODE_ENV=development
SSL_KEY_PATH=certificates/privatekey.pem
SSL_CERT_PATH=certificates/certificate.pem
```

Do not move the `.env` file into the `config` folder.

Do not move the certificate files into the `config` folder.

The certificate files should remain inside:

```text
backend/certificates
```

---

# 🎮 Part 9: Move the Temporary Games Collection

Inside the `data` folder, create:

```text
games.js
```

Move the temporary in-memory games collection from `server.js` into this file.

The file should contain the existing sample games from Learning Unit 1.

The purpose of this file is to separate temporary data from:

* Routes.
* Controllers.
* Validation.
* Server configuration.

This data remains temporary.

It will still be lost whenever the server restarts.

MongoDB will replace this file in a later section of Learning Unit 2.

The expected location is:

```text
backend/data/games.js
```

Other files that need access to the games collection should import it from this data file.

Do not create a second games array in another file.

The entire application should use one shared in-memory collection.

---

# 🎛️ Part 10: Create the System Controller

Inside the `controllers` folder, create:

```text
systemController.js
```

This controller will contain the logic for the non-game endpoints.

Move the response logic for the following endpoints into this controller:

* Root endpoint.
* About endpoint.
* Health-check endpoint.

The controller should be responsible for:

* Reading any required application information.
* Preparing the response data.
* Returning the appropriate HTTP status code.
* Returning the JSON response.

The controller should not define the route URL.

For example, the controller should not decide whether the route is:

```text
/
```

or:

```text
/about
```

The route file will decide the URL.

The controller should only contain the function that handles the request and response.

---

# 🎮 Part 11: Create the Game Controller

Inside the `controllers` folder, create:

```text
gameController.js
```

Move the game-related request logic from `server.js` into this controller.

At this stage, the controller should contain separate functions for:

* Retrieving all games.
* Retrieving one game by ID.
* Creating a new game.

Each operation should have its own controller function.

The controller should be responsible for:

* Accessing the games collection.
* Finding games.
* Creating a new game.
* Returning successful responses.
* Returning game-specific errors where appropriate.

The controller should not:

* Start the server.
* Read certificate files.
* Define the `/games` route path.
* Contain unrelated system routes.
* Contain the entire application configuration.

Later, additional controller functions will be added for:

* Updating a game.
* Deleting a game.

---

# 🛣️ Part 12: Create the System Routes File

Inside the `routes` folder, create:

```text
systemRoutes.js
```

This route file should define the routes for:

* Root.
* About.
* Health check.

The route file should:

* Create an Express router.
* Import the required system controller functions.
* Connect each route path to the correct controller function.
* Export the router.

The route file should not contain the complete response logic.

Its purpose is to answer the question:

```text
Which controller function should handle this URL and HTTP method?
```

The system routes will later be registered inside `app.js`.

---

# 🎯 Part 13: Create the Game Routes File

Inside the `routes` folder, create:

```text
gameRoutes.js
```

This route file should define the game-related routes.

At this stage, it should support:

* Retrieving all games.
* Retrieving one game by ID.
* Creating a new game.

The route file should connect:

* The correct HTTP method.
* The correct route path.
* Any required validation middleware.
* The correct controller function.

The route file should not:

* Store game data.
* Generate game IDs.
* Contain the complete validation process.
* Start the server.
* Read environment variables.

The route file decides where a request goes.

The controller decides what happens when the request arrives.

---

# 🛡️ Part 14: Create the Game Validation Middleware

Inside the `middleware` folder, create:

```text
validateGame.js
```

Move the validation logic for creating a game into this middleware file.

The validation middleware should check the incoming request body before the request reaches the game controller.

It should continue to validate:

* Required fields.
* Text data types.
* Title length.
* Genre length.
* Platform length.
* Release year type.
* Release year range.
* Age-rating values.
* Empty or whitespace-only values.

The validation middleware should:

* Treat all incoming values as untrusted.
* Stop the request when validation fails.
* Return a suitable error message.
* Return an appropriate HTTP status code.
* Allow the request to continue when validation succeeds.

The controller should only create the game after the validation middleware has approved the request.

This improves separation of concerns.

The middleware handles validation.

The controller handles creation.

---

# ❓ Part 15: Create the Not-Found Middleware

Inside the `middleware` folder, create:

```text
notFound.js
```

Move the invalid-route handling logic from `server.js` into this file.

This middleware should run when no valid route matches the request.

It should return:

* An appropriate HTTP status code.
* A simple JSON error response.
* No unnecessary server details.

The not-found middleware must be registered after all valid routes.

If it is registered before the valid routes, every request may be treated as invalid.

The expected order in `app.js` should be:

```text
1. Express middleware
2. Valid routes
3. Not-found middleware
4. Error-handling middleware
```

---

# ⚠️ Part 16: Create the Error-Handling Middleware

Inside the `middleware` folder, create:

```text
errorHandler.js
```

This file will provide a central location for unexpected application errors.

The error handler should be designed to:

* Receive errors passed by the application.
* Choose an appropriate status code.
* Return a controlled JSON response.
* Avoid exposing unnecessary technical details.
* Display more information during development where appropriate.
* Hide sensitive error details in production.

This middleware may not be used extensively yet, but it establishes the correct structure for later work.

It will become more important when the project introduces:

* MongoDB.
* Asynchronous database operations.
* Authentication.
* File operations.
* More complex controllers.

The error handler must appear after the not-found middleware and all valid routes.

---

# 🔗 Part 17: Register the Routes in app.js

Once the route files have been created, register them inside:

```text
backend/app.js
```

The system routes should continue to provide:

```text
/
```

```text
/about
```

```text
/health
```

The game routes should be mounted under:

```text
/games
```

This means the routes should continue to work using the same addresses as before:

```text
GET https://localhost:4000/games
```

```text
GET https://localhost:4000/games/1
```

```text
POST https://localhost:4000/games
```

Refactoring should not unnecessarily change the public API.

The purpose is to reorganise the backend internally while preserving its external behaviour.

---

# 🧹 Part 18: Clean server.js

After all logic has been moved successfully, review `server.js`.

It should no longer contain:

* The games array.
* Root route logic.
* About route logic.
* Health route logic.
* Game route logic.
* Game validation logic.
* The not-found response.
* Express JSON middleware configuration.

It should mainly contain:

* Environment configuration.
* The imported application.
* The imported HTTPS options.
* The HTTPS port.
* The application name.
* The HTTPS server startup.

A smaller `server.js` is an expected result of successful refactoring.

Do not remove the HTTPS server.

The application must continue to use:

```text
https://localhost:4000
```

---

# 🧪 Part 19: Start the Refactored Application

After completing the refactoring, confirm that the terminal is inside:

```text
GameVault/backend
```

Start the development server:

```bash
npm run dev
```

The terminal should still display a secure startup message.

The exact wording may differ, but it should confirm that GameVault is running using HTTPS.

For example:

```text
GameVault API is running securely on https://localhost:4000
```

If nodemon repeatedly restarts, inspect the terminal for syntax errors or missing file paths.

---

# 🔎 Part 20: Test the Existing Endpoints

Refactoring is not complete until all existing behaviour has been retested.

Use the existing Postman collection from Learning Unit 1.

Test:

```text
GET https://localhost:4000/
```

Expected result:

```text
200 OK
```

Test:

```text
GET https://localhost:4000/about
```

Expected result:

```text
200 OK
```

Test:

```text
GET https://localhost:4000/health
```

Expected result:

```text
200 OK
```

Test:

```text
GET https://localhost:4000/games
```

Expected result:

```text
200 OK
```

Test:

```text
GET https://localhost:4000/games/1
```

Expected result:

```text
200 OK
```

Test:

```text
GET https://localhost:4000/games/999
```

Expected result:

```text
404 Not Found
```

Test:

```text
GET https://localhost:4000/games/test
```

Expected result:

```text
400 Bad Request
```

Test:

```text
POST https://localhost:4000/games
```

Use a valid game request.

Expected result:

```text
201 Created
```

Test an invalid game request.

Expected result:

```text
400 Bad Request
```

Test an invalid route:

```text
GET https://localhost:4000/invalid-route
```

Expected result:

```text
404 Not Found
```

All routes should behave in the same way they did before the refactoring.

---

# 📋 Part 21: Create a New Postman Collection

Create a new Postman collection named:

```text
GameVault LU2
```

You may keep the LU1 collection for comparison.

Add a folder named:

```text
Part A - Refactored Backend
```

Include requests for:

* Root endpoint.
* About endpoint.
* Health endpoint.
* Retrieve all games.
* Retrieve one game.
* Invalid game ID.
* Missing game.
* Create a valid game.
* Create an invalid game.
* Invalid route.

Use:

```text
https://localhost:4000
```

Do not change the requests back to HTTP.

Because the local certificate is self-signed, Postman may require SSL certificate verification to remain disabled for these local requests.

---

# 📦 Part 22: Confirm That No New Package Is Required

The basic refactoring process does not require an additional npm package.

The existing packages should still be sufficient:

```text
express
dotenv
nodemon
```

Do not install packages simply because new folders were created.

Folders such as `controllers`, `routes`, and `middleware` are organisational structures, not npm packages.

You can confirm the installed packages using:

```bash
npm list --depth=0
```

The output should include the packages already used by the project.

---

# 🔄 Part 23: Reinstall Dependencies if Necessary

If `node_modules` is missing or the project has been cloned onto another computer, run:

```bash
npm install
```

This command reads:

```text
package.json
```

and:

```text
package-lock.json
```

It then recreates the required dependencies.

Do not copy `node_modules` from another computer.

Do not upload `node_modules` to GitHub.

---

# 🚫 Part 24: Review .gitignore

Confirm that the main project `.gitignore` still excludes:

```text
backend/node_modules/
backend/.env
backend/certificates/
frontend/node_modules/
frontend/.env
```

The following new folders must not be ignored:

```text
backend/config/
backend/controllers/
backend/data/
backend/middleware/
backend/routes/
```

These folders contain source code and must be uploaded to GitHub.

The following new files must also be tracked:

```text
backend/app.js
backend/config/httpsConfig.js
backend/controllers/gameController.js
backend/controllers/systemController.js
backend/data/games.js
backend/middleware/errorHandler.js
backend/middleware/notFound.js
backend/middleware/validateGame.js
backend/routes/gameRoutes.js
backend/routes/systemRoutes.js
```

---

# 🧾 Part 25: Review the Final Structure

Before committing your work, confirm that the project resembles:

```text
GameVault
├── .gitignore
└── backend
    ├── certificates
    │   ├── certificate.pem
    │   └── privatekey.pem
    ├── config
    │   └── httpsConfig.js
    ├── controllers
    │   ├── gameController.js
    │   └── systemController.js
    ├── data
    │   └── games.js
    ├── middleware
    │   ├── errorHandler.js
    │   ├── notFound.js
    │   └── validateGame.js
    ├── routes
    │   ├── gameRoutes.js
    │   └── systemRoutes.js
    ├── node_modules
    ├── .env
    ├── .env.example
    ├── app.js
    ├── package-lock.json
    ├── package.json
    └── server.js
```

The following files should remain local and should not appear on GitHub:

```text
backend/.env
backend/certificates/certificate.pem
backend/certificates/privatekey.pem
backend/node_modules/
```

---

# ☁️ Part 26: Commit the Refactoring

Check the current repository status:

```bash
git status
```

Review the listed files carefully.

Confirm that the private certificate files and `.env` do not appear as files waiting to be committed.

Stage the changes:

```bash
git add .
```

Check the staged files:

```bash
git status
```

Commit the refactoring:

```bash
git commit -m "Refactor backend into routes controllers and middleware"
```

Push the changes:

```bash
git push
```

Use a meaningful commit message that explains what changed.

Avoid vague messages such as:

```text
update
```

```text
changes
```

```text
fixed stuff
```

---

# 🛠️ Part 27: Troubleshooting

## The server cannot find app.js

Check that `app.js` is located directly inside:

```text
backend
```

It should not be inside:

```text
backend/routes
```

or:

```text
backend/controllers
```

Also check that the filename uses the correct capitalisation.

---

## A route returns 404 after refactoring

Check that:

* The router was exported.
* The router was imported into `app.js`.
* The router was registered in `app.js`.
* The correct base path was used.
* The not-found middleware appears after valid routes.

---

## A controller function is undefined

Check that:

* The function was exported from the controller.
* The correct function name was imported.
* The spelling matches exactly.
* The correct controller file path was used.

---

## The games collection is empty or duplicated

Check that:

* Only one games collection exists.
* The game controller imports the shared collection.
* A new games array is not recreated inside every controller function.
* The original array was removed from `server.js`.

---

## Validation no longer runs

Check that:

* The validation middleware is imported into the route file.
* It appears before the controller function.
* It allows valid requests to continue.
* It returns a response when validation fails.

---

## Every request returns “Route not found”

The not-found middleware may have been registered too early.

It must appear after all valid routes.

---

## HTTPS returns an ENOENT error

Check that the certificate paths in `.env` still match:

```text
SSL_KEY_PATH=certificates/privatekey.pem
SSL_CERT_PATH=certificates/certificate.pem
```

Confirm that the files exist inside:

```text
backend/certificates
```

---

## The application works with HTTP but not HTTPS

The server may still be using the old Express listening method.

Confirm that the application is started through the HTTPS server configuration.

The test URL must begin with:

```text
https://
```

not:

```text
http://
```

---

# ✅ Part 28: Completion Checklist

Before continuing to the next section of Learning Unit 2, confirm that:

* The current server has been stopped before refactoring.
* The terminal is running from `GameVault/backend`.
* The `config` folder exists.
* The `controllers` folder exists.
* The `data` folder exists.
* The `middleware` folder exists.
* The `routes` folder exists.
* `app.js` exists.
* HTTPS configuration has been moved into its own file.
* The temporary games collection has been moved into its own file.
* System controller functions have been created.
* Game controller functions have been created.
* System routes have been created.
* Game routes have been created.
* Game validation has been moved into middleware.
* Not-found middleware has been created.
* Error-handling middleware has been created.
* The routes are registered in `app.js`.
* `server.js` only handles server startup responsibilities.
* HTTPS still works.
* Every Learning Unit 1 endpoint still works.
* Valid game creation still works.
* Invalid game creation is still rejected.
* Invalid routes still return a safe error.
* The new files are tracked by Git.
* `.env` is not tracked.
* The certificate files are not tracked.
* `node_modules` is not tracked.
* The refactored project has been committed and pushed.

---

# 📋 Terminal Command Summary

Run these commands from:

```text
GameVault/backend
```

Create the new folders:

```bash
mkdir config
mkdir controllers
mkdir data
mkdir middleware
mkdir routes
```

Check the installed packages:

```bash
npm list --depth=0
```

Install existing project dependencies if required:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

Stop the server:

```text
Ctrl + C
```

Check the repository:

```bash
git status
```

Stage the refactored files:

```bash
git add .
```

Commit the changes:

```bash
git commit -m "Refactor backend into routes controllers and middleware"
```

Push the changes:

```bash
git push
```

---

