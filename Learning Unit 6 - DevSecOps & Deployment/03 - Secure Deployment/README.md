# 🔐 GameVault — Secure Deployment

In this section, you will take GameVault from a development application to an application that is prepared to run securely in a deployed environment.

You have already worked with:

```text
GameVault
-> React frontend
-> Node.js / Express backend
-> MongoDB
-> authentication and JWT
-> HTTPS
-> automated testing
-> ESLint
-> Docker
-> Docker Compose
-> GitHub Actions
-> static code analysis
-> logging
-> monitoring
```

Deployment does not simply mean putting the application on a server and making it publicly accessible.

A secure deployment process should make sure that:

```text
code is tested
-> security checks pass
-> secrets are protected
-> production settings are used
-> containers are built
-> application is deployed
-> HTTPS protects communication
-> logs and monitoring remain available
```

The exact cloud platform used to host GameVault can differ. The important part of this section is understanding how the application itself must be prepared for secure deployment.

---

# 🌍 Step 01 — Understand Development vs Production

## 1. Your development environment is not your production environment

While developing GameVault, you may currently have something similar to:

```text
frontend
-> http://localhost:5173

backend
-> https://localhost:4000

MongoDB
-> localhost or development database
```

This is suitable for development.

A deployed application might instead look like:

```text
user
-> https://gamevault.example.com
-> deployed frontend
-> deployed backend
-> production MongoDB database
```

The production environment should have its own configuration and credentials.

Do not simply copy your local `.env` file to production.

---

# 🗂️ Step 02 — Separate Your Environments

GameVault should have clearly separated environments.

For example:

```text
development
-> local development

test
-> automated tests / CI

production
-> deployed application
```

This applies to configuration as well.

For example:

```text
development MongoDB
-> gamevault

testing MongoDB
-> gamevault_test

production MongoDB
-> separate production database
```

The same principle applies to secrets:

```text
local JWT secret
-> development only

CI JWT secret
-> GitHub Actions only

production JWT secret
-> deployed backend only
```

Do not reuse one secret everywhere.

---

# 🔐 Step 03 — Check Your Environment Variables

## 2. Review the backend configuration

Your backend will probably require values similar to:

```env
NODE_ENV=production
PORT=4000
MONGODB_URI=...
JWT_SECRET=...
JWT_EXPIRES_IN=1h
BCRYPT_ROUNDS=12
FRONTEND_URL=...
```

Your exact variable names may differ.

Use the names already used by your GameVault project.

A safe `.env.example` could contain:

```env
NODE_ENV=production
PORT=4000
MONGODB_URI=<production-mongodb-uri>
JWT_SECRET=<generate-a-long-random-secret>
JWT_EXPIRES_IN=1h
BCRYPT_ROUNDS=12
FRONTEND_URL=<deployed-frontend-url>
```

Notice that this file shows what is required, but does not contain real secrets.

---

# 🚫 3. Never commit the production `.env`

Your `.gitignore` should include:

```text
.env
.env.local
.env.production
```

You can commit:

```text
.env.example
```

but not:

```text
.env
```

Before deploying, check your Git repository carefully.

Run:

```bash
git status
```

Make sure you are not about to commit:

```text
.env
private keys
TLS private certificates
database passwords
JWT secrets
API keys
authentication tokens
generated logs
```

---

# ⚠️ 4. `.gitignore` does not remove previously committed secrets

This is important.

Suppose you accidentally committed:

```env
JWT_SECRET=my-real-secret
```

and then added `.env` to `.gitignore`.

That does not make the old secret safe.

The secret may still exist in Git history.

If a real secret has been committed:

```text
secret committed
-> assume it has been exposed
-> revoke or rotate it
-> create a new secret
-> store new secret securely
```

Do not simply delete the line and continue using the same secret.

---

# 🔑 Step 04 — Generate a Strong JWT Secret

Do not use values such as:

```text
secret
gamevault
gamevault123
Password123
myjwtsecret
```

Generate a random secret instead.

With Node.js you can run:

```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

This generates a random value.

Store the result in the deployment platform's secret/environment variable configuration.

Do not paste the production secret into:

```text
server.js
config.js
Dockerfile
compose.yaml
GitHub
README.md
```

---

# 🛑 Step 05 — Fail Securely When Configuration Is Missing

A production application should not quietly start with unsafe default values.

Avoid:

```javascript
const jwtSecret = process.env.JWT_SECRET || "gamevault123";
```

If `JWT_SECRET` is missing, GameVault now silently uses:

```text
gamevault123
```

That is unsafe.

Instead, check important configuration when the application starts.

For example, create:

```text
backend/config/validateEnv.js
```

```javascript
const required = [
    "MONGODB_URI",
    "JWT_SECRET"
];

const validateEnv = () => {

    const missing = required.filter((name) => !process.env[name]);

    if (missing.length > 0) {
        throw new Error(
            `missing environment variables: ${missing.join(", ")}`
        );
    }

    if (process.env.JWT_SECRET.length < 32) {
        throw new Error("jwt secret is too short");
    }
};

module.exports = validateEnv;
```

Then call it before starting the server:

```javascript
const validateEnv = require("./config/validateEnv");

validateEnv();
```

Now:

```text
required configuration available
-> application starts
```

but:

```text
JWT_SECRET missing
-> startup fails
-> insecure application does not start
```

This is much safer than falling back to weak defaults.

---

# 🏭 Step 06 — Set Production Mode

Your deployed backend should run with:

```env
NODE_ENV=production
```

Your application can then behave differently depending on the environment.

For example:

```javascript
const isProduction = process.env.NODE_ENV === "production";
```

You might use this when deciding how much information should be logged internally.

However, do not rely on `NODE_ENV` to make unsafe code safe.

For example, your API should never return passwords or secrets, regardless of the environment.

---

# ❌ Step 07 — Do Not Return Internal Errors

Your development environment may provide more information when something goes wrong.

Your production API should return controlled responses.

For example:

```javascript
const logger = require("../utils/logger");

const errorHandler = (err, req, res, next) => {

    logger.error("request failed", {
        message: err.message,
        method: req.method,
        path: req.originalUrl
    });

    res.status(err.status || 500).json({
        message: err.status ? err.message : "something went wrong"
    });
};

module.exports = errorHandler;
```

The client should not receive:

```text
stack traces
file paths
MongoDB connection strings
environment variables
server configuration
internal dependency information
```

For example, avoid:

```javascript
res.status(500).json({
    message: err.message,
    stack: err.stack
});
```

The stack trace may be useful to developers, but it should not be sent to normal users.

---

# 🔏 Step 08 — Protect Sensitive Logs

Your logging setup must also be production-safe.

Do not log:

```text
passwords
JWTs
JWT secrets
MongoDB passwords
private keys
full Authorization headers
```

Avoid:

```javascript
logger.info("login request", {
    body: req.body
});
```

because the body may contain:

```json
{
    "email": "student@example.com",
    "password": "Password123!"
}
```

Instead:

```javascript
logger.info("login attempt", {
    email: req.body.email
});
```

For a successful login:

```javascript
logger.info("user logged in", {
    userId: user._id.toString()
});
```

We want enough information to investigate problems without creating another source of sensitive data.

---

# 🔐 Step 09 — Production HTTPS

## 5. Do not use your local self-signed certificate in production

During development, GameVault used a local self-signed certificate.

That was useful for learning HTTPS.

A deployed application should use a certificate trusted by browsers.

Production traffic should look like:

```text
browser
-> HTTPS
-> trusted TLS certificate
-> GameVault
```

not:

```text
browser
-> certificate warning
-> user manually accepts warning
-> GameVault
```

A production user should not have to bypass certificate warnings.

---

# 🌐 6. HTTPS may be handled before Express

Depending on your deployment environment, HTTPS may be terminated by:

```text
cloud platform
reverse proxy
load balancer
web server
```

The public request can therefore be:

```text
browser
-> HTTPS
-> reverse proxy / platform
-> GameVault backend
```

This means your production Express application does not necessarily need to manage certificate files itself.

Do not automatically copy your development:

```text
cert.pem
key.pem
```

onto a production server.

Use the TLS configuration supported by your deployment platform.

---

# 🪖 Step 10 — Keep Helmet Enabled

Your Express backend should continue using Helmet.

For example:

```javascript
const helmet = require("helmet");

