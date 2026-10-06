# 🐳 GameVault — Docker & Docker Compose

In this section, you will containerise the GameVault application using Docker.

At this point, GameVault consists of multiple parts:

```text
GameVault
├── backend
├── frontend
└── postman
```

The backend uses Node.js/Express and connects to MongoDB, while the frontend uses React with Vite.

Normally, another developer who clones your project has to install Node.js, install the correct dependencies, configure environment variables, make sure MongoDB is available and then start the frontend and backend separately.

Docker helps us create a more consistent environment.

Our final setup will look like:

```text
browser
-> frontend container
-> backend container
-> mongodb container
```

We will first containerise the applications individually and then use Docker Compose to run the complete GameVault environment together.

---

# 🐳 Step 01 — Understanding Docker

## 1. What is Docker?

Docker allows an application and the environment it needs to run to be packaged into a container.

There are two important terms you need to understand.

### Docker image

An image is the set of instructions used to create a container.

Think of it as the packaged template for the application.

For example:

```text
node image
-> copy GameVault backend
-> install dependencies
-> start server
```

### Docker container

A container is a running instance of an image.

The relationship is:

```text
Dockerfile
-> build image
-> create container
-> application runs
```

The Dockerfile tells Docker how to build the image.

---

# 💻 2. Install Docker Desktop

Install Docker Desktop for your operating system.

After installation, open Docker Desktop and wait for the Docker engine to start.

Open a terminal and run:

```bash
docker --version
```

You should receive a Docker version.

Then run:

```bash
docker compose version
```

You should also receive a Docker Compose version.

If either command is not recognised, restart your terminal after installing Docker Desktop.

On Windows, Docker Desktop may ask you to enable WSL 2. Follow the Docker Desktop setup prompts and restart the computer if required.

---

# 📁 3. Check your GameVault structure

Before creating Docker files, your project should roughly contain:

```text
GameVault
│
├── backend
│   ├── certificates
│   ├── config
│   ├── controllers
│   ├── middleware
│   ├── models
│   ├── routes
│   ├── tests
│   ├── utils
│   ├── .env
│   ├── app.js
│   ├── package.json
│   └── server.js
│
├── frontend
│   ├── src
│   ├── package.json
│   └── vite.config.js
│
└── postman
```

Your exact files may differ slightly depending on your implementation.

Do not restructure a working project just to make it identical to this example.

---

# 📦 Step 02 — Dockerise the Backend

## 4. Create the backend Dockerfile

Inside:

```text
GameVault/backend
```

create a file named exactly:

```text
Dockerfile
```

Do not name it:

```text
Dockerfile.txt
```

Add:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 4000

