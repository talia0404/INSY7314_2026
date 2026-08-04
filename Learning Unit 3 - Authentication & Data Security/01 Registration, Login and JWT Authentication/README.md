# 🔐 GameVault: Registration, Login and JWT Authentication

## 📌 Overview

GameVault currently allows users to access most backend functionality without identifying themselves.

In this section, you will add authentication so that users can:

* Register a GameVault account.
* Log in using an email address and password.
* Receive a JSON Web Token after successful authentication.
* Use the token to access protected endpoints.
* View their own profile.
* Create games only when authenticated.

The backend will continue using temporary in-memory data for now. This means registered users will disappear whenever the server restarts.

The authentication work will follow the same structured backend approach already used in GameVault: routes receive requests, middleware checks requests, controllers perform the main logic, data files store temporary records, and utility files contain reusable logic. 

---

# 🧠 Part 1: Understand Authentication and Authorisation

## Authentication

Authentication answers:

```text
Who is the user?
```

Examples:

* Registering an account.
* Logging in.
* Verifying a token.
* Identifying the current user.

## Authorisation

Authorisation answers:

```text
What is the user allowed to do?
```

Examples:

* Only admins may delete users.
* Only authenticated users may create games.
* Only the owner of a resource may edit it.
* Only certain roles may access restricted routes.

This section mainly focuses on authentication.

Role information will be included in the user record and token so that authorisation can be added later.

---

# 🔄 Part 2: Understand the Authentication Flow

## Registration flow

```text
POST /auth/register
-> JSON body is read
-> Registration data is validated
-> Existing email is checked
-> Password is hashed
-> User is stored
-> JWT is generated
-> Token and safe user information are returned
```

## Login flow

```text
POST /auth/login
-> JSON body is read
-> Login data is validated
-> User is found by email
-> Password is compared with the stored hash
-> JWT is generated
-> Token and safe user information are returned
```

## Protected route flow

```text
Client sends a Bearer token
-> Authentication middleware reads the token
-> JWT signature and expiry are checked
-> User identity is added to the request
-> Protected controller runs
-> Response is returned
```

---

# 🧭 Part 3: Files You Will Add

Create the following files:

```text
backend
├── controllers
│   └── authController.js
├── data
│   └── users.js
├── middleware
│   ├── authenticateToken.js
│   └── validateAuth.js
├── routes
│   └── authRoutes.js
├── utils
│   └── generateToken.js
└── app.js
```

You already created:

```text
backend/utils/generateToken.js
```

You will now connect it to the rest of the authentication system.

---

# 📦 Part 4: Install the Required Packages

Stop the running server:

```text
Ctrl + C
```

Confirm that the terminal is inside:

```text
GameVault/backend
```

Install bcrypt and JSON Web Token support:

```bash
npm install bcrypt jsonwebtoken
```

Confirm that both packages were installed:

```bash
npm list bcrypt jsonwebtoken
```

## Why bcrypt?

bcrypt is used to hash passwords before storage.

A password hash is not the same as encryption.

The original password is not stored and should not be recoverable.

During login, bcrypt compares the entered password with the stored hash.

## Why jsonwebtoken?

The `jsonwebtoken` package is used to:

* Create signed tokens.
* Add token expiry.
* Verify token signatures.
* Read the authenticated user's identity.
* Reject invalid or expired tokens.

---

# 🔑 Part 5: Add Authentication Environment Variables

Open:

```text
backend/.env
```

Add configuration values for:

```text
JWT_SECRET
JWT_EXPIRES_IN
BCRYPT_ROUNDS
```

Recommended values:

```text
JWT_EXPIRES_IN=1h
BCRYPT_ROUNDS=12
```

The JWT secret must be long and unpredictable.

Generate one using:

```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

Copy the generated value into the `JWT_SECRET` environment variable.

Do not:

* Hardcode the JWT secret in JavaScript.
* Upload the real secret to GitHub.
* Use a weak secret such as `password123`.
* Share the secret in screenshots.
* Reuse a secret from another project.

## Update `.env.example`

Open:

```text
backend/.env.example
```

Add the same environment-variable names, but use placeholder values.

The real `.env` file remains private.

The `.env.example` file must be committed so that other developers know which values are required.

---

# 🎟️ Part 6: Understand `generateToken.js`

File:

```text
backend/utils/generateToken.js
```

## Purpose

This file contains reusable JWT generation logic.

Its responsibility is to:

* Import the JWT package.
* Read the JWT secret from the environment.
* Confirm that the secret exists.
* Create a token payload.
* Sign the token.
* Add expiry, issuer and audience information.
* Return the completed token.

## What should go into the token payload?

Only include information required to identify and authorise the user:

```text
userId
email
role
```

Do not include:

```text
password
passwordHash
JWT secret
private key
banking information
sensitive personal information
```

A JWT payload is encoded and signed, but it is not normally encrypted.

This means a client can decode and read the payload.

## Why place this logic in `utils`?

The token-generation process is used after both:

```text
Registration
Login
```

Placing it in a utility file prevents the same logic from being repeated in the authentication controller.

The communication flow is:

```text
authController.js
-> calls generateToken(user)
-> generateToken.js signs the JWT
-> token is returned to authController.js
-> controller sends the token to the client
```

---

# 👤 Part 7: Create Temporary User Storage

Create:

```text
backend/data/users.js
```

## Purpose

This file stores registered users temporarily in memory.

It should contain:

* One shared users array.
* An export of that array.

Do not add sample users with plain-text passwords.

Users should enter the array only through the registration process.

## How it connects

```text
users.js
-> exports the shared users array
-> authController.js imports the array
-> registration adds users
-> login searches users
-> profile searches users
```

Because the data is stored in memory:

```text
Server restarts
-> users array resets
-> registered accounts disappear
```

This is expected for the current stage.

---

# 🛡️ Part 8: Create Authentication Validation Middleware

Create:

```text
backend/middleware/validateAuth.js
```

## Purpose

This file validates authentication request bodies before the controller runs.

It should contain two middleware functions:

```text
validateRegistration
validateLogin
```

---

## Registration validation responsibilities

The registration middleware should:

* Confirm that a request body exists.
* Confirm that the body is not empty.
* Require a name.
* Require an email address.
* Require a password.
* Confirm that these values contain text.
* Trim unnecessary spaces.
* Convert the email address to lowercase.
* Validate the name length.
* Validate the general email format.
* Apply a password policy.
* Assign the default user role.
* Store cleaned values on the request object.
* Call `next()` when validation succeeds.

A suitable password policy should require:

```text
8 to 128 characters
At least one uppercase letter
At least one lowercase letter
At least one number
At least one special character
```

## Role security

Public registration should always create a normal user.

The client should not be allowed to submit:

```text
role: admin
```

The backend should assign:

```text
role: user
```

This prevents users from promoting themselves through the request body.

## Login validation responsibilities

The login middleware should:

* Confirm that the body exists.
* Confirm that the body is not empty.
* Require an email address.
* Require a password.
* Confirm that both contain text.
* Trim the email address.
* Convert the email address to lowercase.
* Validate the general email format.
* Store the cleaned values on the request object.
* Call `next()`.

## How validation communicates with the controller

Registration flow:

```text
Postman sends req.body
-> validateRegistration reads req.body
-> values are checked and cleaned
-> cleaned values are stored in req.validatedRegistration
-> next() is called
-> register controller reads req.validatedRegistration
```

Login flow:

```text
Postman sends req.body
-> validateLogin reads req.body
-> values are checked and cleaned
-> cleaned values are stored in req.validatedLogin
-> next() is called
-> login controller reads req.validatedLogin
```

When validation fails:

```text
Validation middleware
-> returns 400 response
-> does not call next()
-> controller does not run
```

---

# 🎛️ Part 9: Create the Authentication Controller

Create:

```text
backend/controllers/authController.js
```

## Purpose

This file performs the main registration, login and profile logic.

It should import:

```text
bcrypt
users.js
generateToken.js
```

It should contain three controller functions:

```text
register
login
getProfile
```

---

## Register controller responsibilities

The registration controller should:

1. Read cleaned data from `req.validatedRegistration`.
2. Search the users array for the same email address.
3. Reject duplicate emails.
4. Read the bcrypt rounds from the environment.
5. Hash the password asynchronously.
6. Generate a user ID.
7. Create the user record.
8. Store the password hash instead of the password.
9. Add the user to the users array.
10. Generate a JWT.
11. Return the token.
12. Return only safe user information.
13. Pass unexpected errors to the central error handler.

A duplicate email should return:

```text
409 Conflict
```

The stored user record should contain fields such as:

```text
id
name
email
passwordHash
role
createdAt
```

It must not contain:

```text
password
```

The response must not contain:

```text
password
passwordHash
JWT secret
```

---

## Login controller responsibilities

The login controller should:

1. Read cleaned data from `req.validatedLogin`.
2. Search for the user by email.
3. Return a generic error if the user does not exist.
4. Compare the entered password with the stored hash.
5. Return the same generic error if the password is incorrect.
6. Generate a new token after successful login.
7. Return the token and safe user information.
8. Pass unexpected errors to the central error handler.

Use one error message for both cases:

```text
Invalid email or password.
```

This avoids revealing whether a particular email address is registered.

An invalid login should return:

```text
401 Unauthorised
```

---

## Profile controller responsibilities

The profile controller should:

1. Read the authenticated user ID from `req.user`.
2. Find the corresponding user in the users array.
3. Return `404` if the user no longer exists.
4. Return only safe profile information.

This controller must not run unless token verification succeeds.

---

# 🪪 Part 10: Create JWT Verification Middleware

Create:

```text
backend/middleware/authenticateToken.js
```

## Purpose

This middleware protects routes that require authentication.

Its responsibility is to:

* Read the `Authorisation` request header.
* Confirm that the header exists.
* Confirm that the Bearer format is used.
* Extract the token.
* Read the JWT secret.
* Verify the token.
* Verify the expected algorithm.
* Verify the issuer.
* Verify the audience.
* Verify the expiry.
* Store the decoded token on `req.user`.
* Call `next()` when verification succeeds.
* Return a controlled error when verification fails.

## Expected header format

```text
Authorisation: Bearer token-value
```

The header contains:

```text
Bearer
One space
JWT value
```

## Token verification flow

```text
Protected request
-> authenticateToken reads Authorisation header
-> header is split into scheme and token
-> token is verified
-> decoded payload is stored in req.user
-> next() is called
-> protected controller runs
```

## Required error cases

No token:

```text
401 Unauthorised
Authentication token is required.
```

Incorrect Bearer format:

```text
401 Unauthorised
Authorisation header must use the Bearer token format.
```

Expired token:

```text
401 Unauthorised
Authentication token has expired.
```

Invalid token:

```text
401 Unauthorised
Authentication token is invalid.
```

Missing JWT configuration:

```text
500 Internal Server Error
```

Do not return detailed token-library errors to the client.

---

# 🛣️ Part 11: Create Authentication Routes

Create:

```text
backend/routes/authRoutes.js
```

## Purpose

This file connects authentication URLs to middleware and controllers.

It should import:

```text
Express
register
login
getProfile
validateRegistration
validateLogin
authenticateToken
```

It should define:

```text
POST /register
POST /login
GET /profile
```

Because the router will later be mounted under `/auth`, the complete endpoints will be:

```text
POST /auth/register
POST /auth/login
GET /auth/profile
```

## Registration route flow

```text
POST /auth/register
-> validateRegistration
-> register
-> response
```

## Login route flow

```text
POST /auth/login
-> validateLogin
-> login
-> response
```

## Profile route flow

```text
GET /auth/profile
-> authenticateToken
-> getProfile
-> response
```

The profile route must use authentication middleware before the controller.

---

# ⚙️ Part 12: Register Authentication Routes in `app.js`

Open:

```text
backend/app.js
```

Import:

```text
authRoutes
```

Register it under:

```text
/auth
```

The route registration order should resemble:

```text
Security middleware
-> JSON request parser
-> System routes
-> Authentication routes
-> Game routes
-> Not-found middleware
-> Error handler
```

This creates:

```text
POST /auth/register
POST /auth/login
GET /auth/profile
```

The authentication routes must be registered before the not-found middleware.

Otherwise, every authentication request will return `404`.

---

# 🔒 Part 13: Protect Game Creation

Open:

```text
backend/routes/gameRoutes.js
```

Import:

```text
authenticateToken
```

Update the POST game route so that authentication runs before game validation.

The flow should be:

```text
POST /games
-> authenticateToken
-> validateGame
-> createGame
-> response
```

## Why authenticate before validating the game?

An unauthenticated user is not allowed to create a game.

There is no reason to spend time validating the game body if the requester has not authenticated.

When the token is missing or invalid:

```text
POST /games
-> authenticateToken
-> 401 response
```

The game-validation middleware and controller do not run.

---

# 🔄 Part 14: Complete Application Flow

## Registration

```text
Postman
-> HTTPS server
-> app.js
-> express.json()
-> authRoutes.js
-> validateRegistration
-> authController.register
-> users.js
-> bcrypt.hash()
-> generateToken.js
-> JWT returned
```

## Login

```text
Postman
-> HTTPS server
-> app.js
-> express.json()
-> authRoutes.js
-> validateLogin
-> authController.login
-> users.js
-> bcrypt.compare()
-> generateToken.js
-> JWT returned
```

## Protected profile request

```text
Postman sends Bearer token
-> HTTPS server
-> app.js
-> authRoutes.js
-> authenticateToken
-> jwt.verify()
-> req.user is created
-> authController.getProfile
-> users.js
-> safe profile response
```

## Protected game creation

```text
Postman sends Bearer token and game JSON
-> HTTPS server
-> app.js
-> gameRoutes.js
-> authenticateToken
-> validateGame
-> gameController.createGame
-> games.js
-> 201 response
```

---

# 🧪 Part 15: Test Registration in Postman

Create:

```text
POST https://localhost:4000/auth/register
```

Use a JSON body containing:

```text
name
email
password
```

Use a password that satisfies the configured policy.

Expected result:

```text
201 Created
```

The response should contain:

```text
success
message
token
safe user object
```

Confirm that the response does not contain:

```text
password
passwordHash
JWT_SECRET
```

---

# 🧪 Part 16: Test Duplicate Registration

Send the same registration request again.

Expected result:

```text
409 Conflict
```

This confirms that the application prevents duplicate email addresses.

---

# 🧪 Part 17: Test Invalid Registration

Test the following cases:

* Missing name.
* Missing email.
* Missing password.
* Invalid email format.
* Weak password.
* Password missing an uppercase letter.
* Password missing a number.
* Password missing a special character.
* Empty request body.
* Values using incorrect data types.

Expected result:

```text
400 Bad Request
```

---

# 🧪 Part 18: Test Login

Create:

```text
POST https://localhost:4000/auth/login
```

Use the email address and password used during registration.

Expected result:

```text
200 OK
```

Copy the token from the response.

The token will be required for protected requests.

---

# 🧪 Part 19: Test Invalid Login

Test:

* Incorrect password.
* Unknown email.
* Invalid email format.
* Missing password.
* Empty request body.

Incorrect credentials should return:

```text
401 Unauthorised
```

The response should not reveal whether the email or password was incorrect.

---

# 👤 Part 20: Test the Protected Profile Route

Create:

```text
GET https://localhost:4000/auth/profile
```

In Postman:

1. Open the Authorisation tab.
2. Select Bearer Token.
3. Paste the token.
4. Send the request.

Expected result:

```text
200 OK
```

Remove the token and send the request again.

Expected result:

```text
401 Unauthorised
```

---

# 🎮 Part 21: Test Protected Game Creation

Create:

```text
POST https://localhost:4000/games
```

First send the request without a token.

Expected result:

```text
401 Unauthorised
```

Then add the token using:

```text
Authorisation -> Bearer Token
```

Send the same game request again.

Expected result:

```text
201 Created
```

This proves that:

```text
Valid request body alone is not enough
-> authenticated identity is also required
```

---

# 🕒 Part 22: Test Token Expiry

Temporarily change:

```text
JWT_EXPIRES_IN
```

to a short value such as:

```text
30s
```

Restart the server.

Register or log in again to receive a new token.

Immediately test the protected profile route.

Then wait longer than the configured expiry and test again.

Expected result after expiry:

```text
401 Unauthorised
Authentication token has expired.
```

After testing, restore:

```text
JWT_EXPIRES_IN=1h
```

Restart the server again.

---

# ⚠️ Part 23: Current Limitations

This authentication system is suitable for learning, but it is not yet production-ready.

Current limitations include:

* Users are stored in memory.
* Users disappear when the server restarts.
* Email ownership is not verified.
* Password reset is not implemented.
* Refresh tokens are not implemented.
* Tokens cannot be revoked before expiry.
* Account lockout is not implemented.
* Login rate limiting is not yet configured.
* Audit logging is not implemented.
* User data is not stored in a database.

These limitations will be addressed as the application develops.

---

# 🛠️ Part 24: Troubleshooting

## bcrypt cannot be found

Run:

```bash
npm install bcrypt
```

## jsonwebtoken cannot be found

Run:

```bash
npm install jsonwebtoken
```

## JWT secret error

Confirm that `.env` contains:

```text
JWT_SECRET
```

Restart the backend after changing environment variables.

## Login always fails

The server may have restarted.

Because users are stored in memory, register the user again before logging in.

## Profile always returns 401

Check that the header uses:

```text
Authorisation: Bearer token-value
```

There must be one space between `Bearer` and the token.

## `req.validatedRegistration` is undefined

Check the registration route order:

```text
validateRegistration
-> register
```

## `req.validatedLogin` is undefined

Check the login route order:

```text
validateLogin
-> login
```

## `req.user` is undefined

Check that:

```text
authenticateToken
```

runs before the protected controller.

## Existing tokens stopped working

The JWT secret may have changed.

Tokens signed using the old secret will no longer verify.

Log in again to receive a new token.

---

# ✅ Part 25: Completion Checklist

Confirm that:

* bcrypt is installed.
* jsonwebtoken is installed.
* JWT configuration exists in `.env`.
* `.env.example` has been updated.
* A strong JWT secret is used.
* `users.js` stores one shared temporary array.
* `generateToken.js` creates signed tokens.
* Registration data is validated.
* Login data is validated.
* Email addresses are normalised.
* Public registration assigns the normal user role.
* Duplicate emails are rejected.
* Passwords are hashed asynchronously.
* Plain-text passwords are never stored.
* Password hashes are never returned.
* Login uses bcrypt comparison.
* Registration returns a token.
* Login returns a token.
* JWT payloads contain no sensitive information.
* JWT verification checks expiry, issuer, audience and algorithm.
* `req.user` stores the verified identity.
* The profile route is protected.
* Game creation is protected.
* Invalid tokens are rejected.
* Expired tokens are rejected.
* Existing HTTPS, Helmet, CSP and CORS configuration still works.

---

# ☁️ Part 26: Commit the Authentication Work

