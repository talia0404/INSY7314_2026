# 🔐 Role-Based Access Control (RBAC) 

Role-based access control is the next security layer after authentication.

Your authentication middleware already answers:

```text
who is this user?
```

RBAC answers:

```text
what is this user allowed to do?
```

For GameVault, the simplest setup is to use two roles:

```text
user
admin
```

A normal user can access ordinary authenticated features, while an admin can perform more sensitive actions such as creating, updating, and deleting games.

---

# 🧠 Step 1 — Understand the difference between authentication and authorisation

Before implementing RBAC, make sure the distinction is clear.

Authentication checks whether the user is logged in.

For example:

```text
request
-> authenticateToken
-> jwt is verified
-> decoded user details are stored in req.user
```

Authorisation checks whether that authenticated user has permission to perform the requested action.

For example:

```text
request
-> authenticateToken
-> user is authenticated
-> authoriseRoles
-> user's role is checked
-> request continues or is denied
```

A user can therefore be:

```text
authenticated
but
not authorised
```

For example, a normal user may be successfully logged in but still not be allowed to delete a game.

---

# 👤 Step 2 — Confirm that users have a role

Your `User` model should already include a role field.

The role should normally support values such as:

```text
user
admin
```

The default should be:

```text
user
```

This is important because newly registered users should not automatically receive administrator permissions.

The database should therefore contain users similar to:

```text
name: GameVault User
email: user@gamevault.com
role: user
```

and:

```text
name: GameVault Admin
email: admin@gamevault.com
role: admin
```

---

# 🔑 Step 3 — Confirm that the JWT contains the user's role

Your JWT must contain the user's role.

When the token is generated after registration or login, the payload should contain information such as:

```text
userId
email
role
```

A simplified payload might resemble:

```javascript
{
    userId: user._id,
    email: user.email,
    role: user.role
}
```

> Why? Because the authorisation middleware needs access to the role later.

The flow is:

```text
user logs in
-> database returns user
-> token is generated
-> role is stored inside token
-> token is returned to client
```

---

# 🔓 Step 4 — Confirm that `authenticateToken` stores the decoded token

Your authentication middleware should verify the JWT and store the decoded payload on the request object.

You already do this using:

```javascript
req.user = decodedToken;
```

That means later middleware can access:

```javascript
req.user.userId
req.user.email
req.user.role
```

This is exactly what RBAC needs.

The flow becomes:

```text
request contains jwt
-> authenticateToken
-> jwt.verify()
-> decoded token
-> req.user
-> next()
```

---

# 📁 Step 5 — Create a new authorisation middleware file

Inside:

```text
backend/middleware
```

create a new file called:

```text
authoriseRoles.js
```

The purpose of this file is to determine whether the authenticated user's role is allowed to access a particular route.

Your middleware folder will now include something like:

```text
middleware
-> authenticateToken.js
-> authoriseRoles.js
-> errorHandler.js
-> notFound.js
-> validateAuth.js
-> validateGame.js
-> validateGameId.js
```

---

# 🏭 Step 6 — Make `authoriseRoles` accept allowed roles

The authorisation middleware should not be hard-coded to always allow only admins.

Instead, create it so that the route can specify which roles are accepted.

For example:

```javascript
authoriseRoles("admin")
```

should mean:

```text
only admin users may continue
```

Later, if you had more roles, you could support:

```javascript
authoriseRoles(
    "admin",
    "moderator"
)
```

This means the function should accept multiple role values.

A rest parameter is useful for this:

```javascript
...allowedRoles
```

This collects the supplied roles into an array.

For example:

```text
authoriseRoles("admin", "moderator")
-> allowedRoles
-> ["admin", "moderator"]
```

---

# 🧩 Step 7 — Return an Express middleware function

`authoriseRoles` should return another function containing:

```text
req
res
next
```

This is because you first configure the middleware:

```javascript
authoriseRoles("admin")
```

and then Express executes the returned middleware when a request arrives.

Conceptually:

```text
authoriseRoles("admin")
-> creates middleware
-> request reaches route
-> middleware checks req.user.role
```

This type of function is often called a middleware factory because one function creates another middleware function.

---

# 🚫 Step 8 — Handle cases where `req.user` does not exist

The authorisation middleware should assume that `authenticateToken` normally runs before it.

However, it is still useful to check whether:

```javascript
req.user
```

exists.

If it does not exist, return:

```text
401 Unauthorised
```

This means the user's identity has not been successfully established.

For example:

```text
request
-> authoriseRoles
-> req.user missing
-> 401
```

This check also makes the middleware safer if it is accidentally placed on a route without authentication.

---

# 🔍 Step 9 — Read the authenticated user's role

Once `req.user` exists, retrieve:

```javascript
req.user.role
```

For example, it might contain:

```text
admin
```

or:

```text
user
```

Store that value in a variable so that the authorisation check is easy to read.

---

# ✅ Step 10 — Check whether the role is allowed

Compare the user's role against the list of allowed roles.

The core idea is:

```javascript
allowedRoles.includes(
    req.user.role
)
```

This produces either:

```text
true
```

or:

```text
false
```

For example:

```text
allowedRoles = ["admin"]
req.user.role = "admin"

-> true
```

But:

```text
allowedRoles = ["admin"]
req.user.role = "user"

-> false
```

---

# ⛔ Step 11 — Return 403 when the user lacks permission

If the user is authenticated but does not have the required role, return:

```text
403 Forbidden
```

This is different from `401`.

Use this distinction:

```text
401
-> the user is not successfully authenticated

403
-> the user is authenticated but does not have permission
```

For example:

```text
normal user
-> valid jwt
-> role = user
-> tries DELETE /games/:id
-> admin role required
-> 403
```

A suitable response could include:

```text
success: false
error: you do not have permission to perform this action
```

---

# ▶️ Step 12 — Call `next()` when authorisation succeeds

If the user's role is allowed, call:

```javascript
next();
```

This tells Express:

```text
authorisation succeeded
-> continue to the next middleware/controller
```

The complete conceptual flow is:

```text
request
-> req.user exists?
-> no -> 401

-> yes
-> check req.user.role
-> role allowed?
-> no -> 403
-> yes -> next()
```

---

# 📤 Step 13 — Export `authoriseRoles`

Export the function from:

```text
authoriseRoles.js
```

Make sure your export style matches the way you intend to import it.

If you use a direct import such as:

```javascript
const authoriseRoles =
    require("../middleware/authoriseRoles");
```

then export the function directly.

This avoids the same type of import/export mismatch you encountered earlier.

---

# 🛣️ Step 14 — Import RBAC into `gameRoutes.js`

Open:

```text
backend/routes/gameRoutes.js
```

Import:

```text
authoriseRoles
```

from your new middleware file.

Your game routes already have:

```text
authenticateToken
```

so RBAC should be placed immediately after it on routes that require administrator permissions.

---

# 🎮 Step 15 — Decide which GameVault routes need admin permissions

Keep your read-only routes public:

```text
GET /games
GET /games/:id
```

Require administrator access for routes that modify game data:

```text
POST /games
PUT /games/:id
PATCH /games/:id
DELETE /games/:id
```

Your intended permissions are therefore:

```text
GET /games
-> public

GET /games/:id
-> public

POST /games
-> authenticated
-> admin

PUT /games/:id
-> authenticated
-> admin

PATCH /games/:id
-> authenticated
-> admin

DELETE /games/:id
-> authenticated
-> admin
```

---

# 🧱 Step 16 — Place the middleware in the correct order

Middleware order matters.

For your POST route, use this logical order:

```text
POST /games
-> authenticateToken
-> authoriseRoles("admin")
-> validateCreateGame
-> createGame
```

For PUT:

```text
PUT /games/:id
-> authenticateToken
-> authoriseRoles("admin")
-> validateGameId
-> validateFullGameUpdate
-> replaceGame
```

For PATCH:

```text
PATCH /games/:id
-> authenticateToken
-> authoriseRoles("admin")
-> validateGameId
-> validatePartialGameUpdate
-> updateGame
```

For DELETE:

```text
DELETE /games/:id
-> authenticateToken
-> authoriseRoles("admin")
-> validateGameId
-> deleteGame
```

This order is intentional.

First:

```text
who are you?
```

Then:

```text
are you allowed to do this?
```

Then:

```text
is the supplied data valid?
```

Then:

```text
perform the database operation
```

---

# 🛑 Step 17 — Do not allow users to select their own role during registration

This is one of the most important parts of your implementation.

Do not allow someone to register using:

```json
{
    "name": "name",
    "email": "name@example.com",
    "password": "Password123!",
    "role": "admin"
}
```

Otherwise anyone could make themselves an administrator.

Your public registration process should always create:

```text
role = user
```

The role should be controlled by your backend, not by the client.

Conceptually:

```text
registration request
-> validate name
-> validate email
-> validate password
-> hash password
-> create user
-> role automatically set to user
```

Not:

```text
registration request
-> accept role from client
```

---

# 👑 Step 18 — Create an admin account for testing

Because new registrations should always create normal users, you need another method to create an administrator for testing.

The easiest classroom method is:

```text
register a normal account
-> open mongodb atlas or compass
-> find that user
-> change role from user to admin
-> save
```

For example:

```text
role: user
```

becomes:

```text
role: admin
```

---

# 🔄 Step 19 — Login again after changing the role

This is very important.

Suppose a user logs in while their database role is:

```text
user
```

Their token will contain:

```text
role = user
```

You then change the MongoDB document to:

```text
role = admin
```

The old JWT does **not** automatically change.

It still contains:

```text
role = user
```

So after changing the role:

```text
change mongodb role
-> login again
-> generate new jwt
-> new jwt contains admin
```

Use the new token for your Postman tests.

---

# 🧪 Step 20 — Test without a token

Try:

```text
POST /games
```

without an Authorisation header.

Expected:

```text
401 Unauthorised
```

Flow:

```text
POST /games
-> authenticateToken
-> no jwt
-> 401
```

RBAC should not even be reached because authentication failed first.

---

# 🧪 Step 21 — Test with a normal user's token

Login with a user whose role is:

```text
user
```

Copy their JWT.

Use:

```text
Authorisation: Bearer YOUR_TOKEN
```

Then try:

```text
POST /games
```

Expected:

```text
403 Forbidden
```

Flow:

```text
POST /games
-> authenticateToken
-> valid jwt
-> req.user.role = user
-> authoriseRoles("admin")
-> role not allowed
-> 403
```

This proves that authentication and authorisation are working separately.

---

# 🧪 Step 22 — Test with an admin token

Login using your administrator account.

Copy the newly generated JWT.

Send:

```text
POST /games
```

with:

```text
Authorisation: Bearer ADMIN_TOKEN
```

Expected flow:

```text
POST /games
-> authenticateToken
-> jwt valid
-> req.user.role = admin
-> authoriseRoles("admin")
-> role allowed
-> validateCreateGame
-> createGame
-> mongodb
-> 201
```

---

# 🧪 Step 23 — Test all administrator routes

Do not stop after testing POST.

Test:

```text
POST /games
PUT /games/:id
PATCH /games/:id
DELETE /games/:id
```

For each route, test:

```text
no token
-> 401

normal user token
-> 403

admin token
-> request allowed
```

This gives you a complete RBAC test.

---

# 🔎 Step 24 — Confirm that GET requests still work

Also confirm that:

```text
GET /games
GET /games/:id
```

still work without a JWT if you want those routes to remain public.

That proves you have not accidentally placed RBAC on the entire router.

---

# 🔐 Final RBAC request flow

Your protected game routes should ultimately follow:

```text
request
-> gameRoutes.js
-> authenticateToken
-> verify jwt
-> req.user created
-> authoriseRoles
-> inspect req.user.role
-> permission granted
-> validation middleware
-> controller
-> mongoose model
-> mongodb
-> response
```

A normal user attempting an admin action:

```text
DELETE /games/:id
-> authenticateToken
-> jwt valid
-> req.user.role = user
-> authoriseRoles("admin")
-> 403 forbidden
-> request stops
```

An admin performing the same action:

```text
DELETE /games/:id
-> authenticateToken
-> jwt valid
-> req.user.role = admin
-> authoriseRoles("admin")
-> next()
-> validateGameId
-> deleteGame
-> mongodb
-> 200
```

# 🎓 What you should understand after implementing this

The key concept is not simply that you have added another middleware file.

```text
jwt
-> carries authenticated user information

authenticateToken
-> verifies the jwt
-> creates req.user

authoriseRoles
-> checks req.user.role
-> decides whether access is allowed

controller
-> only runs after authentication and authorisation succeed
```

That separation is the main reason RBAC fits cleanly into your existing Express structure.
