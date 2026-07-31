# 🧱 GameVault Learning Unit 2 Part A: Refactoring the Backend

## 📌 Overview

In Learning Unit 1, the complete GameVault backend was written inside one file:

```text
backend/server.js
```

The file currently performs several different jobs:

* Loads environment variables.
* Creates the Express application.
* enables JSON request bodies.
* Stores the temporary game data.
* Defines the routes.
* Validates incoming data.
* Creates responses.
* Handles invalid routes.
* Reads the HTTPS certificate files.
* Creates the HTTPS server.
* Starts the server.

The application works, but `server.js` now has too many responsibilities.

In this section, we are going to **refactor** the backend.

Refactoring means:

> Improving the structure of existing code without changing what the application does.

The endpoints should still work exactly as before:

```text
GET  /
GET  /about
GET  /health
GET  /games
GET  /games/:id
POST /games
```

We are not rebuilding the backend from the beginning.

We are going to look at the current `server.js`, identify each section and move that section into a more suitable file.

---

# 📚 Learning Outcomes

By the end of this section, you should be able to:

* Explain separation of concerns.
* Explain why large backend files become difficult to maintain.
* Identify the different responsibilities inside `server.js`.
* Move code from one file into appropriate folders.
* Explain how files communicate using `require()` and `module.exports`.
* Explain how a request moves through an Express application.
* Separate routes from controllers.
* Separate validation from controller logic.
* Separate Express configuration from server startup.
* Maintain HTTPS after reorganising the backend.
* Test the application after each refactoring step.

---

# 🧠 Part 1: Understand the Goal

Before refactoring, the request flow is simple:

```text
Postman
   ->
server.js
   ->
Response
```

Everything happens inside `server.js`.

After refactoring, the request will move through several specialised files:

```text
Postman
   ->
server.js
   ->
app.js
   ->
routes
   ->
middleware
   ->
controllers
   ->
data
   ->
Response
```

This may look more complicated at first, but each file has one clear responsibility.

The files communicate using:

```javascript
require()
module.exports
req
res
next
```

We will move the code in the same order that a request flows through the backend.

---

# 🔍 Part 2: Review `server.js` Before Refactoring

Open:

```text
backend/server.js
```

Your file should contain sections similar to the following.

The exact comments or formatting may be slightly different, but the responsibilities should be the same.

```javascript
/*
Loads environment variables from the .env file.
*/
require("dotenv").config();

/*
Imports the required modules.
*/
const express = require("express");
const https = require("https");
const fs = require("fs");
const path = require("path");

/*
Creates the Express application.
*/
const app = express();

/*
Allows Express to read JSON request bodies.
*/
app.use(express.json());

/*
Temporary in-memory game data.
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
System routes.
*/
app.get("/", (req, res) => {

    const appName =
        process.env.APP_NAME || "GameVault API";

    return res.status(200).json({
        application: appName,
        message: "Welcome to the GameVault API"
    });

});

app.get("/about", (req, res) => {

    const appName =
        process.env.APP_NAME || "GameVault API";

    return res.status(200).json({
        application: appName,
        description:
            "GameVault is a secure video game collection and review platform."
    });

});

app.get("/health", (req, res) => {

    const appName =
        process.env.APP_NAME || "GameVault API";

    return res.status(200).json({
        application: appName,
        status: "OK",
        protocol: "HTTPS",
        environment:
            process.env.NODE_ENV || "development",
        timestamp: new Date().toISOString()
    });

});

/*
Returns all games.
*/
app.get("/games", (req, res) => {

    return res.status(200).json({
        count: games.length,
        data: games
    });

});

/*
Returns one game using its ID.
*/
app.get("/games/:id", (req, res) => {

    const gameId = Number(req.params.id);

    if (!Number.isInteger(gameId)) {
        return res.status(400).json({
            error: "Game ID must be a whole number."
        });
    }

    const game = games.find(
        currentGame => currentGame.id === gameId
    );

    if (!game) {
        return res.status(404).json({
            error: "Game not found."
        });
    }

    return res.status(200).json({
        data: game
    });

});

/*
Creates a new game.
*/
app.post("/games", (req, res) => {

    const {
        title,
        genre,
        platform,
        releaseYear,
        ageRating
    } = req.body;

    /*
    Checks that the required fields were supplied.
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
    Checks that the text values contain strings.
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

    const cleanedTitle = title.trim();
    const cleanedGenre = genre.trim();
    const cleanedPlatform = platform.trim();
    const cleanedAgeRating =
        ageRating.trim().toUpperCase();

    const currentYear =
        new Date().getFullYear();

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

    const allowedAgeRatings = [
        "E",
        "E10+",
        "T",
        "M",
        "18"
    ];

    if (
        !allowedAgeRatings.includes(
            cleanedAgeRating
        )
    ) {
        return res.status(400).json({
            error:
                `Age rating must be one of: ${allowedAgeRatings.join(", ")}.`
        });
    }

    const nextId =
        games.length > 0
            ? Math.max(
                ...games.map(game => game.id)
            ) + 1
            : 1;

    const newGame = {
        id: nextId,
        title: cleanedTitle,
        genre: cleanedGenre,
        platform: cleanedPlatform,
        releaseYear,
        ageRating: cleanedAgeRating,
        available: true
    };

    games.push(newGame);

    return res.status(201).json({
        message: "Game created successfully.",
        data: newGame
    });

});

/*
Handles requests that do not match a valid route.
*/
app.use((req, res) => {

    return res.status(404).json({
        error: "Route not found."
    });

});

/*
Reads the HTTPS certificate files.
*/
const backendDirectory = __dirname;

const httpsOptions = {
    key: fs.readFileSync(
        path.resolve(
            backendDirectory,
            process.env.SSL_KEY_PATH
        )
    ),
    cert: fs.readFileSync(
        path.resolve(
            backendDirectory,
            process.env.SSL_CERT_PATH
        )
    )
};

/*
Reads the port from the environment.
*/
const HTTPS_PORT =
    process.env.HTTPS_PORT || 4000;

/*
Creates the HTTPS server.
*/
const server =
    https.createServer(httpsOptions, app);

/*
Starts the server.
*/
server.listen(HTTPS_PORT, () => {

    console.log(
        `GameVault API is running securely on https://localhost:${HTTPS_PORT}`
    );

});
```

This file works, but it mixes several unrelated responsibilities.

The existing guide already identifies these responsibilities and the final folder structure. The purpose of the new process is to show where each existing section moves and how it remains connected. 

---

# 🗂️ Part 3: Identify What Will Move

Before moving any code, identify the destination for each section.

```text
Current code inside server.js          Destination

