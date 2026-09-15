# 🎮 GameVault Frontend — Registration and Login 

## 🎯 What are we implementing?

The GameVault frontend can currently communicate with the backend using the health endpoint.

The existing communication is:

```text
App.jsx
-> api.js
-> GET /health
-> Express backend
-> systemRoutes.js
-> systemController.js
-> JSON response
-> api.js
-> App.jsx
```

We now want users to be able to **register and log in through the React frontend** instead of relying on Postman.

Registration should follow this process:

```text
user completes registration form
-> React collects name, email and password
-> api.js
-> POST /auth/register
-> Express backend
-> registration validation
-> password hashing
-> MongoDB
-> JWT generated
-> response returned
-> frontend receives JWT
-> JWT stored
-> authenticated interface displayed
```

Login should follow:

```text
user completes login form
-> React collects email and password
-> api.js
-> POST /auth/login
-> Express backend
-> credentials checked
-> MongoDB
-> password compared
-> JWT generated
-> response returned
-> frontend receives JWT
-> JWT stored
-> authenticated interface displayed
```

The existing health check should remain in the application.

---

# 📁 Step 1 — Create the component structure

Inside:

```text
frontend/src
```

create a new folder called:

```text
components
```

Inside this folder, create two components:

```text
LoginForm.jsx
RegisterForm.jsx
```

The frontend should now have approximately this structure:

```text
src
├── components
│   ├── LoginForm.jsx
│   └── RegisterForm.jsx
├── services
│   └── api.js
├── App.jsx
├── index.css
└── main.jsx
```

Each file has a different responsibility:

```text
api.js
-> handles communication with the backend

RegisterForm.jsx
-> collects registration information
-> starts registration requests

LoginForm.jsx
-> collects login information
-> starts login requests

App.jsx
-> manages the overall interface
-> manages authentication state
-> receives successful authentication results
-> handles logout

main.jsx
-> starts the React application
```

Separating these responsibilities prevents `App.jsx` from becoming responsible for every part of the application.

---

# 🌐 Step 2 — Keep the backend URL centralised in `api.js`

The existing `api.js` already contains the backend location:

```javascript
const API_BASE_URL =
    "https://localhost:4000";
```

Keep this for the current implementation.

The purpose of `api.js` is to provide **one central location for backend communication**.

Without this separation, different components could end up doing things such as:

```text
LoginForm.jsx
-> independently connects to backend

RegisterForm.jsx
-> independently connects to backend

App.jsx
-> independently connects to backend
```

Instead, use:

```text
LoginForm.jsx
-> api.js
-> backend

RegisterForm.jsx
-> api.js
-> backend

App.jsx
-> api.js
-> backend
```

The components request an action, while `api.js` handles the HTTP communication.

---

# 🐛 Step 3 — Correct the existing `getHealth()` function

Before adding authentication, check the current `getHealth()` function.

The backend response must be converted from JSON into a JavaScript object **before returning the data**.

The general order should be:

```text
fetch health endpoint
-> wait for response
-> convert response from JSON
-> check whether response was successful
-> unsuccessful -> throw error
-> successful -> return data
```

In the current implementation, the statement that converts the response to JSON is inside the unsuccessful-response block and appears after an error is thrown.

Anything after:

```javascript
throw new Error(...)
```

inside the same block will not execute.

Move the JSON conversion outside that block so that the resulting `data` variable is available when the function reaches:

```javascript
return data;
```

---

# 📡 Step 4 — Add a registration function to `api.js`

Create a new asynchronous function responsible specifically for registration.

A useful name is:

```javascript
registerUser
```

The function should receive one parameter representing the registration information.

For example:

```javascript
registerUser(registrationData)
```

The supplied object will eventually contain:

```text
name
email
password
```

The function must send a request to:

```text
/auth/register
```

using the backend base URL already stored in `api.js`.

The request method must be:

```text
POST
```

because registration creates a new user.

---

# 📦 Step 5 — Configure the registration request

The registration request needs three important pieces of information:

```text
method
headers
body
```

