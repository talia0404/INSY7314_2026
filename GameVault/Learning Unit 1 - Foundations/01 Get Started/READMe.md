# 🎮 GameVault Learning Unit 1: Foundations of Secure Application Development

## Overview

In Learning Unit 1, you will begin building **GameVault**, a secure video game collection platform.

GameVault will eventually contain two main parts:

```text
GameVault
├── backend
└── frontend
```

During Learning Units 1 to 3, you will mainly work inside the `backend` folder.

The `frontend` folder will be created later when React is introduced.

At this stage, GameVault will not yet use a database, authentication, user accounts, or a frontend. The purpose of this learning unit is to establish the foundation of the backend application and help you understand how a web server and API work.

You will create a basic Node.js and Express application that:

* Runs a local web server.
* Responds to HTTP requests.
* Returns game-related information in JSON format.
* Accepts information from clients.
* Uses environment variables.
* Applies basic input validation.
* Handles invalid routes and errors safely.
* Demonstrates secure-by-design thinking.

The project created in this learning unit will continue to grow throughout the semester.

---

# 📚 Learning Outcomes

By the end of this learning unit, you should be able to:

* Explain the basic architecture of a MERN application.
* Explain the relationship between a client, server, API, and database.
* Create and initialise a Node.js project.
* Install and use Express.
* Create a basic Express server.
* Create routes and endpoints.
* Explain HTTP requests and responses.
* Return text and JSON responses.
* Accept JSON data from a client.
* Use environment variables for configuration.
* Apply basic input validation.
* Identify untrusted input.
* Return appropriate HTTP status codes.
* Handle invalid routes without exposing unnecessary system details.

---

# 🧱 Part 1: Create the GameVault Project

## Step 1: Create the main project folder

Create a new folder named:

```text
GameVault
```

This will be the main folder for the entire semester project.

You may create the folder manually using File Explorer or create it using the terminal.

### Using the terminal

Navigate to the location where you want to store the project.

Create the main project folder:

Move into the folder:

```bash
cd GameVault
```

---

## Step 2: Create the backend folder

Inside the `GameVault` folder, create another folder named:

```text
backend
```

Move into the backend folder:

```bash
cd backend
```

Your terminal path should now end with:

```text
GameVault\backend
```

The project structure should currently look like this:

```text
GameVault
└── backend
```

All Node.js, Express, environment, route, controller, middleware, and database files must be placed inside the `backend` folder.

Do not initialise the Node.js project in the main `GameVault` folder.

Before running any Node.js or npm commands, confirm that the terminal path ends in:

```text
GameVault\backend
```

---

# 💻 Part 2: Verify the Required Software

Before initialising the project, verify that Node.js and npm are installed.

Run:

```bash
node -v
```

You should receive a Node.js version number.

Example:

```text
v22.14.0
```

Then run:

```bash
npm -v
```

You should receive an npm version number. If not switch to cmd instead of powershell.

Example:

```text
10.9.2
```

You may also verify that Git is installed:

```bash
git --version
```

If any command is not recognised:

1. Confirm that the required software is installed.

---

# 📦 Part 3: Initialise the Backend Project

Confirm that the terminal is inside:

```text
GameVault/backend
```

Initialise the folder as a Node.js project:

```bash
npm init -y
```

The `-y` option accepts the default project settings automatically.

After running the command, the following file should be created:

```text
package.json
```

The project structure should now look like this:

```text
GameVault
└── backend
    └── package.json
```

Open `package.json`.

It should initially look similar to:

```json
{
  "name": "backend",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Update the project information later so that:

* The description identifies the project as the GameVault backend API.
* The main application file is `server.js`.
* A start script launches `server.js`.

Do not manually add dependencies to `package.json`.

Dependencies must be installed using npm commands.

---

# 🔧 Part 4: Install the Required Packages

## Install Express

Ensure that the terminal is still inside:

```text
GameVault/backend
```

Install Express:

```bash
npm install express
```

Express is the framework that will be used to create the GameVault server, routes, and API endpoints.

After installation, the backend folder should contain:

```text
GameVault
└── backend
    ├── node_modules
    ├── package-lock.json
    └── package.json
```

Open `package.json` and confirm that Express appears inside the dependencies section.

The installed version may differ depending on when the command is run.

---

## Install dotenv

Install dotenv:

```bash
npm install dotenv
```

The dotenv package allows the application to read configuration values from a `.env` file.

It will be used to manage values such as:

* Application port.
* Application name.
* Database connection string in later learning units.
* JWT secrets in later learning units.
* Other sensitive configuration values.

After installation, confirm that both `express` and `dotenv` appear inside the dependencies section of `package.json`.

---

## Install nodemon as a development dependency

Install nodemon:

```bash
npm install --save-dev nodemon
```

Nodemon automatically restarts the backend server whenever a file is saved.

This is useful during development because you do not need to stop and manually restart the server after every change.

Because nodemon is only needed during development, install it as a development dependency rather than a normal dependency.

Confirm that nodemon appears inside the `devDependencies` section of `package.json`.

---

## Understand the installation commands

The following command installs a package required when the application runs:

```bash
npm install package-name
```

For example:

```bash
npm install express
```

The following command installs a package used only while developing the application:

```bash
npm install --save-dev package-name
```

For example:

```bash
npm install --save-dev nodemon
```

The difference will become more important when the application is tested, containerised, and deployed.

---

# 📁 Part 5: Understand the Generated Files

After installing the packages, the backend folder should contain:

```text
GameVault
└── backend
    ├── node_modules
    ├── package-lock.json
    └── package.json
