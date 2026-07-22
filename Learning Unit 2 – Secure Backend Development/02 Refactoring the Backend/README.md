# 🧱 GameVault Learning Unit 2 Part A: Refactoring the Backend

## Overview

In Learning Unit 1, the complete GameVault backend was placed inside:

```text
backend/server.js
```

The file currently:

* Loads environment variables.
* Creates the Express application.
* configures HTTPS.
* Reads the private key and certificate.
* Stores the temporary games collection.
* Defines the system routes.
* Defines the game routes.
* Validates incoming game data.
* Handles invalid routes.
* Starts the HTTPS server.

This was suitable while learning the foundations of Express. However, the file will become difficult to maintain as more features are added.

In Learning Unit 2 Part A, the existing backend will be divided into smaller files and folders.

The refactoring must not change the public behaviour of the API. The existing endpoints must continue to work through HTTPS.

---

# 📚 Learning Outcomes

By the end of this section, you should be able to:

* Explain separation of concerns.
* Divide an Express application into appropriate folders.
* Separate application configuration from server startup.
* Separate routes from controllers.
* Create reusable validation middleware.
* Create central not-found and error-handling middleware.
* Move temporary data into a separate module.
* Import and export modules using CommonJS.
* Test an application after refactoring.
* Maintain HTTPS after reorganising a backend.

---

# 🔍 Part 1: Understand Separation of Concerns

Separation of concerns means that each file or section of an application should have one clear responsibility.

For example:

* A route file determines which endpoint handles a request.
* A controller decides what happens when the request reaches that endpoint.
* Middleware performs work before or after a controller.
* A data file stores temporary application data.
* A configuration file manages application configuration.
* `app.js` configures Express.
* `server.js` starts the server.

This structure makes the application easier to:

* Read.
* Debug.
* Test.
* Extend.
* Secure.
* Maintain.

---

# 🧭 Part 2: Final Backend Structure

At the end of Part A, the project should resemble:

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

---

# 🛑 Part 3: Stop the Existing Server

Before creating or moving files, stop the server.

Click inside the terminal and press:

```text
Ctrl + C
```

Confirm that the terminal returns to the command prompt.

---

# 📍 Part 4: Confirm the Terminal Location

All terminal commands must be run from:

```text
GameVault/backend
```

Move into the backend folder where necessary:

```bash
cd backend
```

The terminal path should end with:

```text
GameVault\backend
```

---

# 📁 Part 5: Create the Required Folders

Run the following commands from the `backend` folder:

```bash
mkdir config
mkdir controllers
mkdir data
mkdir middleware
mkdir routes
```

The project should now contain:

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

No new npm packages are required for this refactoring.

---

# 🎮 Part 6: Create the Temporary Games Data File

Inside the `data` folder, create:

```text
games.js
```

The complete path should be:

```text
backend/data/games.js
```

Add the existing temporary games collection:

```javascript
/*
Temporary in-memory game data.

The data is stored in an array while the project does not yet
use MongoDB.

Any games added while the server is running will be lost when
the server restarts.
*/
const games = [
    {
        id: 1,
        title: "The Legend of Zelda: Breath of the Wild",
        genre: "Action Adventure",
        platform: "Nintendo Switch",
        releaseYear: 2017,
        ageRating: "E10+",
        available: true
    },
    {
        id: 2,
        title: "Marvel's Spider-Man 2",
        genre: "Action Adventure",
        platform: "PlayStation 5",
        releaseYear: 2023,
        ageRating: "T",
        available: true
    },
    {
        id: 3,
        title: "Forza Horizon 5",
        genre: "Racing",
        platform: "Xbox Series X/S",
        releaseYear: 2021,
        ageRating: "E",
        available: false
    }
];

/*
Exports the games array.

Other files can import and use the same shared array.
*/
module.exports = games;
```

## Why this file is required

Previously, the games array was stored inside `server.js`.

It is now separated because:

* Data should not be mixed with server startup logic.
* Controllers need access to the games.
* One shared array should be used throughout the application.
* MongoDB will replace this file later.

Do not create another games array in the controller.

---

# 🎛️ Part 7: Create the System Controller

Inside the `controllers` folder, create:

```text
systemController.js
```

The complete path should be:

```text
backend/controllers/systemController.js
```

Add:

```javascript
/*
Returns information for the root endpoint.

The controller receives the request and response objects from
the route file.
*/
const getRoot = (req, res) => {

    const appName = process.env.APP_NAME || "GameVault API";

    return res.status(200).json({
        application: appName,
        message: "Welcome to the GameVault API"
    });

};

/*
Returns information about the GameVault application.
*/
const getAbout = (req, res) => {

    const appName = process.env.APP_NAME || "GameVault API";

    return res.status(200).json({
        application: appName,
        description:
            "GameVault is a secure video game collection and review platform.",
        currentStage:
            "Learning Unit 2 - Refactoring the Backend"
    });

};

/*
Returns the current health status of the application.

The timestamp is generated whenever the endpoint is requested.
*/
const getHealth = (req, res) => {

    const appName = process.env.APP_NAME || "GameVault API";

    return res.status(200).json({
        application: appName,
        status: "OK",
        protocol: "HTTPS",
        environment: process.env.NODE_ENV || "development",
        timestamp: new Date().toISOString()
    });

};

/*
Exports the controller functions.

The system route file will import these functions and connect
them to the correct endpoints.
*/
module.exports = {
    getRoot,
    getAbout,
    getHealth
};
```

## Controller responsibility

This file handles the response logic for:

```text
GET /
GET /about
GET /health
```

It does not define the route URLs. The URLs will be defined inside `systemRoutes.js`.

---

# 🛣️ Part 8: Create the System Routes

Inside the `routes` folder, create:

```text
systemRoutes.js
```

The complete path should be:

```text
backend/routes/systemRoutes.js
```

Add:

```javascript
const express = require("express");

/*
Imports the controller functions that will handle each system
route.
*/
const {
    getRoot,
    getAbout,
    getHealth
} = require("../controllers/systemController");

/*
Creates a smaller Express router.

This router will later be registered inside app.js.
*/
const router = express.Router();

/*
GET /

Calls the getRoot controller function.
*/
router.get("/", getRoot);

/*
GET /about

Calls the getAbout controller function.
*/
router.get("/about", getAbout);

/*
GET /health

Calls the getHealth controller function.
*/
router.get("/health", getHealth);

/*
Exports the router so that app.js can use it.
*/
module.exports = router;
```

## Route responsibility

The route file decides:

* Which HTTP method is used.
* Which route path is used.
* Which controller handles the request.

It does not contain the complete response logic.

---

# 🛡️ Part 9: Create the Game Validation Middleware

Inside the `middleware` folder, create:

```text
validateGame.js
```

The complete path should be:

```text
backend/middleware/validateGame.js
```

Add:

```javascript
/*
Validates the request body before a new game is created.

Middleware runs before the controller function.
*/
const validateGame = (req, res, next) => {

    /*
    Extracts the expected values from the request body.
    */
    const {
        title,
        genre,
        platform,
        releaseYear,
        ageRating
    } = req.body;

    /*
    Checks that all required fields were supplied.

    releaseYear is compared with undefined because a number
    should not be checked in exactly the same way as text.
    */
    if (
        !title ||
        !genre ||
        !platform ||
        releaseYear === undefined ||
        !ageRating
    ) {
        return res.status(400).json({
            error:
                "Title, genre, platform, release year and age rating are required."
        });
    }

    /*
    Checks that all expected text fields contain strings.
    */
    if (
        typeof title !== "string" ||
        typeof genre !== "string" ||
        typeof platform !== "string" ||
        typeof ageRating !== "string"
    ) {
        return res.status(400).json({
            error:
                "Title, genre, platform and age rating must be text."
        });
    }

    /*
    Removes unnecessary spaces from the beginning and end of
    each text value.

    The age rating is also changed to uppercase so that values
    such as "t" and "m" can be handled consistently.
    */
    const cleanedTitle = title.trim();
    const cleanedGenre = genre.trim();
    const cleanedPlatform = platform.trim();
    const cleanedAgeRating = ageRating.trim().toUpperCase();

    /*
    Validates the length of the game title.
    */
    if (
        cleanedTitle.length < 2 ||
        cleanedTitle.length > 100
    ) {
        return res.status(400).json({
            error:
                "Title must contain between 2 and 100 characters."
        });
    }

    /*
    Validates the length of the genre.
    */
    if (
        cleanedGenre.length < 2 ||
        cleanedGenre.length > 50
    ) {
        return res.status(400).json({
            error:
                "Genre must contain between 2 and 50 characters."
        });
    }

    /*
    Validates the length of the platform.
    */
    if (
        cleanedPlatform.length < 2 ||
        cleanedPlatform.length > 50
    ) {
        return res.status(400).json({
            error:
                "Platform must contain between 2 and 50 characters."
        });
    }

    /*
    Creates the maximum permitted release year dynamically.

    Games announced for the near future may use a release year
    up to two years after the current year.
    */
    const currentYear = new Date().getFullYear();

    /*
    Validates that releaseYear:

    - Is a number.
    - Is a whole number.
    - Is not earlier than 1950.
    - Is not more than two years into the future.
    */
    if (
        typeof releaseYear !== "number" ||
        !Number.isInteger(releaseYear) ||
        releaseYear < 1950 ||
        releaseYear > currentYear + 2
    ) {
        return res.status(400).json({
            error:
                `Release year must be a whole number between 1950 and ${currentYear + 2}.`
        });
    }

    /*
    Stores the age ratings accepted by GameVault.
    */
    const allowedAgeRatings = [
        "E",
        "E10+",
        "T",
        "M",
        "18"
    ];

    /*
    Rejects an age rating that is not included in the accepted
    list.
    */
    if (!allowedAgeRatings.includes(cleanedAgeRating)) {
        return res.status(400).json({
            error:
                `Age rating must be one of: ${allowedAgeRatings.join(", ")}.`
        });
    }

    /*
    Stores the cleaned and validated values on the request
    object.

    The controller can use these values instead of cleaning
    the request body again.
    */
    req.validatedGame = {
        title: cleanedTitle,
        genre: cleanedGenre,
        platform: cleanedPlatform,
        releaseYear,
        ageRating: cleanedAgeRating
    };

    /*
    Passes the request to the next function.

    In this case, the next function will be the createGame
    controller.
    */
    next();

};

/*
Exports the middleware function.
*/
module.exports = validateGame;
```

## How middleware works

The middleware receives:

```text
req
res
next
```

* `req` contains information about the incoming request.
* `res` is used to send a response.
* `next` passes control to the next middleware or controller.

When validation fails, the middleware returns an error response.

When validation succeeds, it calls:

```javascript
next();
```

---

# 🎮 Part 10: Create the Game Controller

Inside the `controllers` folder, create:

```text
gameController.js
```

The complete path should be:

```text
backend/controllers/gameController.js
```

Add:

```javascript
/*
Imports the shared temporary games collection.
*/
const games = require("../data/games");

/*
Returns all games.
*/
const getAllGames = (req, res) => {

    return res.status(200).json({
        count: games.length,
        data: games
    });

};

/*
Returns one game using the ID supplied in the route.
*/
const getGameById = (req, res) => {

    /*
    Route parameters are received as text.

    Number converts the supplied value into a number.
    */
    const gameId = Number(req.params.id);

    /*
    Rejects values that cannot be converted into a whole-number
    game ID.
    */
    if (!Number.isInteger(gameId)) {
        return res.status(400).json({
            error: "Game ID must be a whole number."
        });
    }

    /*
    Searches the games array for a game with the requested ID.
    */
    const game = games.find(
        currentGame => currentGame.id === gameId
    );

    /*
    Returns 404 when no matching game exists.
    */
    if (!game) {
        return res.status(404).json({
            error: "Game not found."
        });
    }

    /*
    Returns the matching game.
    */
    return res.status(200).json({
        data: game
    });

};

/*
Creates a new game.

The request reaches this function only after it has passed
through the validateGame middleware.
*/
const createGame = (req, res) => {

    /*
    Retrieves the cleaned values prepared by the validation
    middleware.
    */
    const {
        title,
        genre,
        platform,
        releaseYear,
        ageRating
    } = req.validatedGame;

    /*
    Generates the next available game ID.

    If games exist, the largest current ID is increased by one.
    If the array is empty, the first ID will be 1.
    */
    const nextId =
        games.length > 0
            ? Math.max(...games.map(game => game.id)) + 1
            : 1;

    /*
    Creates the new game object.
    */
    const newGame = {
        id: nextId,
        title,
        genre,
        platform,
        releaseYear,
        ageRating,
        available: true
    };

    /*
    Adds the new game to the temporary array.
    */
    games.push(newGame);

    /*
    Returns a 201 Created response.

    The response includes the game that was created.
    */
    return res.status(201).json({
        message: "Game created successfully.",
        data: newGame
    });

};

/*
Exports the controller functions so that the route file can use
them.
*/
module.exports = {
    getAllGames,
    getGameById,
    createGame
};
```