Set the method to:

```text
POST
```

Add a `Content-Type` header indicating:

```text
application/json
```

This tells Express:

```text
request received
-> body contains JSON
-> Express can process it as JSON
```

The request body should contain the registration object.

However, `fetch()` cannot simply send the JavaScript object as the JSON request body.

Convert it using:

```javascript
JSON.stringify(...)
```

The conceptual request becomes:

```text
registrationData
-> JSON.stringify()
-> request body
-> POST /auth/register
```

---

# 📥 Step 6 — Process the registration response

After calling `fetch()`, wait for the backend response.

Convert the response from JSON into a JavaScript object using:

```javascript
response.json()
```

Then check:

```javascript
response.ok
```

`response.ok` indicates whether the HTTP response represents a successful request.

If registration fails, the backend may return responses such as:

```text
400
-> invalid registration information

409
-> email already exists

429
-> too many requests
```

If `response.ok` is false, throw an error.

Where possible, use the error message returned by the backend.

A useful pattern is:

```javascript
data.error || "Registration failed."
```

This means:

```text
backend supplied an error message
-> use backend message

no backend error message
-> use general frontend message
```

If the request succeeds, return the backend response data.

---

# 📤 Step 7 — Export the registration function

Your `api.js` currently exports:

```text
getHealth
```

Update the export so that it also exports:

```text
registerUser
```

Later, the registration component will import this function.

At this stage:

```text
api.js

getHealth()
-> GET /health

registerUser()
-> POST /auth/register
```

---

# 📝 Step 8 — Build the registration component

Open:

```text
RegisterForm.jsx
```

Import React's:

```javascript
useState
```

The component needs state because it must remember information while the user interacts with the form.

Create state for:

```text
name
email
password
error
loading
```

Initial values should logically be:

```text
name
-> empty string

email
-> empty string

password
-> empty string

error
-> empty string

loading
-> false
```

The first three store user input.

`error` stores a message when registration fails.

`loading` records whether a registration request is currently being processed.

---

# 🧩 Step 9 — Import `registerUser()` into the component

Import:

```text
registerUser
```

from:

```text
../services/api
```

The component itself should **not** contain the complete backend URL or duplicate the `fetch()` configuration.

The responsibility remains:

```text
RegisterForm.jsx
-> collects information
-> asks api.js to register user

api.js
-> performs HTTP request
```

---

# ✏️ Step 10 — Create the registration inputs

Create a form containing:

```text
Name
Email
Password
Register button
```

Use appropriate HTML input types:

```text
name
-> text

email
-> email

password
-> password
```

Connect each input to its corresponding React state.

For example, the name input should conceptually use:

```jsx
value={name}
```

and its `onChange` event should call:

```javascript
setName(...)
```

using:

```javascript
event.target.value
```

Repeat the same pattern for email and password.

---

# 🧠 Step 11 — Understand controlled inputs

When a user types their name, React should keep the state synchronised with the input.

For example:

```text
user types "T"
-> onChange
-> event.target.value = "T"
-> setName("T")

user continues typing
-> event.target.value = "Talia"
-> setName("Talia")
```

React now knows the current value of the input.

The same applies to:

```text
email
password
```

This means that when the user eventually submits the form, the component already has all three values stored in state.

---

# 📤 Step 12 — Create the registration submission function

Inside `RegisterForm.jsx`, create an asynchronous function that handles form submission.

A suitable name is:

```javascript
handleSubmit
```

The function must receive the form submission event.

The first action should be:

```javascript
event.preventDefault();
```

A normal HTML form attempts to perform its own submission, which can cause the browser to reload or navigate.

React should control the submission instead.

Therefore:

```text
user submits form
-> prevent normal HTML submission
-> React processes submission
```

---

# ⏳ Step 13 — Prepare the registration request state

At the beginning of the submission process:

```text
clear previous error
-> set error to empty string

request starts
-> set loading to true
```

This ensures an old error does not remain on screen while a new request is being attempted.

The loading state can also be used to disable the Register button temporarily.

