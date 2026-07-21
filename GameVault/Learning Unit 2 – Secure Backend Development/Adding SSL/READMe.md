

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
const https = require('https');
const fs = require('fs');
const app = require('./app'); // Your Express app
require('dotenv').config();

const PORT = process.env.PORT || 5000;

const options = {
  key: fs.readFileSync('ssl/privatekey.pem'),
  cert: fs.readFileSync('ssl/certificate.pem'),
};

https.createServer(options, app).listen(PORT, () => {
  console.log(`Secure API running at https://localhost:${PORT}`);
});
```

---

### Vite Frontend (`vite.config.js`)

Update your Vite config to enable HTTPS:

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import fs from 'fs'

export default defineConfig({
  plugins: [react()],
  server: {
    https: {
      key: fs.readFileSync('ssl/privatekey.pem'),
      cert: fs.readFileSync('ssl/certificate.pem'),
    },
    port: 5173
  }
});
```

---

## 🧪 Testing HTTPS Locally

Start your backend and frontend:

* Backend: `npm run dev`
* Frontend: `npm run dev` (or `vite`)

Visit:

* [https://localhost:5000](https://localhost:5000) (API)
* [https://localhost:5173](https://localhost:5173) (Frontend)

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
