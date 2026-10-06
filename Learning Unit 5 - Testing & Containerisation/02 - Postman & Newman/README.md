# 📮 Step 02 — Postman & Newman

You have already used Postman throughout GameVault.

So far, you have probably been doing something like:

```text
open Postman
-> choose request
-> click Send
-> inspect response
```

Now we are going to make those tests reusable and executable automatically.

---

# 📁 11. Organise your Postman collection

Create or update a collection called:

```text
GameVault API
```

Organise the requests into folders.

For example:

```text
GameVault API

├── System
│   └── Health Check
│
├── Authentication
│   ├── Register - Valid
│   ├── Register - Duplicate Email
│   ├── Login - Valid
│   └── Login - Invalid Password
│
├── Games
│   ├── Get Games
│   ├── Get Game
│   ├── Create Game
│   ├── Update Game
│   └── Delete Game
│
└── Protected Routes
    ├── Profile - Valid Token
    └── Profile - No Token
```

Avoid keeping all your test scenarios inside one request and manually changing the body every time.

Each important scenario should be a saved request.

---

# 🌐 12. Create a Postman environment

Instead of writing this repeatedly:

```text
https://localhost:4000
```

create a Postman environment variable:

```text
baseUrl
```

Set its value to:

```text
https://localhost:4000
```

Then requests can use:

```text
{{baseUrl}}/auth/login
```

or:

```text
{{baseUrl}}/games
```

This makes the collection easier to move between environments.

---

# 🔑 13. Automatically save the JWT

We don't want to manually copy the JWT every time we log in.

Open your Login - Valid request.

Go to:

```text
Scripts
-> Post-response
```

Depending on your Postman version, this may also appear as the Tests area.

Add:

```javascript
const response = pm.response.json();

pm.environment.set("token", response.token);
```

If your GameVault response looks like:

```json
{
    "data": {
        "token": "..."
    }
}
```

then use:

```javascript
const response = pm.response.json();

pm.environment.set("token", response.data.token);
```

Use whichever matches your API response.

Now the flow becomes:

```text
login
-> api returns jwt
-> postman saves jwt
-> later requests use saved jwt
```

For a protected request, add:

```text
Authorization: Bearer {{token}}
```

You no longer need to copy and paste tokens manually.

---

# ✅ 14. Add Postman tests

Postman can automatically inspect responses.

For example, your successful login request could contain:

```javascript
pm.test("status should be 200", function () {
    pm.response.to.have.status(200);
});
```

You could also verify that a token exists:

```javascript
pm.test("login should return a token", function () {

    const body = pm.response.json();

    pm.expect(body).to.have.property("token");

});
```

Again, change this if your token is inside `data`.

For an invalid login:

```javascript
pm.test("invalid login should be rejected", function () {
    pm.response.to.have.status(401);
});
```

This changes Postman from:

```text
send request
-> student visually checks response
```

into:

```text
send request
-> postman checks response
-> test passes or fails
```

---

# 🧪 15. Add tests to the important requests

At minimum, your collection should automatically check:

```text
registration
-> correct success status
-> user returned
-> password/password hash not returned

login
-> correct success status
-> jwt returned

invalid login
-> 401

games
-> correct response status
-> expected data returned

protected route without jwt
-> 401

protected route with jwt
-> 200

admin-only action as normal user
-> 403

admin-only action as admin
-> succeeds
```

You can add more tests depending on your implementation.

---

# 📤 16. Export the collection

In Postman, export your collection as:

```text
Collection v2.1
```

Place it somewhere sensible in GameVault:

```text
GameVault
├── backend
├── frontend
└── postman
    └── GameVault.postman_collection.json
```

If you use a Postman environment, export that as well:

```text
postman
├── GameVault.postman_collection.json
└── GameVault.postman_environment.json
```

Do not export real secrets or real authentication tokens into GitHub.

Check the exported files before committing them.

---

# 🤖 17. Install Newman

Postman is mainly a graphical application.

Newman allows us to run a Postman collection from the terminal.

Install it:

```bash
npm install --save-dev newman
```

You can install it in the backend project so everyone receives the same version through `npm install`.

---

# ▶️ 18. Run the collection with Newman

First make sure the GameVault backend is running.

For example:

```bash
npm start
```

Then open another terminal.

From the project directory, run:

```bash
npx newman run postman/GameVault.postman_collection.json
```

If you exported an environment:

```bash
npx newman run postman/GameVault.postman_collection.json -e postman/GameVault.postman_environment.json
```

Newman will execute the collection from top to bottom.

You should see a report showing:

```text
requests
assertions
passed tests
failed tests
```

This means we can now test the API without manually clicking through every Postman request.

The process becomes:

```text
start backend
-> run newman
-> collection executes
-> assertions execute
-> terminal reports failures
```

---

# 🔒 19. HTTPS certificates and Newman

GameVault uses a locally generated self-signed HTTPS certificate.

Because the certificate isn't issued by a public certificate authority, Newman may reject it during local development.

For local development only, you can run:

```bash
npx newman run postman/GameVault.postman_collection.json --insecure
```

If you also need the environment:

```bash
npx newman run postman/GameVault.postman_collection.json -e postman/GameVault.postman_environment.json --insecure
```

`--insecure` is appropriate here because we deliberately created a local self-signed certificate.

It is not something you should automatically use against a deployed production application.

---