app.use(helmet());
```

Helmet adds several security-related HTTP headers.

Do not remove your existing security middleware simply because the application is now deployed.

Production should normally have stronger, not weaker, security controls.

---

# 🌐 Step 11 — Configure CORS for Production

During development, your frontend may run from:

```text
http://localhost:5173
```

Your deployed frontend will have a different origin.

Instead of allowing everything:

```javascript
app.use(cors({
    origin: "*"
}));
```

configure the expected frontend origin.

For example:

```javascript
const cors = require("cors");

app.use(cors({
    origin: process.env.FRONTEND_URL
}));
```

Your production environment could then provide:

```env
FRONTEND_URL=https://gamevault.example.com
```

The idea is:

```text
expected GameVault frontend
-> allowed

random website
-> not automatically trusted
```

If your application legitimately needs several origins, use an allowlist rather than changing the setting to `*`.

---

# 🚦 Step 12 — Keep Rate Limiting Enabled

Your authentication endpoints should remain rate limited.

For example:

```javascript
const rateLimit = require("express-rate-limit");

const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    limit: 20,
    standardHeaders: true,
    legacyHeaders: false
});

app.use("/api/auth", authLimiter);
```

This helps reduce repeated automated requests against sensitive endpoints such as login.

Your exact limits should make sense for your application.

Do not choose an extremely low number that blocks normal users simply because "lower is more secure".

---

# 🔑 Step 13 — Review JWT Security

Before deploying, verify that your authentication flow still follows the expected process:

```text
user logs in
-> credentials verified
-> password compared using bcrypt
-> JWT generated
-> JWT returned
-> protected request includes JWT
-> backend verifies JWT
-> user identified
-> authorisation checked
-> request allowed or rejected
```

Check that:

```text
passwords are hashed
JWT secret comes from environment
JWT has an expiry
JWT is verified on protected routes
admin routes check the user's role
invalid/expired tokens are rejected
```

Do not simply decode a JWT and trust its contents.

Protected routes must verify it.

---

# 👑 Step 14 — Secure Admin Access

GameVault has different user roles.

A normal user should not be able to make themselves an administrator by sending:

```json
{
    "name": "student",
    "email": "student@example.com",
    "password": "Password123!",
    "role": "ADMIN"
}
```

Public registration should assign the permitted default role on the server.

For example:

```javascript
const user = await User.create({
    name,
    email,
    passwordHash,
    role: "CLIENT"
});
```

Do not trust a public registration request to decide whether someone should become an administrator.

Admin creation should use a separate controlled process.

---

# 🗄️ Step 15 — Secure the Production Database

Your production MongoDB database should not simply be your lecturer/student development database.

Create a separate production database and separate production credentials.

The production connection should be supplied through:

```text
MONGODB_URI
```

not hard-coded:

```javascript
mongoose.connect("mongodb+srv://admin:password123@...");
```

Use:

```javascript
mongoose.connect(process.env.MONGODB_URI);
```

---

# 👤 16. Apply least privilege to database credentials

The account used by GameVault should have the permissions GameVault actually needs.

The principle is:

```text
application requires certain database operations
-> account receives required permissions
-> unnecessary privileges are not granted
```

Do not use a highly privileged database administration account for ordinary application traffic when a more restricted application account can do the job.

This is the principle of least privilege.

---

# 🌐 17. Restrict database network access

A production database should not unnecessarily accept connections from everywhere.

Where supported by your hosting/database provider, restrict network access to the systems that actually require it.

The desired relationship is:

```text
GameVault backend
-> production database
```

rather than:

```text
entire internet
-> production database
```

The exact configuration depends on the provider used for deployment.

---

# 🐳 Step 16 — Prepare the Backend Docker Image

Your development Dockerfile may work, but review it before deployment.

A simple backend Dockerfile could be:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY . .

ENV NODE_ENV=production

EXPOSE 4000

USER node

CMD ["npm", "start"]
```

There are a few important differences here.

---

# 📦 18. Install production dependencies

We use:

```dockerfile
RUN npm ci --omit=dev
```

rather than installing all development dependencies.

