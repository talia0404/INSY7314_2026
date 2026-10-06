# 🚀 GameVault — GitHub Actions & CI/CD Pipelines

In the previous sections, you added automated testing, Postman/Newman, React testing, ESLint and Docker to GameVault.

At this point, you can manually run checks such as:

```text
backend
-> npm test
-> npm run lint

frontend
-> npm test
-> npm run lint

api
-> newman

docker
-> docker compose build
```

The problem is that developers have to remember to run all of these checks before pushing or merging code.

GitHub Actions allows GitHub to run these checks automatically.

Our workflow will eventually look like:

```text
developer changes code
-> developer pushes to GitHub
-> GitHub Actions starts
-> backend tests run
-> frontend tests run
-> ESLint runs
-> application is built
-> workflow passes or fails
```

This is the beginning of a CI/CD pipeline.

---

# 🧠 Step 01 — Understanding CI/CD

## 1. What is CI?

CI stands for Continuous Integration.

The idea is that developers regularly merge their changes into the shared repository and automated checks run whenever those changes are pushed.

For GameVault:

```text
student pushes code
-> GitHub receives the commit
-> automated tests run
-> linting runs
-> build is checked
-> GitHub reports the result
```

If everything works, the pipeline passes.

If something is broken, the pipeline fails and the team can investigate before merging or deploying the change.

---

## 2. What is CD?

CD normally refers to Continuous Delivery or Continuous Deployment.

The important distinction is:

```text
continuous integration
-> automatically test and verify changes

continuous delivery
-> application is kept ready for deployment

continuous deployment
-> successful changes can automatically be deployed
```

For GameVault, we will first focus on building a reliable CI pipeline.

Once that works, we can prepare the project for a deployment stage.

Do not rush into automatic deployment before your tests and build checks are reliable.

---

# 🐙 Step 02 — Create Your First GitHub Actions Workflow

## 3. Create the workflow folder

GitHub Actions looks for workflow files inside a specific directory.

At the root of GameVault, create:

```text
GameVault
├── .github
│   └── workflows
│       └── ci.yml
│
├── backend
├── frontend
├── postman
└── compose.yaml
```

The spelling is important:

```text
.github
-> workflows
-> ci.yml
```

Do not place `ci.yml` inside `backend` or `frontend`.

It belongs inside the repository-level `.github/workflows` directory.

---

# 📝 4. Start the workflow

Open:

```text
.github/workflows/ci.yml
```

Start with:

```yaml
name: GameVault CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main
```

This gives the workflow a name and tells GitHub when it should run.

In this case:

```text
push to main
-> run pipeline

pull request targeting main
-> run pipeline
```

This is useful for group work because a pull request can be checked before it is merged into `main`.

---

# 🖥️ 5. Add the backend job

Under the trigger, add:

```yaml
jobs:

  backend:
    name: Backend Tests
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: backend
```

A job is a group of steps that GitHub Actions performs.

This job is called:

```text
backend
```

and GitHub will run it on an Ubuntu machine.

This:

```yaml
defaults:
  run:
    working-directory: backend
```

means the commands in this job will run from the backend folder.

Without it, commands such as:

```bash
npm ci
```

would run from the root GameVault directory instead.

---

# 📥 6. Check out the repository

Add the first step:

```yaml
steps:

  - name: Checkout repository
    uses: actions/checkout@v4
```

GitHub Actions starts with a clean runner.

It therefore needs to obtain your repository before it can test anything.

The process is:

```text
GitHub creates runner
-> repository checked out
-> GameVault files available
-> remaining steps run
```

---

# 🟢 7. Install Node.js

Add:

```yaml
  - name: Setup Node
    uses: actions/setup-node@v4
    with:
      node-version: 22
      cache: npm
      cache-dependency-path: backend/package-lock.json
```

This installs Node.js on the runner.

We're also enabling npm caching so repeated workflow runs do not have to download everything from scratch unnecessarily.

The dependency path tells the action which lock file belongs to this job.

---

# 📦 8. Install backend dependencies