```

## `node_modules`

The `node_modules` folder contains all installed packages and their dependencies.

Do not:

* Manually edit files in this folder.
* Copy the folder between computers.
* Upload the folder to GitHub.

The folder can be recreated using:

```bash
npm install
```

The command reads the dependencies listed in `package.json` and installs them.

---

## `package-lock.json`

The `package-lock.json` file records the exact versions of installed packages and their dependencies.

This helps ensure that developers working on the same project use consistent package versions.

Do not manually edit this file.

This file must normally be committed to GitHub.

---

## `package.json`

The `package.json` file describes the Node.js project.

It contains:

* Project name.
* Version.
* Description.
* Main application file.
* Scripts.
* Dependencies.
* Development dependencies.

This file must be committed to GitHub.

---

# 🖥️ Part 6: Create the Express Server File

Inside the `backend` folder, create:

```text
server.js
```

The project structure should now look like this:

```text
GameVault
└── backend
    ├── node_modules
    ├── package-lock.json
    ├── package.json
    └── server.js
```

The `server.js` file will initially be responsible for:

1. Loading environment variables.
2. Importing Express.
3. Creating the Express application.
4. Enabling JSON request bodies.
5. Defining the application port.
6. Creating the first API routes.
7. Starting the backend server.

As the project becomes larger, routes, controllers, middleware, models, and configuration will be moved into separate folders.

---

# ⚙️ Part 7: Add the Project Scripts

Open:

```text
backend/package.json
```

Locate the `scripts` section.

Configure scripts for:

* Starting the application normally.
* Starting the application in development mode using nodemon.

The intended commands should be:

```bash
npm start
```

for the normal server, and:

```bash
npm run dev
```

for the development server.

When using `npm run dev`, nodemon should restart the server automatically whenever `server.js` is saved.

After configuring the scripts, test both commands separately.

Only one server should run at a time.

Stop the current server before starting it using a different command.

To stop the server, click inside the terminal and press:

```text
Ctrl + C
```

---

# 🔐 Part 8: Create the Environment Files

Inside the `backend` folder, create:

```text
.env
```

This file will store the real environment values used on your computer.

Add configuration values for:

* The backend application port.
* The application name.
* The environment mode.

The development environment mode should be:

```text
development
```

Do not upload the real `.env` file to GitHub.

---

## Create `.env.example`

Inside the `backend` folder, create:

```text
.env.example
```

This file must contain the same environment-variable names as `.env`, but it must not contain private or sensitive values.

The purpose of `.env.example` is to show other developers which variables they need to configure after cloning the repository.

The `.env.example` file must be uploaded to GitHub.

---

# 🚫 Part 9: Create the `.gitignore` File

Create a `.gitignore` file inside the main `GameVault` folder.

The recommended location is:

```text
GameVault/.gitignore
```

This allows one file to control ignored content for both the backend and the frontend later in the semester.

Configure it to ignore:

```text
backend/node_modules/
backend/.env
frontend/node_modules/
frontend/.env
```

The frontend entries may be added now even though the frontend folder does not exist yet.

The structure should now look like this:

```text
GameVault
├── .gitignore
└── backend
    ├── node_modules
    ├── .env
    ├── .env.example
    ├── package-lock.json
    ├── package.json
    └── server.js
```

Confirm that:

* `node_modules` is ignored.
* `.env` is ignored.
* `.env.example` is not ignored.
* `package.json` is not ignored.
* `package-lock.json` is not ignored.
* `server.js` is not ignored.

---

# 🧭 Part 10: Final Initial Structure

Before creating the GameVault routes, the project should have the following structure:

```text
GameVault
├── .gitignore
└── backend
    ├── node_modules
    ├── .env
    ├── .env.example
    ├── package-lock.json
    ├── package.json
    └── server.js
```

All backend commands must be run from:

```text
GameVault/backend
```

Examples include:

```bash
npm install
npm start
npm run dev
```

If an npm command produces an error saying that `package.json` cannot be found, check the terminal path.

The terminal is probably running from the wrong folder.

Move into the backend folder:

```bash
cd backend
```

Then run the command again.

---

# 📋 Initialisation and Installation Command Summary

Run these commands in order when creating the project from scratch:

```bash
mkdir GameVault
cd GameVault
mkdir backend
cd backend
npm init -y
npm install express
npm install dotenv
npm install --save-dev nodemon
```

To open the complete project in Visual Studio Code from the backend folder, first move back to the main folder:

```bash
cd ..
code .
```

To install all dependencies after cloning an existing copy of the project:

```bash
cd backend
npm install
```

To start the server normally:

```bash
npm start
```

To start the development server with automatic restarting:

```bash
npm run dev
```

To stop the server:

```text
Ctrl + C
```

---

# ➡️ Continue With the GameVault API

After completing the initialisation and installation process, continue with the remaining Learning Unit 1 tasks:

1. Configure `server.js`.
2. Load environment variables.
3. Create the Express application.
4. Enable JSON request-body processing.
5. Start the server.
6. Create the root route.
7. Create the about route.
8. Create the health-check route.
9. Create the in-memory games collection.
10. Create the game endpoints.
11. Validate external input.
12. Test the API in Postman.
13. Push the project to GitHub.

All files created for these backend tasks must remain inside the `backend` folder unless the instructions specifically state otherwise.