Packages used only for:

```text
testing
linting
development tools
```

do not necessarily need to exist in the production runtime image.

This can reduce the image size and the amount of unnecessary software inside the production container.

---

# 👤 19. Don't run the application as root

Notice:

```dockerfile
USER node
```

The official Node image provides a non-root `node` user.

Running the application as a non-root user reduces the privileges available to the application inside the container.

The principle is:

```text
application compromised
-> attacker gets application-level access
-> unnecessary root privileges are not automatically available
```

This is another example of least privilege.

---

# 🚫 Step 17 — Improve `.dockerignore`

Your backend `.dockerignore` should prevent unnecessary/sensitive files from entering the image.

For example:

```text
node_modules
npm-debug.log
.git
.github
.env
.env.*
logs
coverage
tests
*.log
```

Be careful with certificate files as well.

Do not accidentally bake development private keys into the production image.

---

# ⚛️ Step 18 — Build the React Frontend for Production

During development, the frontend runs using:

```bash
npm run dev
```

That starts the Vite development server.

The Vite development server is not your production deployment.

For production, build the React application:

```bash
npm run build
```

Vite generates:

```text
frontend/dist
```

This contains the production frontend files.

The flow becomes:

```text
React source
-> npm run build
-> dist
-> static production files
```

---

# 🏗️ Step 19 — Use a Multi-Stage Frontend Dockerfile

A production frontend Dockerfile can use one stage to build React and another stage to serve the final files.

For example:

```dockerfile
FROM node:22-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build


FROM nginx:alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

The first stage:

```text
Node
-> install dependencies
-> build React
```

The second stage:

```text
Nginx
-> receive built files
-> serve frontend
```

The final image does not need the complete Node development environment just to serve static React files.

---

# ⚙️ Step 20 — Configure the Production API URL

During development, your frontend may call:

```text
https://localhost:4000
```

That will not work correctly for normal production users.

The production frontend needs the deployed backend address.

For a Vite application, you might use:

```env
VITE_API_URL=https://api.gamevault.example.com
```

Then:

```javascript
const API_URL = import.meta.env.VITE_API_URL;
```

Your API service can use:

```javascript
fetch(`${API_URL}/api/games`);
```

or configure your existing Axios instance using the same value.

Remember:

> Vite frontend environment variables are included in the client build.

Therefore, do not put secrets in variables beginning with `VITE_`.

This would be unsafe:

```env
VITE_JWT_SECRET=...
```

Anything sent to the browser must be treated as public information.

---

# 🔒 Step 21 — Understand Frontend Secrets

There is effectively no safe secret stored inside normal browser JavaScript.

Do not put:

```text
JWT signing secret
database password
private API key
private TLS key
admin password
```

inside React.

React code is delivered to the user's browser.

A user can inspect it.

The frontend should only receive information it is allowed to know.

Sensitive operations remain on the backend.

---

# 🧪 Step 22 — Run All Checks Before Deployment

Before building a production release, run your checks.

Backend:

```bash
npm ci
npm run lint
npm test
```

Frontend:

```bash
npm ci
npm run lint
npm test -- --run
npm run build
```

Static analysis:

```bash
semgrep scan --config auto .
```

Dependency checks:

```bash
npm audit
```

API tests:

```bash
npx newman run postman/GameVault.postman_collection.json --insecure
```

The `--insecure` option here is only relevant to your local self-signed development HTTPS setup.

Your deployed production service should have trusted HTTPS and should not require this workaround.

---

# 🚀 Step 23 — Use the CI/CD Pipeline

You should not depend only on someone manually remembering the deployment checklist.

Your GitHub Actions pipeline should verify the application first.

A deployment pipeline should conceptually follow:

```text
push / merge
-> backend lint
-> backend tests
-> frontend lint
-> frontend tests
-> frontend build
-> static analysis
-> Docker build
-> deployment
-> health check
```

Deployment should happen after the required checks pass.

---

# 🚦 Step 24 — Separate CI and Deployment

Your pipeline has two broad responsibilities.

### Continuous Integration

```text
code pushed
-> lint
-> test
-> static analysis
-> build
```

### Deployment

```text
verified code
-> create production artefact/image
-> authenticate with deployment environment
-> deploy
-> check application health
```

Do not deploy a commit that has already failed the required CI checks.

---

# 🌿 Step 25 — Deploy from the Correct Branch

A simple GameVault workflow could use:

```text
feature branches
-> development work

