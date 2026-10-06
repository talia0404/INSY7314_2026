# 🔎 GameVault — Static Code Analysis, Logging & Monitoring

In this section, you will improve the security, maintainability and observability of GameVault.

We have already introduced automated testing and ESLint. These help us find problems before code is released. However, once an application becomes larger, we also need ways to:

```text
analyse the source code
-> identify potential security/code-quality problems
-> record what the application is doing
-> detect when the running application has problems
```

We will therefore introduce three related concepts:

```text
Static Code Analysis
-> inspect source code without running the application

Logging
-> record important events while the application is running

Monitoring
-> use application information to determine whether the system is healthy
```

These are different techniques, but they work together.

---

# 🔎 Step 01 — Static Code Analysis

## 1. What is static code analysis?

Static code analysis examines source code without executing the application.

For example, a static analysis tool may inspect this:

```javascript
const password = "Password123!";
```

and identify that a credential appears to have been hard-coded.

Or it may identify:

```text
unused variables
-> suspicious code patterns
-> security problems
-> duplicated code
-> maintainability problems
-> vulnerable coding practices
```

Static analysis is different from automated testing.

```text
automated testing
-> runs code
-> checks behaviour

static analysis
-> examines code
-> looks for problems in the source
```

We will use more than one level of static analysis in GameVault.

---

# 🧹 Step 02 — ESLint as Basic Static Analysis

## 2. Start with ESLint

You already configured ESLint in the previous section.

Run it from the backend:

```bash
npm run lint
```

Then run it from the frontend:

```bash
npm run lint
```

ESLint is itself a form of static analysis.

It can identify problems such as:

```text
undefined variables
unused variables
unreachable code
incorrect React Hook usage
other suspicious JavaScript patterns
```

For example:

```javascript
const game = await Game.findById(req.params.id);
const user = req.user;

res.json(game);
```

If `user` is never used, ESLint can report it.

Do not immediately disable a rule because it produces a warning.

First check whether the warning identifies a genuine problem.

---

# 🔐 Step 03 — Add Security-Focused Static Analysis

ESLint is useful, but we also want to inspect GameVault specifically for potential security problems.

One option for this is Semgrep.

Semgrep analyses source code using security and code-quality rules.

It can help identify patterns involving areas such as:

```text
hard-coded secrets
-> unsafe code
-> injection risks
-> insecure configuration
-> dangerous functions
```

It does not replace manual security review, but it gives us another automated check.

---

# 📦 4. Install Semgrep

Semgrep is not a normal Node package that needs to become part of GameVault.

If Python is available on your computer, you can install the Semgrep CLI with:

```bash
python -m pip install semgrep
```

Depending on your Python installation, you may instead use:

```bash
py -m pip install semgrep
```

Check that it works:

```bash
semgrep --version
```

If the command is not recognised immediately, restart your terminal.

You can also run Semgrep through a supported container if you prefer not to install it directly.

---

# 🔍 5. Scan GameVault

From the root GameVault directory, run:

```bash
semgrep scan --config auto .
```

Semgrep will inspect the repository and select relevant rules.

The process is:

```text
GameVault source
-> Semgrep scans files
-> rules inspect code patterns
-> findings reported
```

Do not assume every finding automatically means your application is vulnerable.

Static analysis tools can produce:

```text
true positive
-> real problem

false positive
-> tool reports something that is safe in this context
```

Every finding must be reviewed.

---

# 🚨 6. Investigate findings

Suppose a tool identifies something like:

```javascript
const JWT_SECRET = "gamevault123";
```

That is a genuine problem.

The secret should come from the environment:

```javascript
const JWT_SECRET = process.env.JWT_SECRET;
```

Your `.env` could contain the actual development value:

```env
JWT_SECRET=your_long_random_development_secret
```

and `.env` should remain excluded from Git.

Another example could involve logging:

```javascript
console.log(req.body);
```

That might look harmless, but a login request body could contain:

```json
{
    "email": "student@example.com",
    "password": "Password123!"
}
```