---

# 📦 Step 14 — Build the registration object

Inside the submission function, create an object containing:

```text
name
email
password
```

The property names must match what your backend's registration validation expects.

Conceptually:

```javascript
const registrationData = {
    name,
    email,
    password
};
```

If the user entered:

```text
Name: Test User
Email: test@example.com
Password: Password123!
```

the resulting request data represents:

```text
name
-> Test User

email
-> test@example.com

password
-> Password123!
```

Do **not** include:

```text
role
```

Public users must not be allowed to select their own role.

Your backend should continue assigning:

```text
role
-> user
```

automatically.

---

# 🔐 Step 15 — Do not hash passwords in React

The registration component should send the password to your HTTPS backend.

Do not install or use bcrypt in the React frontend.

The intended flow is:

```text
password entered
-> React
-> HTTPS request
-> Express backend
-> backend validation
-> bcrypt
-> passwordHash
-> MongoDB
```

The frontend must also never contain:

```text
JWT_SECRET
MongoDB connection string
database credentials
private backend configuration
```

These belong on the server.

---

# 📡 Step 16 — Call the registration API function

Inside a `try` block, call:

```javascript
registerUser(registrationData)
```

Because it is asynchronous, use `await`.

Store the successful result in a variable such as:

```javascript
data
```

The response should contain the information returned by your backend after successful registration, including the JWT if your backend is currently configured to return one.

---

# ❌ Step 17 — Handle registration errors

Add a `catch` block.

When `registerUser()` throws an error, retrieve the error's message and store it in the registration component's `error` state.

The flow becomes:

```text
backend rejects registration
-> api.js receives unsuccessful response
-> api.js throws error
-> RegisterForm catches error
-> error state updated
-> React re-renders
-> error displayed
```

Use a `finally` block to set:

```text
loading
-> false
```

because the request is finished whether it succeeded or failed.

---

# 🚦 Step 18 — Use the loading state

While registration is taking place, disable the Register button.

You can also change its text.

For example:

```text
loading = false
-> Register

loading = true
-> Registering...
```

This prevents users from repeatedly submitting the same form while waiting for the backend.

This is particularly useful now that the backend also has rate limiting.

---

# 🔑 Step 19 — Create `LoginForm.jsx`

The login component follows almost the same structure as registration.

Import:

```text
useState
loginUser
```

Create state for:

```text
email
password
error
loading
```

Login does not require a name because the backend identifies the account using the supplied credentials.

Create a form containing:

```text
Email
Password
Login button
```

Connect the email and password inputs to React state in the same way as the registration form.

---

# 🌐 Step 20 — Add `loginUser()` to `api.js`

Return to:

```text
services/api.js
```

Create another asynchronous function called:

```javascript
loginUser
```

It should receive:

```javascript
loginData
```

and send a request to:

```text
/auth/login
```

Use:

```text
POST
```

and send:

```text
Content-Type
-> application/json
```

Convert the login object into JSON before sending it.

After receiving the response:

```text
wait for response
-> convert JSON response
-> check response.ok
-> unsuccessful -> throw error
-> successful -> return data
```

Export `loginUser` together with the other API functions.

Your service now provides:

```text
getHealth()
-> GET /health

registerUser()
-> POST /auth/register

loginUser()
-> POST /auth/login
```

---

# 🔓 Step 21 — Implement login submission

Inside `LoginForm.jsx`, create an asynchronous submission handler.

The process should be:

```text
form submitted
-> preventDefault()
-> clear previous error
-> loading = true
-> create loginData
-> call loginUser()
-> wait for backend
```

The login object should contain only:

```text
email
password
```

If authentication fails:

```text
catch error
-> store error.message
-> display error
```

Whether it succeeds or fails:

```text
finally
-> loading = false
```

---

# 🎟️ Step 22 — Understand what happens after successful authentication

Your backend should already generate a JWT after successful registration/login.

Check the successful responses inside:

```text
backend/controllers/authController.js
```

Determine the exact property name used for the JWT.