games array                            data/games.js

Root, about and health logic           controllers/systemController.js

Game response logic                    controllers/gameController.js

System route paths                     routes/systemRoutes.js

Game route paths                       routes/gameRoutes.js

POST validation                        middleware/validateGame.js

Invalid-route handling                 middleware/notFound.js

Unexpected-error handling              middleware/errorHandler.js

HTTPS certificate setup                config/httpsConfig.js

Express setup and route registration   app.js

HTTPS server startup                   server.js
```

The most important principle is:

> We are moving code, not duplicating it.

When a section is moved successfully, it should be removed from `server.js`.

---

# 🧭 Part 4: The Refactoring Order

We will refactor in this order:

```text
1. Move the games array
2. Move the system response functions
3. Create the system routes
4. Move the game response functions
5. Move validation into middleware
6. Create the game routes
7. Move invalid-route handling
8. Add central error handling
9. Move Express setup into app.js
10. Move HTTPS configuration
11. Leave server startup inside server.js
```

After each major step, we will test the application.

This allows us to identify exactly which change caused an error.

---

# 🛑 Part 5: Stop the Existing Server

Before moving code, stop the running server.

Click inside the terminal and press:

```text
Ctrl + C
```

Confirm that the terminal returns to the command prompt.

---

# 📍 Part 6: Confirm the Terminal Location

All commands must be run from:

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

# 📁 Part 7: Create the Destination Folders

Run:

```bash
mkdir config
mkdir controllers
mkdir data
mkdir middleware
mkdir routes
```

The backend should now contain:

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

At this stage, the folders are empty.

We will now move existing code into them.

---

# 🎮 Part 8: Move the Games Array

## 8.1 Find the array in `server.js`

Locate:

```javascript
const games = [
    // Game objects
];
```

Cut the complete array from `server.js`.

Do not leave a second copy behind.

## 8.2 Create the data file

Create:

```text
backend/data/games.js
```

Paste the array into the file:

```javascript
/*
Temporary in-memory game data.

This array was previously stored inside server.js.

It has been moved because server.js should not be responsible
for storing application data.
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
Exports the array.

Another file can import and use this same array.
*/
module.exports = games;
```

## 8.3 Import the array into `server.js`

For now, add this near the top of `server.js`:

```javascript
/*
Imports the shared games array from data/games.js.
*/
const games = require("./data/games");
```

The routes inside `server.js` can still use:

```javascript
games
```

The only difference is that the array now lives in another file.

## 8.4 How the files communicate

Inside `games.js`:

```javascript
module.exports = games;
```

This means:

> Make the games array available to another file.

Inside `server.js`:

```javascript
const games = require("./data/games");
```

This means:

> Import the value exported by `data/games.js`.

The communication flow is:

```text
data/games.js
      ->
