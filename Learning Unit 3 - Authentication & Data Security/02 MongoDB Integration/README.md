# 🍃 GameVault: Integrating MongoDB and Mongoose

## 📌 Overview

GameVault currently stores information temporarily in memory.

At the moment, data is stored using files such as:

```text
backend/data/games.js
backend/data/users.js
```

This means:

```text
Server restarts
-> arrays reset
-> registered users disappear
-> newly added games disappear
```

The next step is to introduce **persistent storage** using MongoDB.

MongoDB will store the data outside the Node.js process, so data can remain available even after the backend stops and restarts.

Mongoose will be used between GameVault and MongoDB.

The new flow will become:

```text
Postman
-> HTTPS server
-> app.js
-> routes
-> middleware
-> controllers
-> Mongoose models
-> MongoDB
-> response
```
---

# 🧠 Part 1: Understand MongoDB and Mongoose

MongoDB is the actual database.

Its structure can be understood as:

```text
Database
-> Collection
-> Document
```

For GameVault, the database might contain:

```text
gamevault
-> users collection
-> games collection
```

A user document may contain:

```text
_id
name
email
passwordHash
role
createdAt
updatedAt
```

A game document may contain:

```text
_id
title
genre
platform
releaseYear
ageRating
available
createdAt
updatedAt
```

MongoDB stores documents rather than rows.

Mongoose is an **Object Data Modelling library** used by Node.js applications to work with MongoDB.

The relationship is:

```text
Controller
-> Mongoose model
-> MongoDB collection
```

A **schema** describes the structure and validation rules for a document.

A **model** gives the application functions for interacting with that collection.

For example:

```text
User model
-> users collection
```

and:

```text
Game model
-> games collection
```

---

# 📦 Part 2: Install Mongoose

Stop the running server:

```text
Ctrl + C
```

Confirm that the terminal is inside:

```text
GameVault/backend
```

Install Mongoose:

```bash
npm install mongoose
```

Confirm that it installed correctly:

```bash
npm list mongoose
```

Mongoose should now appear under the project's dependencies.

You do not need to manually install the MongoDB Node.js driver separately for this project because Mongoose uses it internally.

---

# ☁️ Part 3: Create a MongoDB Atlas Project

Go to MongoDB Atlas and sign in.

Create a project named:

```text
GameVault
```

Inside the project, create a development database deployment.

Use the free development option where appropriate.

During setup, MongoDB Atlas will ask you to create a database user.

This account is used by the GameVault backend to connect to MongoDB.

It is not the same as a GameVault application user.

You therefore have two completely different types of users:

```text
MongoDB database user
-> allows the backend to connect to MongoDB

GameVault user
-> registers and logs into the GameVault application
```

Keep that distinction clear.

---

# 🔐 Part 4: Configure Database Access

Inside MongoDB Atlas, open the database access section.

Create a database user specifically for GameVault.

The user should have permission to read and write the GameVault database.

Record the:

```text
Database username
Database password
```

Do not:

* Hardcode these values in JavaScript.
* Add them to the README.
* Push them to GitHub.
* Include them in screenshots.
* Share them with other groups.

These values will eventually form part of the MongoDB connection string.

---

# 🌍 Part 5: Configure Network Access

MongoDB Atlas also controls which IP addresses are allowed to connect.

Open the network access section.

For local development, add your current IP address.

You may also see an option to allow access from anywhere.

That option is convenient for classroom development, but it is less secure because it allows connection attempts from any IP address.

For production systems, network access should be as restrictive as possible.

For now, configure access according to the development environment being used in class.

---

# 🔗 Part 6: Get the MongoDB Connection String

From your Atlas database deployment:

```text
Connect
-> Drivers
-> Node.js
```

Copy the connection string.

It will resemble:

```text
mongodb+srv://<username>:<password>@<cluster-address>/?retryWrites=true&w=majority
```

Replace:

```text
<username>
<password>
```

with your database credentials.

Add the database name:

```text
gamevault
```

The final URI should conceptually resemble:

```text
mongodb+srv://USERNAME:PASSWORD@CLUSTER/gamevault?...
```

Do not place the real URI directly inside JavaScript.

---

# ⚠️ Part 7: Be Careful with Special Characters