CMD ["npm", "start"]
```

If your backend uses a different port, obviously replace `4000` with the port your application actually uses.

---

# 🔍 5. Understand the Dockerfile

This:

```dockerfile
FROM node:22-alpine
```

tells Docker which base image to use.

GameVault requires Node.js, so instead of manually installing Node inside the container, we start from an existing Node image.

`alpine` is a lightweight Linux-based image.

---

This:

```dockerfile
WORKDIR /app
```

creates and selects:

```text
/app
```

inside the container.

The remaining commands will run from there.

---

Next:

```dockerfile
COPY package*.json ./
```

copies:

```text
package.json
package-lock.json
```

into the container.

Then:

```dockerfile
RUN npm ci
```

installs the exact dependency versions from `package-lock.json`.

For a project with a valid lock file, `npm ci` is preferable for a reproducible Docker build.

---

Then:

```dockerfile
COPY . .
```

copies the rest of the backend into the image.

The sequence is deliberately:

```text
copy package files
-> install dependencies
-> copy application
```

Docker can cache build steps. If you change one controller but don't change `package.json`, Docker may be able to reuse the dependency installation layer rather than reinstalling everything.

---

Finally:

```dockerfile
EXPOSE 4000
```

documents the port used by the application.

And:

```dockerfile
CMD ["npm", "start"]
```

specifies what should run when the container starts.

Make sure your `package.json` actually contains a `start` script.

For example:

```json
"scripts": {
    "start": "node server.js"
}
```

---

# 🚫 6. Create `.dockerignore`

Inside:

```text
GameVault/backend
```

create:

```text
.dockerignore
```

Add:

```text
node_modules
npm-debug.log
.git
.gitignore
.env
coverage
tests
```

This prevents unnecessary files from being copied into the Docker image.

The important one is:

```text
node_modules
```

We do not want Docker copying your Windows `node_modules` folder into a Linux container.

Docker should install its own dependencies using:

```text
package.json
-> package-lock.json
-> npm ci
```

Also notice that `.env` is ignored.

Secrets should not be permanently baked into a Docker image.

---

# 🔨 7. Build the backend image

Open a terminal inside:

```text
GameVault/backend
```

Run:

```bash
docker build -t gamevault-backend .
```

The final `.` is important.

It tells Docker:

```text
use the current directory as the build context
```

Docker will:

```text
read Dockerfile
-> download Node image if necessary
-> create /app
-> copy package files
-> install dependencies
-> copy backend
-> create GameVault backend image
```

When it finishes, run:

```bash
docker images
```

You should see:

```text
gamevault-backend
```

---

# ▶️ 8. Run the backend container

You could run the image manually with:

```bash
docker run --name gamevault-backend -p 4000:4000 gamevault-backend
```

However, GameVault also requires environment variables and MongoDB, so we will shortly use Docker Compose instead.

The important thing to understand is the port syntax:

```text
4000:4000
```

means:

```text
host port : container port
```

Therefore:

```text
localhost:4000
-> container port 4000
-> GameVault backend
```

---

# 🔐 9. A note about GameVault HTTPS

Your existing GameVault backend uses HTTPS with local certificates.

That creates an extra consideration when using Docker.

If your `server.js` expects files such as:

```text
certificates/privatekey.pem
certificates/certificate.pem
```

those files must exist inside the container at the paths expected by the application.

Do not solve this by committing private keys to GitHub.

For the initial Docker setup, you can mount the local certificate directory into the container through Docker Compose.

We will do that shortly.

---

# ⚛️ Step 03 — Dockerise the React Frontend

## 10. Create the frontend Dockerfile

Go to:

```text
GameVault/frontend
```

Create:

```text
Dockerfile
```

For our development container, use:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

---

# 🌐 11. Why do we use `--host 0.0.0.0`?

Normally Vite may listen in a way that works perfectly when you run it directly on your computer.

Inside Docker, we need Vite to accept connections coming from outside its container.

Therefore:

```text
npm run dev
```

becomes:

```text
npm run dev -- --host 0.0.0.0
```

The browser will still access the application using something like:

```text
http://localhost:5173
```

You are not expected to type `0.0.0.0` into the browser.

---

# 🚫 12. Add the frontend `.dockerignore`

Inside:

```text
GameVault/frontend
```

create:

```text
.dockerignore
```

Add:

```text
node_modules
dist
coverage
.git
.gitignore
.env
```

Again, this keeps unnecessary local files out of the image.

---

# 🔨 13. Build the frontend image

From:

```text
GameVault/frontend
```

run:

```bash
docker build -t gamevault-frontend .
```

Then:

```bash
docker images
```

You should now have at least:

```text
gamevault-backend
gamevault-frontend
```

At this stage we *could* manually start every container.

But once an application has:

```text
frontend
backend
database
```

manually managing each container becomes inconvenient.

This is where Docker Compose becomes useful.

---

# 🧩 Step 04 — Docker Compose

## 14. What is Docker Compose?

Docker Compose allows multiple related containers to be defined in one file.

Instead of manually doing:

```text
start MongoDB
-> start backend
-> start frontend
-> connect everything
```

we define the services once and then use:

```bash
docker compose up
```

Our GameVault Compose environment will contain:

```text
gamevault-frontend
-> gamevault-backend
-> gamevault-mongodb
```

---

# 📄 15. Create `compose.yaml`

Return to the main:

```text
GameVault
```

folder.

Create:

```text
compose.yaml
```

Your structure should now include:

```text
GameVault
├── backend
│   ├── Dockerfile
│   └── .dockerignore
│
├── frontend
│   ├── Dockerfile
│   └── .dockerignore
│
├── postman
│
└── compose.yaml
```

---

# 🍃 16. Add MongoDB

Start the Compose file with:

```yaml
services:

  mongodb:
    image: mongo:8
    container_name: gamevault-mongodb

    ports:
      - "27017:27017"

    volumes:
      - gamevault-mongo-data:/data/db