---

# 🎯 Part 11: Create the Game Routes

Inside the `routes` folder, create:

```text
gameRoutes.js
```

The complete path should be:

```text
backend/routes/gameRoutes.js
```

Add:

```javascript
const express = require("express");

/*
Imports the game controller functions.
*/
const {
    getAllGames,
    getGameById,
    createGame
} = require("../controllers/gameController");

/*
Imports the validation middleware used when creating a game.
*/
const validateGame = require("../middleware/validateGame");

/*
Creates the game router.
*/
const router = express.Router();

/*
GET /games

The /games base path will be added inside app.js.
Therefore, this route only needs "/".
*/
router.get("/", getAllGames);

/*
GET /games/:id

The value after /games/ will be available through req.params.id.
*/
router.get("/:id", getGameById);

/*
POST /games

The request first passes through validateGame.

If validation succeeds, createGame runs next.
If validation fails, createGame is not called.
*/
router.post("/", validateGame, createGame);

/*
Exports the router.
*/
module.exports = router;
```

## Route execution order

For a valid POST request, Express runs:

```text
POST /games
        ↓
validateGame
        ↓
createGame
        ↓
Response
```

For an invalid POST request:

```text
POST /games
        ↓
validateGame
        ↓
400 response
```

The controller does not run when validation fails.

---

# ❓ Part 12: Create the Not-Found Middleware

Inside the `middleware` folder, create:

```text
notFound.js
```

The complete path should be:

```text
backend/middleware/notFound.js
```

Add:

```javascript
/*
Handles requests that do not match any valid route.

This middleware must be registered after all valid routes.
*/
const notFound = (req, res) => {

    return res.status(404).json({
        error: "Route not found."
    });

};

/*
Exports the middleware.
*/
module.exports = notFound;
```

The not-found middleware should not expose:

* File paths.
* Stack traces.
* Certificate information.
* Environment variables.
* Server configuration.

---

# ⚠️ Part 13: Create the Error-Handling Middleware

Inside the `middleware` folder, create:

```text
errorHandler.js
```

The complete path should be:

```text
backend/middleware/errorHandler.js
```

Add:

```javascript
/*
Handles unexpected errors passed through the Express
application.

An Express error-handling middleware function must have four
parameters:

err, req, res and next
*/
const errorHandler = (err, req, res, next) => {

    /*
    Uses the status code attached to the error where available.

    If no status code exists, the server returns 500.
    */
    const statusCode = err.statusCode || 500;

    /*
    Checks whether the application is running in development
    mode.
    */
    const isDevelopment =
        process.env.NODE_ENV === "development";

    /*
    Returns a controlled JSON response.

    The stack trace is included only during development.
    Production users should not receive internal technical
    information.
    */
    return res.status(statusCode).json({
        error:
            statusCode === 500
                ? "An unexpected server error occurred."
                : err.message,
        stack: isDevelopment ? err.stack : undefined
    });

};

/*
Exports the error-handling middleware.
*/
module.exports = errorHandler;
```

## Why `next` is included

Although this function may not directly call `next`, Express identifies it as an error handler because it contains four parameters:

```javascript
(err, req, res, next)
```

Removing one of these parameters may cause Express to treat it as normal middleware.

---

# 🔐 Part 14: Create the HTTPS Configuration File

Inside the `config` folder, create:

```text
httpsConfig.js
```

The complete path should be:

```text
backend/config/httpsConfig.js
```

Add:

```javascript
/*
Imports Node.js modules.

fs reads the certificate files.
path creates reliable full file paths.
*/
const fs = require("fs");
const path = require("path");

/*
Creates the full path to the backend folder.

__dirname currently refers to backend/config.

Moving one level upward produces the backend folder.
*/
const backendDirectory = path.resolve(__dirname, "..");

/*
Reads the relative certificate paths from the environment
variables.

Fallback values are supplied when an environment variable is
not available.
*/
const sslKeyPath =
    process.env.SSL_KEY_PATH ||
    "certificates/privatekey.pem";

const sslCertPath =
    process.env.SSL_CERT_PATH ||
    "certificates/certificate.pem";

/*
Combines the backend directory with each relative certificate
path.

This produces full paths to the certificate files.
*/
const resolvedKeyPath = path.resolve(
    backendDirectory,
    sslKeyPath
);

const resolvedCertPath = path.resolve(
    backendDirectory,
    sslCertPath
);

/*
Reads the private key and certificate files.

The HTTPS server requires both files.
*/
const httpsOptions = {
    key: fs.readFileSync(resolvedKeyPath),
    cert: fs.readFileSync(resolvedCertPath)
};

/*
Exports the HTTPS configuration object.
*/
module.exports = httpsOptions;
```