module.exports
      ->
require("./data/games")
      ->
server.js
```

## 8.5 Important detail

Do not create another array inside `server.js`.

Both GET and POST must use the same shared array.

## 8.6 Test the application

Start the server:

```bash
npm run dev
```

Test:

```text
GET https://localhost:4000/games
GET https://localhost:4000/games/1
POST https://localhost:4000/games
```

The routes should still work.

Stop the server before continuing:

```text
Ctrl + C
```

---

# 🎛️ Part 9: Move the System Response Logic

The system routes currently look similar to:

```javascript
app.get("/", (req, res) => {
    // Response logic
});
```

This block contains two responsibilities:

```text
app.get("/")
    ->
Route information

(req, res) => { ... }
    ->
Controller logic
```

The route determines:

* The HTTP method.
* The path.

The controller determines:

* What the application should do.
* Which response should be returned.

We will separate them.

## 9.1 Create the controller

Create:

```text
backend/controllers/systemController.js
```

Move the callback logic from the three system routes into named functions:

```javascript
/*
Contains the response logic that was previously inside:

app.get("/", ...)
*/
const getRoot = (req, res) => {

    const appName =
        process.env.APP_NAME || "GameVault API";

    return res.status(200).json({
        application: appName,
        message: "Welcome to the GameVault API"
    });

};

/*
Contains the response logic that was previously inside:

app.get("/about", ...)
*/
const getAbout = (req, res) => {

    const appName =
        process.env.APP_NAME || "GameVault API";

    return res.status(200).json({
        application: appName,
        description:
            "GameVault is a secure video game collection and review platform.",
        currentStage:
            "Learning Unit 2 - Refactoring the Backend"
    });

};

/*
Contains the response logic that was previously inside:

app.get("/health", ...)
*/
const getHealth = (req, res) => {

    const appName =
        process.env.APP_NAME || "GameVault API";

    return res.status(200).json({
        application: appName,
        status: "OK",
        protocol: "HTTPS",
        environment:
            process.env.NODE_ENV || "development",
        timestamp: new Date().toISOString()
    });

};

/*
Exports the controller functions.

The route file will import these functions.
*/
module.exports = {
    getRoot,
    getAbout,
    getHealth
};
```

## 9.2 What changed?

Before:

```javascript
app.get("/", (req, res) => {

    return res.status(200).json({
        message: "Welcome"
    });

});
```

After:

```javascript
const getRoot = (req, res) => {

    return res.status(200).json({
        message: "Welcome"
    });

};
```

The response logic has not changed.

It has simply been:

* Moved.
* Given a name.
* Exported.

---

# 🛣️ Part 10: Create the System Routes

Create:

```text
backend/routes/systemRoutes.js
```

Add:

```javascript
const express = require("express");

/*
Imports the controller functions.

These functions contain the response logic.
*/
const {
    getRoot,
    getAbout,
    getHealth
} = require("../controllers/systemController");

/*
Creates a smaller Express router.

This router handles only the system endpoints.
*/
const router = express.Router();

/*
Connects each route to the correct controller function.
*/
router.get("/", getRoot);
router.get("/about", getAbout);
router.get("/health", getHealth);

/*
Exports the router.
*/
module.exports = router;
```

## 10.1 How routes speak to controllers

This line:

```javascript
router.get("/", getRoot);
```

means:

```text
When a GET request reaches /
        ->
Run the getRoot function
        ->
getRoot sends the response
```

The route file knows:

* The method is `GET`.
* The path is `/`.
* The controller is `getRoot`.

The controller knows:

* What data to return.
* Which status code to use.

## 10.2 Temporarily connect the router to `server.js`

At the top of `server.js`, add:

```javascript
const systemRoutes =
    require("./routes/systemRoutes");
