# ⚙️ 1. Configure `server.js`

Inside the `backend` folder, `server.js` will act as the entry point for your backend application.

Configure the file so that it:

* Imports the required packages.
* Creates the Express application.
* Loads environment variables.
* Enables JSON request-body processing.
* Starts the server.
* Contains all API routes for Learning Unit 1.

As the project grows throughout the semester, additional functionality will be moved into separate folders such as routes, controllers, middleware, and models.

---

# 🌍 2. Load Environment Variables

Configure your application to load values from the `.env` file.

The application should retrieve configuration values such as:

* Application port.
* Application name.
* Environment.

Where appropriate, provide sensible fallback values if an environment variable is not available.

Environment variables should be used for configuration instead of hardcoding values directly into the application.

---

# 🚀 3. Create the Express Application

Create the Express application that will serve as the backend for GameVault.

The application should:

* Initialise Express.
* Prepare the application for future routes.
* Be used throughout the semester as the main backend application.

---

# 📥 4. Enable JSON Request-Body Processing

Configure Express so that it can receive JSON data from clients.

This allows the API to accept request bodies sent from applications such as:

* Postman
* React
* Other APIs

Without this configuration, the server will not be able to correctly process JSON request bodies.

---

# 🖥️ 5. Start the Server

Configure the application so that it listens on the port defined in the environment variables.

Start the server using the appropriate npm command.

Once the server is running:

* Confirm that the terminal displays a successful startup message.
* Open the application in your browser using the correct localhost address.
* Verify that the server is running successfully.

If changes are made while using nodemon, the server should restart automatically.

---

# 🏠 6. Create the Root Route

Create a root endpoint for the application.

This route should:

* Respond to a GET request.
* Confirm that the GameVault API is running.
* Return a friendly welcome message or application information.
* Return an appropriate HTTP status code.

Test the route using both your browser and Postman.

---

# ℹ️ 7. Create the About Route

Create an endpoint that provides information about the GameVault application.

This route should briefly explain:

* What GameVault is.
* The purpose of the application.
* The current development stage.

Return the information as a JSON response.

---

# ❤️ 8. Create the Health-Check Route

Create a health-check endpoint that allows developers to verify that the application is running correctly.

The response should include useful information such as:

* Application status.
* Application name.
* Current date and time.

This endpoint is commonly used in production environments for monitoring and automated deployments.

Test the endpoint in both your browser and Postman.

---

# 🎮 9. Create the In-Memory Games Collection

Create an in-memory collection that stores sample game data.

Include at least three different games.

Each game should contain information such as:

* Unique ID.
* Title.
* Genre.
* Platform.
* Release year.
* Age rating.
* Availability status.

Remember that this data is temporary and will be lost whenever the server restarts.

A database will be introduced later in the semester.

---

# 🔍 10. Create the Game Endpoints

Create API endpoints that allow clients to interact with the games collection.

Your API should support:

* Retrieving all games.
* Retrieving a single game using its ID.
* Adding a new game.

Use appropriate HTTP methods and return suitable HTTP status codes.

Test each endpoint thoroughly before moving on.

---

# 🛡️ 11. Validate External Input

Remember that all information received from clients should be treated as untrusted.

Before creating a new game, validate the incoming data.

Ensure that:

* Required fields are present.
* Text fields contain valid values.
* Numeric fields contain valid numbers.
* Invalid values are rejected.
* Appropriate error messages are returned.
* The correct HTTP status code is used.

The API should never accept incomplete or invalid data.

---

# 🧪 12. Test the API in Postman

Create a Postman collection named:

```text
GameVault LU1
```

Include requests that test:

* Root endpoint.
* About endpoint.
* Health-check endpoint.
* Retrieve all games.
* Retrieve a game by ID.
* Add a valid game.
* Add an invalid game.
* Invalid routes.

For every request, verify:

* Request method.
* URL.
* Request body (where applicable).
* Response body.
* HTTP status code.

Do not test only successful scenarios. Ensure that invalid requests are handled correctly.

---

# ☁️ 13. Push the Project to GitHub

Once Learning Unit 1 has been completed:

* Verify that your project builds and runs successfully.
* Confirm that `.gitignore` is correctly configured.
* Ensure that `node_modules` and `.env` are not being tracked.
* Commit your completed work using a clear and meaningful commit message.
* Push the latest version of your project to your GitHub repository.

Your repository should include all source files required to run the project, excluding any files that should remain private or can be regenerated automatically.
