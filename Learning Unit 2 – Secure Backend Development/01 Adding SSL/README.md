# 🔐 Adding HTTPS to the GameVault Backend

## Overview

Web applications should protect information while it travels between a client and a server.

GameVault currently communicates using HTTP. In this activity, you will update the backend so that it communicates using HTTPS.

HTTP uses an address such as:

```text
http://localhost:4000
```

HTTPS uses an address such as:

```text
https://localhost:4000
```

HTTPS encrypts communication using TLS. Although the term **SSL** is still commonly used, modern applications use **TLS**.

For local development, GameVault will use a **self-signed certificate**. This certificate encrypts the connection, but browsers and API clients will not automatically trust it because it was not issued by a recognised certificate authority.

Self-signed certificates are suitable for:

* Local development.
* Classroom demonstrations.
* Testing HTTPS configuration.

They must not be used for a public production application.

---

## 📚 Learning Outcomes

By the end of this activity, you should be able to:

* Explain the difference between HTTP and HTTPS.
* Explain the purpose of TLS certificates.
* Install OpenSSL on Windows.
* Generate a private key and self-signed certificate.
* Store certificate paths in environment variables.
* Configure a Node.js and Express application to use HTTPS.
* Test a self-signed certificate in a browser and Postman.
* Protect private keys from being uploaded to GitHub.
* Explain how production HTTPS differs from local HTTPS.

---

# 🔎 Part 1: Research Activity

Spend approximately 10–15 minutes researching the following questions:

* What are SSL and TLS?
* What is the difference between SSL and TLS?
* Why is HTTPS more secure than HTTP?
* What risks arise when a web application does not use HTTPS?
* What can happen when a certificate is expired or incorrectly configured?
* Can you find a real-world security incident involving missing or misconfigured HTTPS?

Write a summary containing **four to six complete sentences**.

Create the following file in the main GameVault folder:

```text
ssl_research.md
```

Your project should resemble:

```text
GameVault
├── ssl_research.md
├── .gitignore
└── backend
```

Commit this file to your repository when the activity is complete.

---

# 🧰 Part 2: Install OpenSSL on Windows

OpenSSL is used to generate the private key and certificate required by the GameVault HTTPS server.

## Option A: Check Whether OpenSSL Is Already Available

Before installing anything, open Command Prompt, PowerShell, Git Bash or the Visual Studio Code terminal.

Run:

```bash
openssl version
```

If OpenSSL is available, a version number should appear.

For example:

```text
OpenSSL 3.x.x
```

You may continue to the certificate-generation section if the command works.

If Windows displays an error such as:

```text
'openssl' is not recognized as an internal or external command
```

OpenSSL is either not installed or its installation folder has not been added to the Windows `Path`.

---

## Option B: Try Git Bash

Git for Windows commonly provides OpenSSL through Git Bash.

To test this option:

1. Open the GameVault project folder in File Explorer.
2. Right-click inside the folder.
3. Select **Open Git Bash here**.
4. Run:

```bash
openssl version
```

If a version number appears, you may use Git Bash for the remaining OpenSSL commands.

You do not need to install a second copy of OpenSSL when it already works in Git Bash.

---

## Option C: Install OpenSSL Using a Windows EXE Installer

When OpenSSL is unavailable, download an approved 64-bit Windows OpenSSL installer.

Most modern Windows 10 and Windows 11 computers use a 64-bit operating system. Select a **Win64 OpenSSL** installer unless your lecturer or system administrator instructs you otherwise.

During installation:

1. Run the downloaded `.exe` installer.
2. Approve the Windows security prompt.
3. Accept the licence agreement.
4. Keep the default installation folder where possible.

A common installation location is:

```text
C:\Program Files\OpenSSL-Win64
```

5. When asked where OpenSSL DLL files should be stored, select the OpenSSL binaries directory when that option is available.
6. Complete the installation.

Do not download OpenSSL installers from unknown websites.

---

## Add OpenSSL to the Windows Path

OpenSSL may be installed successfully but still be unavailable in the terminal.

To add it to the Windows `Path`:

1. Open the Windows Start menu.
2. Search for:

```text
environment variables
```

3. Select **Edit the system environment variables**.
4. Select **Environment Variables**.
5. Under either **User variables** or **System variables**, select `Path`.
6. Select **Edit**.
7. Select **New**.
8. Add the OpenSSL `bin` folder.

For example:

```text
C:\Program Files\OpenSSL-Win64\bin
```

9. Select **OK** on every open window.
10. Close Visual Studio Code and all open terminals.
11. Reopen Visual Studio Code.