Some characters have special meaning inside URLs.

Examples include:

```text
@
:
/
?
#
%
```

If one of these characters appears inside the database password, the password may need to be URL encoded before it can be used in the connection string.

If the URI fails to connect, check:

```text
Username
-> Password
-> Database name
-> Cluster address
-> Special characters
-> Network access rules
```

Do not weaken a password unnecessarily just to avoid encoding.

---

# 🌍 Part 8: Add the MongoDB URI to `.env`

Open:

```text
backend/.env
```

Add a new environment variable named:

```text
MONGODB_URI
```

The value should contain the real MongoDB connection string.

Your `.env` file should now contain configuration for:

```text
HTTPS
-> application details
-> frontend origin
-> JWT
-> bcrypt
-> MongoDB
```

Do not commit the `.env` file.

---

# 📝 Part 9: Update `.env.example`

Open:

```text
backend/.env.example
```

Add:

```text
MONGODB_URI
```

Use a safe placeholder value rather than the real database credentials.

The purpose of `.env.example` is to tell another developer:

```text
These variables are required
-> create your own .env file
-> insert your own private values
```

---

# 📁 Part 10: Create the Models Folder

Inside:

```text
backend
```

create:

```text
models
```

Using the terminal:

```bash
mkdir models
```

Your structure should now include:

```text
backend
├── config
├── controllers
├── middleware
├── models
├── routes
├── utils
├── app.js
└── server.js
```

The `models` folder will later contain:

```text
User.js
Game.js
```

These model files will replace the current dependency on temporary arrays.

---

# 🗄️ Part 11: Create the Database Connection File

Create:

```text
backend/config/database.js
```

This file should be responsible only for database connection logic.

It should:

* Import Mongoose.
* Read `MONGODB_URI` from the environment.
* Confirm that the URI exists.
* Attempt to connect using Mongoose.
* Wait for the connection to succeed.
* Display a safe success message.
* Allow connection errors to be handled by the server startup process.
* Avoid printing the real connection string.

The file should export one reusable function:

```text
connectDatabase
```

The purpose of this function is:

```text
server.js
-> calls connectDatabase()
-> database.js connects to MongoDB
-> successful connection returns
-> server.js continues starting the backend
```

## Why use a separate database file?

`server.js` already has one clear responsibility:

```text
Start the application
```

Database connection configuration should therefore remain inside:

```text
config/database.js
```

This keeps responsibilities separated.

---

# 🔄 Part 12: Understand Why the Database Connection Is Asynchronous

A database connection is not immediate.

The application must:

```text
Contact MongoDB Atlas
-> establish a network connection
-> authenticate the database user
-> select the deployment
-> establish the database session
```

This takes time.

Therefore, database connection logic must use asynchronous programming.

You will use:

```text
async
await
```

The important idea is:

```text
Call database connection
-> wait
-> connection succeeds
-> continue
```

Without waiting, the application could behave like this:

```text
Start HTTPS server
-> receive API request
-> controller tries to query MongoDB
-> database is not ready
-> request fails
```

The correct order is:

```text
Load configuration
-> connect to MongoDB
-> wait for connection
-> start HTTPS server
```

---

# 🚀 Part 13: Update `server.js`

Open:

```text
backend/server.js
```

Import the new database connection function.

The server startup logic should now be reorganised so that database connection happens before HTTPS begins listening for requests.

Your startup process should:

1. Load environment variables.
2. Import the HTTPS module.
3. Import the configured Express application.
4. Import the HTTPS certificate options.
5. Import `connectDatabase`.
6. Read the port and application name.
7. Create an asynchronous startup function.
8. Call `connectDatabase()` inside that function.
9. Wait for the database connection.
10. Create the HTTPS server only after the connection succeeds.
11. Start listening on the configured port.
12. Handle HTTPS server errors.
13. Handle database-startup failures.
14. Stop the Node.js process if the database connection fails.

The important behaviour should be:

```text
Database fails
-> backend does not start
```

rather than:

```text
Database fails
-> backend still claims it started successfully
```

This prevents the application from accepting requests when one of its required services is unavailable.

---

# 🔄 Part 14: Understand the New Startup Flow

Before MongoDB, starting GameVault followed:

```text
npm run dev
-> server.js
-> load .env
-> load app.js
-> load HTTPS configuration
-> create HTTPS server
-> begin listening
```

After MongoDB integration, the startup sequence becomes:

```text
npm run dev
-> server.js
-> load .env
-> load app.js
-> load HTTPS configuration
-> call connectDatabase()
-> Mongoose attempts MongoDB connection
-> MongoDB authenticates connection
-> connection succeeds
-> create HTTPS server
-> begin listening
```

If MongoDB cannot be reached:

```text
npm run dev
-> connectDatabase()
-> connection fails
-> startup error is handled
-> HTTPS server does not start
```

This ensures GameVault does not run in a partially working state.

---

# 👤 Part 15: Create the User Model

Create:

```text
backend/models/User.js
```

This file will replace the temporary user data structure.

The User schema should define the structure of a user document.

Include the following fields:

```text
name
email
passwordHash
role
```

## Name field

Configure the name field so that it:

* Uses the String type.
* Is required.
* Removes unnecessary surrounding spaces.
* Has a minimum length.
* Has a maximum length.

Keep the length rules aligned with the registration validation middleware already used by GameVault.

## Email field

Configure the email so that it:

* Uses the String type.
* Is required.
* Removes surrounding spaces.
* Is converted to lowercase.
* Must be unique.

The application should still check for existing emails before creating an account.

The database uniqueness rule provides an additional layer of protection.

## Password hash field

The field must store:

```text
passwordHash
```

Do not create a database field named:

```text
password
```

The password-hash field should:

* Use the String type.
* Be required.
* Be excluded from normal database queries by default.

This prevents controllers from accidentally returning password hashes when retrieving user records.

## Role field

The role should:

* Use the String type.
* Allow only recognised GameVault roles.
* Default to the normal user role.

The public registration route should still control role assignment rather than trusting the client.

## Timestamps

Enable automatic Mongoose timestamps.

Mongoose should automatically add:

```text
createdAt
updatedAt
```

This removes the need to manually generate timestamps when registering a user.

## Export the model

At the end of the file:

```text
Create User model
-> associate it with the user schema
-> export the model
```

The rest of the application will then communicate with MongoDB through the User model.

---

# 🔐 Part 16: Understand Why `passwordHash` Should Be Excluded by Default

The password hash is sensitive information.

Most user queries do not need it.

For example:

```text
Display profile
-> needs name
-> needs email
-> needs role
-> does not need password hash
```

Therefore, configure the schema so that the password-hash field is not selected automatically.

The login process is the exception.

Login needs the hash because bcrypt must compare:

```text
Entered password
-> bcrypt comparison
-> stored password hash
```

During login, explicitly request the password-hash field.

This creates a safer default behaviour:

```text
Normal query
-> passwordHash excluded

Login query
-> explicitly request passwordHash
```

---

# 🎮 Part 17: Create the Game Model

Create:

```text
backend/models/Game.js
```

This file will describe how game documents must be stored.

The Game schema should include:

```text
title
genre
platform
releaseYear
ageRating
available
```

## Title

Configure it to:

* Use String.
* Be required.
* Trim spaces.
* Apply minimum and maximum lengths.

Use the same limits already used in your existing GameVault validation middleware.

## Genre

Configure it to:

* Use String.
* Be required.
* Trim spaces.
* Apply appropriate length limits.

## Platform

Configure it to:

* Use String.
* Be required.
* Trim spaces.
* Apply appropriate length limits.

## Release year

Configure it to:

* Use Number.
* Be required.
* Have a minimum permitted year.
* Have an upper limit based on the current year plus the permitted future range.

## Age rating

Configure it to:

* Use String.
* Be required.
* Trim spaces.
* Store the value in uppercase.
* Restrict values to the GameVault-approved age ratings.

## Availability

Configure it to:

* Use Boolean.
* Default to `true`.

## Timestamps

Enable automatic timestamps.

MongoDB game documents should automatically receive:

```text
createdAt
updatedAt
```

## Export the model

Create and export a Game model so the controllers can use it.

The flow becomes:

```text
gameController.js
-> Game model
-> games collection
```

---

# 🛡️ Part 18: Understand the Two Layers of Validation

