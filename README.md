# Zorrow Salon – Backend

> Internal project of **Zorrow Tech**. Confidential. Do not share outside the organization.

A microservice-based backend that powers salon operations: appointments, services, staff and customers, billing, offers, inventory, notifications, reviews and reports.

| | |
|---|---|
| **Project owner** | Zorrow Tech |
| **Status** | In development (Phase 3 – Documentation) |
| **Repository** | https://github.com/zorrowtechitsolutions/saloonBackend |
| **Last updated** | 2026-11-24 |

---

## 1. Overview

The Salon Management System replaces manual booking and billing with a single platform used by three groups:

- **Admins** manage services, staff, offers and reports.
- **Staff** (stylists, receptionists) manage schedules and appointments.
- **Customers** book appointments, pay, use offers and leave reviews.

This repository contains the backend only. It exposes REST APIs through a single API gateway.

## 2. Key Features

- Role-based access control (Super admin, Admin, staff, user)
- Appointment booking and stylist scheduling
- Service  management
- Invoicing and payments (Cash, Card, UPI)
- Offers, coupons and redemption tracking
- Inventory, suppliers and purchase orders
- SMS/Email notifications and reviews
- Business reports and performance metrics

## 3. Architecture Summary

The system uses **5 consolidated microservices**, each owning its own database. Services communicate through the API gateway (synchronous) and events (asynchronous).

| Service | Responsibility |
|---|---|
| `identity-user-service` | Users, roles, permissions, staff and customer profiles |
| `salon-operations-service` |  services, appointments, schedules |
| `billing-promotions-service` | Invoices, payments, offers, coupons |
| `inventory-analytics-service` | Products, stock levels, suppliers, purchase orders then  Reports and metrics built from events |
| `engagement-service` | Notifications, templates, reviews, moderation |

Details: [architecture.md](./architecture.md)

## 4. Tech Stack

| Layer | Technology |
|---|---|
| Language / Runtime | Node.js ,Typescript  |
| Framework | Express.js |
| Database |  PostgreSQL  |
| ORM |  Sequelize |
| Password Hashing | bcrypt  |
| Real-time Communication|  Socket.IO |
| Caching / Session Store |  Redis |
| Hosting |  DigitalOcean |
| Message Broker |  RabbitMQ  |
| API Gateway/Reverse Proxy |  Nginx  |
| Authentication | JWT |
| Version Control | Git, GitHub |
| Containerization | Docker, Docker Compose |
| File Storage | DigitalOcean Spaces (S3-compatible) |
| Testing | postman |

## 5. Project Structure

```text
salon-backend/
├── services/
│   ├── api-gateway/
│   ├── identity-user-service/
│   ├── salon-operations-service/
│   ├── billing-promotions-service/
│   ├── inventory-service/
│   ├── engagement-service/
│   └── analytics-service/
├── docs/
│   ├── requirements.md
│   ├── architecture.md
│   ├── modules.md
│   ├── database-design.md
│   ├── api-documentation.md
│   ├── business-rules.md
│   └── development-plan.md
├── scripts/
│   └── backup/                # Database backup scripts
├── .github/
│   └── workflows/             # CI/CD pipelines
├── docker-compose.yml         # Local development
├── docker-compose.demo.yml    # Demo environment
├── docker-compose.prod.yml    # Production
├── .env.example
└── README.md
```


## 6. Getting Started

### Prerequisites
## 6. Getting Started

### Prerequisites

- Node.js 20 or higher
- npm (installed together with Node.js)
- Docker and Docker Compose
- Git
- Postman (optional, for testing APIs)

### Setup

```bash
# 1. Clone the repository
git clone [Repo URL]
cd salon-backend

# 2. Create your environment file
cp .env.example .env

# 3. Start all services
docker-compose up --build

# 4. Run database migrations 
[migration command]
```

### Verify

Open `http://localhost:3000/health`. All services should return `OK`.

## 7. Environment Variables

| Variable | Description |
|---|---|
| `DB_URL_*` | Database connection string for each service |
| `JWT_SECRET` | Secret used to sign tokens |
| `MESSAGE_BROKER_URL` | Event broker connection |
| `SMS_API_KEY` / `EMAIL_API_KEY` | Notification providers |

Never commit real secrets. Use `.env.example` for placeholders only.

## 8. Running Tests

APIs are tested with Postman. Import the collection from `docs/postman/` and run it against your local environment.

All new APIs must be tested in Postman and pass before merging.

## 9. API Access

- Base URL (local): `http://localhost:3000/api/v1`
- All requests go through the API gateway.
- Protected endpoints need the header `Authorization: Bearer <token>`.

Full endpoint list: [api-documentation.md](./api-documentation.md)

## 10. Project Documentation

| File | Purpose |
|---|---|
| [requirements.md](./requirements.md) | Functional and non-functional requirements |
| [architecture.md](./architecture.md) | Backend architecture |
| [modules.md](./modules.md) | Module descriptions |
| [database-design.md](./database-design.md) | Tables and relationships |
| [api-documentation.md](./api-documentation.md) | API endpoints |
| [business-rules.md](./business-rules.md) | Salon-specific rules |
| [development-plan.md](./development-plan.md) | Development phases and tasks |

Read these before writing code.

## 11. Development Guidelines

- **Branches:**
  - `main`: latest tested code, where all work is merged first
  - `demo`: code from `main` deployed to the demo environment for review
  - `production`: code from `demo` deployed to the live environment
  - `feature/<module>-<short-name>`: new features, branched from `main`
  - `bugfix/<short-name>`: bug fixes, branched from `main`
- **Release flow:** `feature/*` → `main` → `demo` → `production`. Code only moves forward in this order, and never goes to `production` without passing through `demo`.
- **Commits:** `type(scope): message`, for example `feat(offers): add coupon validation`
- **Pull requests:** at least one reviewer and passing tests. No direct pushes to `main`, `demo` or `production`.
- **Boundaries:** a service must not read another service's database. Use APIs or events.
- **Naming:** services in `kebab-case`, tables in `snake_case`, API paths in plural nouns (`/appointments`)

## 12. Team and Contact

| Role | Name | Contact |
|---|---|---|
| Project Manager / Tech Lead | Safwan | zorrowtech@gmail.com |
| Backend Developer | Suhaila Thasni | suhailathasni0@gmail.com |
| Backend Team Channel | - | #salon (Pumble) |

---

© 2026 zorrowtech. All rights reserved.