Now the plaintext password may be written to your logs.

Static analysis can help us notice patterns like these, but developers still need to understand why the pattern may be unsafe.

---

# 📄 7. Don't blindly ignore findings

If a finding is not relevant, investigate it before suppressing it.

Do not do this:

```text
scanner reports 12 findings
-> ignore all 12
-> pipeline becomes easier
```

The correct process is:

```text
scanner reports finding
-> inspect affected code
-> determine whether it is genuine
-> fix genuine issue
-> document/suppress only if justified
-> scan again
```

---

# 🐙 Step 04 — Add Static Analysis to GitHub Actions

We do not want static analysis to depend entirely on someone remembering to run it.

It can become part of CI.

Create a separate workflow:

```text
.github
└── workflows
    ├── ci.yml
    └── security.yml
```

Inside:

```text
.github/workflows/security.yml
```

you can create:

```yaml
name: GameVault Security Scan

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:

  semgrep:
    name: Static Analysis
    runs-on: ubuntu-latest

    steps:

      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install Semgrep
        run: pip install semgrep

      - name: Scan code
        run: semgrep scan --config auto .
```

Now:

```text
push / pull request
-> GitHub Actions
-> Semgrep installed
-> source code scanned
-> findings reported
```

This complements the existing pipeline:

```text
ESLint
-> code-quality checks

Jest / Vitest
-> automated behaviour tests

Semgrep
-> security-focused static analysis
```

---

# 📦 Step 05 — Check Dependencies Too

Static source analysis is only one part of checking GameVault.

GameVault also depends on third-party npm packages.

Run:

```bash
npm audit
```

inside the backend.

Then run:

```bash
npm audit
```

inside the frontend.

npm will inspect the installed dependency tree against known vulnerability information.

You may see output such as:

```text
0 vulnerabilities
```

or findings with severity levels.

Do not automatically run:

```bash
npm audit fix --force
```

without understanding the changes.

`--force` can install breaking dependency versions.

Instead:

```text
read vulnerability
-> identify affected package
-> check whether GameVault actually uses affected version/path
-> update dependency where appropriate
-> test application again
```

---

# 📝 Step 06 — Logging

## 8. What is logging?

Logging means recording events that happen while GameVault is running.

You have probably already used:

```javascript
console.log("server started");
```

That is technically logging, but we want something more structured.

Useful logs can tell us:

```text
when request happened
-> which endpoint was requested
-> request method
-> response status
-> how long request took
-> whether an error occurred
```

For security-related events, logs can also tell us things such as:

```text
failed login
-> unauthorised request
-> forbidden admin request
-> server error
```

However, logging must be done carefully.

Logs should not contain:

```text
passwords
JWTs
JWT secrets
database passwords
credit card details
private keys
full authentication headers
```

A log file can itself become a security problem if sensitive information is written into it.

---

# 📦 Step 07 — Install Winston

For GameVault, we can use Winston for application logging.

Open:

```text
GameVault/backend
```

Install:

```bash
npm install winston
```

---

# 📁 9. Create the logger

Create:

```text
backend
└── utils
    └── logger.js
```

Add:

```javascript
const winston = require("winston");

const logger = winston.createLogger({

    level: process.env.LOG_LEVEL || "info",

    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),

    transports: [
        new winston.transports.Console()
    ]

});

module.exports = logger;
```

This gives GameVault a reusable logger.

Instead of having:

```javascript
console.log("something happened");
```

throughout the project, we can use:

```javascript
logger.info("something happened");
```

---

# 📊 10. Understand log levels

Winston supports different levels of logging.

The ones we will mainly use are:

```text
error
-> something failed

warn
-> something suspicious or unexpected happened

info
-> useful normal application event

debug
-> detailed information mainly useful during development
```

Examples:

```javascript
logger.info("server started");
```

```javascript
logger.warn("login failed");
```

```javascript
logger.error("database connection failed");
```

Do not use `error` for every message.

The log level should match the importance of the event.

---

# 🧾 Step 08 — Add Useful Context to Logs

Instead of:

```javascript
logger.info("request");
```

we can provide structured information:

```javascript
logger.info("request completed", {
    method: req.method,
    path: req.originalUrl,
    status: res.statusCode
});
```

Because Winston is using JSON formatting, this information is stored in a structured way.

A log may resemble:

```json
{
    "level": "info",
    "message": "request completed",
    "method": "GET",
    "path": "/api/games",
    "status": 200,
    "timestamp": "2026-10-06T06:30:00.000Z"
}
```

Structured logs are easier to search and analyse later.

---

# ⏱️ Step 09 — Create Request Logging Middleware

Create:

```text
backend
└── middleware
    └── requestLogger.js
```

Add:

```javascript
const logger = require("../utils/logger");

const requestLogger = (req, res, next) => {

    const start = Date.now();

    res.on("finish", () => {

        const time = Date.now() - start;

        logger.info("request completed", {
            method: req.method,
            path: req.originalUrl,
            status: res.statusCode,
            durationMs: time
        });

    });

    next();
};

module.exports = requestLogger;
```

The middleware records when the request started.

Once the response finishes:

```text
request received
-> timer starts
-> route/controller runs
-> response finishes
-> duration calculated
-> request logged
```

---

# ⚙️ 11. Add the middleware to Express

Open:

```text
backend/app.js
```

Import it:

```javascript
const requestLogger = require("./middleware/requestLogger");
```

Then add:

```javascript
app.use(requestLogger);
```

Place it early enough that it can observe the requests you want logged.

For example:

```javascript
app.use(express.json());

app.use(requestLogger);

app.use("/api/auth", authRoutes);
app.use("/api/games", gameRoutes);
```

Now requests are logged automatically.

You do not need to manually add a log statement to every GET, POST, PATCH and DELETE route just to record basic request information.

---

# 🔐 Step 10 — Log Authentication Events Carefully

Authentication events can be useful security logs.

For example, after a successful login:

```javascript
logger.info("user logged in", {
    userId: user._id.toString()
});
```

For a failed login:

```javascript
logger.warn("login failed", {
    email: req.body.email
});
```

Be careful with what you record.

Never do:

```javascript
logger.warn("login failed", {
    email: req.body.email,
    password: req.body.password
});
```

The password must never be logged.

Also avoid:

```javascript
logger.info("token created", {
    token
});
```

The JWT is a credential.

Do not put it into your logs.

A better security log contains enough information to investigate an event without recording the secret itself.

---

# 🛡️ 12. Log authorisation failures

Suppose GameVault has an admin middleware.

Instead of silently returning `403`, you can also log the event:

```javascript
logger.warn("admin access denied", {
    userId: req.user.id,
    path: req.originalUrl
});

return res.status(403).json({
    message: "access denied"
});
```

Now:

```text
normal user requests admin route
-> RBAC middleware checks role
-> access denied
-> warning logged
-> 403 returned
```

This can be useful when investigating suspicious behaviour.

---

# ❌ Step 11 — Improve Error Logging

Your global error handler should also use the logger.

For example:

```javascript
const logger = require("../utils/logger");

const errorHandler = (err, req, res, next) => {

    logger.error("request failed", {
        message: err.message,
        method: req.method,
        path: req.originalUrl
    });

    res.status(500).json({
        message: "something went wrong"
    });
};

module.exports = errorHandler;
```

Notice that the response sent to the user is controlled:

```json
{
    "message": "something went wrong"
}
```

We do not return:

```javascript
err.stack
```

to the client.

The client does not need your internal stack trace.

---

# ⚠️ 13. Don't log the entire request body

This is particularly important.

Avoid:

```javascript
logger.error("request failed", {
    body: req.body
});
```

Why?

Imagine the failing request was:

```text
POST /api/auth/login
```

The request body could contain:

```json
{
    "email": "student@example.com",
    "password": "Password123!"
}
```

Now the password is stored in your logs.

Rather log the information you specifically need:

```javascript
logger.error("request failed", {
    method: req.method,
    path: req.originalUrl
});
```