After Mongoose is introduced, GameVault will validate data in two places.

The first layer is your existing middleware:

```text
Client request
-> validateGame.js
-> controller
```

This protects the API boundary.

The second layer is the Mongoose schema:

```text
Controller
-> Game model
-> schema validation
-> MongoDB
```

This protects the database.

Using both is intentional.

It is an example of:

```text
Defence in depth
```

If invalid data somehow reaches the controller, the database schema still provides another validation layer.

Do not remove the existing request validation simply because Mongoose has schema validation.

---

# 👤 Part 19: Update the Authentication Controller

Open:

```text
backend/controllers/authController.js
```

The controller currently imports:

```text
backend/data/users.js
```

Remove that import.

Import:

```text
User model
```

from:

```text
backend/models/User.js
```

The controller will now communicate with MongoDB through the model instead of manipulating an array.

Before:

```text
authController
-> users array
```

After:

```text
authController
-> User model
-> MongoDB
```

Keep the following existing dependencies:

```text
bcrypt
generateToken
```

---

# 📝 Part 20: Update Registration to Use MongoDB

Your registration logic currently performs operations similar to:

```text
Search users array
-> calculate numeric ID
-> hash password
-> push new user into array
```

That must be replaced.

The new registration flow should be:

```text
Receive validated registration data
-> search MongoDB for matching email
-> reject duplicate email if found
-> hash password using bcrypt
-> create a User document
-> save document to MongoDB
-> generate JWT
-> return safe user information
```

## Find an existing user

Use the User model to search the users collection using the normalised email address.

Because database operations are asynchronous:

```text
register controller
-> must remain async
-> database query must use await
```

If a matching user exists:

```text
Return 409 Conflict
```

## Hash the password

Keep the existing bcrypt behaviour.

Read the bcrypt rounds from:

```text
BCRYPT_ROUNDS
```

Hash the validated password.

Do not store the original password.

## Create the user

Use the User model to create the new database document.

Provide:

```text
name
email
passwordHash
role
```

Do not manually create:

```text
id
createdAt
updatedAt
```

MongoDB and Mongoose now handle these.

MongoDB creates:

```text
_id
```

Mongoose creates the timestamps.

## Generate the token

Pass the newly created user document to:

```text
generateToken
```

Return:

```text
token
safe user details
```

Do not return the password hash.

## Handle duplicate-key errors

Although the controller checks for an existing email first, the database also has a unique email constraint.

Two requests could theoretically attempt to create the same email at almost the same time.

MongoDB may then return a duplicate-key error.

Handle that error and return:

```text
409 Conflict
```

Do not return the raw database error to the client.

---

# 🔑 Part 21: Update Login to Use MongoDB

The old login flow searches the temporary users array.

Remove that logic.

The new flow should be:

```text
Receive validated login details
-> query User model using email
-> explicitly include passwordHash
-> user found?
-> compare entered password using bcrypt
-> generate token
-> return safe user details
```

## Find the user

Query MongoDB by email.

Because `passwordHash` is excluded by default, the login query must explicitly request it.

If no user exists:

```text
401 Unauthorized
```

Use the existing generic message:

```text
Invalid email or password.
```

## Compare passwords

Use bcrypt to compare:

```text
Entered plain-text password
-> stored password hash
```

Do not manually hash the login password and compare strings.

Use bcrypt's comparison function.

If the password does not match:

```text
401 Unauthorized
```

Return the same generic message.

## Generate the token

If authentication succeeds:

```text
MongoDB user
-> generateToken
-> JWT
```

Return safe user information only.

---

# 🎟️ Part 22: Update `generateToken.js` for MongoDB IDs

Open:

```text
backend/utils/generateToken.js
```

Previously, temporary users used numeric IDs.

MongoDB users use:

```text
_id
```

Update the user ID claim so that the token uses the MongoDB document identifier.

Convert the identifier to a string before placing it inside the token.

The JWT payload should still contain only:

```text
userId
email
role
```

The flow becomes:

```text
MongoDB User document
-> _id
-> convert to string
-> userId claim
-> signed JWT
```

Do not place the entire user document inside the JWT.

---

# 👤 Part 23: Update the Profile Controller

The profile controller currently searches the temporary users array using:

```text
req.user.userId
```

Replace that lookup with a User-model query.

The profile flow should become:

```text
GET /auth/profile
-> authenticateToken
-> JWT verified
-> req.user.userId
-> User model
-> MongoDB query by ID
-> user document
-> safe profile response
```

The controller should:

* Remain asynchronous.
* Read the authenticated user ID from `req.user`.
* Query MongoDB using the ID.
* Return `404` if the user no longer exists.
* Return safe profile information only.
* Pass unexpected database errors to the central error handler.

Because `passwordHash` is excluded by default, it should not appear in this query result.

---

# 🎮 Part 24: Update the Game Controller

Open:

```text
backend/controllers/gameController.js
```

Remove the import for:

```text
backend/data/games.js
```

Import the new Game model instead.

Before:

```text
gameController
-> games array
```

After:

```text
gameController
-> Game model
-> MongoDB
```

Every controller function that interacts with the database must now be asynchronous.

Update:

```text
getAllGames
getGameById
createGame
replaceGame
updateGame
deleteGame
```

so they use Mongoose instead of array methods.

---

# 📖 Part 25: Update Get All Games

The current implementation reads directly from the games array.

Replace that with a query through the Game model.

The flow should be:

```text
GET /games
-> gameController
-> Game model
-> retrieve all game documents
-> MongoDB returns documents
-> return count and data
```

The controller should:

* Be asynchronous.
* Query all games.
* Wait for the query result.
* Return `200 OK`.
* Return the number of games.
* Return the documents.
* Pass unexpected database errors to the error handler.

The original array method is no longer required.

---

# 🔍 Part 26: Update Get Game by ID

The biggest change is the identifier format.

Previously, GameVault used:

```text
1
2
3
```

MongoDB uses ObjectIds similar to:

```text
66bf1fc0f47b67d7425ba624
```

The existing numeric-ID validation is therefore no longer suitable.

The new flow should be:

```text
GET /games/:id
-> validate MongoDB ObjectId format
-> invalid format?
-> return 400
-> valid format
-> query Game model by ID
-> document exists?
-> return game
-> document missing?
-> return 404
```

The distinction remains important:

```text
Malformed MongoDB ID
-> 400 Bad Request
```

```text
Valid MongoDB ID but no matching document
-> 404 Not Found
```

Do not attempt to convert MongoDB IDs into numbers.

---

# ➕ Part 27: Update Create Game

The current controller does:

```text
Create object
-> push object into games array
```

That should be removed.

The new flow should be:

```text
POST /games
-> authenticateToken
-> validateGame
-> createGame controller
-> read req.validatedGame
-> Game model creates document
-> schema validation runs
-> MongoDB stores document
-> created document returned
-> 201 Created
```

Do not:

* Generate a numeric ID.
* Push into an array.
* Manually generate timestamps.

MongoDB and Mongoose handle:

```text
_id
createdAt
updatedAt
```

Use the validated request values to create the document.

---

# ✏️ Part 28: Update PUT

If you already implemented:

```text
PUT /games/:id
```

it must now update MongoDB rather than replace an array entry.

The flow should be:

```text
PUT /games/:id
-> authenticateToken
-> validate ObjectId
-> validate complete update body
-> controller
-> find matching MongoDB document
-> apply complete validated update
-> run Mongoose validators
-> return updated document
```

Your existing full-update middleware should still enforce the difference between PUT and PATCH.

For PUT:

```text
All editable fields required
```

The update operation should:

* Return the updated document.
* Run schema validators.
* Return `404` if no document exists.
* Return `400` for invalid IDs.
* Pass unexpected database errors to the error handler.

One important concept:

The Mongoose operation you use may technically perform an update rather than a literal MongoDB document replacement.

For the purpose of GameVault, the **PUT behaviour is enforced by your full-update validation middleware**.

---

# 🩹 Part 29: Update PATCH

PATCH continues to mean:

```text
Update only supplied fields
```

The flow should be:

```text
PATCH /games/:id
-> authenticateToken
-> validate ObjectId
-> validate partial update
-> controller
-> update only supplied values
-> Mongoose validation
-> return updated document
```

Keep the existing rules:

* Empty PATCH bodies are rejected.
* Unexpected fields are rejected.
* Only supplied fields change.
* Missing fields remain unchanged.

Use a Mongoose update operation that:

* Finds the document by ID.
* Applies `req.validatedGame`.
* Returns the updated document.
* Runs schema validation.

Return:

```text
200 OK
```

when successful.

Return:

```text
404 Not Found
```

when the ID format is valid but no matching game exists.

---

# 🗑️ Part 30: Update DELETE

The current delete controller removes an item from an array.

Replace that with a database delete operation.

The flow should be:

```text
DELETE /games/:id
-> authenticateToken
-> validate ObjectId
-> controller
-> find and delete matching game
-> document existed?
-> return deleted game
-> no document?
-> return 404
```

The controller should:

* Be asynchronous.
* Delete by MongoDB ID.
* Return a controlled success response.
* Return `404` if nothing was deleted.
* Pass unexpected database errors to the central error handler.

The deletion is now persistent.

Restarting the server will not restore the deleted game.

---

# 🔢 Part 31: Replace Numeric ID Validation

If you currently have:

```text
backend/middleware/validateGameId.js
```

that checks for positive whole numbers, that validation must change.

The old rule:

```text
ID must be a positive whole number
```

is no longer correct.

The new middleware should validate:

```text
MongoDB ObjectId
```

It should:

* Read `req.params.id`.
* Check whether it has a valid MongoDB ObjectId format.
* Return `400 Bad Request` when invalid.
* Call `next()` when the format is valid.

The middleware should not query the database.

Its only responsibility is:

```text
Does this value have a valid ObjectId format?
```

The controller remains responsible for:

```text
Does a document with this valid ID actually exist?
```

---

# ♻️ Part 32: Keep MongoDB ID Validation Reusable

The same ID validation will be required for:

```text
GET /games/:id
PUT /games/:id
PATCH /games/:id
DELETE /games/:id
```

Therefore, keep it as reusable middleware rather than repeating the same validation inside every controller.

The route flow should resemble:

```text
GET /games/:id
-> validateGameId
-> getGameById
```

```text
PUT /games/:id
-> authenticateToken
-> validateGameId
-> full-update validation
-> replaceGame
```

```text
PATCH /games/:id
-> authenticateToken
-> validateGameId
-> partial-update validation
-> updateGame
```

```text
DELETE /games/:id
-> authenticateToken
-> validateGameId
-> deleteGame
```

This keeps the controllers focused on database operations.

---

# 🧹 Part 33: Remove Temporary Data Files Carefully

Once both models work correctly, the application should no longer depend on:

```text
backend/data/games.js
backend/data/users.js
```

Do not delete these files immediately.

Move one resource at a time.

Recommended sequence:

```text
Migrate users to MongoDB
-> test registration
-> test login
-> restart server
-> confirm user persistence
-> confirm no controller imports users.js
```

Then:

```text
Migrate games to MongoDB
-> test GET
-> test POST
-> test PUT
-> test PATCH
-> test DELETE
-> restart server
-> confirm persistence
-> confirm no controller imports games.js
```

Only after those tests succeed should the temporary data files be removed.

This makes debugging much easier.

---

# 🧪 Part 34: Test the Database Connection

Start GameVault:

```bash
npm run dev
```

The terminal should confirm:

```text
MongoDB connection successful
-> HTTPS server started
```

The exact wording may differ depending on your startup messages.

If MongoDB fails:

```text
Database error
-> server should not start
```

Check:

* `MONGODB_URI`.
* Database username.
* Database password.
* Network access.
* Cluster availability.
* Database name.
* Special characters in the URI.

Never print the full URI while troubleshooting.

---

# 👤 Part 35: Test User Registration

Send:

```text
POST https://localhost:4000/auth/register
```

Register a new user.

Expected result:

```text
201 Created
```

Open MongoDB Atlas.

Locate:

```text
gamevault
-> users
```

Confirm that a document was created.

Check that the document contains:

```text
_id
name
email
passwordHash
role
createdAt
updatedAt
```

Confirm that it does not contain a plain-text password.

---

# 🔁 Part 36: Prove User Persistence

This is an important test.

First:

```text
Register user
-> verify user exists in Atlas
```

