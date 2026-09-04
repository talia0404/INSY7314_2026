# 🚦 Rate Limiting 

Rate limiting is another security layer for the GameVault API.

It controls:

```text
how many requests a client can make
-> within a particular period
```

<img width="521" height="326" alt="image" src="https://github.com/user-attachments/assets/9985655f-5e16-4c83-a2cb-cacbdba3faa6" />


For example, without rate limiting, someone could repeatedly send thousands of requests to:

```text
POST /auth/login
```

This could be used for:

```text
brute-force login attempts
automated abuse
excessive API requests
unnecessary server load
```

For GameVault, we will implement **two rate limits**:

```text
general API rate limit
-> protects the entire API

authentication rate limit
-> stricter limit
-> protects registration and login
```

---

# 🧠 Step 1 — Understand what rate limiting does

Consider a client repeatedly sending:

```text
POST /auth/login
POST /auth/login
POST /auth/login
POST /auth/login
POST /auth/login
...
```

Without rate limiting:

```text
request
-> Express
-> login controller
-> MongoDB
-> bcrypt
-> response

request
-> Express
-> login controller
-> MongoDB
-> bcrypt
-> response

request
-> Express
-> login controller
-> MongoDB
-> bcrypt
-> response
```

The server continues processing the requests.

Rate limiting places a checkpoint before the expensive application logic:

```text
request
-> rate limiter
-> has the client exceeded the limit?
-> no
-> continue

request
-> rate limiter
-> has the client exceeded the limit?
-> yes
-> reject request
```

The rejected request should receive:

```text
429 Too Many Requests
```

---

# 📦 Step 2 — Install the rate-limiting package

GameVault does not currently have dedicated rate-limiting middleware.

We will use:

```text
express-rate-limit
```

Install it from the `backend` directory using npm.

The command will follow the normal pattern:

```bash
npm install PACKAGE-NAME
```

Replace `PACKAGE-NAME` with:

```text
express-rate-limit
```

After installation, check:

```text
backend/package.json
```

You should see `express-rate-limit` listed under the project's dependencies.

Do not manually type a version number into `package.json`. Let npm install the appropriate version and update the dependency files.

---

# 📁 Step 3 — Create a rate limiter middleware file

Inside:

```text
backend/middleware
```

create:

```text
rateLimiters.js
```

Your middleware folder should now include:

```text
middleware
-> authenticateToken.js
-> authoriseRoles.js
-> errorHandler.js
-> notFound.js
-> rateLimiters.js
-> validateAuth.js
-> validateGame.js
-> validateGameId.js
```

We are using one file called `rateLimiters.js` because we are going to create more than one limiter.

---

# 📥 Step 4 — Import `rateLimit` from the package

At the top of `rateLimiters.js`, import the `rateLimit` function from:

```text
express-rate-limit
```

You can use destructuring:

```javascript
const {
    rateLimit
} = require("express-rate-limit");
```

The `rateLimit` function allows us to configure things such as:

```text
how long the counting period lasts
how many requests are allowed
what response is returned when the limit is exceeded
what rate-limit information is sent in the response headers
```

---

# 🌐 Step 5 — Create a general API limiter

The first limiter should protect the **entire GameVault API**.

Give it a meaningful name such as:

```text
apiLimiter
```

Create it by calling:

```javascript
rateLimit({
    // configuration goes here
});
```

This limiter should eventually be applied globally in:

```text
app.js
```

The idea is:

```text
any incoming API request
-> apiLimiter
-> route
```

---

# ⏱️ Step 6 — Configure the general time window

The limiter needs to know the period during which requests should be counted.

For GameVault, use:

```text
15 minutes
```

The configuration property for this is:

```javascript
windowMs
```

It expects milliseconds.

You therefore need to convert:

```text
15 minutes
-> 15 × 60 seconds
-> 15 × 60 × 1000 milliseconds
```

This can be written directly as the calculation rather than manually calculating the final number.

The benefit is readability:

```text
15 * 60 * 1000
```

clearly represents:

```text
15 minutes
```

---

# 🔢 Step 7 — Configure the general request limit

Next, decide how many requests a client can make during that window.

For this exercise, use:

```text
100 requests
-> every 15 minutes
```

The appropriate configuration property is:

```javascript
limit
```

The general idea is therefore:

```text
windowMs
-> 15 minutes

limit
-> 100 requests
```

This means:

```text
client makes request 1
-> allowed

client makes request 2
-> allowed

...

client makes request 100
-> allowed

client exceeds the configured limit
-> 429 Too Many Requests
```

---

# 📋 Step 8 — Configure rate-limit response headers

Rate limiting can provide information about the limit through HTTP response headers.

Configure:

```javascript
standardHeaders
```

to use the modern standard headers supported by the package.

For our implementation, use:

```text
draft-8
```

Also disable the older legacy headers using:

```javascript
legacyHeaders
```

with a Boolean value of:

```text
false
```

Conceptually:

```text
standardHeaders
-> enabled

legacy headers
-> disabled
```

You do not need to manually create these headers. The package handles them.

---

# 🚫 Step 9 — Create the 429 response

You should control what GameVault returns when a client exceeds the limit.

The HTTP status code must be:

```text
429
```

which means:

```text
Too Many Requests
```

Use the limiter's:

```javascript
handler
```

configuration.

The handler should receive:

```text
req
res
```

and return a JSON response.

Keep it consistent with the rest of GameVault:

```text
success
-> false

error
-> meaningful message explaining that too many requests were made
```

For example, your message could tell the client to try again later.

---

# 🔐 Step 10 — Create a separate authentication limiter

The general API limiter is not enough for sensitive authentication endpoints.

Create a second limiter in the same file.

Give it a name such as:

```text
authLimiter
```

This limiter will specifically protect:

```text
POST /auth/register
POST /auth/login
```

Why should these have a stricter limit?

Because authentication endpoints process credentials.

An attacker could repeatedly attempt:

```text
email + password
-> login fails

different password
-> login fails

different password
-> login fails

different password
-> login fails
```

Rate limiting makes this type of automated attack more difficult.

---

# ⏱️ Step 11 — Configure the authentication time window

Use the same:

```text
15-minute window
```

for the authentication limiter.

Again, configure this using:

```javascript
windowMs
```

and express the value as a calculation converting minutes to milliseconds.

---

# 🔢 Step 12 — Give authentication a lower request limit

The authentication limiter should be stricter than the general API limiter.

For the teaching implementation, use:

```text
10 authentication requests
-> every 15 minutes
```

This gives us:

```text
general API
-> 100 requests per 15 minutes

authentication
-> 10 requests per 15 minutes
```

The difference is intentional.

Normal API use might generate many requests, whereas repeated authentication attempts deserve tighter restrictions.

---

# 🔐 Step 13 — Decide whether successful authentication requests count

The package provides an option called:

```javascript
skipSuccessfulRequests
```

For this implementation, keep it:

```text
false
```

That means successful requests are still counted.

So:

```text
successful login
-> counted

failed login
-> counted
```

---

# 🚨 Step 14 — Give authentication its own 429 response

The authentication limiter should also return:

```text
429 Too Many Requests
```

when the limit is exceeded.

However, use a message specifically related to authentication attempts.

For example, communicate that:

```text
too many authentication attempts were made
-> try again later
```

Again, keep your JSON response consistent:

```text
success: false
error: ...
```

---

# 📤 Step 15 — Export both limiters

At the bottom of:

```text
rateLimiters.js
```

export:

```text
apiLimiter
authLimiter
```

Because there are two values being exported from one file, you can export them together as properties of an object.

Your other files can then destructure whichever limiter they need.

Conceptually:

```text
rateLimiters.js
-> apiLimiter
-> authLimiter
```

---

# 🌐 Step 16 — Import the general limiter into `app.js`

Open:

```text
backend/app.js
```

Import:

```text
apiLimiter
```

from:

```text
./middleware/rateLimiters
```

Remember that `rateLimiters.js` exports multiple values, so your import should match that export structure.

This is another place where you should be careful about the import/export mismatch we encountered earlier.

After importing it, the variable should represent a middleware function.

---

# 🧱 Step 17 — Apply the general limiter before the routes

The general limiter should execute before requests reach your application routes.

Your `app.js` currently has security and parsing middleware such as:

```text
Helmet
CORS
express.json()
```

After those, apply:

```text
apiLimiter
```

before mounting the application's routers.

The broad flow should become:

```text
incoming request
-> Helmet
-> CORS
-> express.json()
-> apiLimiter
-> system routes / auth routes / game routes
-> controller
-> response
```

This means requests to all of these are covered:

```text
GET /health
GET /games
GET /games/:id
POST /games
POST /auth/login
POST /auth/register
...
```

---

# 🔍 Step 18 — Understand what the general limiter does when a request arrives

When a request reaches:

```text
apiLimiter
```

the middleware determines whether that client has exceeded the configured request allowance.

If the client is below the limit:

```text
request
-> apiLimiter
-> limit not exceeded
-> next()
-> appropriate route
```

If the limit has been exceeded:

```text
request
-> apiLimiter
-> limit exceeded
-> 429 response
-> request stops
```

The controller is therefore never reached for a blocked request.

---

# 🔑 Step 19 — Import `authLimiter` into `authRoutes.js`

Now open:

```text
backend/routes/authRoutes.js
```

Import:

```text
authLimiter
```

from:

```text
../middleware/rateLimiters
```

You do **not** need to import `apiLimiter` here.

Why?

Because:

```text
apiLimiter
-> already applied globally in app.js
```

while:

```text
authLimiter
-> only needed on selected authentication routes
```

---

# 🛣️ Step 20 — Add the limiter to the registration route

Your registration route currently has middleware responsible for validating registration data before calling the registration controller.

Insert:

```text
authLimiter
```

before that validation middleware.

The flow should become:

```text
POST /auth/register
-> authLimiter
-> validateRegistration
-> register
```

Remember that the request will actually have already passed through the global limiter in `app.js`.

So the complete flow is:

```text
POST /auth/register
-> apiLimiter
-> authRoutes
-> authLimiter
-> validateRegistration
-> register
-> MongoDB
-> response
```

---

# 🔑 Step 21 — Add the limiter to the login route

Do the same for:

```text
POST /auth/login
```

Place:

```text
authLimiter
```

before the login validation middleware.

The route-level flow becomes:

```text
POST /auth/login
-> authLimiter
-> validateLogin
-> login
```

The complete application flow becomes:

```text
POST /auth/login
-> app.js
-> apiLimiter
-> authRoutes.js
-> authLimiter
-> validateLogin
-> login controller
-> User model
-> MongoDB
-> bcrypt
-> generateToken
-> response
```

---

# 👤 Step 22 — Do not put the authentication limiter on `/profile`

Your route:

```text
GET /auth/profile
```

does not represent a login or registration attempt.

It should still have:

```text
authenticateToken
```

because the user must be logged in.

But it does not need:

```text
authLimiter
```

The general API limiter already protects it.

So:

```text
GET /auth/profile
-> apiLimiter
-> authenticateToken
-> getProfile
```

rather than:

```text
GET /auth/profile
-> authLimiter
```

This demonstrates an important principle:

```text
different routes
-> different risks
-> different security controls
```

---

# 🧠 Step 23 — Understand the two layers of rate limiting

Login requests now pass through **two** rate limiters.

This is intentional.

A login request follows:

```text
POST /auth/login
-> apiLimiter
-> authLimiter
-> login processing
```

The general limiter asks:

```text
has this client made too many requests to GameVault overall?
```

The authentication limiter asks:

```text
has this client made too many authentication requests?
```

The two limiters solve related but slightly different problems.

---

# 🧪 Step 24 — Temporarily lower the authentication limit for testing

Waiting until you have sent more than 10 requests is inconvenient during a classroom demonstration.

Temporarily change the authentication limit from:

```text
10
```

to:

```text
3
```

Do not change the 15-minute window.

You now have:

```text
authLimiter
-> 3 requests
-> 15 minutes
```

Restart the backend if necessary.

---

# 🧪 Step 25 — Send repeated login requests

Using Postman, send:

```text
POST /auth/login
```

multiple times from the same client.

For example:

```text
request 1
-> allowed

request 2
-> allowed

request 3
-> allowed

next request beyond the configured allowance
-> blocked
```