## Important path detail

Because this file is inside:

```text
backend/config
```

`__dirname` refers to the `config` folder.

The code first moves one level upward to reach:

```text
backend
```

It can then locate:

```text
backend/certificates/privatekey.pem
backend/certificates/certificate.pem
```

---

# ⚙️ Part 15: Create app.js

Inside the main `backend` folder, create:

```text
app.js
```

The complete path should be:

```text
backend/app.js
```

Add:

```javascript
const express = require("express");

/*
Imports the route files.
*/
const systemRoutes = require("./routes/systemRoutes");
const gameRoutes = require("./routes/gameRoutes");

/*
Imports the middleware that handles invalid routes and
unexpected errors.
*/
const notFound = require("./middleware/notFound");
const errorHandler = require("./middleware/errorHandler");

/*
Creates the Express application.
*/
const app = express();

/*
Enables the application to read JSON request bodies.

Without this middleware, req.body may be undefined when a
client sends JSON.
*/
app.use(express.json());

/*
Registers the system routes.

The system router defines:

GET /
GET /about
GET /health
*/
app.use("/", systemRoutes);

/*
Registers the game routes under the /games base path.

The routes inside gameRoutes.js are combined with this path.

For example:

router.get("/") becomes GET /games
router.get("/:id") becomes GET /games/:id
router.post("/") becomes POST /games
*/
app.use("/games", gameRoutes);

/*
Handles requests that do not match a valid route.

This must be registered after the valid routes.
*/
app.use(notFound);

/*
Handles unexpected application errors.

This must be registered after the routes and not-found
middleware.
*/
app.use(errorHandler);

/*
Exports the configured Express application.

server.js will import the application and use it to create the
HTTPS server.
*/
module.exports = app;
```

## app.js responsibility

`app.js`:

* Creates Express.
* Enables JSON processing.
* Registers routes.
* Registers middleware.
* Exports the application.

It does not start the server.

---

# 🚀 Part 16: Replace server.js

Once all the new files have been created, replace the existing contents of:

```text
backend/server.js
```

with:

```javascript
/*
Loads environment variables before the other application files
are imported.

This ensures that the HTTPS configuration and controllers can
access process.env values.
*/
require("dotenv").config();

/*
Imports the built-in Node.js HTTPS module.
*/
const https = require("https");

/*
Imports the configured Express application from app.js.
*/
const app = require("./app");

/*
Imports the private-key and certificate configuration.
*/
const httpsOptions = require("./config/httpsConfig");

/*
Reads the server configuration from the environment variables.

Fallback values are provided when the environment variables are
not available.
*/
const HTTPS_PORT = process.env.HTTPS_PORT || 4000;
const APP_NAME =
    process.env.APP_NAME || "GameVault API";

/*
Creates and starts the HTTPS server.

httpsOptions contains the private key and certificate.

app contains the configured Express application.
*/
const server = https.createServer(httpsOptions, app);

/*
Starts listening for incoming HTTPS requests.
*/
server.listen(HTTPS_PORT, () => {

    console.log(
        `${APP_NAME} is running securely on https://localhost:${HTTPS_PORT}`
    );

});

/*
Handles server startup errors.

For example, EADDRINUSE occurs when another application is
already using the selected port.
*/
server.on("error", error => {

    console.error("The GameVault server could not start.");
    console.error(error.message);

});
```

The new `server.js` is much smaller because it now has one main responsibility:

```text
Starting the HTTPS server
```

---

# 🌍 Part 17: Review the Environment File

Your existing `backend/.env` file should remain:

```env
HTTPS_PORT=4000
APP_NAME=GameVault API
NODE_ENV=development
SSL_KEY_PATH=certificates/privatekey.pem
SSL_CERT_PATH=certificates/certificate.pem
```

Your `backend/.env.example` should contain the same variable names:

```env
HTTPS_PORT=4000
APP_NAME=GameVault API
NODE_ENV=development
SSL_KEY_PATH=certificates/privatekey.pem
SSL_CERT_PATH=certificates/certificate.pem
```

The `.env` file should not be committed to GitHub.

The `.env.example` file should be committed.

---

# 🚫 Part 18: Review .gitignore

The `.gitignore` in the main `GameVault` folder should contain:

```gitignore
backend/node_modules/
backend/.env
backend/certificates/