Add the folder containing `openssl.exe`, not the executable itself.

Correct:

```text
C:\Program Files\OpenSSL-Win64\bin
```

Incorrect:

```text
C:\Program Files\OpenSSL-Win64\bin\openssl.exe
```

---

## Verify the Installation

Open a new terminal and run:

```bash
openssl version
```

You should see the installed OpenSSL version.

You can also check where Windows found the executable:

```cmd
where.exe openssl
```

The result may resemble:

```text
C:\Program Files\OpenSSL-Win64\bin\openssl.exe
```

If the normal command still does not work, test the executable directly in PowerShell:

```powershell
& "C:\Program Files\OpenSSL-Win64\bin\openssl.exe" version
```

If this command works, OpenSSL is installed correctly, but the Windows `Path` still needs to be corrected.

---

# 📁 Part 3: Prepare the Certificate Folder

Your GameVault project should already contain a `backend` folder.

Move into it:

```bash
cd backend
```

Confirm that the terminal path ends with:

```text
GameVault\backend
```

Create a folder named:

```text
certificates
```

Using the terminal:

```bash
mkdir certificates
```

Your project should now resemble:

```text
GameVault
├── .gitignore
├── ssl_research.md
└── backend
    ├── certificates
    ├── node_modules
    ├── .env
    ├── .env.example
    ├── package-lock.json
    ├── package.json
    └── server.js
```

All certificate-generation commands must be run from:

```text
GameVault/backend
```

---

# 🔑 Part 4: Generate the Private Key and Certificate

Run the following command as one complete line:

```bash
openssl req -x509 -nodes -newkey rsa:2048 -sha256 -keyout certificates/privatekey.pem -out certificates/certificate.pem -days 365 -subj "/C=ZA/ST=KwaZulu-Natal/L=Durban/O=GameVault/OU=Development/CN=localhost" -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

This command creates:

```text
backend/certificates/privatekey.pem
backend/certificates/certificate.pem
```

The project should now resemble:

```text
GameVault
└── backend
    ├── certificates
    │   ├── certificate.pem
    │   └── privatekey.pem
    ├── node_modules
    ├── .env
    ├── .env.example
    ├── package-lock.json
    ├── package.json
    └── server.js
```

## Understanding the Command

The command uses the following options:

* `req` starts the certificate-generation process.
* `-x509` creates a self-signed certificate.
* `-nodes` creates a private key without a passphrase.
* `-newkey rsa:2048` generates a new 2048-bit RSA key.
* `-sha256` signs the certificate using SHA-256.
* `-keyout` specifies the private-key filename.
* `-out` specifies the certificate filename.
* `-days 365` makes the certificate valid for 365 days.
* `CN=localhost` identifies the certificate as belonging to localhost.
* `subjectAltName` allows the certificate to work with both `localhost` and `127.0.0.1`.

The private key must remain private.

The certificate may be shared with clients that need to trust the local server, but it must never be confused with the private key.

---

## Confirm That the Files Were Created

Run:

```bash
dir certificates
```

In Git Bash, you may use:

```bash
ls certificates
```

You should see:

```text
certificate.pem
privatekey.pem
```

Inspect the certificate using:

```bash
openssl x509 -in certificates/certificate.pem -text -noout
```

Confirm that the output includes:

```text
CN = localhost
```

It should also include a Subject Alternative Name section containing:

```text
DNS:localhost
IP Address:127.0.0.1
```

---

# 🚫 Part 5: Protect the Certificate Files

The private key must not be committed to GitHub.

Open the `.gitignore` file located in the main GameVault folder:

```text
GameVault/.gitignore
```

Ensure that it contains:

```gitignore
backend/node_modules/
backend/.env
backend/certificates/