```

Notice that MongoDB doesn't need a Dockerfile.

We are using the official MongoDB image.

This:

```yaml
volumes:
  - gamevault-mongo-data:/data/db
```

is very important.

Containers are disposable.

If we simply delete a MongoDB container, we don't want all our GameVault data disappearing with it.

A Docker volume allows the database files to live separately from the container.

The relationship becomes:

```text
mongodb container
-> /data/db
-> gamevault-mongo-data volume
```

---

# 🗄️ 17. Add the backend service

Below MongoDB, add:

```yaml
  backend:
    build:
      context: ./backend

    container_name: gamevault-backend

    ports:
      - "4000:4000"

    environment:
      NODE_ENV: development
      HTTPS_PORT: 4000
      MONGODB_URI: mongodb://mongodb:27017/gamevault
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRES_IN: 1h
      BCRYPT_ROUNDS: 12

    depends_on:
      - mongodb
```

Adjust variable names if your existing GameVault backend uses different names.

Do not rename variables in working application code simply to match this example.

---

# 🔗 18. Understand the MongoDB connection

This is extremely important.

Outside Docker, you may have used:

```text
mongodb://localhost:27017/gamevault
```

Inside the backend container, localhost does not mean your computer.

It means:

```text
this backend container
```

MongoDB is running in a different container.

Docker Compose gives services their own internal names.

Because our MongoDB service is named:

```yaml
mongodb:
```

the backend can connect using:

```text
mongodb://mongodb:27017/gamevault
```

The flow is:

```text
backend container
-> hostname mongodb
-> mongodb container
-> port 27017
```

This is one of the most common mistakes when students first use Docker Compose.

---

# 🔑 19. Keep the JWT secret outside `compose.yaml`

Notice:

```yaml
JWT_SECRET: ${JWT_SECRET}
```

We are not writing the actual JWT secret directly into `compose.yaml`.

At the root of GameVault, create:

```text
.env
```

For example:

```env
JWT_SECRET=put_your_long_random_secret_here
```

Use a genuinely random secret rather than literally using that value.

Your root `.gitignore` should contain:

```text
.env
```

You can provide:

```text
.env.example
```

with:

```env
JWT_SECRET=<generate-a-long-random-secret>
```

The flow becomes:

```text
root .env
-> Docker Compose
-> backend container
-> process.env.JWT_SECRET
```

This avoids hard-coding the secret into the Compose file.

---

# 🔐 20. Mount the HTTPS certificates

If your backend still expects your local GameVault certificate directory, add a volume to the backend service.

For example:

```yaml
    volumes:
      - ./backend/certificates:/app/certificates:ro
```

`ro` means:

```text
read only
```

The container can read the certificate files but cannot modify them.

The backend section would therefore contain:

```yaml
  backend:
    build:
      context: ./backend

    container_name: gamevault-backend

    ports:
      - "4000:4000"

    environment:
      NODE_ENV: development
      HTTPS_PORT: 4000
      MONGODB_URI: mongodb://mongodb:27017/gamevault
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRES_IN: 1h
      BCRYPT_ROUNDS: 12

    volumes:
      - ./backend/certificates:/app/certificates:ro

    depends_on:
      - mongodb
```

If your certificate directory has a different name, use the path from your own project.

Also make sure the environment variables used by `httpsConfig.js` point to the same location expected inside the container.

---

# ⚛️ 21. Add the frontend service

Now add:

```yaml
  frontend:
    build:
      context: ./frontend

    container_name: gamevault-frontend

    ports:
      - "5173:5173"

    depends_on:
      - backend