Add:

```yaml
  - name: Install dependencies
    run: npm ci
```

For CI environments, prefer:

```bash
npm ci
```

rather than:

```bash
npm install
```

`npm ci` installs the dependency versions recorded in `package-lock.json`.

This helps make the pipeline consistent.

The flow is:

```text
package.json
-> package-lock.json
-> npm ci
-> dependencies installed
```

Your `package-lock.json` should therefore be committed to GitHub.

---

# 🧹 9. Run ESLint

Add:

```yaml
  - name: Run ESLint
    run: npm run lint
```

This assumes the backend `package.json` already contains:

```json
"lint": "eslint ."
```

If ESLint finds an error that causes the command to exit unsuccessfully:

```text
npm run lint
-> error
-> step fails
-> backend job fails
```

This is exactly what we want.

The pipeline should not pretend everything is fine when a required quality check fails.

---

# 🧪 10. Run the backend tests

Add:

```yaml
  - name: Run tests
    run: npm test -- --runInBand
```

For Jest, `--runInBand` makes the tests run sequentially in one process. This can make CI behaviour simpler and easier to debug for a project at this scale.

If your existing test setup does not need it, this is also acceptable:

```yaml
  - name: Run tests
    run: npm test
```

Use whichever works correctly with the GameVault tests you created.

The backend job now does:

```text
checkout repository
-> setup Node
-> install dependencies
-> run ESLint
-> run automated tests
```

---

# ⚛️ Step 03 — Add the Frontend Pipeline

## 11. Create a separate frontend job

Do not put every check into one enormous job.

Add another job at the same indentation level as `backend`:

```yaml
  frontend:
    name: Frontend Tests
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: frontend
```

Now we have:

```text
GameVault CI

backend job
-> backend checks

frontend job
-> frontend checks
```

These jobs can run independently.

---

# 📥 12. Check out and configure Node

Inside the frontend job:

```yaml
    steps:

      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci
```

This is similar to the backend.

Remember that each job receives its own runner.

The frontend job cannot assume that the backend job already installed something for it.

---

# 🧹 13. Run frontend ESLint

Add:

```yaml
      - name: Run ESLint
        run: npm run lint
```

If your Vite project already had ESLint configured, this will use the configuration you previously set up.

---

# 🧪 14. Run React tests

Add:

```yaml
      - name: Run tests
        run: npm test -- --run
```

We use:

```text
--run
```

because Vitest normally supports watch behaviour during development.

A CI pipeline should:

```text
start tests
-> run tests once
-> report result
-> finish
```

It should not sit waiting for someone to edit a file.

If your frontend `package.json` contains:

```json
"test": "vitest"
```

then:

```bash
npm test -- --run
```

will run Vitest once.

---

# 🏗️ 15. Test the React production build

Tests passing does not automatically mean the application can build.

Add:

```yaml
      - name: Build frontend
        run: npm run build
```

Vite should create:

```text
frontend/dist
```

This checks whether the React application can actually produce its build.

Your frontend pipeline becomes:

```text
install dependencies
-> lint
-> test
-> build
```

If someone introduces a broken import or another build problem, GitHub Actions can catch it before the code is merged.

---

# 📄 Step 04 — Check the Complete CI File

At this stage, your `.github/workflows/ci.yml` should look similar to this:

```yaml
name: GameVault CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:

  backend:
    name: Backend Tests
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: backend

    steps:

      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          cache-dependency-path: backend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run tests
        run: npm test -- --runInBand


  frontend:
    name: Frontend Tests
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: frontend

    steps:

      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run tests
        run: npm test -- --run

      - name: Build frontend
        run: npm run build
```

Do not blindly copy this file if your scripts have different names.

Check your own:

```text
backend/package.json
frontend/package.json
```

and make sure the commands match.

---

# 📤 Step 05 — Push the Workflow to GitHub

## 16. Commit the workflow

From the GameVault repository:

```bash
git add .
git commit -m "ci: add github actions workflow"
git push
```