pull request
-> automated checks

main
-> approved stable code
-> production deployment
```

For example:

```text
feature/game-reviews
-> pull request
-> GitHub Actions
-> tests pass
-> review
-> merge to main
-> deployment pipeline
```

This prevents every unfinished feature branch from automatically becoming a production release.

---

# 🔑 Step 26 — Store Deployment Credentials as GitHub Secrets

Your deployment process may eventually require credentials.

For example:

```text
cloud deployment token
container registry username
container registry password/token
production service credentials
```

Do not write them directly into:

```text
.github/workflows/deploy.yml
```

Use:

```text
GitHub repository
-> Settings
-> Secrets and variables
-> Actions
```

Then your workflow can refer to a secret:

```yaml
${{ secrets.DEPLOY_TOKEN }}
```

The actual value remains outside the workflow file.

---

# 🐳 Step 27 — Build a Production Image in CI/CD

Your deployment job might first build the backend image:

```yaml
- name: Build backend image
  run: docker build -t gamevault-backend ./backend
```

and frontend:

```yaml
- name: Build frontend image
  run: docker build -t gamevault-frontend ./frontend
```

At this point:

```text
source code
-> tests passed
-> production images built
-> images ready for deployment
```

The exact steps for pushing those images depend on the container registry and deployment provider you use.

---

# 🏷️ Step 28 — Tag Deployments Properly

Avoid relying only on a tag such as:

```text
latest
```

If every deployment is simply called `latest`, it becomes harder to determine exactly which version is running.

A better process is to associate the image with a specific commit/version.

For example:

```text
gamevault-backend:4f93c21
```

where the tag identifies the Git commit.

This gives you traceability:

```text
production problem
-> identify deployed image
-> identify Git commit
-> inspect exact code
```

---

# ❤️ Step 29 — Keep the Health Endpoint

Your deployed GameVault backend should continue to expose a safe health endpoint.

For example:

```javascript
router.get("/health", (req, res) => {

    const databaseConnected = mongoose.connection.readyState === 1;

    res.status(databaseConnected ? 200 : 503).json({
        status: databaseConnected ? "ok" : "degraded",
        database: databaseConnected ? "connected" : "disconnected",
        timestamp: new Date().toISOString()
    });

});
```

After deployment:

```text
deployment completes
-> health endpoint requested
-> 200 returned
-> application considered healthy
```

If the health check returns `503`, investigate before considering the deployment successful.

---

# 🩺 Step 30 — Add a Post-Deployment Check

A deployment finishing successfully does not necessarily mean the application works.

The deployment pipeline should verify the deployed application.

Conceptually:

```text
deployment reports success
-> request production /health
-> health check passes
-> deployment considered successful
```

For example, a workflow step may eventually resemble:

```yaml
- name: Check deployment
  run: curl --fail https://api.gamevault.example.com/health
```

`--fail` makes `curl` return an unsuccessful exit code for HTTP error responses.

Do not use `-k` against the production application simply to bypass an invalid certificate.

A production TLS problem should be fixed.

---

# 📮 Step 31 — Run Smoke Tests

You can also perform a small number of smoke tests after deployment.

Smoke testing asks:

> Are the application's most important functions basically working?

For example:

```text
health endpoint
-> responds

games endpoint
-> responds

login
-> responds correctly
```

You do not necessarily need to execute every destructive test against production.

For example, avoid having a deployment pipeline repeatedly:

```text
create 50 users
delete production games
modify real user information
```

Use carefully designed production-safe checks.

---

# 📊 Step 32 — Keep Monitoring After Deployment

Deployment is not the end of the process.

Your monitoring from the previous section now becomes especially important.

GameVault can expose:

```text
/health
-> application health