```

Your browser can then access:

```text
http://localhost:5173
```

---

# ⚠️ 22. Understand frontend URLs inside Docker

This part causes a lot of confusion.

Suppose the React frontend contains:

```javascript
const API_BASE_URL = "https://localhost:4000";
```

Should you automatically change that to:

```text
https://backend:4000
```

No.

Why?

Because your React code runs in the user's browser, not inside the frontend container after the page has loaded.

The browser understands:

```text
localhost:4000
```

because Docker publishes the backend to your computer.

The browser generally does not know what:

```text
backend
```

means.

`backend` is the Docker Compose service name used for communication between containers.

So:

```text
backend container -> mongodb:27017
```

works because both are inside Docker's network.

But the browser should continue using:

```text
https://localhost:4000
```

for your current local development setup.

---

# 📄 23. Complete `compose.yaml`

A simplified complete version will look similar to:

```yaml
services:

  mongodb:
    image: mongo:8
    container_name: gamevault-mongodb

    ports:
      - "27017:27017"

    volumes:
      - gamevault-mongo-data:/data/db


  backend:
    build:
      context: ./backend

    container_name: gamevault-backend

    ports:
      - "4000:4000"

    environment:
      NODE_ENV: development
      HTTPS_PORT: 4000
      MONGODB_URI: mongodb://mongodb:27017/gamevault
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRES_IN: 1h
      BCRYPT_ROUNDS: 12

    volumes:
      - ./backend/certificates:/app/certificates:ro

    depends_on:
      - mongodb


  frontend:
    build:
      context: ./frontend

    container_name: gamevault-frontend

    ports:
      - "5173:5173"

    depends_on:
      - backend


volumes:
  gamevault-mongo-data:
```

This is a starting point.

You may need to change:

```text
port numbers
environment variable names
certificate paths
MongoDB database name
```

to match your GameVault implementation.

---

# ▶️ Step 05 — Run GameVault with Docker Compose

## 24. Build the complete application

Open a terminal from:

```text
GameVault
```

Run:

```bash
docker compose build
```

Docker Compose will read:

```text
compose.yaml
```

and build the services that have a `build` section.

Then start everything:

```bash
docker compose up
```

The flow is now:

```text
Docker Compose
-> MongoDB container
-> backend container
-> frontend container
```

The terminal will display logs from the different services.

---

# 🌐 25. Test the application

Once everything has started, test the frontend:

```text
http://localhost:5173
```

Then test the backend.

Because GameVault uses HTTPS:

```text
https://localhost:4000
```

Try one of your existing public endpoints, for example:

```text
https://localhost:4000/games
```

or your health route.

You may receive a browser warning because GameVault uses a self-signed development certificate.

That is expected for your local certificate.

Then test the complete flow:

```text
open frontend
-> register/login
-> frontend sends request
-> backend processes request
-> backend connects to mongodb
-> response returned
-> frontend updates
```

---

# 📋 26. View running containers

Open another terminal and run:

```bash
docker ps
```

You should see containers similar to:

```text
gamevault-frontend
gamevault-backend
gamevault-mongodb
```

You can also view them visually inside Docker Desktop.

---

# 📜 27. View logs

To see logs from all services:

```bash
docker compose logs
```

To continuously follow them:

```bash
docker compose logs -f
```

For only the backend:

```bash
docker compose logs backend
```

For MongoDB:

```bash
docker compose logs mongodb
```

For the frontend:

```bash
docker compose logs frontend
```

This is extremely useful when debugging.

For example:

```text
frontend not loading games
-> inspect frontend logs
-> inspect backend logs
-> check whether backend received request
-> inspect mongodb logs if database connection failed
```

---

# ⏹️ 28. Stop the application

If `docker compose up` is running directly in your terminal, you can usually stop it using:

```text
Ctrl + C
```

Then run:

```bash
docker compose down
```

This stops and removes the containers and Compose network.

Your MongoDB volume remains.

That means:

```text
docker compose down
-> containers removed
-> database volume remains
-> data remains
```

---

# 💾 29. Understand MongoDB persistence

Start GameVault:

```bash
docker compose up
```

Register a user or add some GameVault data.

Then:

```bash
docker compose down
```

Start it again:

```bash
docker compose up
```

The data should still exist because of:

```yaml
volumes:
  - gamevault-mongo-data:/data/db
