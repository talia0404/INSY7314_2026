# 🛡️ GameVault Learning Unit 2 Part C: Securing the Backend with Helmet, CSP and CORS

## 📌 Overview

Your GameVault backend is now structured into separate routes, controllers, middleware and configuration files.

Although the application is functional, it still needs additional security measures that protect it when communicating with browsers and future frontend applications.

In this section you will add three important security technologies:

* Helmet
* Content Security Policy (CSP)
* Cross-Origin Resource Sharing (CORS)

These technologies do not replace HTTPS or input validation.

Instead, they add another layer of protection to your application.

---

# 📚 What we're doing?

* Explain the purpose of Helmet.
* Explain what HTTP security headers are.
* Explain Content Security Policy (CSP).
* Explain Cross-Origin Resource Sharing (CORS).
* Configure Helmet.
* Configure a basic Content Security Policy.
* Configure CORS for a frontend application.
* Test security headers using Postman.

---

# 🏗️ Part 1: Understand Where These Features Belong

Open your project.

These features belong inside:

```text
backend/app.js
```

Do not add them to:

```text
server.js
```

Remember the responsibilities of each file:

| File        | Responsibility                       |
| ----------- | ------------------------------------ |
| server.js   | Starts the HTTPS server              |
| app.js      | Configures the Express application   |
| routes      | Maps URLs to controllers             |
| controllers | Contains business logic              |
| middleware  | Executes before or after controllers |

Helmet and CORS are application-level middleware.

They must be configured inside app.js before your routes.

---

# 📦 Part 2: Install the Required Packages

Open a terminal inside:

```text
GameVault/backend
```

Install Helmet:

```bash
npm install helmet
```

Install CORS:

```bash
npm install cors
```

Verify the installation:

```bash
npm list helmet cors
```

Both packages should now appear in your project dependencies.

---

# 🪖 Part 3: Add Helmet

Open:

```text
backend/app.js
```

Import the Helmet package.

Next, register Helmet as application middleware.

Helmet should be configured immediately after the Express application is created and before any routes are registered.

Your middleware order should begin to resemble:

```text
Create Express App

->

Helmet

->

JSON Middleware

->

Routes
```

---

# 🧠 Why Are We Adding Helmet?

Express itself provides very few security headers.

Helmet automatically adds several important HTTP response headers that improve browser security.

Examples include:

* Content Security Policy
* X-Frame-Options
* X-Content-Type-Options
* Referrer-Policy
* Strict-Transport-Security

These headers help reduce risks such as:

* Clickjacking
* MIME sniffing
* Some Cross-Site Scripting (XSS) attacks
* Information leakage

Helmet does not:

* Encrypt traffic
* Validate user input
* Authenticate users
* Replace HTTPS

It is simply another security layer.

---

# 🔐 Part 4: Configure a Content Security Policy (CSP)

Helmet allows you to customise the Content Security Policy that will be sent to browsers.

Rather than using Helmet's default configuration, create your own policy.

Your policy should:

* Use the current origin as the default source.
* Allow scripts only from the current origin.
* Allow styles only from the current origin.
* Allow fonts only from the current origin.
* Allow images from the current origin and embedded data URLs.
* Restrict browser connections to the current origin.
* Block embedded plugins.
* Prevent the application from being displayed inside frames.
* Restrict the document base URL.
* Restrict where forms may submit.

Configure this policy using the Helmet middleware.

Do not create a separate CSP package.

Helmet already provides this functionality.

---

# 🧠 Understanding Content Security Policy

Content Security Policy tells browsers where resources are allowed to come from.

Examples include:

* JavaScript
* Images
* CSS
* Fonts
* Videos

Instead of trusting every website on the internet, the browser only trusts the locations you explicitly allow.

For example:

```text
'default-src self'
```

means:

> Only load resources from this website.

This helps reduce the impact of attacks such as Cross-Site Scripting (XSS).

---

# 🌍 Part 5: Configure CORS

Eventually your React frontend will communicate with your Express backend.

Browsers enforce the Same-Origin Policy, which means that websites cannot freely communicate with servers running on different origins.

To allow your frontend to communicate with GameVault, configure CORS.

---

## Create a New Environment Variable

Open:

```text
backend/.env
```

Add:

```env
CLIENT_ORIGIN=https://localhost:5173
```

Also update:

```text
backend/.env.example
```

Include the same variable name.

---

## Configure CORS

Import the CORS package into:

```text
app.js
```

Create a configuration object that:

* Reads the frontend URL from the environment variables.
* Allows only that origin.
* Allows the REST API methods used by GameVault.
* Allows JSON request headers.
* Supports future Authorization headers.
* Returns the appropriate response for browser preflight requests.

Finally, register the CORS middleware before your routes.

---

# 🧠 Understanding CORS

An origin consists of:

```text
Protocol
Hostname
Port
```

Example:

```text
https://localhost:4000
```

The frontend might run on:

```text
https://localhost:5173
```

Even though both use localhost, they are considered different origins because the ports are different.

Without CORS, browser JavaScript would not be allowed to read responses from your backend.

---

# ⚠️ Important

CORS only affects browsers.

Applications such as:

* Postman
* curl
* Another backend server

are not restricted by CORS.

CORS is not authentication.

It simply tells browsers which websites are allowed to access your API.

---

# 🔄 Part 6: Check the Middleware Order

Your middleware should now execute in the following order:

```text
HTTPS Server

->

Helmet

->

Content Security Policy

->

CORS

->

JSON Request Parser

->

Routes

->

Controllers

->

Not Found Middleware

->

Error Handler
```

The order is important.

Helmet and CORS should run before the routes so that every response includes the appropriate security headers.

---

# 🧪 Part 7: Test Helmet

Start the application:

```bash
npm run dev
```

Open Postman.

Test:

```text
GET https://localhost:4000/health
```

Open the Headers tab in the response.

Confirm that security headers such as the following are present:

* Content-Security-Policy
* Strict-Transport-Security
* X-Content-Type-Options
* X-Frame-Options
* Referrer-Policy

Your JSON response should remain unchanged.

---

# 🧪 Part 8: Test CORS

In Postman, create a request to:

```text
GET https://localhost:4000/games
```

Add a request header:

```text
Origin: https://localhost:5173
```

Send the request.

Confirm that the response includes an appropriate:

```text
Access-Control-Allow-Origin
```

header.

Next, change the Origin header to a different website.

Observe how the response headers change.

Remember that Postman will still display the response because it does not enforce browser CORS rules.

---

# 📋 Part 9: Completion Checklist

Before moving on, confirm that:

* Helmet is installed.
* CORS is installed.
* Helmet is configured in `app.js`.
* A custom Content Security Policy has been configured.
* CORS reads the frontend URL from `.env`.
* `.env.example` has been updated.
* Helmet executes before the routes.
* CORS executes before the routes.
* Security headers appear in Postman.
* Existing GameVault endpoints still work correctly.
* HTTPS continues to function.

---

# ☁️ Part 10: Commit and Push Your Changes