```

Remove the original three system route blocks:

```javascript
app.get("/", ...);
app.get("/about", ...);
app.get("/health", ...);
```

Replace them with:

```javascript
/*
Sends system requests to systemRoutes.js.
*/
app.use("/", systemRoutes);
```

## 10.3 How the route path is built

Inside `server.js`:

```javascript
app.use("/", systemRoutes);
```

Inside `systemRoutes.js`:

```javascript
router.get("/about", getAbout);
```

Combined route:

```text
/ + /about = /about
```

## 10.4 Test the system routes

Start the backend:

```bash
npm run dev
```

Test:

```text
GET https://localhost:4000/
GET https://localhost:4000/about
GET https://localhost:4000/health
```

The responses should remain the same.

Stop the server before continuing.

---

# 🎮 Part 11: Move the Game Controller Logic

The game routes currently contain both route information and game-processing logic.

We will move the processing logic into:

```text
backend/controllers/gameController.js
```

Create the file and add:

```javascript
/*
Imports the shared games array.

The controller does not create a new array.
It uses the array exported by data/games.js.
*/
const games = require("../data/games");

/*
Contains the logic previously used by:

GET /games
*/
const getAllGames = (req, res) => {

    return res.status(200).json({
        count: games.length,
        data: games
    });

};

/*
Contains the logic previously used by:

GET /games/:id
*/
const getGameById = (req, res) => {

    /*
    Route parameters arrive as text.

    Number converts the supplied value into a number.
    */
    const gameId = Number(req.params.id);

    if (!Number.isInteger(gameId)) {
        return res.status(400).json({
            error: "Game ID must be a whole number."
        });
    }

    const game = games.find(
        currentGame => currentGame.id === gameId
    );

    if (!game) {
        return res.status(404).json({
            error: "Game not found."
        });
    }

    return res.status(200).json({
        data: game
    });

};

/*
Contains the creation logic previously used by:

POST /games

Validation will happen before this function runs.
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

    const nextId =
        games.length > 0
            ? Math.max(
                ...games.map(game => game.id)
            ) + 1
            : 1;

    const newGame = {
        id: nextId,
        title,
        genre,
        platform,
        releaseYear,
        ageRating,
        available: true
    };

    games.push(newGame);

    return res.status(201).json({
        message: "Game created successfully.",
        data: newGame
    });

};

/*
Exports the controller functions.
*/
module.exports = {
    getAllGames,
    getGameById,
    createGame
};
```

## 11.1 How the controller speaks to the data file

Inside `gameController.js`:

```javascript
const games = require("../data/games");
```

This imports the same array exported by `games.js`.

The request flow is:

```text
gameController.js
        -> require
data/games.js
        ->
shared games array
```

When `createGame` runs:

```javascript
games.push(newGame);
```

the shared array changes.

When `getAllGames` runs later, it reads the same changed array.

---

# 🛡️ Part 12: Move Validation into Middleware

The current POST route contains two different responsibilities:

```text
Validate the incoming request
Create the new game
```

These should be separated.

The validation should run before the controller.

Create:

```text
backend/middleware/validateGame.js
```

Move the validation code from the old POST route into this file:

```javascript
/*
Validates the request body before a new game is created.

Middleware runs before the controller.
*/
const validateGame = (req, res, next) => {

    const {
        title,
        genre,
        platform,
        releaseYear,
        ageRating
    } = req.body;

    /*
    Checks that all required fields were supplied.
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
    Checks that the expected text values are strings.
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
    Cleans the text values.
    */
    const cleanedTitle = title.trim();
    const cleanedGenre = genre.trim();
    const cleanedPlatform = platform.trim();
    const cleanedAgeRating =
        ageRating.trim().toUpperCase();

    /*
    Validates title length.
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
    Validates genre length.
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
    Validates platform length.
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

    const currentYear =
        new Date().getFullYear();

    /*
    Validates the release year.
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

    const allowedAgeRatings = [
        "E",
        "E10+",
        "T",
        "M",
        "18"
    ];

    /*
    Validates the age rating.
    */
    if (
        !allowedAgeRatings.includes(
            cleanedAgeRating
        )
    ) {
        return res.status(400).json({
            error:
                `Age rating must be one of: ${allowedAgeRatings.join(", ")}.`
        });
    }

    /*
    Adds the cleaned values to the request object.

    The same request object will continue to the controller.
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
    */
    next();

};

/*
Exports the middleware.
*/
module.exports = validateGame;
```

---

# 🔄 Part 13: Understand `req`, `res` and `next`

Middleware receives:

```javascript
(req, res, next)
```

## `req`

`req` contains information about the incoming request.

Examples:

```javascript
req.body
req.params
req.headers
req.method
req.path
```

## `res`

`res` is used to send a response.

Example:

```javascript
return res.status(400).json({
    error: "Invalid data."
});
```

## `next`

`next()` tells Express:

> This middleware has finished successfully. Continue to the next function.

For the POST route, the flow will be:

```text
Postman sends req.body
        ->