```

You can inspect volumes with:

```bash
docker volume ls
```

---

# ⚠️ 30. Be careful with `down -v`

You may see this command online:

```bash
docker compose down -v
```

The `-v` matters.

It tells Docker to remove the Compose volumes as well.

For our MongoDB container, this can remove the stored GameVault database data.

So understand the difference:

```text
docker compose down
-> remove containers
-> keep database volume
```

```text
docker compose down -v
-> remove containers
-> remove volumes
-> database data may be deleted
```

Do not randomly use `-v` when you want to keep your local data.

---

# 🔄 31. Rebuild after changing dependencies

Suppose you run:

```bash
npm install some-new-package
```

and `package.json` changes.

Your existing Docker image doesn't magically receive that new dependency.

Rebuild:

```bash
docker compose build
```

Then:

```bash
docker compose up
```

You can also do both with:

```bash
docker compose up --build
```

This is a useful command while developing:

```text
change Dockerfile/dependencies
-> docker compose up --build
-> image rebuilt
-> containers started
```

---

# 🌙 32. Run containers in the background

Normally:

```bash
docker compose up
```

keeps your terminal attached to the logs.

You can instead use:

```bash
docker compose up -d
```

`-d` means detached mode.

The containers continue running in the background.

Check them with:

```bash
docker ps
```

View logs with:

```bash
docker compose logs -f
```

Stop everything with:

```bash
docker compose down
```

---

# ❤️ Step 06 — Improve Service Start-up

## 33. `depends_on` does not mean MongoDB is ready

We currently have:

```yaml
depends_on:
  - mongodb
```

This tells Compose that the MongoDB container should be started before the backend.

However:

```text
mongodb container started
```

does not necessarily mean:

```text
mongodb is ready to accept connections
```

There may be a short delay while MongoDB starts.

For a more reliable setup, add a health check.

---

# 🩺 34. Add a MongoDB health check

Update the MongoDB service:

```yaml
  mongodb:
    image: mongo:8
    container_name: gamevault-mongodb

    ports:
      - "27017:27017"

    volumes:
      - gamevault-mongo-data:/data/db

    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
```

Then update the backend dependency:

```yaml
    depends_on:
      mongodb:
        condition: service_healthy
```

The flow becomes:

```text
mongodb container starts
-> health check runs
-> mongodb responds
-> mongodb marked healthy
-> backend starts
```

This gives us a more reliable Compose environment.

---

# 🩺 35. Add a backend health check

GameVault already has a health endpoint.

We can also use that when managing containers.

If your backend health endpoint is:

```text
/health
```

you can add a health check to the backend service.

The exact command depends on what is available inside your image. For this project, you can either add a small Node-based check or keep the health check simple until the deployment stage.

The important idea is:

```text
container running
```

does not always mean:

```text
application healthy
```

A proper health endpoint gives us a way to check the actual application.

---

# 🧪 Step 07 — Docker + Testing

## 36. Run your tests before building

Containerising an application doesn't replace testing.

A good workflow is:

```text
change backend
-> npm test
-> npm run lint

change frontend
-> npm test
-> npm run lint

test api
-> newman

everything passes
-> docker compose build
-> docker compose up
```

Docker answers:

> Can we package and run the application consistently?

Automated tests answer:

> Does the application behave correctly?

ESLint answers:

> Does the code meet our quality rules?

Newman answers:

> Do our API requests and assertions still pass?

These tools complement one another.

---

# 📮 37. Test the containerised API with Newman

Once the containers are running:

```bash
docker compose up -d
```

you can run your existing GameVault Postman collection:

```bash
npx newman run postman/GameVault.postman_collection.json --insecure
```

If using an environment:

```bash
npx newman run postman/GameVault.postman_collection.json -e postman/GameVault.postman_environment.json --insecure
```

The request flow is now:

```text
Newman
-> https://localhost:4000
-> Docker published port
-> backend container
-> Express
-> MongoDB container
-> response
-> Newman assertions
```

This is a useful way to verify that the containerised version still behaves like the version you previously ran directly through Node.

---

# 🔎 Step 08 — Useful Docker Commands

These are the commands you should become familiar with:

```bash
docker --version
```

Check Docker.

```bash
docker images
```

View local images.

```bash
docker ps
```

View running containers.

```bash
docker ps -a
```

View running and stopped containers.

```bash
docker compose build
```

Build the application images.

```bash
docker compose up
```

Start the Compose environment.

```bash
docker compose up --build
```

Rebuild and start.

```bash
docker compose up -d
```

Start in the background.

```bash
docker compose logs -f
```

Follow logs.

```bash
docker compose down
```

Stop and remove the Compose containers.

---

# 🛠️ Step 09 — Common Problems

## ❌ Backend cannot connect to MongoDB

Check your connection string.

Inside Docker, don't use:

```text
mongodb://localhost:27017/gamevault
```

Use:

```text
mongodb://mongodb:27017/gamevault
```

Remember:

```text
localhost inside backend container
-> backend container itself