For example, if your backend returns:

```javascript
{
    success: true,
    token: "..."
}
```

then the frontend can access:

```javascript
data.token
```

Do not simply assume the property is called `token`. It must match your actual backend response.

---

# 🔄 Step 23 — Allow the forms to communicate with `App.jsx`

`App.jsx` should control the overall authentication state.

Therefore, when either form successfully authenticates the user, it needs to inform `App.jsx`.

Have both components receive a function through props.

A suitable prop name is:

```javascript
onAuthSuccess
```

Conceptually:

```text
App.jsx
-> provides onAuthSuccess to LoginForm

App.jsx
-> provides onAuthSuccess to RegisterForm
```

After registration succeeds:

```text
RegisterForm
-> onAuthSuccess(data)
```

After login succeeds:

```text
LoginForm
-> onAuthSuccess(data)
```

This allows both forms to use the same authentication logic in the parent component.

---

# 🏠 Step 24 — Add authentication state to `App.jsx`

Keep your existing:

```text
health
loading
error
```

state.

Add another state value controlling which authentication form is visible.

For example:

```text
authView
```

Its possible values can be:

```text
login
register
```

Start with:

```text
login
```

Then:

```text
authView = login
-> display LoginForm

authView = register
-> display RegisterForm
```

---

# 💾 Step 25 — Add token state

`App.jsx` also needs to know whether the browser currently has an authentication token.

Create state for:

```text
token
```

Instead of always starting it as `null`, check browser `localStorage` for a previously stored GameVault token.

Use a meaningful storage key such as:

```text
gamevaultToken
```

The logic should be:

```text
App.jsx starts
-> check localStorage
-> token exists -> initialise token state with stored value
-> token missing -> initialise token state as null
```

---

# 💾 Step 26 — Understand why we are using `localStorage`

React state exists while the application is running.

If you only use React state:

```text
login
-> token stored in state
-> page refreshed
-> React application restarts
-> state recreated
-> token lost
```

`localStorage` persists browser data.

Therefore:

```text
login
-> token stored in localStorage
-> page refreshed
-> App.jsx checks localStorage
-> token found
-> authenticated state restored
```

For this learning implementation, this provides a clear demonstration of JWT persistence.

For production systems, secure HttpOnly cookies are another common approach and can reduce exposure of tokens to JavaScript. That requires a different authentication architecture and is outside the current implementation.

---

# 🔐 Step 27 — Create `handleAuthSuccess()`

Inside `App.jsx`, create a function that receives the successful authentication response from either form.

It should:

```text
receive backend data
-> retrieve JWT
-> confirm JWT exists
-> save JWT in localStorage
-> save JWT in React state
```

You will need methods such as:

```javascript
localStorage.setItem(...)
```

and your React state setter.

Why do both?

```text
React state
-> updates the interface immediately

localStorage
-> keeps token after refresh
```

---

# 👁️ Step 28 — Change the interface according to authentication state

Use the token state to decide what the user sees.

The logic should be:

```text
token exists
-> display authenticated interface

token does not exist
-> display login/register interface
```

For now, the authenticated interface can simply display:

```text
You are logged in.
Logout
```

We are not yet building the complete GameVault dashboard.

The objective of this implementation is to establish authentication first.

---

# 🔀 Step 29 — Add Login and Register controls

When no token exists, display two controls:

```text
Login
Register
```

Clicking Login should change:

```text
authView
-> login
```

Clicking Register should change:

```text
authView
-> register
```

Then use conditional rendering.

Conceptually:

```jsx
authView === "login"
    ? <LoginForm />
    : <RegisterForm />
```

Both components should receive:

```text
onAuthSuccess
```

from `App.jsx`.

---

# 🧠 Step 30 — Understand the parent/child relationship

The relationship now becomes:

```text
App.jsx
-> passes handleAuthSuccess as a prop
-> LoginForm receives it as onAuthSuccess
-> user logs in
-> LoginForm receives backend response
-> LoginForm calls onAuthSuccess(data)
-> App.jsx receives data
-> JWT stored
-> authentication state changes
```