frontend/node_modules/
frontend/.env
```

The following source folders must not be ignored:

```text
backend/config
backend/controllers
backend/data
backend/middleware
backend/routes
```

---

# ▶️ Part 19: Start the Refactored Backend

Confirm that the terminal is inside:

```text
GameVault/backend
```

Start the development server:

```bash
npm run dev
```

The terminal should display:

```text
GameVault API is running securely on https://localhost:4000
```

Use:

```text
https://localhost:4000
```

Do not use:

```text
http://localhost:4000
```

---

# 🧪 Part 20: Test the System Endpoints

Test the following requests in Postman.

## Root route

```text
GET https://localhost:4000/
```

Expected status:

```text
200 OK
```

## About route

```text
GET https://localhost:4000/about
```

Expected status:

```text
200 OK
```

## Health route

```text
GET https://localhost:4000/health
```

Expected status:

```text
200 OK
```

The health response should show:

```text
protocol: HTTPS
```

---

# 🎯 Part 21: Test the Game Endpoints

## Retrieve all games

```text
GET https://localhost:4000/games
```

Expected status:

```text
200 OK
```

## Retrieve one game

```text
GET https://localhost:4000/games/1
```

Expected status:

```text
200 OK
```

## Retrieve a missing game

```text
GET https://localhost:4000/games/999
```

Expected status:

```text
404 Not Found
```

## Use an invalid ID

```text
GET https://localhost:4000/games/test
```

Expected status:

```text
400 Bad Request
```

---

# 📥 Part 22: Test Valid Game Creation

Send:

```text
POST https://localhost:4000/games
```

Select:

```text
Body → raw → JSON
```

Use:

```json
{
    "title": "Minecraft",
    "genre": "Sandbox",
    "platform": "PC",
    "releaseYear": 2011,
    "ageRating": "E10+"
}
```

Expected status:

```text
201 Created
```

After adding the game, send:

```text
GET https://localhost:4000/games
```

The new game should appear in the array.

The game will disappear when the server restarts because MongoDB has not yet been added.

---

# 🛡️ Part 23: Test Invalid Game Creation

## Missing title

```json
{
    "genre": "Sandbox",
    "platform": "PC",
    "releaseYear": 2011,
    "ageRating": "E10+"
}
```

Expected status:

```text
400 Bad Request
```

## Invalid release year

```json
{
    "title": "Example Game",
    "genre": "Adventure",
    "platform": "PC",
    "releaseYear": "2024",
    "ageRating": "T"
}
```

The release year is text rather than a number.

Expected status:

```text
400 Bad Request
```

## Invalid age rating

```json
{
    "title": "Example Game",
    "genre": "Adventure",
    "platform": "PC",
    "releaseYear": 2024,
    "ageRating": "PG"
}
```

Expected status:

```text
400 Bad Request
```

## Whitespace-only title

```json
{
    "title": "   ",
    "genre": "Adventure",
    "platform": "PC",
    "releaseYear": 2024,
    "ageRating": "T"
}
```

Expected status:

```text
400 Bad Request
```

---

# ❓ Part 24: Test the Not-Found Middleware

Send:

```text
GET https://localhost:4000/invalid-route
```

Expected status:

```text
404 Not Found
```

Expected response:

```json
{
    "error": "Route not found."
}
```

This confirms that valid routes are checked before the not-found middleware runs.

---

# 🔄 Part 25: Understand the Request Flow

When the following request is sent:

```text
POST /games
```

the request flows through the application in this order:

```text
HTTPS server
    ↓
app.js
    ↓
JSON middleware
    ↓
gameRoutes.js
    ↓
validateGame.js
    ↓
gameController.js
    ↓
games.js
    ↓
JSON response
```

When an invalid route is requested:

```text
HTTPS server
    ↓
app.js
    ↓
Valid routes checked
    ↓
