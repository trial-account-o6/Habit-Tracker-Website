# Habit-Tracker-Website

Monorepo boilerplate for a Habit Tracker web app.

## Project structure

- `frontend/` — React + Vite + TypeScript
- `backend/` — Express + TypeScript

## Prerequisites

- Node.js 20+
- npm 10+

## Local development setup

### 1) Clone the repository

```bash
git clone https://github.com/trial-account-o6/Habit-Tracker-Website.git
cd Habit-Tracker-Website
```

### 2) Install dependencies

Install all workspace dependencies from the repo root:

```bash
npm install
```

### 3) Start the app in development mode

Run frontend and backend together from the root:

```bash
npm run dev
```

Services:

- Frontend: http://localhost:5173
- Backend health endpoint: http://localhost:3001/health

### 4) Build the project

```bash
npm run build
```

## Useful workspace commands

Run frontend only:

```bash
npm run dev -w frontend
```

Run backend only:

```bash
npm run dev -w backend
```

Run tests:

```bash
npm run test
```
