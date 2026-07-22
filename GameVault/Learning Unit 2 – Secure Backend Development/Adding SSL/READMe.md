

# 🔐 Adding SSL (HTTPS) to Your GameVault Project

## Background

Web security is non-negotiable in today's development landscape. Whether you're handling login forms, blogs, or API endpoints, HTTPS ensures confidentiality and integrity in your application.

This guide helps you add SSL (more accurately, **TLS**) for **local development** on your backend and frontend projects using a **self-signed certificate**.

---

## 🔎 Research Activity: Why SSL?

Spend 10–15 minutes researching the following:

* What is SSL/TLS, and how do they differ?
* Why is HTTPS more secure than HTTP?
* What are the risks of not using SSL in web applications?
* Have there been any real-world incidents involving missing or misconfigured SSL?

📝 **Write a 4–6 sentence summary** of your findings and commit it as `ssl_research.md` in your repository.

**Useful links:**

* [https://letsencrypt.org/docs/why-https/](https://letsencrypt.org/docs/why-https/)
* [https://developer.mozilla.org/en-US/docs/Web/Security/HTTP\_strict\_transport\_security](https://developer.mozilla.org/en-US/docs/Web/Security/HTTP_strict_transport_security)

---


## ⚙️ Setting Up SSL for Local Development

### Step 1 and 2: Generate the Self-Signed Certificate and Private Key (If not done previously)

Use **Git Bash** in your project root:

```bash
openssl req -x509 -nodes -newkey rsa:2048 -keyout certificates/privatekey.pem -out certificates/certificate.pem -days 365 -subj "/C=ZA/ST=KwaZulu-Natal/L=Durban/O=GameVault/OU=Dev/CN=localhost" -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

✅ You should now see `privatekey.pem` and `certificate.pem` inside `backend/ssl`.

---

### Step 3: Share Certificate with the Frontend

If your frontend is in a separate folder:

1. Create a folder named `ssl` inside your frontend directory.
2. Copy `privatekey.pem` and `certificate.pem` from the backend's `ssl/` folder.

```bash
cp backend/ssl/*.pem frontend/ssl/
```

📝 You can reuse the same certificate for both backend and frontend if served from the same domain (`localhost`).

---

## 🔧 Configuring the Application to Use HTTPS

### Node.js Backend (`server.js`)

Update your `server.js`:

```js
require("dotenv").config();

const express = require("express");

//creates the https server
const https = require("https");
//reads the cert and key files
const fs = require("fs");
//creates relaible path files
const path = require("path");

const app = express();

app.use(express.json());

const HTTPS_PORT = process.env.HTTPS_PORT || 4000;
const APP_NAME = process.env.APP_NAME || "GameVault API";

const sslKeyPath =
    process.env.SSL_KEY_PATH || "certificates/privatekey.pem";

const sslCertPath =
    process.env.SSL_CERT_PATH || "certificates/certificate.pem";

const resolvedKeyPath = path.resolve(__dirname, sslKeyPath);
const resolvedCertPath = path.resolve(__dirname, sslCertPath);

const httpsOptions = {
    key: fs.readFileSync(resolvedKeyPath),
    cert: fs.readFileSync(resolvedCertPath)
};

/*
Temporary in-memory data.

This data will be lost whenever the server restarts.
MongoDB will replace this array in a later learning unit.
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
GET /

Confirms that the API is running.
*/
app.get("/", (req, res) => {
    return res.status(200).json({
        application: APP_NAME,
        message: "Welcome to the GameVault API"
    });
});

/*
GET /about

Returns information about the application.
*/
app.get("/about", (req, res) => {
    return res.status(200).json({
        application: APP_NAME,
        description: "GameVault is a secure video game collection and review platform.",
        currentStage: "Learning Unit 1 - Backend Foundations"
    });
});

/*
GET /health

Provides a basic application health check.
*/
app.get("/health", (req, res) => {
    return res.status(200).json({
        application: APP_NAME,
        status: "OK",
        environment: process.env.NODE_ENV || "development",
        timestamp: new Date().toISOString()
    });
});

/*
GET /games

Returns all games.
*/
app.get("/games", (req, res) => {
    return res.status(200).json({
        count: games.length,
        data: games
    });
});

/*
GET /games/:id

Returns a single game using its ID.
*/
app.get("/games/:id", (req, res) => {

    const gameId = Number(req.params.id);

    if (!Number.isInteger(gameId)) {
        return res.status(400).json({
            error: "Game ID must be a whole number."
        });
    }

    const game = games.find(currentGame => currentGame.id === gameId);

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
POST /games

Validates the request and creates a new game.
*/
app.post("/games", (req, res) => {

    const { title, genre, platform, releaseYear, ageRating } = req.body;

    if (
        !title ||
        !genre ||
        !platform ||
        releaseYear === undefined ||
        !ageRating
    ) {
        return res.status(400).json({
            error: "Title, genre, platform, release year and age rating are required."
        });
    }

    if (
        typeof title !== "string" ||
        typeof genre !== "string" ||
        typeof platform !== "string" ||
        typeof ageRating !== "string"
    ) {
        return res.status(400).json({
            error: "Title, genre, platform and age rating must be text."
        });
    }

    const cleanedTitle = title.trim();
    const cleanedGenre = genre.trim();
    const cleanedPlatform = platform.trim();
    const cleanedAgeRating = ageRating.trim().toUpperCase();

    if (cleanedTitle.length < 2 || cleanedTitle.length > 100) {
        return res.status(400).json({
            error: "Title must contain between 2 and 100 characters."
        });
    }

    if (cleanedGenre.length < 2 || cleanedGenre.length > 50) {
        return res.status(400).json({
            error: "Genre must contain between 2 and 50 characters."
        });
    }

    if (cleanedPlatform.length < 2 || cleanedPlatform.length > 50) {
        return res.status(400).json({
            error: "Platform must contain between 2 and 50 characters."
        });
    }

    const currentYear = new Date().getFullYear();

    if (
        typeof releaseYear !== "number" ||
        !Number.isInteger(releaseYear) ||
        releaseYear < 1950 ||
        releaseYear > currentYear + 2
    ) {
        return res.status(400).json({
            error: `Release year must be a whole number between 1950 and ${currentYear + 2}.`
        });
    }

    const allowedAgeRatings = [
        "E",
        "E10+",
        "T",
        "M",
        "18"
    ];

    if (!allowedAgeRatings.includes(cleanedAgeRating)) {
        return res.status(400).json({
            error: `Age rating must be one of: ${allowedAgeRatings.join(", ")}.`
        });
    }

    const newGame = {
        id: games.length > 0 ? Math.max(...games.map(game => game.id)) + 1 : 1,
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
Handles routes that do not exist.
This middleware must appear after all valid routes.
*/
app.use((req, res) => {

    return res.status(404).json({
        error: "Route not found."
    });

});

/*
Starts the GameVault HTTPS server.
*/
https.createServer(httpsOptions, app).listen(HTTPS_PORT, () => {
    console.log(
        `${APP_NAME} is running securely on https://localhost:${HTTPS_PORT}`
    );
});
```

---


## 🧪 Testing HTTPS Locally

Start your backend:

* Backend: `npm run dev`

Visit:

* [https://localhost:5000](https://localhost:5000) (API)

🔒 You’ll likely see a **browser warning**—this is expected for self-signed certs.

---

## 🛡️ Trusting the Certificate (Windows Only)

To avoid security warnings in browsers:

1. Press `Win + R` → type `certmgr.msc` → Enter
2. Navigate to `Trusted Root Certification Authorities > Certificates`
3. Right-click and choose `All Tasks > Import`
4. Import `ssl/cert.pem`
5. Select **Trusted Root Certification Authorities** as the destination
6. Complete the wizard and restart your browser

✅ Now HTTPS will show a green padlock (trusted cert).

🧪 If you **still see the "Not Secure" warning**, check these:

🔁 Refresh Cache

* Open Dev Tools → Right-click Reload Button → Select **Empty Cache and Hard Reload**

🧼 Clear HSTS Settings (Rare)

If you've visited localhost before with an untrusted cert:

1. Visit: `chrome://net-internals/#hsts`
2. Scroll to **Delete domain security policies**
3. Enter `localhost` and click “Delete”

---

## 📌 Best Practices and Reminders

* ❌ **Never** use self-signed certs in production.
* 🔐 **Do not** commit `key.pem` or `cert.pem`. Add `ssl/` to your `.gitignore`.
* ☁️ Use **Let's Encrypt** or another Certificate Authority (CA) for live production sites.
* ✅ SSL in production is often managed by NGINX, Apache, or your cloud provider (e.g. Vercel, Netlify).

---

## 🧠 Reflection Questions

* Did adding HTTPS change how your app behaves?
* Were there any setup issues or certificate trust problems?
* What would need to change for a production deployment?

---