A useful rule is:

> log the information you need, not everything you have.

---

# 📄 Step 12 — Log to Files During Development

So far, Winston logs to the console.

You can also add file transports.

Update:

```text
utils/logger.js
```

For example:

```javascript
const winston = require("winston");

const logger = winston.createLogger({

    level: process.env.LOG_LEVEL || "info",

    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),

    transports: [

        new winston.transports.Console(),

        new winston.transports.File({
            filename: "logs/error.log",
            level: "error"
        }),

        new winston.transports.File({
            filename: "logs/app.log"
        })

    ]

});

module.exports = logger;
```

Winston will now write general logs to:

```text
logs/app.log
```

and error-level logs to:

```text
logs/error.log
```

---

# 📁 14. Create the logs directory

Inside the backend:

```text
backend
└── logs
```

You should not normally commit generated log files.

Add this to:

```text
backend/.gitignore
```

```text
logs/
```

The logs are runtime output, not source code.

---

# 🐳 Step 13 — Logging with Docker

When GameVault runs in Docker, console logging becomes especially useful.

You can view backend logs using:

```bash
docker compose logs backend
```

Follow them live:

```bash
docker compose logs -f backend
```

This works because Winston's console transport writes to the container's standard output.

The flow becomes:

```text
GameVault backend
-> Winston
-> console output
-> Docker captures output
-> docker compose logs
```

This is one reason you should not depend only on local `.log` files inside a container.

Containers can be replaced.

Console-based structured logs can later be collected by a proper monitoring/logging platform.

---

# 📈 Step 14 — Monitoring

## 15. What is monitoring?

Logging answers:

> What happened?

Monitoring answers questions such as:

> Is GameVault currently healthy?

> Is the API responding?

> Are requests suddenly failing?

> Is the server becoming slow?

> Is MongoDB connected?

Monitoring uses information from the running system to help us understand its health.

For GameVault, we will start with simple application-level monitoring.

---

# ❤️ Step 15 — Create a Health Endpoint

Your GameVault project may already have a basic health endpoint.

If not, create one.

For example:

```javascript
router.get("/health", (req, res) => {

    res.status(200).json({
        status: "ok",
        service: "gamevault-api",
        timestamp: new Date().toISOString()
    });

});
```

The response could be:

```json
{
    "status": "ok",
    "service": "gamevault-api",
    "timestamp": "2026-10-06T06:30:00.000Z"
}
```

A monitoring system can repeatedly request this endpoint.

If it receives `200`:

```text
API responding
-> basic health check passes
```

If it cannot connect or receives a failure:

```text
API unavailable
-> health check fails
-> investigate
```

---

# 🍃 Step 16 — Include Database Health

A running Express server does not necessarily mean the whole application is healthy.

For example:

```text
Express running
-> MongoDB disconnected
-> users cannot log in
-> games cannot load
```

The process exists, but the application is not fully operational.

If you are using Mongoose, you can inspect the database connection.

For example:

```javascript
const mongoose = require("mongoose");

router.get("/health", (req, res) => {

    const databaseConnected = mongoose.connection.readyState === 1;

    const status = databaseConnected ? "ok" : "degraded";

    res.status(databaseConnected ? 200 : 503).json({
        status,
        database: databaseConnected ? "connected" : "disconnected",
        timestamp: new Date().toISOString()
    });

});
```

Now the endpoint provides more useful information.

Healthy:

```json
{
    "status": "ok",
    "database": "connected",
    "timestamp": "2026-10-06T06:30:00.000Z"
}
```

Database problem:

```json
{
    "status": "degraded",
    "database": "disconnected",
    "timestamp": "2026-10-06T06:31:00.000Z"
}
```

---

# ⚠️ 17. Don't expose sensitive health information

A public health endpoint should not return things like:

```text
MongoDB username
database password
full connection string
JWT secret
server file paths
environment variables
stack traces
```

This would be unsafe:

```json
{
    "database": "mongodb+srv://admin:password123@...",
    "jwtSecret": "gamevault123"
}
```