validateGame reads req.body
        ->
validateGame checks the values
        ->
validateGame creates req.validatedGame
        ->
next()
        ->
createGame reads req.validatedGame
        ->
Response
```

The same `req` object travels through the middleware and controller.

That is why the middleware can write:

```javascript
req.validatedGame = {
    // Cleaned data
};
```

and the controller can later read:

```javascript
req.validatedGame
```

## When validation fails

When validation fails, the middleware returns a response:

```javascript
return res.status(400).json({
    error: "Invalid data."
});
```

It does not call:

```javascript
next();
```

Therefore, the controller does not run.

---

# 🎯 Part 14: Create the Game Routes

Create:

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
Imports the validation middleware.
*/
const validateGame =
    require("../middleware/validateGame");

/*
Creates a router for game-related endpoints.
*/
const router = express.Router();

/*
Returns all games.
*/
router.get("/", getAllGames);

/*
Returns one game.
*/
router.get("/:id", getGameById);

/*
Creates a new game.

Express runs the functions from left to right.
*/
router.post(
    "/",
    validateGame,
    createGame
);

/*
Exports the router.
*/
module.exports = router;
```

## 14.1 Explain the POST execution order

This route:

```javascript
router.post(
    "/",
    validateGame,
    createGame
);
```

does not run everything at the same time.

Express runs the functions from left to right:

```text
POST /
   ->
validateGame
   ->
next()
   ->
createGame
   ->
Response
```

When validation fails:

```text
POST /
   ->
validateGame
   ->
400 response
```

`createGame` is skipped.

## 14.2 Connect the route to `server.js`

At the top of `server.js`, add:

```javascript
const gameRoutes =
    require("./routes/gameRoutes");
```

Remove the original game route blocks:

```javascript
app.get("/games", ...);
app.get("/games/:id", ...);
app.post("/games", ...);
```

Replace them with:

```javascript
app.use("/games", gameRoutes);
```

## 14.3 How the paths combine

Inside `server.js`:

```javascript
app.use("/games", gameRoutes);
```

Inside `gameRoutes.js`:

```javascript
router.get("/", getAllGames);
```

Combined:

```text
/games + / = /games
```

Inside `gameRoutes.js`:

```javascript
router.get("/:id", getGameById);
```

Combined:

```text
/games + /:id = /games/:id
```

Inside `gameRoutes.js`:

```javascript
router.post("/", validateGame, createGame);
```

Combined:

```text
POST /games
```

---

# 🧪 Part 15: Test the Game Routes

Start the server:

```bash
npm run dev
```

Test:

```text
GET https://localhost:4000/games
GET https://localhost:4000/games/1
GET https://localhost:4000/games/999
GET https://localhost:4000/games/test
POST https://localhost:4000/games
```

Test a valid POST body:

```json
{
    "title": "Minecraft",
    "genre": "Sandbox",
    "platform": "PC",
    "releaseYear": 2011,
    "ageRating": "E10+"
}
```

Also test an invalid body:

```json
{
    "genre": "Sandbox",
    "platform": "PC",
    "releaseYear": 2011,
    "ageRating": "E10+"
}
```

Stop the server before continuing.

---

# ❓ Part 16: Move the Not-Found Handler

The old `server.js` contains:

```javascript
app.use((req, res) => {

    return res.status(404).json({
        error: "Route not found."
    });

});
```

Move this function into:

```text
backend/middleware/notFound.js
```

Add:

```javascript
/*
Handles requests that do not match any valid route.
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

For now, import it into `server.js`:

```javascript
const notFound =
    require("./middleware/notFound");
```

Then replace the old anonymous not-found function with:

```javascript
app.use(notFound);
```

## Why must this come after the routes?

Express checks middleware in the order in which it is registered.

Correct order:

```text
Valid routes
    ->
notFound
```

Incorrect order:

```text
notFound
    ->
Valid routes
```

If `notFound` is registered first, every request immediately receives `404`.

---

# ⚠️ Part 17: Add Central Error Handling

Not-found handling and unexpected-error handling are not the same.

A not-found error means:

```text
The requested route does not exist.
```

An unexpected server error means:

```text
Something failed while processing a valid request.
```

Create:

```text
backend/middleware/errorHandler.js
```

Add:

```javascript
/*
Handles unexpected errors passed through Express.

Express recognises error-handling middleware because it has
four parameters:

err, req, res and next
*/
const errorHandler = (err, req, res, next) => {

    const statusCode =
        err.statusCode || 500;

    const isDevelopment =
        process.env.NODE_ENV === "development";

    return res.status(statusCode).json({
        error:
            statusCode === 500
                ? "An unexpected server error occurred."
                : err.message,
        stack:
            isDevelopment
                ? err.stack
                : undefined
    });

};