Once the workflow file reaches GitHub, GitHub should recognise it automatically.

You do not run `ci.yml` manually using Node.

GitHub Actions reads the workflow.

---

# 👀 17. View the workflow

Open your repository on GitHub.

Go to:

```text
Actions
```

You should see:

```text
GameVault CI
```

Open the workflow run.

You should see separate jobs similar to:

```text
Backend Tests
Frontend Tests
```

Open a job to see its individual steps.

For example:

```text
Backend Tests
-> Checkout repository
-> Setup Node
-> Install dependencies
-> Run ESLint
-> Run tests
```

A green tick means the job passed.

A red cross means something failed.

---

# ❌ Step 06 — Understand Pipeline Failures

## 18. Do not treat a failed pipeline as a GitHub problem

Suppose you see:

```text
Run tests
X Process completed with exit code 1
```

Open the failed step and read the output.

You might find:

```text
Expected: 200
Received: 500
```

That means:

```text
GitHub Actions worked
-> test ran
-> test found a problem
-> pipeline correctly failed
```

Do not delete the test just to make the pipeline green.

Fix the actual problem.

---

# 🧹 19. ESLint failures

You might see something like:

```text
'response' is assigned a value but never used
```

The workflow is telling you that:

```text
npm run lint
```

failed.

Reproduce it locally:

```bash
npm run lint
```

Fix the problem.

Then:

```bash
git add .
git commit -m "fix: resolve lint error"
git push
```

GitHub Actions runs again.

---

# 🧪 20. Test failures

If a test fails in GitHub but passes locally, compare the environments.

Check things such as:

```text
Node version
environment variables
database connection
file paths
case-sensitive filenames
test data
```

Linux file paths are case-sensitive.

For example:

```javascript
require("./Models/Game");
```

is not the same as:

```text
models/Game.js
```

on a Linux runner.

Something may work on Windows and then fail on GitHub's Ubuntu runner because the casing is wrong.

Fix the import rather than trying to work around the CI runner.

---

# 🔐 Step 07 — Environment Variables and GitHub Secrets

## 21. Do not commit secrets for CI

GameVault uses sensitive configuration such as:

```text
JWT_SECRET
MongoDB credentials
deployment credentials
```

Do not put real values directly into:

```text
ci.yml
```

For example, do not do this:

```yaml
env:
  JWT_SECRET: myactualsecret123
```

That secret would become part of the repository.

GitHub provides repository secrets for sensitive values.

---

# 🔑 22. Add a GitHub secret

In your GitHub repository, go to the repository's:

```text
Settings
-> Secrets and variables
-> Actions
```

Create a new repository secret.

For example:

```text
Name:
JWT_SECRET
```

The value should be your test/CI secret.

Do not use your real production secret for automated tests.

---

# ⚙️ 23. Use the secret in the workflow

You can make the secret available to the backend test step.

For example:

```yaml
      - name: Run tests
        run: npm test -- --runInBand
        env:
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
          NODE_ENV: test
```

The relationship is:

```text
GitHub repository secret
-> GitHub Actions
-> environment variable
-> process.env.JWT_SECRET
```

Your application still reads:

```javascript
process.env.JWT_SECRET
```

It does not need to know that GitHub Actions supplied the value.

---

# 🗄️ Step 08 — MongoDB in GitHub Actions

## 24. Tests should not use your real database

If your backend tests require MongoDB, do not connect your GitHub Actions workflow to the database you use for normal development.

Otherwise:

```text
pipeline runs
-> tests create users
-> tests create games
-> tests delete records
-> real development data changes
```

Automated tests should be isolated.

You have several ways of handling this.

For GameVault, one useful approach is to create a MongoDB service container inside the GitHub Actions job.

---

# 🍃 25. Add MongoDB as a service

Inside the backend job, before `defaults`, you can add:

```yaml
    services:

      mongodb:
        image: mongo:8

        ports:
          - 27017:27017
```

The beginning of the backend job becomes:

```yaml
  backend:
    name: Backend Tests
    runs-on: ubuntu-latest

    services:

      mongodb:
        image: mongo:8

        ports:
          - 27017:27017

    defaults:
      run:
        working-directory: backend
```

GitHub will start MongoDB for the job.

Your tests can then use a separate test database such as:

```text
mongodb://localhost:27017/gamevault_test
```

---

# 🧪 26. Provide the test database URI

Update the backend test step:

```yaml
      - name: Run tests
        run: npm test -- --runInBand
        env:
          NODE_ENV: test
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
          MONGODB_URI: mongodb://localhost:27017/gamevault_test
```

Use the environment-variable name that your GameVault backend actually expects.

Now:

```text
GitHub Actions backend job
-> starts MongoDB service
-> GameVault connects to gamevault_test
-> automated tests run
-> job ends
```

This keeps CI testing away from your normal database.

---

# 📮 Step 09 — Add Newman to the Pipeline

You previously created a Postman collection and used Newman to execute it from the terminal.

We can eventually add this to CI as well.

However, Newman requires the API to actually be running.

The flow needs to be:

```text
install backend
-> start MongoDB
-> start GameVault API
-> wait for API
-> run Newman
```

---

# 📦 27. Make sure Newman is installed

Inside the backend, you should already have installed:

```bash
npm install --save-dev newman
```

If not, install it and commit the updated:

```text
package.json
package-lock.json
```

---

# ▶️ 28. Start the API during CI

After the backend tests, you can start the application in the background.

For example:

```yaml
      - name: Start backend
        run: npm start &
        env:
          NODE_ENV: test
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
          MONGODB_URI: mongodb://localhost:27017/gamevault_test
```

The `&` means the process runs in the background so the workflow can continue to the next step.

However, there is a problem.

The next step could begin before GameVault has finished starting.

We therefore need to wait for the API.

---

# ⏳ 29. Wait for GameVault

A simple classroom approach is to use the health endpoint.

For example:

```yaml
      - name: Wait for backend
        run: |
          for i in {1..15}; do
            if curl -k https://localhost:4000/health; then
              exit 0
            fi

            sleep 2
          done

          exit 1
```

`curl` tries to access the GameVault health route.

`-k` is being used because GameVault currently uses a local self-signed HTTPS certificate.

The flow is:

```text
try health endpoint
-> not ready
-> wait 2 seconds
-> try again
-> backend responds
-> continue pipeline
```

If the API never starts, the step fails.

Change the port/path if your GameVault health route is different.

---

# 📮 30. Run Newman

Then add:

```yaml
      - name: Run Postman tests
        run: npx newman run ../postman/GameVault.postman_collection.json --insecure
```

Remember that the backend job's working directory is:

```text
backend
```

so:

```text
../postman
```

moves back to the GameVault root and then into the Postman directory.

If you also use an exported environment:

```yaml
      - name: Run Postman tests
        run: >
          npx newman run ../postman/GameVault.postman_collection.json
          -e ../postman/GameVault.postman_environment.json
          --insecure
```

Do not commit real JWT tokens or secrets inside the exported Postman environment.

---

# ⚠️ 31. Don't add Newman just because it exists

Your Jest/Supertest tests and Newman tests overlap slightly, but they serve different purposes.

For example:

```text
Jest + Supertest
-> automated backend tests
-> close to application code

Postman + Newman
-> execute API collection
-> verify saved API scenarios
```

Both can be useful.

But don't create 50 duplicate tests purely to increase the number of tests.

Focus on useful coverage.

---

# 🐳 Step 10 — Add Docker to CI

We previously created Dockerfiles for the backend and frontend.

A useful CI check is to verify that those Docker images can still be built.

This catches situations where:

```text
application works locally
-> Dockerfile is outdated
-> container build is broken
```

---

# 🏗️ 32. Add a Docker job

Add another job:

```yaml
  docker:
    name: Docker Build
    runs-on: ubuntu-latest

    needs:
      - backend
      - frontend
```

`needs` is important.

It creates this pipeline:

```text
backend tests ─┐
               -> Docker build
frontend tests ┘
```

