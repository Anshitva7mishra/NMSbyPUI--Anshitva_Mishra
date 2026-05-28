# PUIonNPM Backend System

![MongoDB](https://img.shields.io/badge/MongoDB-Database-green)
![Express](https://img.shields.io/badge/Express-Backend-black)
![React](https://img.shields.io/badge/React-Frontend-blue)
![Node.js](https://img.shields.io/badge/Node.js-20-green)
![Clerk](https://img.shields.io/badge/Auth-Clerk-purple)
![Status](https://img.shields.io/badge/status-development-yellow)

> PUIonNPM is a product of **ProjectUI**, designed and built by **Anshitva** (Founder) and **Ananya** (Co-Founder). It is an enterprise-grade MERN stack marketplace platform for distributing reusable backend systems, templates, and components — with engineer verification, revenue sharing, and admin moderation built in.

---

## Table of Contents

- [Overview](#overview)
- [System Roles](#system-roles)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Authentication & Authorization](#authentication--authorization)
- [Engineer Verification Workflow](#engineer-verification-workflow)
- [Project Upload Lifecycle](#project-upload-lifecycle)
- [Revenue & Payout Management](#revenue--payout-management)
- [API Reference](#api-reference)
- [Database Design](#database-design)
- [Background Jobs](#background-jobs)
- [Notifications](#notifications)
- [Security](#security)
- [Performance & Scalability](#performance--scalability)
- [Logging & Monitoring](#logging--monitoring)
- [Folder Structure](#folder-structure)
- [Environment Variables](#environment-variables)
- [Deployment](#deployment)
- [Future Scope](#future-scope)
- [Risks & Assumptions](#risks--assumptions)

---

## Overview

**PUIonNPM** is a scalable developer marketplace for distributing:

- Backend components (Login Systems, RBAC, Payment APIs)
- Full-stack templates (LMS, E-commerce, SaaS, Admin Panels)
- Reusable APIs and starter kits

The platform connects **engineers** who build and upload projects with **users** who purchase and download them, under strict **admin moderation** and a transparent **revenue-sharing model**.

---

## System Roles

| Role | Description |
|---|---|
| `USER` | Browses, purchases, and downloads resources |
| `ENGINEER` | Uploads templates/components and earns revenue |
| `ADMIN` | Manages platform operations, moderation, and payouts |

---

## Features

### User Features
- Clerk-based authentication (social login, magic link, MFA)
- Browse and search templates and components
- Purchase and download resources
- Wishlist and purchase history

### Engineer Features
- Membership request and verification workflow
- Secret key-based dashboard access
- Project upload and management
- Revenue dashboard, sales analytics, and payout history
- Re-validation requests after inactivity

### Admin Features
- Engineer approval and rejection
- Project moderation (approve, reject, disable)
- Revenue analytics and platform profit tracking
- Payout management
- Platform monitoring

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React.js |
| Backend | Node.js + Express.js |
| Database | MongoDB |
| ODM | Mongoose |
| Queue | BullMQ |
| Cache | Redis |
| Storage | AWS S3 |
| Authentication | Clerk |
| Documentation | Swagger |
| Deployment | Docker |

---

## Architecture

```
Client
   ↓
API Gateway
   ↓
Controllers
   ↓
Services
   ↓
Repositories
   ↓
Database (MongoDB)
```

**Base URL:** `/api/v1`

### Response Format

**Success:**
```json
{
  "success": true,
  "message": "Project uploaded successfully",
  "data": {}
}
```

**Error:**
```json
{
  "success": false,
  "message": "Unauthorized access",
  "error": {}
}
```

---

## Authentication & Authorization

### Auth Provider

Authentication is fully managed by **Clerk**. This includes registration, login, logout, session management, password reset, magic links, social OAuth, and MFA — all handled on the Clerk dashboard and SDK, with no custom auth endpoints required.

Clerk session tokens are verified server-side on protected routes using the Clerk Express middleware (`ClerkExpressRequireAuth`).

### Clerk-Protected Backend Middleware

```js
import { ClerkExpressRequireAuth } from '@clerk/clerk-sdk-node';

router.use('/api/engineer', ClerkExpressRequireAuth());
router.use('/api/admin', ClerkExpressRequireAuth());
```

### Role Permissions

| Role | Permissions |
|---|---|
| `USER` | Browse & Purchase |
| `ENGINEER` | Upload & Manage Projects |
| `ADMIN` | Full Platform Control |

---

## Engineer Verification Workflow

```
Engineer Registers
        ↓
Restricted Landing Page
        ↓
Submits Membership Request
        ↓
Admin Validates Submission
        ↓
Secret Key Generated & Sent
        ↓
Engineer Activates Dashboard
```

### Required Submission Fields

| Field | Required |
|---|---|
| Resume | Yes |
| Portfolio | Yes |
| GitHub Profile | Yes |
| Skills | Yes |
| Experience | Yes |
| Social Links | Yes |

### Engineer APIs

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/engineer/request-membership` | Submit membership request |
| `POST` | `/api/engineer/verify-secret` | Verify secret key |
| `POST` | `/api/engineer/revalidate` | Request re-validation |
| `GET` | `/api/engineer/dashboard` | Access engineer dashboard |

### Admin — Engineer Management APIs

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/admin/engineer-requests` | View all pending requests |
| `POST` | `/api/admin/approve-engineer` | Approve engineer |
| `POST` | `/api/admin/reject-engineer` | Reject engineer |
| `POST` | `/api/admin/generate-secret-key` | Generate secret key |
| `POST` | `/api/admin/disable-engineer` | Disable engineer |

### Auto-Invalidation Rule

> If an engineer uploads **no project for 60 days**, their secret key becomes invalid, dashboard access is blocked, and their status is set to **inactive**.

**Re-Validation Flow:**
```
Engineer Requests Re-Validation
            ↓
Admin Receives Notification
            ↓
Admin Reviews Activity
            ↓
Engineer Reactivated
```

---

## Project Upload Lifecycle

### Upload Flow

```
Engineer Uploads Project
        ↓
Status: PENDING
        ↓
Admin Reviews
        ↓
APPROVED → Visible on User Portal
REJECTED → Engineer Notified
```

### Project Status Types

| Status | Meaning |
|---|---|
| `PENDING` | Awaiting admin review |
| `APPROVED` | Live and visible to users |
| `REJECTED` | Rejected by admin |
| `DISABLED` | Hidden from marketplace |

### Engineer — Project APIs

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/engineer/projects` | Upload a new project |
| `PUT` | `/api/engineer/projects/:id` | Update a project |
| `DELETE` | `/api/engineer/projects/:id` | Delete a project |
| `GET` | `/api/engineer/projects` | List own projects |

### Admin — Project APIs

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/admin/projects/pending` | View pending projects |
| `POST` | `/api/admin/projects/:id/approve` | Approve a project |
| `POST` | `/api/admin/projects/:id/reject` | Reject a project |

### User — Project APIs

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/projects` | Browse all projects |
| `GET` | `/api/projects/:id` | View project details |
| `GET` | `/api/templates` | Browse templates |
| `GET` | `/api/components` | Browse components |

---

## Revenue & Payout Management

### Revenue Split

| Entity | Share |
|---|---|
| Engineer | 80% |
| Platform | 20% |

### Engineer — Revenue APIs

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/engineer/revenue` | Total revenue overview |
| `GET` | `/api/engineer/sales` | Sales history |
| `GET` | `/api/engineer/payouts` | Payout history |

### Admin — Revenue APIs

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/admin/revenue` | Platform-wide revenue |
| `GET` | `/api/admin/platform-profit` | Platform profit report |
| `POST` | `/api/admin/release-payout` | Release engineer payout |

### Purchase APIs

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/purchase/create-order` | Create a new order |
| `POST` | `/api/purchase/verify-payment` | Verify payment |
| `GET` | `/api/purchase/history` | View purchase history |

---

## API Reference

Full API documentation is available via **Swagger** at `/api/docs` when the server is running.

---

## Database Design

### User Collection

| Field | Type |
|---|---|
| `_id` | ObjectId |
| `name` | String |
| `email` | String |
| `password` | String (hashed) |
| `role` | Enum (`USER`, `ENGINEER`, `ADMIN`) |
| `purchases` | Array |
| `wishlist` | Array |

### Engineer Collection

| Field | Type |
|---|---|
| `userId` | ObjectId (ref: User) |
| `secretKey` | String |
| `validationStatus` | Enum |
| `isValidated` | Boolean |
| `totalRevenue` | Number |
| `totalSales` | Number |
| `github` | String |
| `portfolio` | String |
| `resume` | String |

### Project Collection

| Field | Type |
|---|---|
| `title` | String |
| `slug` | String |
| `description` | String |
| `category` | String |
| `type` | Enum (`COMPONENT`, `TEMPLATE`) |
| `tags` | Array |
| `price` | Number |
| `engineerId` | ObjectId (ref: Engineer) |
| `status` | Enum |
| `salesCount` | Number |

### Purchase Collection

| Field | Type |
|---|---|
| `userId` | ObjectId (ref: User) |
| `projectId` | ObjectId (ref: Project) |
| `paymentId` | String |
| `totalAmount` | Number |
| `engineerShare` | Number |
| `platformShare` | Number |

---

## Background Jobs

| Job | Purpose |
|---|---|
| Inactivity Scanner | Disables engineers inactive for 60+ days |
| Revenue Settlement | Processes monthly payouts |
| Notification Queue | Dispatches email notifications |
| Cleanup Jobs | Removes temporary uploaded files |

---

## Notifications

### Channels

| Type | Description |
|---|---|
| Email | Mail notifications via queue |
| In-App | Dashboard notification center |
| WebSocket | Real-time push notifications |

### Events

- Engineer Approved / Rejected
- Project Approved / Rejected
- Payout Released
- Purchase Successful
- Re-Validation Requested

---

## Security

| Layer | Implementation |
|---|---|
| Password Hashing | bcrypt |
| Authentication | Clerk |
| API Protection | Rate Limiting |
| HTTP Headers | Helmet |
| File Upload Validation | MIME Type Validation |
| CORS | Enabled |
| Input Sanitization | XSS Protection |

---

## Performance & Scalability

### Targets

| Metric | Target |
|---|---|
| API Response Time | < 300ms |
| Concurrent Users | 10,000+ |
| Uptime | 99.9% |
| Upload Capacity | 1,000+ uploads/day |

### Strategies

- Horizontal scaling support
- Redis caching layer
- CDN integration for file delivery
- BullMQ queue-based background processing
- MongoDB indexing and lazy loading
- Cursor-based pagination

---

## Logging & Monitoring

| Tool | Purpose |
|---|---|
| Winston | Application logs |
| Morgan | HTTP request logs |
| Sentry | Error monitoring |
| Prometheus | Metrics collection |
| Grafana | Metrics visualization |

---

## Folder Structure

```
/backend
│
├── src
│   ├── modules         # Feature modules
│   ├── controllers     # Route handlers
│   ├── services        # Business logic
│   ├── repositories    # Database access layer
│   ├── middlewares     # HTTP middlewares
│   ├── guards          # Auth & role guards
│   ├── interceptors    # Response interceptors
│   ├── validations     # DTO validators
│   ├── jobs            # Background job processors
│   ├── socket          # WebSocket gateway
│   ├── config          # App configuration
│   └── utils           # Shared utilities
│
├── uploads             # Temporary file storage
├── tests               # Unit & integration tests
├── docs                # API documentation
├── package.json
└── README.md
```

---

## Environment Variables

```env
PORT=5000

MONGO_URI=

JWT_SECRET=
JWT_REFRESH_SECRET=

REDIS_URL=

AWS_ACCESS_KEY=
AWS_SECRET_KEY=

RAZORPAY_KEY=
RAZORPAY_SECRET=
```

---

## Deployment

### Infrastructure

| Layer | Technology |
|---|---|
| Containerization | Docker |
| Reverse Proxy | Nginx |
| Process Manager | PM2 |
| CI/CD | GitHub Actions |
| Hosting | AWS / VPS |

### Quick Start

```bash
# Clone the repository
git clone "<Yaha humare Laala and Laaliyo hum apni actual project repo daalenge>"
cd puionnpm-backend

# Install dependencies
npm install

# Copy environment variables
cp .env.example .env

# Run in development
npm run start:dev

# Run with Docker
docker-compose up --build
```

---

## Future Scope

- AI Code Review on uploads
- Subscription Plans for users
- Public API Marketplace
- Plugin Store
- Team Collaboration features
- AI-powered Search and Recommendations
- Version Comparison for projects

---

## Risks & Assumptions

### Risks
- Fake or malicious engineer applications
- Malware embedded in uploaded templates
- Payment fraud attempts
- High cloud storage consumption

### Assumptions
- Stable cloud infrastructure is available
- Payment gateway (Razorpay) remains operational
- Engineers provide genuine application data

---

**PUIonNPM** is a product of [ProjectUI](https://projectui.in).
Conceived and built by **Anshitva** (Founder) and **Ananya** (Co-Founder).

