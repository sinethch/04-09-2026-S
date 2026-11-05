# MERN E-Commerce Platform

A full-stack e-commerce application and DevOps portfolio project. It pairs a React and Vite storefront with an Express and MongoDB API, and demonstrates containerized delivery through GitHub Actions, Docker, and GitHub Container Registry (GHCR).

> **Project focus:** Associate DevOps Engineer — CI/CD automation, container builds, environment-aware image publishing, and deployment workflow design.

## Application Preview

| Product catalog | Product management |
|:---:|:---:|
| ![E-commerce application screenshot 1](Screenshot%20(2).png) | ![E-commerce application screenshot 2](Screenshot%20(3).png) |

## What’s Implemented

### Application
- React 18 and Vite frontend with product catalog, search, category filtering, and product details.
- Express API backed by MongoDB and Mongoose, with product create, read, update, and delete endpoints.
- Health endpoint at `GET /api/health`.
- Modular backend organization using routes, controllers, services, and models.

### DevOps and delivery
- Separate Docker images for the frontend and backend, built from Node.js 20 Alpine base images.
- Docker Compose setup for running the app and MongoDB locally.
- GitHub Actions CI on pull requests to `qa`, `staging`, and `main`, and pushes to `feature/**`, `fix/**`, and `hotfix/**` branches.
- CI installs locked dependencies, checks backend JavaScript syntax, builds the frontend, and verifies both Docker images build.
- GitHub Actions CD on pushes to `qa`, `staging`, or `main`, with an option to start a workflow manually.
- Environment-tagged frontend and backend images published to GHCR, using both commit-specific and `*-latest` tags.
- Optional remote deployment over SSH when the deployment secrets are configured.

## Architecture

```text
React + Vite frontend  ──HTTP──>  Express API  ──>  MongoDB
         │                            │
  Docker image                  Docker image
         └────────── GitHub Actions ──┘
                CI → build/publish to GHCR → optional SSH deploy
```

The repository includes additional auth, cart, and order module files. The current Express app mounts the product API, and the current frontend router renders the product page; the modules not yet wired into those routes are not presented here as completed user-facing flows.

## Run Locally

### Prerequisites
- Node.js 20 or later and npm
- MongoDB, running locally or available through a connection string

### Start the backend

```bash
cd backend
npm ci
```

Set the backend environment variables in `backend/.env`:

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/ecommerce
CLIENT_URL=http://localhost:5173
```

Then run:

```bash
npm run dev
```

The API starts at `http://localhost:5000`. Check it at `http://localhost:5000/api/health`.

### Start the frontend

In a second terminal:

```bash
cd frontend
npm ci
npm run dev
```

Open `http://localhost:5173`. The Vite development server uses `http://localhost:5000/api` as its default API base URL; set `VITE_API_BASE_URL` if your API is hosted elsewhere.

## Run with Docker Compose

From the repository root, build and start MongoDB, the API, and the frontend:

```bash
docker compose up --build
```

The Compose configuration exposes the frontend at `http://localhost:5173` and the API at `http://localhost:5000`. Stop the services with `Ctrl+C`, or run `docker compose down`. MongoDB data is stored in the `mongo-data` named volume.

## CI/CD Environments

| Environment | Branch / trigger | Image tags |
|:--|:--|:--|
| QA | Push to `qa`, or manual workflow selection | `qa-<short-sha>`, `qa-latest` |
| Staging | Push to `staging`, or manual workflow selection | `staging-<short-sha>`, `staging-latest` |
| Main | Push to `main`, or manual workflow selection | `main-<short-sha>`, `main-latest` |

The CD workflow builds and pushes images to `ghcr.io/<owner>/<repository>/backend` and `ghcr.io/<owner>/<repository>/frontend`. Configure GitHub Environments and add the required deployment secrets to enable SSH deployment. Without SSH configuration, the workflow publishes the images and reports that remote deployment was skipped. Configure required reviewers and branch restrictions in GitHub Environment settings if approval gates are needed; these are repository settings rather than automatic guarantees of the workflow file.

See [the CI/CD guide](docs/CI_CD_GUIDE.md) for workflow details, environment configuration, GHCR image use, and deployment setup. The `.env.qa.example`, `.env.staging.example`, and `.env.main.example` files show the expected application configuration keys; replace example values with environment-specific values and keep real secrets out of Git.

## Repository Layout

```text
.github/workflows/    GitHub Actions CI and CD workflows
docs/                 CI/CD setup and deployment guide
backend/              Express API, MongoDB models, and Dockerfile
frontend/             React/Vite app and Dockerfile
docker-compose.yml    Local development stack
docker-compose.deploy.yml
                      Compose configuration for published images
```

## Technology Stack

- **Frontend:** React 18, Vite
- **Backend:** Node.js 20, Express 4
- **Database:** MongoDB, Mongoose
- **Containers:** Docker, Docker Compose
- **CI/CD and registry:** GitHub Actions, GitHub Container Registry

## Notes

- The backend `npm test` script currently performs a Node.js syntax check; it is not a comprehensive automated test suite.
- The CI workflow validates Docker builds but does not run container integration tests.
- Configure `CLIENT_URL`, `VITE_API_BASE_URL`, MongoDB connection strings, and deployment secrets for each target environment before deploying.

---

**Author:** Associate DevOps Engineer portfolio project