frontend/node_modules/
frontend/.env
```

Ignoring the entire certificate folder prevents both local certificate files from being uploaded.

The following files must remain local:

```text
backend/certificates/privatekey.pem
backend/certificates/certificate.pem
```

The private key is the most sensitive file. Anyone who obtains it may be able to impersonate the server using that certificate.

If a private key is accidentally uploaded to GitHub, remove it from the repository and generate a new key and certificate.

---

# 🌍 Part 6: Update the Environment Variables

Open:

```text
backend/.env
```

Add or update the following values:

```env
HTTPS_PORT=4000
APP_NAME=GameVault API
NODE_ENV=development
SSL_KEY_PATH=certificates/privatekey.pem
SSL_CERT_PATH=certificates/certificate.pem
```

## Environment Variable Purposes

`HTTPS_PORT` specifies the port used by the secure server:

```text
4000
```

`APP_NAME` stores the application name:

```text
GameVault API
```

`NODE_ENV` identifies the current environment:

```text
development
```

`SSL_KEY_PATH` identifies the location of the private key:

```text
certificates/privatekey.pem
```

`SSL_CERT_PATH` identifies the location of the certificate:

```text
certificates/certificate.pem
```

The `.env` file must not be uploaded to GitHub.

---

## Update .env.example

Open:

```text
backend/.env.example
```

Add the same variable names:

```env
HTTPS_PORT=4000
APP_NAME=GameVault API
NODE_ENV=development
SSL_KEY_PATH=certificates/privatekey.pem
SSL_CERT_PATH=certificates/certificate.pem
```

The `.env.example` file should be committed because it shows other developers which variables are required.

It does not contain the certificate or private key itself.

---

# 🔧 Part 7: Configure the GameVault Backend

Open:

```text
backend/server.js
```

Use the following complete implementation:

```javascript
/*
Loads the values stored in the .env file.

This must run before the environment variables are accessed.
*/
require("dotenv").config();

const express = require("express");

/*
The https module creates the secure HTTPS server.

This is a built-in Node.js module and does not need to be
installed separately.
*/
const https = require("https");

/*
The fs module reads the private key and certificate files.

This is also a built-in Node.js module.
*/
const fs = require("fs");

/*
The path module creates reliable file paths.

It allows the application to locate the certificate files even
when the server is started from a different terminal location.
*/
const path = require("path");

/*
Creates the Express application.
*/
const app = express();

/*
Enables the application to read JSON request bodies.
*/
app.use(express.json());

/*
Reads the port and application name from the environment
variables.

Fallback values are used when the environment variables are
not available.
*/
const HTTPS_PORT = process.env.HTTPS_PORT || 4000;
const APP_NAME = process.env.APP_NAME || "GameVault API";

/*
Reads the relative private-key and certificate paths from the
environment variables.
*/
const sslKeyPath =
    process.env.SSL_KEY_PATH ||
    "certificates/privatekey.pem";

const sslCertPath =
    process.env.SSL_CERT_PATH ||
    "certificates/certificate.pem";

/*
Converts the relative paths into complete paths.

__dirname refers to the folder containing server.js.
*/
const resolvedKeyPath = path.resolve(
    __dirname,
    sslKeyPath
);

const resolvedCertPath = path.resolve(
    __dirname,
    sslCertPath
);

/*
Reads the private key and certificate.

Both files are required to create the HTTPS server.
*/
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
        description:
            "GameVault is a secure video game collection and review platform.",
        currentStage:
            "Learning Unit 1 - Backend Foundations"
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
        protocol: "HTTPS",
        environment:
            process.env.NODE_ENV || "development",
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

