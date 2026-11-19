# School Management System — MEAN Stack

A deployable starter project using **MongoDB, Express, Angular and Node.js**.

## Included
- Dashboard summary
- Student management (create, list, search, delete)
- Class management (create and list)
- Attendance recording and listing
- JWT login with a seeded administrator
- MongoDB connection through `MONGODB_URI`
- Azure App Service deployment configuration and GitHub Actions workflow

> This is a learning/demo starter. Before production, add password reset, stronger validation, audit logs, pagination, role-based permissions, backups, monitoring and security review.

## Requirements
- Node.js 20 or 22
- npm
- MongoDB Atlas database (recommended) or local MongoDB for development
- Azure subscription for deployment

## 1. Configure MongoDB and backend

Create a MongoDB Atlas cluster and database user. Add your development IP to Atlas Network Access.

```bash
cd backend
cp .env.example .env
```

Edit `backend/.env` and set:
- `MONGODB_URI` — Atlas connection string
- `JWT_SECRET` — a long random secret
- `ADMIN_EMAIL` and `ADMIN_PASSWORD` — initial administrator credentials

Install and run:

```bash
npm install
npm run seed
npm run dev
```

API runs at `http://localhost:5000`. Health check: `http://localhost:5000/api/health`.

## 2. Run Angular frontend

In a second terminal:

```bash
cd frontend
npm install
npm start
```

Open `http://localhost:4200`. The Angular development proxy forwards `/api` requests to port 5000.

The demo administrator is the email/password configured in `backend/.env`. Do not use the sample credentials in a real environment.

## 3. Build frontend for Azure

Set `frontend/src/environments/environment.ts` `apiUrl` to `/api` (already configured).

```bash
cd frontend
npm run build
```

The Angular production output is copied into `backend/public` by the build script. The Express server serves the Angular app and API from the same Azure App Service.

## 4. Deploy to Azure App Service

Recommended simple architecture:

- One Azure App Service (Linux, Node 20/22)
- MongoDB Atlas (or Azure Cosmos DB for MongoDB, after compatibility testing)
- Angular static assets served by Express from the same App Service

### Azure App Service configuration

Create a Linux Web App with Node 20 or 22. In **App Service → Settings → Environment variables**, add:

| Name | Value |
|---|---|
| `NODE_ENV` | `production` |
| `MONGODB_URI` | Your Atlas connection string |
| `JWT_SECRET` | Long random secret |
| `ADMIN_EMAIL` | Administrator email |
| `ADMIN_PASSWORD` | Strong initial password |
| `WEBSITE_NODE_DEFAULT_VERSION` | `~20` |

Do not commit `.env` or secrets to Git.

### Deploy manually with ZIP

From the project root after building Angular:

```bash
cd frontend
npm install
npm run build
cd ..
cd backend
npm install --omit=dev
cd ..
```

Create a deployment ZIP containing the **contents** of the `backend` folder, including `public/`, `package.json` and `package-lock.json`. Deploy it using Azure App Service Deployment Center or:

```bash
az webapp deploy --resource-group YOUR_RESOURCE_GROUP \
  --name YOUR_WEB_APP_NAME --src-path backend-deploy.zip --type zip
```

Set the App Service startup command to:

```bash
npm start
```

The backend listens on `process.env.PORT`, as required by App Service.

### GitHub Actions
The included `.github/workflows/azure-deploy.yml` expects:
- GitHub secret `AZURE_WEBAPP_PUBLISH_PROFILE`
- GitHub variable `AZURE_WEBAPP_NAME`

Download the publish profile from your App Service and add it as a GitHub secret. Review the workflow and change the Node version if your App Service uses another supported version.

## API endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/auth/login` | Login and receive JWT |
| GET | `/api/dashboard` | Dashboard counts |
| GET/POST | `/api/students` | List/create students |
| DELETE | `/api/students/:id` | Delete student |
| GET/POST | `/api/classes` | List/create classes |
| GET/POST | `/api/attendance` | List/record attendance |
| GET | `/api/health` | Health check |

All endpoints except login and health require `Authorization: Bearer <token>`.

## Important deployment notes
- Configure MongoDB Atlas Network Access for Azure outbound connectivity. For a learning project, allowing access broadly is convenient but less secure; use appropriate network restrictions for production.
- Use App Service HTTPS and store secrets only in App Service settings or GitHub secrets.
- This starter uses one App Service for simplicity. For larger systems, host Angular separately (for example Azure Static Web Apps) and the API in App Service.