The blocked request should return:

```text
429 Too Many Requests
```

and your custom JSON error response.

---

# 🔎 Step 26 — Inspect the response headers

In Postman, inspect the response headers.

Because you enabled standard rate-limit headers, you should see rate-limiting information provided by the middleware.

Use this to explain that rate limiting is not only something happening invisibly inside the server.

The HTTP response can communicate rate-limit information back to the client.

---

# 🔄 Step 27 — Understand why restarting the server may reset your test

For this simple classroom implementation, the rate limiter uses its standard in-memory storage unless you explicitly configure another store.

That means the counters belong to the running backend process.

During development:

```text
server running
-> rate-limit counters exist in memory

server restarted
-> in-memory counters reset
```

It also means this simple configuration is appropriate for your current learning exercise, but a larger production system running multiple server instances would normally require a shared rate-limit store.

---

# 🔧 Step 28 — Restore the authentication limit after testing

Once you have demonstrated the `429` response, change:

```text
3 requests
```

back to:

```text
10 requests
```

The low value is useful for testing but would become irritating during normal development.

---

# 🧪 Step 29 — Confirm ordinary API requests still work

Test:

```text
GET /health
GET /games
GET /games/:id
```

They should still work normally while the general API limit has not been exceeded.

Then test authenticated functionality:

```text
POST /auth/login
GET /auth/profile
POST /games
PATCH /games/:id
```

The addition of rate limiting should not replace your existing security controls.

Instead, they work together.

---

# 🔐 Step 30 — Understand how rate limiting works with JWT and RBAC

After implementing everything, an admin creating a game goes through several independent security controls:

```text
POST /games
-> Helmet
-> CORS
-> apiLimiter
-> gameRoutes
-> authenticateToken
-> authoriseRoles("admin")
-> validateCreateGame
-> createGame
-> Game model
-> MongoDB
-> response
```

Each layer asks a different question.

```text
apiLimiter
-> has this client made too many requests?

authenticateToken
-> is this user authenticated?

authoriseRoles
-> does this authenticated user have permission?

validateCreateGame
-> is the supplied game data valid?

controller
-> perform the requested operation
```

---

# 🧪 Step 31 — Test the important rate-limiting scenarios

Before considering the implementation complete, test these scenarios:

```text
normal GET request
-> allowed

normal login request
-> allowed

repeated login requests below limit
-> allowed

login requests exceeding auth limit
-> 429

normal authenticated request
-> allowed

normal admin request
-> allowed if general limit has not been exceeded
```

Also confirm that the existing authentication responses still work when the rate limit has not been exceeded:

```text
missing token
-> 401

invalid token
-> 401

normal user attempting admin action
-> 403

too many requests
-> 429
```

---

# 🚨 Step 32 — Understand why rate limiting does not replace authentication

A common misconception is that rate limiting makes authentication safer enough on its own.

It does not.

Rate limiting cannot determine whether someone knows the correct password.

It simply restricts how frequently requests can be made.

Therefore:

```text
rate limiting
-> controls request frequency

bcrypt
-> protects stored passwords

jwt
-> carries authenticated identity information

authenticateToken
-> verifies authentication

rbac
-> controls permissions
```

These controls work together rather than replacing one another.

---

# 🏁 Final Rate-Limiting Structure

Your implementation should ultimately look conceptually like this:

```text
rateLimiters.js
-> apiLimiter
   -> 15-minute window
   -> 100-request limit
   -> standard headers
   -> 429 response

-> authLimiter
   -> 15-minute window
   -> 10-request limit
   -> standard headers
   -> 429 response
```

Then:

```text
app.js
-> Helmet
-> CORS
-> express.json()
-> apiLimiter
-> routes
```

And:

```text
authRoutes.js

POST /register
-> authLimiter
-> validateRegistration
-> register

POST /login
-> authLimiter
-> validateLogin
-> login

GET /profile
-> authenticateToken
-> getProfile
```

Because `apiLimiter` is already applied in `app.js`, the actual login flow is:

```text
POST /auth/login
-> apiLimiter
-> authLimiter
-> validateLogin
-> login controller
-> MongoDB
-> bcrypt
-> JWT
-> response
```
