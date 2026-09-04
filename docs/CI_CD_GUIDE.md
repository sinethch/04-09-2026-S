# CI/CD Pipeline & Multi-Environment Deployment Guide

This repository is configured with a robust GitHub Actions CI/CD pipeline supporting three isolated environments:
1. **QA (`qa`)**
2. **Staging (`staging`)**
3. **Main / Production (`main`)**

---

## 1. Branching & Deployment Strategy

```text
 feature/* or fix/*  ──(Pull Request)──> [CI Pipeline: Tests, Lint, Docker Build Check]
                                                        │
                  ┌─────────────────────────────────────┴─────────────────────────────────────┐
                  ▼                                     ▼                                     ▼
             [qa branch]                         [staging branch]                       [main branch]
                  │                                     │                                     │
                  ▼                                     ▼                                     ▼
        Deploy to QA (Auto)                  Deploy to Staging (Auto)             Deploy to Production (Auto/Approved)
```

| Environment | Trigger Branch | Primary Purpose | GitHub Environment Protection |
| :--- | :--- | :--- | :--- |
| **QA** | `qa` | Functional & QA validation | None (immediate auto-deploy) |
| **Staging** | `staging` | Pre-production validation & staging tests | Optional approval |
| **Main** | `main` | Production deployment | Recommended: Required reviewers |

---

## 2. GitHub Actions Workflows

### Continuous Integration (`.github/workflows/ci.yml`)
- **Trigger**: Pull requests targeting `qa`, `staging`, or `main`, or push to `feature/**`, `fix/**`, `hotfix/**`.
- **Backend Job**: Sets up Node.js 20, caches npm modules, installs dependencies via `npm ci`, and runs tests/syntax verification.
- **Frontend Job**: Sets up Node.js 20, caches npm modules, installs dependencies via `npm ci`, and executes `npm run build`.
- **Docker Verification Job**: Verifies both backend and frontend Dockerfiles build without error.

### Continuous Deployment (`.github/workflows/cd.yml`)
- **Trigger**:
  1. Direct push / merge to `qa`, `staging`, or `main`.
  2. Manual execution via `workflow_dispatch` with an environment selector (`qa`, `staging`, `main`).
- **Container Publishing**:
  - Automatically logs in to **GitHub Container Registry (`ghcr.io`)** using the built-in `GITHUB_TOKEN`.
  - Builds and tags Docker containers:
    - `ghcr.io/<owner>/<repo>/backend:<env>-<short_sha>`
    - `ghcr.io/<owner>/<repo>/backend:<env>-latest`
    - `ghcr.io/<owner>/<repo>/frontend:<env>-<short_sha>`
    - `ghcr.io/<owner>/<repo>/frontend:<env>-latest`
- **Environment Targeting**:
  - Deploys targeting the corresponding GitHub Environment (`qa`, `staging`, or `main`), utilizing that environment's configured secrets and variables.
  - Generates a step summary report on GitHub Actions.

---

## 3. Configuring GitHub Environments & Secrets

To leverage environment isolation, configure your environments in GitHub:

1. Navigate to your repository on GitHub: `https://github.com/<owner>/<repo>`
2. Go to **Settings** > **Environments**.
3. Create three environments:
   - `qa`
   - `staging`
   - `main`

### Recommended Protection Rules
- **For `main`**:
  - Check **Required reviewers** and assign designated tech leads or DevOps reviewers.
  - Set deployment branch rule to allow only the `main` branch.
- **For `staging`**:
  - Set deployment branch rule to allow only the `staging` branch.
- **For `qa`**:
  - Set deployment branch rule to allow only the `qa` branch.

### Environment Variables & Secrets Configuration

For each environment (`qa`, `staging`, `main`), add the corresponding secrets and variables (templates available in `.env.<env>.example`):

#### Environment Secrets
| Secret Name | Description |
| :--- | :--- |
| `MONGO_URI` | MongoDB connection string for the environment |
| `JWT_SECRET` | JWT signing secret for backend authentication |
| `DEPLOY_HOST` | Remote server hostname or IP (optional, for SSH deploys) |
| `DEPLOY_SSH_KEY` | SSH private key for remote server deployment (optional) |

#### Environment Variables
| Variable Name | Description | Example (QA) | Example (Staging) | Example (Main) |
| :--- | :--- | :--- | :--- | :--- |
| `APP_URL` | Application Frontend URL | `https://qa.example.com` | `https://staging.example.com` | `https://app.example.com` |
| `CLIENT_URL` | Allowed client origin in CORS | `https://qa.example.com` | `https://staging.example.com` | `https://app.example.com` |
| `PORT` | Backend service port | `5000` | `5000` | `5000` |

---

## 4. Manual Deployment Trigger

You can deploy any branch or commit to any environment manually at any time:
1. Go to **Actions** in your GitHub repository.
2. Select **CD Pipeline** from the left sidebar.
3. Click **Run workflow**.
4. Select the target branch and choose the environment (`qa`, `staging`, or `main`) from the dropdown.
5. Click **Run workflow**.

---

## 5. Local and Server Deployment with `docker-compose.deploy.yml`

To deploy the published containers on your target host:

```bash
# Set environment variables (or supply via .env.qa, .env.staging, etc.)
export BACKEND_IMAGE=ghcr.io/<owner>/<repo>/backend:qa-latest
export FRONTEND_IMAGE=ghcr.io/<owner>/<repo>/frontend:qa-latest
export MONGO_URI="mongodb://mongodb:27017/ecommerce"
export CLIENT_URL="https://qa.example.com"

# Pull latest images and restart services
docker compose -f docker-compose.deploy.yml pull
docker compose -f docker-compose.deploy.yml up -d
```