The health endpoint only needs enough information to indicate health.

For example:

```json
{
    "status": "ok",
    "database": "connected"
}
```

is enough.

---

# ⏱️ Step 17 — Monitor Response Times

Remember our request logging middleware:

```javascript
const start = Date.now();

res.on("finish", () => {

    const time = Date.now() - start;

    logger.info("request completed", {
        method: req.method,
        path: req.originalUrl,
        status: res.statusCode,
        durationMs: time
    });

});
```

We are already collecting a simple performance measurement:

```text
durationMs
```

Suppose normal requests take:

```text
GET /api/games
-> 40ms
```

but suddenly logs show:

```text
GET /api/games
-> 4200ms
```

The endpoint still technically works, but there may be a performance problem.

Monitoring is not only about whether the application is completely offline.

We also care about:

```text
availability
response time
error rate
database health
resource usage
```

---

# 🚨 Step 18 — Monitor HTTP Errors

Because our request logger records:

```text
status
```

we can identify patterns such as:

```text
200
-> successful

400
-> bad request

401
-> authentication problem

403
-> authorisation problem

404
-> resource not found

500
-> server error
```

One `500` does not automatically mean the entire system has failed.

But:

```text
normal traffic
-> 2% errors
```

changing to:

```text
normal traffic
-> 70% errors
```

would be a serious warning.

This is where logging becomes useful for monitoring.

---

# 📊 Step 19 — Add Basic Application Metrics

For more detailed application monitoring, we can expose metrics that monitoring tools understand.

A common Node package for this is:

```text
prom-client
```

Install it in the backend:

```bash
npm install prom-client
```

Create:

```text
backend
└── monitoring
    └── metrics.js
```

Add:

```javascript
const client = require("prom-client");

client.collectDefaultMetrics();

const httpRequests = new client.Counter({
    name: "gamevault_http_requests_total",
    help: "total number of http requests",
    labelNames: ["method", "route", "status"]
});

module.exports = {
    client,
    httpRequests
};
```

This creates a counter for HTTP requests.

---

# 🔢 20. Record requests

Update your request middleware:

```javascript
const logger = require("../utils/logger");
const { httpRequests } = require("../monitoring/metrics");

const requestLogger = (req, res, next) => {

    const start = Date.now();

    res.on("finish", () => {

        const time = Date.now() - start;

        logger.info("request completed", {
            method: req.method,
            path: req.originalUrl,
            status: res.statusCode,
            durationMs: time
        });

        httpRequests.inc({
            method: req.method,
            route: req.route?.path || req.path,
            status: res.statusCode
        });

    });

    next();
};

module.exports = requestLogger;
```

Each completed request now increases a metric.

---

# 📊 21. Create a metrics endpoint

Inside `app.js`, import:

```javascript
const { client } = require("./monitoring/metrics");
```

Then add:

```javascript
app.get("/metrics", async (req, res) => {

    res.set("Content-Type", client.register.contentType);

    res.end(await client.register.metrics());

});
```

Now open:

```text
https://localhost:4000/metrics
```

You should see metric output.

It may contain information about:

```text
Node.js memory
process CPU
event loop
garbage collection
GameVault request totals
```

The output is intended for monitoring software rather than normal application users.

---

# ⚠️ 22. Think about `/metrics` security

A metrics endpoint can reveal information about the application.

For a classroom/local environment, exposing it locally is fine for learning.

For a real deployed application, do not automatically expose detailed internal metrics publicly.

Depending on the environment, you may:

```text
restrict the route
-> keep it on an internal network
-> require monitoring authentication
-> configure infrastructure to access it privately
```

The important idea is that operational information should be treated as application information, not as a public feature.

---

# 📉 Step 23 — Prometheus

A common monitoring tool is Prometheus.

Prometheus can periodically request the GameVault metrics endpoint.

The relationship is:

```text
GameVault
-> /metrics
-> Prometheus collects metrics
-> metrics stored over time
```

This allows us to move from:

```text
what is happening right now?
```

to:

```text
what has been happening over time?
```