mongodb
-> MongoDB Compose service
```

---

## ❌ Frontend cannot reach backend

First check:

```bash
docker ps
```

Make sure the backend port is published.

Then test:

```text
https://localhost:4000
```

directly.

Also check your backend CORS configuration.

Your frontend is still accessed by the browser from:

```text
http://localhost:5173
```

so your backend should permit the appropriate frontend origin.

---

## ❌ Certificate file cannot be found

Check the backend logs:

```bash
docker compose logs backend
```

Then verify that your volume maps the directory to the location expected by `httpsConfig.js`.

For example:

```yaml
volumes:
  - ./backend/certificates:/app/certificates:ro
```

means:

```text
your computer
backend/certificates

->

container
/app/certificates
```

---

## ❌ Port already in use

You may see an error saying that a port is already allocated.

For example, you might already have the backend running normally with:

```bash
npm start
```

on port `4000`.

You then try to start Docker using the same host port.

Both applications cannot use the same host port simultaneously.

Stop the existing Node process before starting the container.

The same applies to:

```text
5173
27017
```

---

## ❌ `npm ci` fails during Docker build

Make sure you have:

```text
package.json
package-lock.json
```

in the relevant project directory.

If your dependency files are inconsistent, run locally:

```bash
npm install
```

and commit the updated `package-lock.json`.

Then rebuild:

```bash
docker compose build
```

Do not solve dependency problems by deleting random parts of the Dockerfile.

---

# 🔐 Step 10 — Security Rules

Docker doesn't automatically make an application secure.

You still need to follow the security practices already implemented in GameVault.

Do not put real secrets directly inside:

```text
Dockerfile
compose.yaml
GitHub
```

Do not write:

```dockerfile
ENV JWT_SECRET=mysecret123
```

and commit it.

Instead:

```text
.env
-> Docker Compose
-> backend environment
```

Your `.env` remains excluded from Git.

Also do not commit:

```text
private TLS keys
real MongoDB credentials
real JWT secrets
authentication tokens
```

Provide safe `.env.example` files showing which variables are required, but not their real values.

---

# 📁 Final GameVault Structure

After completing this section, your project should look roughly like:

```text
GameVault
│
├── backend
│   ├── certificates
│   ├── config
│   ├── controllers
│   ├── middleware
│   ├── models
│   ├── routes
│   ├── tests
│   ├── utils
│   ├── .dockerignore
│   ├── .env
│   ├── Dockerfile
│   ├── app.js
│   ├── package.json
│   └── server.js
│
├── frontend
│   ├── src
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── package.json
│   └── vite.config.js
│
├── postman
│   ├── GameVault.postman_collection.json
│   └── GameVault.postman_environment.json
│
├── .env
├── .env.example
├── .gitignore
└── compose.yaml
```

Your exact structure may differ slightly.

The important part is that another developer should eventually be able to clone the repository, provide the required environment values and run:

```bash
docker compose up --build
```

rather than manually setting up every part of GameVault.

# ✅ What you should have working

By the end of this section, you should be able to demonstrate:

```text
Dockerfile for backend
-> backend image builds successfully

Dockerfile for frontend
-> frontend image builds successfully

Docker Compose
-> MongoDB starts
-> backend starts
-> frontend starts

browser
-> frontend loads

frontend
-> communicates with backend

backend
-> communicates with MongoDB

Docker volume
-> MongoDB data persists

HTTPS
-> backend remains accessible securely

docker compose down
-> containers stop cleanly
```

The main goal is to move GameVault from:

```text
"it works on my computer"
```

to:

```text
clone project
-> configure environment
-> docker compose up --build
-> GameVault runs in a consistent environment
```