The Docker job will not begin until both required jobs have passed.

If backend tests fail:

```text
backend fails
-> Docker job does not run
```

There is little value in preparing a container build for code that already failed its required tests.

---

# 📦 33. Build the Docker images

Add:

```yaml
    steps:

      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build backend image
        run: docker build -t gamevault-backend ./backend

      - name: Build frontend image
        run: docker build -t gamevault-frontend ./frontend
```

Now GitHub verifies that both Dockerfiles still work.

The CI flow becomes:

```text
backend
-> lint
-> tests

frontend
-> lint
-> tests
-> build

both pass
-> build backend Docker image
-> build frontend Docker image
```

---

# 🧩 34. Alternatively, validate Docker Compose

Because GameVault uses Docker Compose, you can also check whether the Compose configuration is valid:

```yaml
      - name: Check Docker Compose
        run: docker compose config
```

This catches certain YAML/configuration problems before deployment.

You could place it before the image builds:

```yaml
      - name: Check Docker Compose
        run: docker compose config

      - name: Build containers
        run: docker compose build
```

If your Compose configuration requires environment variables, provide suitable CI values rather than real production credentials.

---

# 🚀 Step 11 — Build a Proper Pipeline

Once the individual pieces are working, the GameVault pipeline should have clear stages.

A useful structure is:

```text
push / pull request
        |
        |
        -> backend checks
        |     -> install
        |     -> lint
        |     -> test
        |
        -> frontend checks
              -> install
              -> lint
              -> test
              -> build

backend + frontend pass
        |
        -> Docker build
        |
        -> application ready for deployment
```

The main idea is that later stages depend on earlier stages succeeding.

---

# 🚦 Step 12 — Understand Pipeline Gates

A pipeline acts as a gate.

Suppose a student makes a change that breaks login.

The flow becomes:

```text
push code
-> backend tests start
-> login test fails
-> backend job fails
-> pipeline becomes red
-> Docker/deployment stage blocked
```

This is better than:

```text
push broken login
-> deploy application
-> users discover login is broken
```

The purpose of CI/CD is not simply to show a green tick on GitHub.

The pipeline protects later stages from known-bad changes.

---

# 🌿 Step 13 — Use CI with Branches and Pull Requests

For a group project, avoid having everyone make uncontrolled changes directly to `main`.

A better workflow is:

```text
main
-> stable shared version

feature branch
-> individual/group change
```

For example:

```bash
git checkout -b feature/game-reviews
```

Make your changes.

Then:

```bash
git add .
git commit -m "feat: add game reviews"
git push -u origin feature/game-reviews
```

Create a pull request:

```text
feature/game-reviews
-> main
```

Because our workflow contains:

```yaml
pull_request:
  branches:
    - main
```

the CI pipeline runs against the pull request.

The team can then see:

```text
Backend Tests ✓
Frontend Tests ✓
Docker Build ✓
```

before merging.

---

# 🛡️ Step 14 — Protect `main`

If you control the repository settings, GitHub can be configured so that code cannot be merged into `main` until the required checks pass.

The idea is:

```text
pull request created
-> GitHub Actions runs
-> required checks pass
-> merge allowed
```

instead of:

```text
pull request created
-> tests failing
-> merge anyway
```

The exact GitHub options available can depend on the repository/account setup, but the principle is to use the CI checks as merge protection, not just decoration.

---

# 📦 Step 15 — Continuous Delivery

At this stage, we have:

```text
code
-> tests
-> lint
-> build
-> Docker build
```

This is already a useful CI pipeline.

The next stage is delivery.

A deployment platform may need:

```text
Docker image
```

or:

```text
frontend build
```

or:

```text
source repository
```

depending on how the application will eventually be hosted.

The important point is that deployment should come after the verification stages.

```text
code
-> test
-> lint
-> build
-> package
-> deploy
```

not:

```text
code
-> deploy immediately
-> hope it works
```

---

# 🔐 Step 16 — Deployment Secrets