module.exports = errorHandler;
```

Import it into `server.js`:

```javascript
const errorHandler =
    require("./middleware/errorHandler");
```

Register it after `notFound`:

```javascript
app.use(notFound);
app.use(errorHandler);
```

## How errors reach the error handler

A route or controller can pass an error through Express:

```javascript
next(error);
```

Express skips normal middleware and looks for a function with four parameters:

```javascript
(err, req, res, next)
```

The flow is:

```text
Controller
   ->
next(error)
   ->
errorHandler
   ->
Controlled JSON response
```

---

# ⚙️ Part 18: Move Express Configuration into `app.js`

At this stage, `server.js` still:

* Imports Express.
* Creates the Express application.
* Enables JSON processing.
* Registers routes.
* Registers middleware.
* Starts HTTPS.

We will now separate the Express application from the server startup.

Create:

```text
backend/app.js
```

Move the following responsibilities from `server.js` into `app.js`:

```text
Import Express
Create app
Enable JSON
Register routes
Register middleware
Export app
```

Add:

```javascript
const express = require("express");

/*
Imports the route files.
*/
const systemRoutes =
    require("./routes/systemRoutes");

const gameRoutes =
    require("./routes/gameRoutes");

/*
Imports the final middleware.
*/
const notFound =
    require("./middleware/notFound");

const errorHandler =
    require("./middleware/errorHandler");

/*
Creates the Express application.
*/
const app = express();

/*
Allows Express to read JSON request bodies.

This must be registered before routes that use req.body.
*/
app.use(express.json());

/*
Registers the system routes.
*/
app.use("/", systemRoutes);

/*
Registers the game routes.
*/
app.use("/games", gameRoutes);

/*
Handles unmatched routes.

This must appear after all valid routes.
*/
app.use(notFound);

/*
Handles unexpected errors.

This must appear last.
*/
app.use(errorHandler);

/*
Exports the configured Express application.
*/
module.exports = app;
```

## What should be removed from `server.js`?

Remove:

```javascript
const express = require("express");
```

Remove:

```javascript
const app = express();
```

Remove:

```javascript
app.use(express.json());
```

Remove the route imports:

```javascript
const systemRoutes = ...
const gameRoutes = ...
```

Remove the middleware imports:

```javascript
const notFound = ...
const errorHandler = ...
```

Remove:

```javascript
app.use("/", systemRoutes);
app.use("/games", gameRoutes);
app.use(notFound);
app.use(errorHandler);
```

## How `server.js` will receive the app

Add to `server.js`:

```javascript
const app = require("./app");
```

## How the files communicate

Inside `app.js`:

```javascript
module.exports = app;
```

Inside `server.js`:

```javascript
const app = require("./app");
```

The flow is:

```text
app.js
  -> exports configured Express app
server.js
  -> gives app to HTTPS server
https.createServer(httpsOptions, app)
```

---

# 🔐 Part 19: Move HTTPS Configuration

The HTTPS configuration currently uses:

```javascript
fs
path
process.env.SSL_KEY_PATH
process.env.SSL_CERT_PATH
```

This code is not responsible for starting the server.

Its responsibility is only:

> Prepare the HTTPS certificate options.

Create:

```text
backend/config/httpsConfig.js
```

Move the certificate-reading logic into it:

```javascript
/*
Imports built-in Node.js modules.

fs reads files.
path creates reliable file paths.
*/
const fs = require("fs");
const path = require("path");

/*
__dirname refers to backend/config.

Moving one level upward produces the backend folder.
*/
const backendDirectory =
    path.resolve(__dirname, "..");

/*
Reads the certificate paths from the environment.
*/
const sslKeyPath =
    process.env.SSL_KEY_PATH ||
    "certificates/privatekey.pem";

const sslCertPath =
    process.env.SSL_CERT_PATH ||
    "certificates/certificate.pem";

/*
Builds complete paths to the certificate files.
*/
const resolvedKeyPath =
    path.resolve(
        backendDirectory,
        sslKeyPath
    );

const resolvedCertPath =
    path.resolve(
        backendDirectory,
        sslCertPath
    );

/*
Reads the certificate files.
*/
const httpsOptions = {
    key: fs.readFileSync(resolvedKeyPath),
    cert: fs.readFileSync(resolvedCertPath)
};