Registration follows the same process.

This allows:

```text
child component
-> perform specific task

parent component
-> manage overall application state
```

---

# 🚪 Step 31 — Implement logout

Create a logout function inside `App.jsx`.

Logout should perform two operations:

```text
remove token from localStorage
-> remove persistent token

set token state to null
-> update React immediately
```

Use:

```javascript
localStorage.removeItem(...)
```

with the same key used when storing the JWT.

After logout:

```text
token = null
-> React re-renders
-> authenticated interface disappears
-> login/register interface returns
```

You can also reset `authView` to:

```text
login
```

so that the login form appears automatically.

---

# ❤️ Step 32 — Keep the existing health check

Do not remove the existing health functionality.

The page can still display:

```text
Welcome to GameVault

API connection
-> Backend connected

authentication section
-> Login/Register
```

This means students can see whether backend communication is working before trying authentication.

Also correct the current error condition.

The error should display when:

```text
loading = false
error exists
```

rather than when:

```text
loading = true
error exists
```

The request has normally finished by the time the `catch` block stores the error.

---

# 🎨 Step 33 — Add basic form styling

Extend the existing `index.css`.

You will need styling for:

```text
authentication card
authentication navigation
form
labels
inputs
buttons
disabled buttons
errors
```

A simple layout could use:

```text
form
-> display flex
-> flex-direction column
```

This will place:

```text
Name
input

Email
input

Password
input

Register
```

vertically.

Give the Login and Register selection buttons enough spacing to make it clear that they change the active form.

Do not spend too much time on visual design yet.

The priority is:

```text
functionality
-> communication
-> authentication
-> state
-> feedback
```

---

# 🔒 Step 34 — Check CORS before testing

Your React/Vite frontend will normally run on:

```text
http://localhost:5173
```

Your Express backend runs on:

```text
https://localhost:4000
```

Your backend CORS configuration must therefore permit the frontend origin.

Check the **backend** `.env`.

The permitted client origin should match:

```text
http://localhost:5173
```

unless you have explicitly configured Vite itself to use HTTPS.

Restart the backend after changing backend environment variables.

---

# 🔐 Step 35 — Check the development HTTPS certificate

The backend uses a self-signed HTTPS certificate.

Your browser may initially refuse requests from React to:

```text
https://localhost:4000
```

Open the backend health endpoint directly in the browser first:

```text
https://localhost:4000/health
```

If your browser warns about the **localhost certificate that you created for this project**, allow/trust it for local development.

Then return to the React frontend.

Do not treat bypassing certificate warnings as normal behaviour for real websites. This is specifically for the self-signed development certificate created for GameVault.

---

# ▶️ Step 36 — Run both applications

You need the backend and frontend running simultaneously.

Use one terminal for:

```text
backend
-> npm run dev
```

Confirm:

```text
MongoDB connects
-> backend starts
-> HTTPS server listens on port 4000
```

Use another terminal for:

```text
frontend
-> npm run dev
```

Open the Vite URL displayed in the terminal.

---

# 🧪 Step 37 — Test the health connection first

Before registration or login, confirm the existing API status still works.

You should see the backend information being displayed.

This confirms:

```text
React
-> api.js
-> HTTPS
-> Express
-> response received
```

If the health request itself fails, fix that connection before debugging registration.

---

# 🧪 Step 38 — Test registration

Select:

```text
Register
```

Enter:

```text
name
valid email
valid password
```

Remember that the current backend password validation requires:

```text
8–128 characters
uppercase letter
lowercase letter
number
special character
```

Submit the form.

The complete process should be:

```text
RegisterForm
-> handleSubmit
-> registrationData
-> registerUser
-> api.js
-> POST /auth/register
-> Express
-> rate limiter
-> validation
-> auth controller
-> bcrypt
-> User model
-> MongoDB
-> generate JWT
-> response
-> api.js
-> RegisterForm
-> onAuthSuccess
-> App.jsx
-> localStorage
-> React token state
-> authenticated interface
```

