# CI/CD Pipeline Practice Project

A hands-on project built to learn and demonstrate an end-to-end CI/CD workflow from an Associate DevOps Engineer perspective. A small React and Express product-management application provides the workload; the main focus is automating validation, container builds, image publishing, and deployment across QA, staging, and main environments.

The `main` deployment shown below completed successfully, and the application was opened from the deployed server.

## Deployment Evidence

### Successful GitHub Actions deployment to `main`

The workflow completed its environment selection, container image build and publish, and deployment job successfully. The run summary records the commit and the backend and frontend image tags published to GHCR.

![Successful GitHub Actions CD workflow run for main](Screenshot%20(2).png)

### Application running on the deployed host

The product-management application was opened using the server address after deployment.

![Product management application running on the deployed host](Screenshot%20(3).png)

## What This Project Demonstrates

- **Continuous integration:** GitHub Actions runs on pull requests targeting `qa`, `staging`, and `main`, and on pushes to `feature/**`, `fix/**`, and `hotfix/**` branches.
- **Application checks:** CI installs dependencies with `npm ci`, runs the backend's current syntax-check script, and creates a frontend production build.
- **Container validation:** CI builds both Docker images without publishing them.
- **Continuous delivery:** Pushes to `qa`, `staging`, and `main` start the CD workflow; manual runs can select an environment.
- **Image publishing:** The workflow authenticates to GHCR with `GITHUB_TOKEN` and publishes backend and frontend images with environment and commit-specific tags, plus environment `latest` tags.
- **Deployment workflow:** The deployment job targets the matching GitHub Environment. When SSH deployment secrets are configured, it connects to the host, pulls the images, and runs Docker Compose. The successful run captured above reached a successful deployment job, and the application screenshot shows the app running on the host. The workflow's final “health checks” step currently prints a status message; it does not make an HTTP request to the deployed application. The screenshot is separate evidence that the application was opened on the host.
- **Operational hygiene:** Workflow concurrency is configured, registry permissions are scoped to package publishing, and example environment files are provided without real credentials.

## Delivery Flow

```text
Pull request / feature branch
             │
             ▼
  Backend syntax check + frontend build
             │
             ▼
  Build backend and frontend images
             │
             ▼
Push to qa / staging / main (or manual run)
             │
             ▼
  Build and publish environment-tagged images to GHCR
             │
             ▼
  Deploy with Docker Compose over SSH
             │
             ▼
  Application available on the deployment host
```

## Application Workload

The application is a small product-management interface used to exercise the delivery pipeline:

- React 18 and Vite frontend with product listing, name search, category filtering, and add, edit, and delete actions.
- Node.js 20 and Express API with product CRUD endpoints and a health endpoint at `GET /api/health`.
- MongoDB persistence through Mongoose.
- Separate backend and frontend Dockerfiles, plus Compose files for local development and deployment.

The repository contains additional authentication, cart, and order modules, but those modules are not mounted in the current Express app or frontend router. The current running application flow is product management.

## Technology Used

| Area | Tools |
|:--|:--|
| CI/CD | GitHub Actions |
| Containers | Docker, Docker Compose |
| Container registry | GitHub Container Registry (GHCR) |
| Frontend | React 18, Vite |
| API | Node.js 20, Express 4 |
| Database | MongoDB, Mongoose |
| Environments | QA, staging, main |

## CI and CD Configuration

### Continuous integration

Defined in [`.github/workflows/ci.yml`](.github/workflows/ci.yml):

1. Backend job uses Node.js 20, installs locked dependencies, and runs `npm test` (currently `node --check src/server.js`).
2. Frontend job installs locked dependencies and runs `npm run build`.
3. Docker job builds backend and frontend images as validation; it does not push them.

### Continuous delivery

Defined in [`.github/workflows/cd.yml`](.github/workflows/cd.yml):

- Pushes to `qa`, `staging`, and `main` select the corresponding environment. A manual workflow run also allows choosing one.
- Images are published using tags such as `<environment>-<short-sha>` and `<environment>-latest`.
- Image names follow `ghcr.io/<owner>/<repository>/backend` and `ghcr.io/<owner>/<repository>/frontend`.
- SSH deployment uses `SSH_HOST`, `SSH_USER`, `SSH_KEY`, and optionally `SSH_PORT` and `DEPLOY_PATH`. Configure them as repository or environment secrets to deploy to a remote host.
- Without `SSH_HOST`, the workflow publishes images but skips remote deployment. GitHub Environment approval rules and branch restrictions must be configured in repository settings if desired.
- The workflow's final deployment verification step is informational only and does not perform a remote HTTP health check.

For the full setup and deployment configuration, see the [CI/CD guide](docs/CI_CD_GUIDE.md). The `.env.qa.example`, `.env.staging.example`, and `.env.main.example` files are configuration examples; use real environment-specific values outside source control.

## Run Locally

### Requirements

- Node.js 20 or later and npm
- MongoDB locally or a MongoDB connection string

### Start the API

```bash
cd backend
npm ci
```

Create `backend/.env` with local values:

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/ecommerce
CLIENT_URL=http://localhost:5173
```

Then start the API:

```bash
npm run dev
```

The API is available at `http://localhost:5000`; check `http://localhost:5000/api/health` for its health response.

### Start the frontend

In a second terminal:

```bash
cd frontend
npm ci
npm run dev
```

Open `http://localhost:5173`. Vite proxies `/api` requests to the local backend.

### Start the full local stack with Docker Compose

From the repository root:

```bash
docker compose up --build
```

The frontend is available at `http://localhost:5173`, and the API at `http://localhost:5000`. MongoDB data is stored in the `mongo-data` volume.

## Repository Structure

```text
.github/workflows/     CI and CD workflow definitions
backend/               Express API and backend Dockerfile
frontend/              React/Vite application and frontend Dockerfile
docs/                  CI/CD and deployment guide
docker-compose.yml     Local development stack
docker-compose.deploy.yml
                       Deployment stack using published images
.env.*.example         QA, staging, and main configuration examples
```

## Learning Outcomes

This project gave me practical experience with CI/CD workflow design, GitHub Actions, Docker image builds, GHCR publishing, environment-specific tags, deployment secrets, and running a Compose-based application on a remote host. It also highlighted the distinction between a successful image publishing job and a completed remote deployment, and the value of confirming deployment with the running application.

---

**Role focus:** Associate DevOps Engineer | CI/CD, Docker, GitHub Actions, GHCR