/metrics
-> application metrics
```

Prometheus can collect:

```text
request totals
HTTP status codes
process metrics
memory usage
```

Grafana can display them.

The process becomes:

```text
deploy GameVault
-> users access application
-> application generates logs
-> application exposes metrics
-> monitoring collects information
-> problems can be detected
```

---

# 📝 Step 33 — Keep Production Logging Enabled

Production logs should still contain useful events such as:

```text
application started
database connected
request completed
failed login
forbidden access
unexpected server error
```

For example:

```javascript
logger.warn("login failed", {
    email: email
});
```

or:

```javascript
logger.error("request failed", {
    method: req.method,
    path: req.originalUrl,
    message: err.message
});
```

Again, do not log:

```text
password
JWT
JWT secret
database credentials
private keys
```

---

# 📈 Step 34 — Watch for Security-Relevant Patterns

Monitoring is also useful for security.

For example:

```text
normal login failures
-> occasional

sudden hundreds of login failures
-> suspicious
```

Or:

```text
normal server errors
-> very low

deployment happens
-> 500 errors suddenly increase
-> possible deployment problem
```

Useful things to monitor include:

```text
application availability
response times
500 errors
401 responses
403 responses
failed login activity
memory usage
database health
```

---

# 🔄 Step 35 — Think About Rollback

Sometimes a deployment passes initial checks but later causes problems.

For example:

```text
version 1.4 deployed
-> application starts
-> health check passes
-> users begin using feature
-> serious bug discovered
```

You need to know which previous version was stable.

This is another reason to use identifiable image/version tags.

A rollback process is:

```text
new deployment causes problem
-> identify previous stable version
-> redeploy previous version
-> confirm health
-> investigate faulty version separately
```

Do not try to repair a serious production problem by randomly editing files directly on the production server.

Production should remain tied to controlled, versioned deployments.

---

# 💾 Step 36 — Think About Database Changes Separately

Application rollback is relatively straightforward when using versioned Docker images.

Database changes can be more complicated.

Suppose a new release:

```text
changes database structure
-> modifies existing data
-> new application fails
-> old application redeployed
```

The old application may no longer understand the changed data.

Database migrations should therefore be planned carefully.

Before major production changes:

```text
review migration
-> back up important data
-> deploy carefully
-> verify result
```

Do not treat production database changes as something that can always be undone automatically.

---

# 💾 Step 37 — Back Up Important Production Data

A Docker volume is not the same thing as a backup.

A volume helps data survive container replacement.

It does not automatically protect you against:

```text
accidental deletion
bad migration
database corruption
compromised account
infrastructure failure
```

Production data should have an appropriate backup strategy based on the database/hosting provider being used.

You should also understand how restoration works.

A backup that has never been tested for restoration may not be as useful as you think.

---

# 🧹 Step 38 — Remove Development-Only Behaviour

Before production deployment, review the project for temporary development code.

Look for things such as:

```text
console.log(req.body)
temporary test routes
hard-coded users
default admin passwords
development-only authentication bypasses
debug endpoints
sample credentials
unused test accounts
HTTP fallback
stack traces returned to users
```

For example, do not automatically create:

```text
admin@example.com
AdminPass123!
```

every time production starts.

A predictable default administrator account is a security risk.

If the application requires an initial administrator, use a controlled setup process and require a secure credential.

---

# 🚫 Step 39 — Do Not Fall Back to HTTP

During development, some projects use code similar to:

```text
try HTTPS
-> certificate missing
-> start HTTP instead
```

Do not use this approach for secure production deployment.

If secure communication is required and the HTTPS/reverse-proxy configuration is broken:

```text
secure configuration unavailable
-> deployment/startup should fail
```

not:

```text
secure configuration unavailable
-> quietly serve application insecurely
```

Failing securely is preferable to silently reducing security.

---

# 🔍 Step 40 — Run a Final Security Review

Before deployment, check the application systematically.

### Authentication

```text
passwords hashed
JWT secret secure
JWT expires
protected routes verify token
admin routes verify role
public registration cannot create admin
```

### Input

```text
required fields validated
invalid input rejected
malicious input handled safely
unexpected properties handled appropriately
```

### Errors

```text
controlled responses
no stack traces sent to clients
no internal paths exposed
no configuration values exposed
```

### Secrets

```text
no .env committed
no credentials in source
no secrets in React
no secrets in Dockerfile
no secrets in Compose
no secrets in Postman exports
```

### Network

```text
trusted HTTPS
production CORS configured
database access restricted
rate limiting enabled
```

### Containers

```text
production dependencies only where appropriate
non-root user
.dockerignore configured
no private keys baked into images
images versioned
```

### Quality

```text
ESLint passes
tests pass
static analysis reviewed
dependency vulnerabilities reviewed
frontend builds
Docker images build
```

### Runtime

```text
health endpoint works
logging works
sensitive values not logged
monitoring works
database backup considered
rollback possible
```

---

# 🚀 Step 41 — Complete Secure Deployment Flow

Your full GameVault process should now look approximately like this:

```text
developer creates feature
-> local tests
-> ESLint
-> static analysis
-> commit
-> push feature branch
-> pull request