---

# 🐳 Step 24 — Add Prometheus with Docker Compose

At the root of GameVault, create:

```text
monitoring
└── prometheus.yml
```

Add:

```yaml
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: "gamevault-backend"

    static_configs:
      - targets:
          - "backend:4000"
```

Because Prometheus will run inside the same Docker Compose environment, it can access the backend using its Compose service name:

```text
backend
```

rather than:

```text
localhost
```

---

# ⚠️ 25. HTTPS and Prometheus

Because the GameVault backend uses local HTTPS, your Prometheus configuration may need to know that the target uses HTTPS.

For the classroom self-signed certificate setup, the scrape configuration can be adjusted:

```yaml
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: "gamevault-backend"

    scheme: https

    tls_config:
      insecure_skip_verify: true

    static_configs:
      - targets:
          - "backend:4000"
```

`insecure_skip_verify` is being used because our local GameVault certificate is self-signed.

This is for the local classroom environment.

Do not treat disabling certificate verification as the normal production solution.

---

# 🧩 26. Add Prometheus to `compose.yaml`

Add another service:

```yaml
  prometheus:
    image: prom/prometheus

    container_name: gamevault-prometheus

    ports:
      - "9090:9090"

    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro

    depends_on:
      - backend
```

Then start the environment:

```bash
docker compose up --build
```

You now have:

```text
frontend
-> backend
-> MongoDB

Prometheus
-> backend /metrics
```

---

# 🌐 27. Open Prometheus

Open:

```text
http://localhost:9090
```

Prometheus provides an interface where you can query collected metrics.

For example, you can search for:

```text
gamevault_http_requests_total
```

Generate some requests in GameVault and then check the metric again.

You should see the counter change.

---

# 📊 Step 28 — Grafana

Prometheus collects and stores metrics.

A tool such as Grafana can display those metrics using dashboards.

The relationship is:

```text
GameVault
-> exposes metrics
-> Prometheus collects metrics
-> Grafana reads Prometheus
-> dashboards display information
```

This means:

```text
Prometheus
-> monitoring data

Grafana
-> visualisation
```

---

# 🐳 29. Add Grafana to Docker Compose

Add:

```yaml
  grafana:
    image: grafana/grafana

    container_name: gamevault-grafana

    ports:
      - "3001:3000"

    volumes:
      - gamevault-grafana-data:/var/lib/grafana

    depends_on:
      - prometheus
```

Then add another named volume at the bottom of `compose.yaml`:

```yaml
volumes:
  gamevault-mongo-data:
  gamevault-grafana-data:
```

We use host port:

```text
3001
```

because another part of your environment may already use port `3000`.

Start everything again:

```bash
docker compose up --build
```

Grafana should then be accessible at:

```text
http://localhost:3001
```

---

# 🔗 Step 30 — Connect Grafana to Prometheus

Inside Grafana, add Prometheus as a data source.

When Grafana asks for the Prometheus server URL, do not use:

```text
http://localhost:9090
```

Grafana is running inside its own container.

Inside that container:

```text
localhost
-> Grafana container
```

The Prometheus Compose service is called:

```text
prometheus
```

So use:

```text
http://prometheus:9090
```

The connection is:

```text
Grafana container
-> prometheus:9090
-> Prometheus container
```

---

# 📈 31. Build a simple GameVault dashboard

Once Prometheus is connected, create a dashboard.

You could start with information such as:

```text
total API requests
failed requests
requests by status code
Node.js memory usage
CPU/process information
```

As your metrics improve, you could also monitor:

```text
login attempts
failed logins
games created
API response duration
database availability
```

Be sensible about what becomes a metric.

Do not use metric labels containing passwords, JWTs or unnecessary personal information.

---

# 🚨 Step 32 — Logging vs Monitoring

Do not confuse the two.

Suppose a GameVault request fails.

Logging might tell us:

```text
timestamp: 08:32
method: POST
path: /api/games
status: 500
message: request failed
```

Monitoring might tell us:

```text
500 errors have increased significantly
-> application may have a problem
```