When you eventually add deployment, you may need credentials such as:

```text
deployment token
cloud credentials
container registry credentials
production database connection
```

These must not be written directly into the workflow.

Bad:

```yaml
env:
  DEPLOYMENT_PASSWORD: Password123
```

Better:

```yaml
env:
  DEPLOYMENT_PASSWORD: ${{ secrets.DEPLOYMENT_PASSWORD }}
```

The real value stays inside GitHub's secret storage.

Your repository contains only the reference to the secret.

---

# 🌍 Step 17 — Keep Environments Separate

Do not use the same environment for everything.

A mature GameVault setup might eventually have:

```text
development
-> developers work locally

test
-> automated tests and CI

production
-> deployed application
```

For databases:

```text
gamevault
-> development database

gamevault_test
-> automated testing database

production database
-> deployed application
```

For JWT secrets:

```text
development secret
-> local development

CI secret
-> GitHub Actions

production secret
-> deployed application
```

Do not reuse the production credentials in your automated test pipeline.

---

# 🧪 Step 18 — Test the Pipeline Deliberately

Once your pipeline is working, deliberately make a harmless test fail.

For example, if a test expects:

```javascript
expect(response.statusCode).toBe(200);
```

temporarily change it to:

```javascript
expect(response.statusCode).toBe(999);
```

Commit and push the branch.

You should see:

```text
GitHub Actions
-> tests run
-> test fails
-> job becomes red
```

Then restore the correct test and push again.

You should see:

```text
GitHub Actions
-> tests run
-> tests pass
-> job becomes green
```

This proves that your CI pipeline is actually enforcing something.

Do not leave the intentionally broken test in the repository.

---

# 📊 Step 19 — Understand What a Successful Pipeline Means

A green pipeline does not mean:

> GameVault has no bugs.

It means:

> GameVault passed the checks we configured.

If your tests only check one endpoint, a green pipeline only tells you that those limited checks passed.

The usefulness of CI depends on the quality of the checks inside it.

As GameVault grows:

```text
application grows
-> test suite grows
-> pipeline checks more behaviour
-> confidence improves
```

---

# 📁 Step 20 — Final GameVault Structure

After adding GitHub Actions, your repository should roughly contain:

```text
GameVault
│
├── .github
│   └── workflows
│       └── ci.yml
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
│   ├── package.json
│   └── vite.config.js
│
├── postman
│   ├── GameVault.postman_collection.json
│   └── GameVault.postman_environment.json
│
├── compose.yaml
├── .env.example
└── .gitignore
```

---

# 🔄 Final GameVault CI/CD Flow

By the end of this section, you should understand this complete process:

```text
developer creates feature branch
-> code changed
-> tests run locally
-> ESLint run locally
-> changes committed
-> branch pushed to GitHub
-> pull request created

GitHub Actions starts
-> backend dependencies installed
-> backend linted
-> backend tests run

GitHub Actions
-> frontend dependencies installed
-> frontend linted
-> React tests run
-> frontend build checked

required checks pass
-> Docker configuration checked
-> Docker images build
-> application is ready for delivery/deployment

pull request reviewed
-> merge into main
```

## ✅ What you should have working

By the end, you should be able to demonstrate that:

```text
push code
-> GitHub Actions starts automatically

backend
-> dependencies install
-> ESLint runs
-> automated tests run

frontend
-> dependencies install
-> ESLint runs
-> React tests run
-> Vite build succeeds

API testing
-> GameVault can start in CI
-> Newman can execute the Postman collection

Docker
-> backend image builds
-> frontend image builds
-> Compose configuration is valid

failure
-> pipeline becomes red
-> later dependent stages are blocked

success
-> pipeline becomes green
-> code is ready for the next stage
```

The main goal is to move GameVault away from relying on:

> “I ran it on my computer and it seemed fine.”

and towards:

```text
change made
-> change tested automatically
-> code quality checked
-> application build verified
-> container build verified
-> only successful changes continue through the pipeline
```

That is the role of GitHub Actions and CI/CD in GameVault.