GitHub Actions
-> backend lint
-> backend tests
-> frontend lint
-> React tests
-> frontend build
-> static analysis
-> Docker build

checks pass
-> code reviewed
-> merge to main

deployment pipeline
-> production images created
-> deployment credentials loaded securely
-> application deployed
-> production environment variables supplied
-> trusted HTTPS used
-> database connected

post-deployment
-> health check
-> smoke tests
-> logs checked
-> metrics monitored

problem detected
-> investigate logs/metrics
-> rollback if necessary
-> fix code
-> pipeline runs again
-> redeploy
```

---

# 📁 Final GameVault Structure

Your project may now look roughly like:

```text
GameVault
│
├── .github
│   └── workflows
│       ├── ci.yml
│       ├── security.yml
│       └── deploy.yml
│
├── backend
│   ├── config
│   │   └── validateEnv.js
│   ├── controllers
│   ├── middleware
│   │   ├── auth.js
│   │   ├── errorHandler.js
│   │   └── requestLogger.js
│   ├── models
│   ├── monitoring
│   │   └── metrics.js
│   ├── routes
│   ├── tests
│   ├── utils
│   │   └── logger.js
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── eslint.config.js
│   ├── app.js
│   ├── package.json
│   └── server.js
│
├── frontend
│   ├── src
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── eslint.config.js
│   ├── package.json
│   └── vite.config.js
│
├── monitoring
│   └── prometheus.yml
│
├── postman
│   └── GameVault.postman_collection.json
│
├── compose.yaml
├── .env.example
└── .gitignore
```

The exact structure does not have to be identical. Keep the structure that makes sense for your existing GameVault implementation.

# ✅ What You Should Have Working

By the end of this section, you should be able to demonstrate:

```text
CONFIGURATION

development and production separated
-> production environment variables configured
-> strong JWT secret
-> startup rejects missing critical configuration


SECRETS

.env not committed
-> secrets not hard-coded
-> production secrets stored by deployment platform
-> GitHub deployment credentials stored as secrets


BACKEND SECURITY

password hashing enabled
-> JWT verification enabled
-> RBAC enforced
-> Helmet enabled
-> rate limiting enabled
-> production CORS restricted
-> controlled errors
-> no stack traces exposed


FRONTEND SECURITY

production React build created
-> production API URL configured
-> no backend secrets stored in React


DATABASE

production database separated
-> credentials stored securely
-> least privilege considered
-> network access restricted
-> backup strategy considered


CONTAINERS

production Docker images build
-> unnecessary development dependencies excluded where appropriate
-> backend runs as non-root
-> sensitive files excluded from image
-> images can be versioned


CI/CD

tests run before deployment
-> linting runs before deployment
-> static analysis runs
-> Docker images build
-> failed checks block deployment
-> deployment uses secure credentials


PRODUCTION

trusted HTTPS
-> application starts
-> database connects
-> health endpoint returns successfully
-> smoke tests pass
-> logging works
-> monitoring works
-> rollback is possible
```

The important idea is that secure deployment is not one final security setting.

It is the complete process:

```text
secure code
-> automated checks
-> protected secrets
-> hardened configuration
-> secure container
-> controlled deployment
-> trusted HTTPS
-> production monitoring
-> safe rollback
```

GameVault should not only be able to run in production. You should be able to explain why the deployed version is safer than simply taking the development application and making it publicly accessible.
