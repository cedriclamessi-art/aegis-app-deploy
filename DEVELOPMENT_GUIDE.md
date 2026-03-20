# 🛠️ AEGIS SUPREMATIE - Development Guide

## 📋 Table of Contents
1. [Setup Local Development](#setup-local-development)
2. [Project Structure](#project-structure)
3. [API Routes](#api-routes)
4. [Database](#database)
5. [Authentication](#authentication)
6. [Twin Integration](#twin-integration)
7. [Deployment](#deployment)
8. [Troubleshooting](#troubleshooting)

---

## 🚀 Setup Local Development

### Prerequisites
```bash
- Node.js 18+
- PostgreSQL 16+
- Redis 7+
- Docker (optional, for containerization)
```

### Installation

1. **Extract the ZIP**
```bash
unzip aegis-app-dev.zip
cd aegis-app
```

2. **Install Dependencies**
```bash
npm install
# or
pnpm install
```

3. **Setup Environment**
```bash
cp .env.example .env.local
# Edit .env.local with your values
```

4. **Setup Database**
```bash
# Create PostgreSQL database
createdb aegis_dev

# Run Prisma migrations
npx prisma db push

# (Optional) Seed data
npx prisma db seed
```

5. **Start Development Server**
```bash
npm run dev
# Server runs on http://localhost:3000
```

---

## 📁 Project Structure

```
aegis-app/
├── src/
│   ├── pages/
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   │   ├── login.ts
│   │   │   │   ├── register.ts
│   │   │   │   └── me.ts
│   │   │   ├── health.ts
│   │   │   ├── products/
│   │   │   ├── campaigns/
│   │   │   └── twin/
│   │   ├── index.tsx (Landing)
│   │   ├── login.tsx
│   │   ├── signup.tsx
│   │   ├── dashboard.tsx
│   │   ├── analyzer.tsx
│   │   ├── brand-studio.tsx
│   │   └── billing.tsx
│   ├── components/
│   │   └── Layout.tsx
│   ├── lib/
│   │   ├── auth.ts (JWT utilities)
│   │   ├── db.ts (Database helpers)
│   │   └── config.ts (Pricing, guardrails)
│   └── styles/
│       └── globals.css
├── prisma/
│   └── schema.prisma
├── public/
├── .env.example
├── .env.local (local only)
├── package.json
├── tsconfig.json
├── tailwind.config.js
├── next.config.js
├── Dockerfile
├── docker-compose.yml
├── README.md
├── DEPLOYMENT_GUIDE.md
├── DEVELOPMENT_GUIDE.md
└── AEGIS_PRESENTATION_CORRECTED.md
```

---

## 🔌 API Routes

### Authentication

#### POST `/api/auth/register`
```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123",
    "name": "John Doe"
  }'
```

Response:
```json
{
  "user": {
    "id": 1,
    "email": "user@example.com",
    "name": "John Doe",
    "tier": 1,
    "isFounder": false
  },
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

#### POST `/api/auth/login`
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'
```

#### GET `/api/auth/me`
```bash
curl -X GET http://localhost:3000/api/auth/me \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Health Check

#### GET `/api/health`
```bash
curl http://localhost:3000/api/health
```

Response:
```json
{
  "status": "ok",
  "timestamp": "2026-03-19T10:30:00.000Z",
  "uptime": 3600
}
```

---

## 🗄️ Database

### Schema Overview

**Users Table**
```sql
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255),
  tier INT DEFAULT 1,
  isFounder BOOLEAN DEFAULT FALSE,
  subscriptionStatus VARCHAR(50) DEFAULT 'active',
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW() ON UPDATE NOW()
);
```

**Products Table**
```sql
CREATE TABLE products (
  id INT PRIMARY KEY AUTO_INCREMENT,
  userId INT NOT NULL,
  url VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  image VARCHAR(255),
  price DECIMAL(10, 2),
  radarScore INT,
  status VARCHAR(50) DEFAULT 'active',
  createdAt TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (userId) REFERENCES users(id)
);
```

**Campaigns Table**
```sql
CREATE TABLE campaigns (
  id INT PRIMARY KEY AUTO_INCREMENT,
  userId INT NOT NULL,
  productId INT NOT NULL,
  name VARCHAR(255) NOT NULL,
  objective VARCHAR(50),
  status VARCHAR(50) DEFAULT 'active',
  budget DECIMAL(10, 2),
  roas DECIMAL(10, 2),
  revenue DECIMAL(10, 2),
  createdAt TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (userId) REFERENCES users(id),
  FOREIGN KEY (productId) REFERENCES products(id)
);
```

### Migrations

```bash
# Generate migration from schema changes
npx prisma migrate dev --name add_new_field

# Apply pending migrations
npx prisma db push

# View migration status
npx prisma migrate status
```

---

## 🔐 Authentication

### JWT Implementation

**Token Structure**
```typescript
{
  userId: number,
  email: string,
  tier: number,
  isFounder: boolean,
  iat: number,
  exp: number
}
```

**Token Expiry**
- Default: 30 days
- Configurable in `src/lib/auth.ts`

### Protected Routes

```typescript
// Example: Protected API route
import { verifyToken } from '@/lib/auth';

export default async function handler(req, res) {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  const user = verifyToken(token);
  if (!user) {
    return res.status(401).json({ error: 'Invalid token' });
  }
  
  // Protected logic here
  res.status(200).json({ user });
}
```

---

## 🤖 Twin Integration

### Architecture

```
AEGIS Backend
    ↓
Execution Router
    ├─ API (priority 1)
    ├─ AEGIS Internal (priority 2)
    └─ Twin (priority 3 - fallback)
```

### Twin API Endpoints

#### POST `/api/twin/execute`
```bash
curl -X POST http://localhost:3000/api/twin/execute \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "action": "scrape_product",
    "params": {
      "url": "https://amazon.com/dp/B123456789"
    }
  }'
```

#### POST `/api/twin/publish`
```bash
curl -X POST http://localhost:3000/api/twin/publish \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "platform": "facebook",
    "content": {
      "image": "...",
      "copy": "..."
    }
  }'
```

### Twin Execution Rules

```typescript
// src/lib/executionRouter.ts

export async function executeAction(action, params, user) {
  // Rule 1: Try API first
  if (hasAPI(action)) {
    return await executeViaAPI(action, params);
  }
  
  // Rule 2: Try AEGIS internal
  if (isInternalCapable(action)) {
    return await executeInternal(action, params, user);
  }
  
  // Rule 3: Fall back to Twin
  if (isTwinCapable(action)) {
    return await executeTwin(action, params);
  }
  
  throw new Error(`Action ${action} not supported`);
}
```

---

## 🚀 Deployment

### Docker

**Build Image**
```bash
docker build -t aegis-app:latest .
```

**Run Container**
```bash
docker run -p 3000:3000 \
  -e DATABASE_URL="postgresql://..." \
  -e JWT_SECRET="..." \
  aegis-app:latest
```

### Docker Compose (Local)

```bash
docker-compose up -d
# Services: PostgreSQL, Redis, App
```

### Render Deployment

See `DEPLOYMENT_GUIDE.md` for step-by-step instructions.

---

## 🐛 Troubleshooting

### Database Connection Error
```
Error: connect ECONNREFUSED
```

**Solution:**
1. Check PostgreSQL is running
2. Verify `DATABASE_URL` in `.env.local`
3. Ensure database exists: `createdb aegis_dev`

### JWT Token Invalid
```
Error: Invalid token
```

**Solution:**
1. Check token hasn't expired
2. Verify `JWT_SECRET` matches
3. Re-login to get new token

### Twin Execution Failed
```
Error: Twin action failed
```

**Solution:**
1. Check Twin service is running
2. Verify Twin API key in `.env.local`
3. Check action is Twin-capable (not AEGIS-only)

### Build Fails
```
Error: Cannot find module 'next'
```

**Solution:**
```bash
rm -rf node_modules package-lock.json
npm install
npm run build
```

---

## 📝 Development Workflow

### 1. Create Feature Branch
```bash
git checkout -b feature/my-feature
```

### 2. Make Changes
```bash
# Edit files
# Test locally
npm run dev
```

### 3. Test
```bash
npm run test
npm run build
```

### 4. Commit & Push
```bash
git add .
git commit -m "feat: add my feature"
git push origin feature/my-feature
```

### 5. Create Pull Request
- Go to GitHub
- Create PR from feature branch to main
- Wait for CI/CD checks
- Merge when approved

---

## 🔄 Continuous Integration

### GitHub Actions

Automatically runs on every push:
1. Install dependencies
2. Run tests
3. Build project
4. Deploy to Render (if main branch)

---

## 📚 Useful Commands

```bash
# Development
npm run dev              # Start dev server
npm run build            # Build for production
npm start                # Start production server

# Database
npx prisma studio       # Open Prisma Studio (GUI)
npx prisma db push      # Apply migrations
npx prisma db seed      # Seed data

# Testing
npm run test             # Run tests
npm run test:watch      # Watch mode

# Linting
npm run lint             # Run ESLint
npm run format           # Format code with Prettier

# Docker
docker-compose up       # Start services
docker-compose down     # Stop services
docker-compose logs     # View logs
```

---

## 🎓 Next Steps

1. **Understand the Architecture**
   - Read `AEGIS_PRESENTATION_CORRECTED.md`
   - Review Twin integration section

2. **Setup Local Environment**
   - Follow "Setup Local Development" above
   - Test API endpoints

3. **Explore Codebase**
   - Check `src/pages/api/` for routes
   - Review `src/lib/` for utilities
   - Examine `prisma/schema.prisma` for data model

4. **Implement Features**
   - Add new API routes
   - Create new pages
   - Integrate Twin actions

5. **Deploy**
   - Follow `DEPLOYMENT_GUIDE.md`
   - Setup Render account
   - Deploy to production

---

## 📞 Support

- **GitHub Issues**: Report bugs and feature requests
- **Discord**: Join community for discussions
- **Email**: support@aegis-suprematie.com

---

*Last Updated: 2026-03-19*