Then stop the backend:

```text
Ctrl + C
```

Restart it:

```bash
npm run dev
```

Attempt to log in using the account created before the restart.

Expected result:

```text
Login succeeds
```

This demonstrates the difference between temporary memory and persistent database storage.

Before MongoDB:

```text
Restart
-> user lost
```

After MongoDB:

```text
Restart
-> user still stored
-> login succeeds
```

---

# 🎮 Part 37: Test Game Creation

Log in first and obtain a JWT.

Send:

```text
POST https://localhost:4000/games
```

Include the Bearer token.

Submit valid game data.

Expected result:

```text
201 Created
```

Open MongoDB Atlas.

Locate:

```text
gamevault
-> games
```

Confirm that the game document appears.

It should now have a MongoDB-generated:

```text
_id
```

rather than the old numeric ID.

---

# 🔎 Part 38: Test MongoDB IDs

Copy a real game `_id` from Atlas or the POST response.

Send:

```text
GET https://localhost:4000/games/<id>
```

Expected:

```text
200 OK
```

Then test an invalid value:

```text
GET https://localhost:4000/games/not-an-id
```

Expected:

```text
400 Bad Request
```

Then test a correctly formatted ObjectId that does not correspond to a document.

Expected:

```text
404 Not Found
```

Remember:

```text
Malformed ObjectId
-> 400
```

```text
Valid ObjectId but no resource
-> 404
```

---

# 🧪 Part 39: Test Complete CRUD Persistence

Test the full sequence:

```text
POST
-> create a game

GET
-> confirm the game exists

PATCH
-> change one or more fields

GET
-> confirm the change

PUT
-> perform a complete update

GET
-> confirm the complete update

DELETE
-> remove the game

GET
-> confirm the game is gone
```

Restart the backend.

Confirm that MongoDB still reflects the final state.

This proves that all CRUD operations now use persistent storage.

---

# ⚠️ Part 40: Update the Central Error Handler

MongoDB and Mongoose introduce new categories of errors.

Open:

```text
backend/middleware/errorHandler.js
```

Extend the existing error handling so that database errors are converted into controlled API responses.

Handle at least:

```text
Mongoose validation errors
Duplicate-key errors
Unexpected database errors
```

## Validation errors

When Mongoose rejects data because it violates schema rules:

```text
Mongoose validation error
-> return 400 Bad Request
```

Do not return raw schema internals unless they are intentionally formatted into safe client messages.

## Duplicate-key errors

MongoDB may return a duplicate-key error when a unique field, such as email, is inserted twice.

Handle this as:

```text
409 Conflict
```

## Unexpected errors

Keep your existing general server-error handling.

In production:

```text
Do not expose stack traces
Do not expose MongoDB URI
Do not expose credentials
Do not expose internal file paths
```

---

# 🔄 Part 41: Understand the Final Application Flow

## Registration

```text
POST /auth/register
-> validateRegistration
-> authController
-> check User model
-> MongoDB users collection
-> bcrypt password hashing
-> create user document
-> generateToken
-> JWT response
```

## Login

```text
POST /auth/login
-> validateLogin
-> authController
-> User model
-> MongoDB users collection
-> retrieve password hash
-> bcrypt comparison
-> generateToken
-> JWT response
```

## Profile

```text
GET /auth/profile
-> authenticateToken
-> verify JWT
-> req.user
-> authController
-> User model
-> MongoDB
-> safe profile response
```

## Retrieve games

```text
GET /games
-> gameRoutes
-> gameController
-> Game model
-> MongoDB games collection
-> documents returned
-> JSON response
```

## Create game

```text
POST /games
-> authenticateToken
-> validateGame
-> gameController
-> Game model
-> Mongoose schema validation
-> MongoDB
-> saved document
-> 201 response
```

## Update game

```text
PATCH or PUT /games/:id
-> authenticateToken
-> validate MongoDB ID
-> validate request body
-> gameController
-> Game model
-> MongoDB update
-> updated document
-> response
```

## Delete game

```text
DELETE /games/:id
-> authenticateToken
-> validate MongoDB ID
-> gameController
-> Game model
-> MongoDB delete
-> response
```

---

# ☁️ Part 42: Commit the MongoDB Integration