Returns one game using its ID.
*/
app.get("/games/:id", (req, res) => {
    const gameId = Number(req.params.id);

    /*
    Route parameters arrive as text.

    The converted ID must be a whole number.
    */
    if (!Number.isInteger(gameId)) {
        return res.status(400).json({
            error: "Game ID must be a whole number."
        });
    }

    /*
    Searches the temporary games array for a matching ID.
    */
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
POST /games

Validates the request and creates a new game.
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
    Confirms that every required field was supplied.
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
    Confirms that the expected text fields contain strings.
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
    Removes unnecessary spaces and standardises the age rating.
    */
    const cleanedTitle = title.trim();
    const cleanedGenre = genre.trim();
    const cleanedPlatform = platform.trim();
    const cleanedAgeRating =
        ageRating.trim().toUpperCase();

    if (
        cleanedTitle.length < 2 ||
        cleanedTitle.length > 100
    ) {
        return res.status(400).json({
            error:
                "Title must contain between 2 and 100 characters."
        });
    }

    if (
        cleanedGenre.length < 2 ||
        cleanedGenre.length > 50
    ) {
        return res.status(400).json({
            error:
                "Genre must contain between 2 and 50 characters."
        });
    }

    if (
        cleanedPlatform.length < 2 ||
        cleanedPlatform.length > 50
    ) {
        return res.status(400).json({
            error:
                "Platform must contain between 2 and 50 characters."
        });
    }

    const currentYear = new Date().getFullYear();

    /*
    Confirms that the release year is a valid whole number.
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

    if (
        !allowedAgeRatings.includes(cleanedAgeRating)
    ) {
        return res.status(400).json({
            error:
                `Age rating must be one of: ${allowedAgeRatings.join(", ")}.`
        });
    }

    /*
    Generates the next available ID.
    */
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
Handles requests that do not match any valid route.

This middleware must appear after all valid routes.
*/
app.use((req, res) => {
    return res.status(404).json({
        error: "Route not found."
    });
});

/*
Creates and starts the secure HTTPS server.

app.listen() would create a normal HTTP server.

https.createServer() creates an HTTPS server using the private
key, certificate and Express application.
*/
https
    .createServer(httpsOptions, app)
    .listen(HTTPS_PORT, () => {
        console.log(
            `${APP_NAME} is running securely on https://localhost:${HTTPS_PORT}`
        );
    });
```

The `https`, `fs` and `path` modules are built into Node.js.

Do not install them using npm.

---

# ▶️ Part 8: Start the HTTPS Server

Stop any currently running server:

```text
Ctrl + C
```

Confirm that the terminal is inside:

```text
GameVault/backend
```

Start the development server:

```bash
npm run dev
```

The terminal should display:

```text
GameVault API is running securely on https://localhost:4000
```

Open:

```text
https://localhost:4000
```

Do not use:

```text
http://localhost:4000
```

The browser may display a certificate warning. This is expected because the certificate is self-signed.

Only continue past the warning when the address is your own local GameVault server.

---

# 🧪 Part 9: Test HTTPS in the Browser

Test the following addresses:

```text
https://localhost:4000/
```

```text
https://localhost:4000/about
```

```text
https://localhost:4000/health
```

```text
https://localhost:4000/games
```

The health-check response should identify the protocol as HTTPS.

A browser warning does not necessarily mean that encryption failed. It means that the browser does not recognise the organisation that issued the certificate.

In this case, the certificate was issued by you rather than by a trusted certificate authority.

---

# 📬 Part 10: Test HTTPS in Postman

Open the GameVault Postman collection.

Change each request URL from:

```text
http://localhost:4000
```

to:

```text
https://localhost:4000
```

For example:

```text
GET https://localhost:4000/games
```

Postman may display an error because the certificate is self-signed.

For local development:

1. Open the request in Postman.
2. Select the request’s **Settings** tab.
3. Locate **Enable SSL certificate verification**.
4. Turn it off.
5. Send the request again.

This setting should only be disabled for the local self-signed GameVault server.

Do not disable certificate verification when communicating with production APIs.

Test:

```text
GET https://localhost:4000/
```

```text
GET https://localhost:4000/about
```

```text
GET https://localhost:4000/health
```

```text
GET https://localhost:4000/games
```

```text
GET https://localhost:4000/games/1
```

```text
POST https://localhost:4000/games
```

Confirm that HTTPS does not change the intended API behaviour.

The routes, request bodies, status codes and JSON responses should continue to work as before.

---

# 🛡️ Part 11: Trust the Certificate on Windows

This step is optional.

Trusting a self-signed certificate may remove the browser warning on the computer where it is installed.

Only install and trust a certificate that you personally generated and understand.

Do not install unknown certificates into the trusted root certificate store.

To import the GameVault certificate:

1. Press `Win + R`.
2. Enter:

```text
certmgr.msc
```

3. Press Enter.
4. Open **Trusted Root Certification Authorities**.
5. Open **Certificates**.
6. Right-click the certificate list.
7. Select **All Tasks**.
8. Select **Import**.
9. Browse to:

```text
GameVault\backend\certificates\certificate.pem
```

10. Follow the certificate import wizard.
11. Select **Trusted Root Certification Authorities** as the certificate store.
12. Complete the import.
13. Close and reopen the browser.

Some Windows tools may not display `.pem` files by default. Change the file-selection filter to show all files where necessary.

Installing the certificate may require administrator permission.

A modern browser may show a secure connection indicator rather than a green padlock.

---

## Remove the Certificate Later

When the local certificate is no longer required:

1. Open:

```text
certmgr.msc
```

2. Open **Trusted Root Certification Authorities**.
3. Open **Certificates**.
4. Locate the certificate issued to `localhost`.
5. Verify that it is the GameVault development certificate.
6. Delete it.

Do not delete certificates that you cannot identify.

---

# 🛠️ Part 12: Troubleshooting

## OpenSSL is not recognised

Try using Git Bash:

```bash
openssl version
```

If OpenSSL was installed using an EXE installer, confirm that its `bin` folder was added to the Windows `Path`.

A common path is:

```text
C:\Program Files\OpenSSL-Win64\bin
```

Close and reopen all terminals after updating the `Path`.

---

## The certificate command rejects `-addext`

Check the OpenSSL version:

```bash
openssl version
```

An older OpenSSL version may not support this option.

Use a current OpenSSL installation where possible.

---

## The certificates folder does not exist

Confirm that you are inside:

```text
GameVault/backend
```

Then run:

```bash
mkdir certificates
```

Run the certificate-generation command again.

---

## The server reports ENOENT

An `ENOENT` error means that Node.js could not locate one of the files.

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

Check the spelling and capitalisation carefully.

---

## The server reports EADDRINUSE

Another application is already using port `4000`.

Stop the existing server:

```text
Ctrl + C
```

Then run:

```bash
npm run dev
```

Alternatively, change the port in `.env`:

```env
HTTPS_PORT=5000
```

The new address will be:

```text
https://localhost:5000
```

---

## The browser displays a security warning

This is expected for a self-signed certificate.

Confirm that the address is:

```text
https://localhost:4000
```

Only bypass the warning for your own local GameVault application.

---

## Postman rejects the certificate

Disable SSL certificate verification for the local Postman request.

Do not disable it for public or production APIs.

---

## HTTP worked before, but the server no longer responds

The application now uses HTTPS.

Use:

```text
https://localhost:4000
```

Do not use:

```text
http://localhost:4000
```

---

## The browser still warns after importing the certificate

Confirm that:

* The imported file was `certificate.pem`.
* The certificate was placed in the trusted root store.
* The browser was fully closed and reopened.
* The URL uses `localhost`.
* The certificate has not expired.
* The certificate contains `localhost` as a Subject Alternative Name.

Do not import:

```text
privatekey.pem
```

The private key should never be added to the certificate store or shared.

---

# 📌 Part 13: Security Best Practices

Remember the following:

* Never use a self-signed certificate for a public production application.
* Never upload `privatekey.pem` to GitHub.
* Do not email, share or distribute the private key.
* Add the entire local certificate folder to `.gitignore`.
* Generate separate certificates where appropriate.
* Replace certificates before they expire.
* Do not ignore certificate warnings on unknown websites.
* Do not disable certificate verification for production services.
* Use certificates issued by a trusted certificate authority in production.
* Production HTTPS may be managed by a cloud platform, reverse proxy or hosting provider.

The certificate used in this activity proves that the application can be configured to use encrypted communication. It does not provide the same identity verification as a certificate issued by a trusted certificate authority.

---

# ☁️ Part 14: Commit the HTTPS Changes

Check the repository:

```bash
git status
```

Confirm that the following are not listed as files waiting to be committed:

```text
backend/.env
backend/certificates/privatekey.pem
backend/certificates/certificate.pem
backend/node_modules/
```

The following files may be committed:

```text
.gitignore
ssl_research.md
backend/.env.example
backend/server.js
```

Stage the changes:

```bash
git add .
```

Review them again:

```bash
git status
```

Commit the changes:

```bash
git commit -m "Add HTTPS support to GameVault backend"
```

Push the project:

```bash
git push
```

After pushing, inspect the GitHub repository and confirm that the certificate folder and `.env` file are absent.

---

# 🧠 Part 15: Reflection Questions

Answer the following questions:

1. What is the difference between HTTP and HTTPS?
2. What role does the private key play in the HTTPS configuration?
3. What information is stored in the certificate?
4. Why does the browser warn users about self-signed certificates?
5. Did HTTPS change the API routes or only the connection protocol?
6. Why must `privatekey.pem` not be committed to GitHub?
7. Why is disabling certificate verification acceptable only for controlled local testing?
8. What would need to change before GameVault could use HTTPS in production?

Add your answers to:

```text
ssl_research.md
```

---

# ✅ Completion Checklist

Before considering the HTTPS activity complete, confirm that:

* OpenSSL is installed or available through Git Bash.
* `openssl version` works.
* The terminal is inside `GameVault/backend`.
* The `certificates` folder exists.
* `privatekey.pem` has been generated.
* `certificate.pem` has been generated.
* The certificate supports `localhost`.
* The certificate supports `127.0.0.1`.
* The certificate folder is ignored by Git.
* The `.env` file contains the certificate paths.
* The `.env.example` file contains the required variable names.
* The Node.js HTTPS module has been imported.
* The private key and certificate are read successfully.
* The server starts using HTTPS.
* The root endpoint works through HTTPS.
* The health endpoint works through HTTPS.
* The games endpoints work through HTTPS.
* Postman can connect to the HTTPS server.
* The private key is not stored on GitHub.
* The research activity has been completed.
* The reflection questions have been answered.
* The changes have been committed and pushed.
