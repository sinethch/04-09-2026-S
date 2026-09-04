# MERN Stack E-Commerce Application

A full-stack MERN (MongoDB, Express, React, Node.js) application with feature-based frontend and module-based backend architecture.

## Getting Started

### 1. Prerequisites
- **Node.js** (v18+)
- **MongoDB** (Local instance or MongoDB Atlas connection string in `backend/.env`)

### 2. Backend Setup
```bash
cd backend
npm install
npm run dev
```
Backend runs on `http://localhost:5000`.

### 3. Frontend Setup
```bash
cd frontend
npm install
npm run dev
```
Frontend runs on `http://localhost:5173`.

## Features
- **Products CRUD**:
  - Browse and search products by name
  - Filter products by category
  - Create new product with price and stock
  - Edit existing product
  - Delete product
- **Architecture**:
  - Controller-Service-Model MVC pattern for backend
  - Feature-oriented directory structure for frontend

## CI/CD & Environments
This project includes GitHub Actions CI/CD pipelines supporting three environments:
- **`qa`**: Automated testing and deployment for QA validation.
- **`staging`**: Pre-production staging environment.
- **`main`**: Production release with deployment protection rules.

See the complete setup and configuration instructions in [CI/CD Guide](docs/CI_CD_GUIDE.md).