No route matched
    ↓
notFound.js
    ↓
404 response
```

When an unexpected error is passed to Express:

```text
Route or controller
    ↓
Error passed through Express
    ↓
errorHandler.js
    ↓
Controlled error response
```

---

# 📦 Part 26: Confirm the Installed Packages

No additional packages are required for this section.

Run:

```bash
npm list --depth=0
```

The project should still include:

```text
dotenv
express
nodemon
```

The following are built-in Node.js modules and do not need to be installed:

```text
https
fs
path
```

Do not run:

```bash
npm install https
```

Do not run:

```bash
npm install fs
```

Do not run:

```bash
npm install path
```

---

# ☁️ Part 27: Commit the Refactored Backend

Check the files waiting to be committed:

```bash
git status
```

Confirm that the following do not appear:

```text
backend/.env
backend/node_modules
backend/certificates/privatekey.pem
backend/certificates/certificate.pem
```

Stage the source files:

```bash
git add .
```

Review them again:

```bash
git status
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

# 🛠️ Part 28: Troubleshooting

## Cannot find module

An error such as:

```text
Cannot find module '../controllers/gameController'
```

usually means that:

* The file is in the wrong folder.
* The filename is incorrect.
* The relative path is incorrect.
* The function was not exported.

Check the project structure and spelling carefully.

---

## Certificate file cannot be found

An error containing:

```text
ENOENT
```

means Node.js cannot locate a file.

Confirm that these files exist:

```text
backend/certificates/privatekey.pem
backend/certificates/certificate.pem
```

Confirm that `.env` contains:

```env
SSL_KEY_PATH=certificates/privatekey.pem
SSL_CERT_PATH=certificates/certificate.pem
```

---

## Every endpoint returns route not found

The not-found middleware may be registered before the routes.

The order inside `app.js` must be:

```text
JSON middleware
System routes
Game routes
Not-found middleware
Error-handling middleware
```

---

## POST requests do not have a body

Confirm that `app.js` contains JSON request-body middleware before the routes.

The order matters.

JSON processing must be enabled before the game router is registered.

---

## Valid requests never reach the controller

Confirm that the validation middleware calls:

```javascript
next();
```

after all validation checks pass.

Without `next()`, the request stops inside the middleware.

---

## req.validatedGame is undefined

Confirm that:

* `validateGame` runs before `createGame`.
* The middleware assigns the cleaned object to `req.validatedGame`.
* The POST route lists the functions in the correct order.

The order must be:

```text
validateGame
createGame
```

---

## Port 4000 is already in use

Stop the existing server:

```text
Ctrl + C
```

Then start the backend again:

```bash
npm run dev
```

Alternatively, change the port in `.env`.

---

# ✅ Part 29: Completion Checklist

Before continuing, confirm that:

* The backend contains a `config` folder.
* The backend contains a `controllers` folder.
* The backend contains a `data` folder.
* The backend contains a `middleware` folder.
* The backend contains a `routes` folder.
* `app.js` creates and configures Express.
* `server.js` starts the HTTPS server.
* `httpsConfig.js` reads the key and certificate.
* `games.js` exports one shared games array.
* `systemController.js` handles the system responses.
* `gameController.js` handles game operations.
* `systemRoutes.js` defines the system endpoints.
* `gameRoutes.js` defines the game endpoints.
* `validateGame.js` validates incoming game data.
* `notFound.js` handles unknown routes.
* `errorHandler.js` handles unexpected errors.
* HTTPS still works.
* All existing LU1 endpoints still work.
* Valid games can still be added.
* Invalid games are rejected.
* Invalid routes return `404`.
* `.env` is ignored.
* The certificate folder is ignored.
* `node_modules` is ignored.
* The new source files have been committed.
* The changes have been pushed to GitHub.

---

# 📋 Terminal Command Summary

Run these commands from:

```text
GameVault/backend
```

Create the folders:

```bash
mkdir config
mkdir controllers
mkdir data
mkdir middleware
mkdir routes
```

Check installed dependencies:

```bash
npm list --depth=0
```

Start the development server:

```bash
npm run dev
```

Stop the server:

```text
Ctrl + C
```

Check Git:

```bash
git status
```

Stage the changes:

```bash
git add .
```

Commit the refactoring:

```bash
git commit -m "Refactor backend into routes controllers and middleware"
```

Push the project:

```bash
git push
```

---