---

# 🗄️ Step 39 — Check MongoDB

After successful registration, inspect the users collection.

Confirm that the new user exists.

The user should contain information such as:

```text
name
email
passwordHash
role
timestamps
```

The original plaintext password should **not** be stored.

Also confirm:

```text
role
-> user
```

A publicly registered account should not automatically become an administrator.

---

# 🚪 Step 40 — Test logout

Select Logout.

Confirm:

```text
JWT removed from localStorage
-> token state becomes null
-> React re-renders
-> Login/Register interface displayed
```

You can inspect the browser's developer tools to confirm that the stored token has disappeared.

---

# 🔑 Step 41 — Test login

Use the account you registered.

Enter:

```text
email
password
```

Submit the login form.

The expected flow is:

```text
LoginForm
-> loginData
-> loginUser
-> api.js
-> POST /auth/login
-> Express
-> rate limiter
-> validation
-> auth controller
-> MongoDB
-> retrieve user
-> bcrypt compares password
-> generate JWT
-> response
-> frontend
-> localStorage
-> authenticated interface
```

---

# ❌ Step 42 — Test invalid login details

Logout and attempt to log in using an incorrect password.

Expected:

```text
incorrect credentials
-> backend rejects login
-> unsuccessful HTTP response
-> api.js detects response.ok is false
-> error thrown
-> LoginForm catches error
-> error state updated
-> React displays error
```

The user should remain unauthenticated.

---

# 🚦 Step 43 — Test the rate limiter through the frontend

Repeatedly submit login requests.

Because `/auth/login` is protected by your authentication rate limiter:

```text
React
-> POST /auth/login
-> apiLimiter
-> authLimiter
```

eventually the backend should return:

```text
429 Too Many Requests
```

Your frontend should not require special `429` logic for the basic implementation.

The existing error handling can:

```text
receive 429
-> response.ok = false
-> throw error
-> LoginForm catches error
-> display backend message
```

This demonstrates that the security mechanisms implemented on the backend also protect requests coming from the real frontend.

---

# 🔎 Step 44 — Inspect the stored JWT

After successful authentication, open the browser developer tools and inspect Local Storage for the Vite application.

You should find:

```text
gamevaultToken
-> JWT value
```

This demonstrates:

```text
backend generates JWT
-> response sent to frontend
-> React receives response
-> App.jsx extracts JWT
-> JWT stored in browser
```

The JWT should never be manually copied into your frontend source files.

---

# 🔄 Step 45 — Test authentication persistence

While logged in, refresh the browser.

Expected:

```text
browser refresh
-> React application restarts
-> App.jsx initialises
-> localStorage checked
-> gamevaultToken found
-> token state populated
-> authenticated interface displayed
```

The user should therefore still appear logged in.

There is an important limitation to understand:

```text
token exists
-> frontend currently assumes authenticated
```

At this stage, the frontend has **not yet verified that the stored JWT is still valid or unexpired**.

That can be handled in the next implementation when the frontend calls the protected `/auth/profile` endpoint.

---

# 🔐 Step 46 — Understand what has been completed

After this implementation, the frontend supports:

```text
backend health check
-> complete

registration form
-> complete

registration API request
-> complete

login form
-> complete

login API request
-> complete

receive JWT
-> complete

store JWT
-> complete

authentication state
-> complete

logout
-> complete

token persistence after refresh
-> complete
```

We have **not yet implemented**:

```text
retrieve authenticated profile
display user name/email/role
send JWT with protected requests
display games
admin-only frontend controls
create/update/delete games through frontend
```

Those should come afterwards.

---


# 📚 Resources

For more information on the concepts used in this implementation, use these official resources:

* [React — useState documentation](https://react.dev/reference/react/useState) — explains how React components store and update state, which is used for the form inputs, loading state, errors and JWT state.
* [MDN — Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) — explains `fetch()`, HTTP requests, request bodies, headers, responses and error handling.
* [MDN — Window localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) — explains how `localStorage` stores, retrieves and removes browser data.