So:

```text
monitoring
-> tells us there is a problem

logging
-> helps us investigate what happened
```

They work together.

---

# 🔎 Step 33 — Static Analysis vs Logging vs Monitoring

At this point, you should understand where each technique fits.

```text
BEFORE / DURING DEVELOPMENT

source code
-> ESLint
-> Semgrep
-> npm audit
-> problems identified before release


WHILE APPLICATION IS RUNNING

GameVault
-> Winston
-> events/errors recorded


APPLICATION HEALTH

GameVault
-> /health
-> /metrics
-> Prometheus
-> Grafana
-> health and behaviour observed
```

---

# 🔄 Step 34 — Add Everything to the Development Process

Your GameVault development process is now becoming much more complete:

```text
developer changes code
-> ESLint
-> automated tests
-> static analysis
-> commit
-> push
-> GitHub Actions
-> tests run again
-> security scan
-> Docker build
-> application deployed/run
-> logs generated
-> metrics collected
-> application monitored
```

This is much closer to the workflow used when developing and maintaining real applications.

---

# 🐙 Step 35 — CI/CD Integration

Your existing GitHub Actions setup can now include:

```text
backend
-> npm ci
-> npm run lint
-> npm test

frontend
-> npm ci
-> npm run lint
-> npm test
-> npm run build

security
-> Semgrep static analysis
-> dependency checks

Docker
-> build images
-> verify Compose

runtime
-> structured logs
-> health endpoint
-> metrics
-> monitoring
```

Static analysis belongs naturally in CI because we want security/code-quality problems detected before changes progress through the pipeline.

Logging and monitoring are mainly runtime concerns because they help us understand the application after it has started running.

---

# 📁 Final GameVault Structure

After completing this section, GameVault may look roughly like:

```text
GameVault
│
├── .github
│   └── workflows
│       ├── ci.yml
│       └── security.yml
│
├── backend
│   ├── config
│   ├── controllers
│   ├── middleware
│   │   ├── auth.js
│   │   ├── errorHandler.js
│   │   └── requestLogger.js
│   │
│   ├── models
│   ├── monitoring
│   │   └── metrics.js
│   │
│   ├── routes
│   ├── tests
│   ├── utils
│   │   └── logger.js
│   │
│   ├── logs
│   ├── Dockerfile
│   ├── eslint.config.js
│   ├── app.js
│   ├── package.json
│   └── server.js
│
├── frontend
│   ├── src
│   ├── Dockerfile
│   ├── eslint.config.js
│   └── package.json
│
├── monitoring
│   └── prometheus.yml
│
├── postman
├── compose.yaml
├── .env.example
└── .gitignore
```

Your exact structure may be slightly different depending on your existing GameVault implementation.

Do not restructure working code simply to make it identical to this diagram.

---

# ✅ What You Should Have Working

By the end of this section, you should be able to demonstrate:

```text
STATIC CODE ANALYSIS

ESLint
-> analyses JavaScript/React code

Semgrep
-> performs security-focused source analysis

npm audit
-> checks third-party dependencies

GitHub Actions
-> static analysis can run automatically


LOGGING

Winston
-> structured application logs

request middleware
-> method logged
-> path logged
-> status logged
-> response duration logged

security events
-> failed authentication can be logged
-> forbidden access can be logged

error handling
-> errors logged safely
-> passwords/tokens not logged
-> internal stack traces not returned to users


MONITORING

/health
-> reports basic application/database health

/metrics
-> exposes application metrics

Prometheus
-> collects GameVault metrics

Grafana
-> displays monitoring information

Docker Compose
-> runs GameVault and monitoring services together
```

The overall idea is:

```text
static analysis
-> try to find problems before the application runs

logging
-> record important things that happen while it runs

monitoring
-> determine whether the running application is healthy

all three together
-> easier to prevent, detect and investigate problems
```

For GameVault, the goal is no longer only to build an application that works. You should also be able to check the quality of its code, understand what it is doing while it runs, and recognise when something has gone wrong.