/*
Exports the HTTPS options.
*/
module.exports = httpsOptions;
```

## What should be removed from `server.js`?

Remove:

```javascript
const fs = require("fs");
const path = require("path");
```

Remove the existing certificate path and `httpsOptions` code.

Import the completed configuration instead:

```javascript
const httpsOptions =
    require("./config/httpsConfig");
```

## How the files communicate

Inside `httpsConfig.js`:

```javascript
module.exports = httpsOptions;
```

Inside `server.js`:

```javascript
const httpsOptions =
    require("./config/httpsConfig");
```

Then:

```javascript
https.createServer(httpsOptions, app);
```

`server.js` receives two prepared objects:

```text
app
    from app.js

httpsOptions
    from config/httpsConfig.js
```

It combines them to create the HTTPS server.

---

# 🚀 Part 20: Final `server.js`

After moving the other responsibilities, `server.js` should contain only server-startup logic.

Update it to:

```javascript
/*
Loads environment variables before importing application files.

This is important because app.js, controllers and the HTTPS
configuration may use process.env.
*/
require("dotenv").config();

/*
Imports the built-in HTTPS module.
*/
const https = require("https");

/*
Imports the configured Express application.
*/
const app = require("./app");

/*
Imports the prepared HTTPS certificate options.
*/
const httpsOptions =
    require("./config/httpsConfig");

/*
Reads the startup values from the environment.
*/
const HTTPS_PORT =
    process.env.HTTPS_PORT || 4000;

const APP_NAME =
    process.env.APP_NAME || "GameVault API";

/*
Creates the HTTPS server.

httpsOptions supplies the private key and certificate.

app supplies the configured Express application.
*/
const server =
    https.createServer(httpsOptions, app);

/*
Starts the server.
*/
server.listen(HTTPS_PORT, () => {

    console.log(
        `${APP_NAME} is running securely on https://localhost:${HTTPS_PORT}`
    );

});

/*
Handles server startup errors.

For example, EADDRINUSE means that the selected port is already
being used.
*/
server.on("error", error => {

    console.error(
        "The GameVault server could not start."
    );

    console.error(error.message);

});
```

`server.js` now has one main responsibility:

```text
Start the HTTPS server
```

---

# 🧭 Part 21: Final Project Structure

The backend should now resemble:

```text
backend
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

# 🔄 Part 22: Understand the Complete Startup Flow

When you run:

```bash
npm run dev
```

Node starts:

```text
server.js
```

Then the following happens:

```text
server.js
   ->
Loads .env
   ->
Imports app.js
   ->
app.js creates Express
   ->
app.js imports routes
   ->
Routes import controllers
   ->
Controllers import data
   ->
app.js registers middleware
   ->
server.js imports httpsConfig.js
   ->
httpsConfig.js reads certificates
   ->
server.js creates HTTPS server
   ->
Server begins listening
```

---

# 📥 Part 23: Understand a GET Request

For:

```text
GET /games
```

the request flow is:

```text
Postman
   ->
HTTPS server
   ->
app.js
   ->
app.use("/games", gameRoutes)
   ->
gameRoutes.js
   ->
router.get("/", getAllGames)
   ->
gameController.js
   ->
getAllGames
   ->
games.js
   ->
JSON response
```

The files communicate as follows:

```text
app.js imports gameRoutes.js

gameRoutes.js imports gameController.js

gameController.js imports games.js
```

---

# 📥 Part 24: Understand a POST Request

For:

```text
POST /games
```

the request flow is:

```text
Postman
   ->
HTTPS server
   ->
app.js
   ->
express.json()
   ->
gameRoutes.js
   ->
validateGame.js
   ->
createGame
   ->
games.js
   ->
JSON response
```

The detailed flow is:

```text
Postman sends JSON
        ->
express.json() converts JSON into req.body
        ->
validateGame reads req.body
        ->
validateGame cleans and checks the data
        ->
validateGame stores req.validatedGame
        ->
validateGame calls next()
        ->
createGame reads req.validatedGame
        ->
createGame adds a new object to games
        ->
Controller sends 201 response
```

---

# ❌ Part 25: Understand an Invalid POST Request

When the data is invalid:

```text
POST /games
   ->
validateGame
   ->
Validation fails
   ->
400 response
```

The controller does not run because the middleware does not call:

```javascript
next();
```

---

# ❓ Part 26: Understand an Invalid Route

For:

```text
GET /invalid-route
```

the flow is:

```text
HTTPS server
   ->
app.js
   ->
System routes checked
   ->
Game routes checked
   ->
No route matched
   ->
notFound.js
   ->
404 response
```

This is why `notFound` must be registered after all valid routes.

---

# ⚠️ Part 27: Understand an Unexpected Error

When a controller or route passes an error using:

```javascript
next(error);
```

the flow becomes:

```text
Route or controller
   ->
next(error)
   ->
Express skips normal middleware
   ->
errorHandler.js
   ->
Controlled error response
```

The error handler prevents the application from returning uncontrolled technical information to users.

---

# 📦 Part 28: CommonJS Communication Summary

## Export one value

```javascript
module.exports = games;
```

Import it:

```javascript
const games =
    require("../data/games");
```

## Export several functions

```javascript
module.exports = {
    getAllGames,
    getGameById,
    createGame
};
```

Import selected functions:

```javascript
const {
    getAllGames,
    getGameById,
    createGame
} = require("../controllers/gameController");
```

## Relative paths

From:

```text
routes/gameRoutes.js
```

to:

```text
controllers/gameController.js
```

use:

```javascript
require("../controllers/gameController");
```

`..` means:

```text
Move up one folder
```

---

# 🌍 Part 29: Review the Environment Files

The `.env` file should contain:

```env
HTTPS_PORT=4000
APP_NAME=GameVault API
NODE_ENV=development
SSL_KEY_PATH=certificates/privatekey.pem
SSL_CERT_PATH=certificates/certificate.pem
```

The `.env.example` file should contain the same variable names:

```env
HTTPS_PORT=4000
APP_NAME=GameVault API
NODE_ENV=development
SSL_KEY_PATH=certificates/privatekey.pem
SSL_CERT_PATH=certificates/certificate.pem
```

The real `.env` file must not be committed.

---

# 🚫 Part 30: Review `.gitignore`

The project `.gitignore` should contain:

```gitignore
backend/node_modules/
backend/.env
backend/certificates/

frontend/node_modules/
frontend/.env
```

Do not ignore:

```text
backend/config
backend/controllers
backend/data
backend/middleware
backend/routes
```

These contain source code and must be committed.

---

# ▶️ Part 31: Start the Refactored Backend

Run:

```bash
npm run dev
```

Expected output:

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

# 🧪 Part 32: Test the Endpoints

Test:

```text
GET  https://localhost:4000/
GET  https://localhost:4000/about
GET  https://localhost:4000/health
GET  https://localhost:4000/games
GET  https://localhost:4000/games/1
GET  https://localhost:4000/games/999
GET  https://localhost:4000/games/test
POST https://localhost:4000/games
```

Use this valid POST body:

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

Test an invalid route:

```text
GET https://localhost:4000/invalid-route
```

Expected status:

```text
404 Not Found
```

---

# 🛠️ Part 33: Common Problems

## Cannot find module

Example:

```text
Cannot find module '../controllers/gameController'
```

Check:

* The file exists.
* The filename is correct.
* Capitalisation is correct.
* The relative path is correct.
* The function or object was exported.

## `req.validatedGame` is undefined

Check that the POST route is:

```javascript
router.post(
    "/",
    validateGame,
    createGame
);
```

The order must be:

```text
validateGame
createGame
```

Also confirm that the middleware contains:

```javascript
req.validatedGame = {
    // Values
};

next();
```

## Every route returns 404

Check the order inside `app.js`:

```javascript
app.use("/", systemRoutes);
app.use("/games", gameRoutes);
app.use(notFound);
app.use(errorHandler);
```

`notFound` must not appear before the valid routes.

## POST body is undefined

Confirm that this appears before the routes:

```javascript
app.use(express.json());
```

## Certificate file cannot be found

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

# ✅ Part 34: Completion Checklist

Confirm that:

* The original responsibilities in `server.js` were identified.
* The games array was moved into `data/games.js`.
* The controller functions were moved into the `controllers` folder.
* Route definitions were moved into the `routes` folder.
* POST validation was moved into middleware.
* `req.validatedGame` carries data from middleware to the controller.
* `next()` passes the request to the next function.
* `app.js` creates and configures Express.
* `server.js` starts the HTTPS server.
* `httpsConfig.js` reads the key and certificate.
* `notFound.js` handles invalid routes.
* `errorHandler.js` handles unexpected errors.
* All original endpoints still work.
* HTTPS still works.
* The refactoring did not change the public API behaviour.

---

# ☁️ Part 35: Commit and Push the Refactoring

The refactoring is complete when the application behaves exactly as it did before, but the code is now organised according to responsibility.
