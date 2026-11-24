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

> Update this table to match the final decisions.

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
├── api-gateway/
├── services/
│   ├── identity-user-service/
│   ├── salon-operations-service/
│   ├── billing-promotions-service/
│   ├── inventory-service/
│   ├── engagement-service/
│   └── analytics-service/
├── shared/                # Event schemas, common utils and types
├── docker-compose.yml
├── .env.example
└── README.md
```

Each service follows the same layout: `src/` (modules), `migrations/`, `tests/`, `Dockerfile`.
Each module follows: `controller → service → repository`, plus `routes`, `model` and `dto/`.

## 6. Getting Started

### Prerequisites

- [Runtime, e.g. Node.js 20+] 
- Docker and Docker Compose
- Git

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

```bash
[unit test command]
[integration test command]
```

All new code must include tests and pass before merging.

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

- **Branches:** `main` (stable), `develop` (integration), `feature/<module>-<short-name>`, `bugfix/<short-name>`
- **Commits:** `type(scope): message`, for example `feat(offers): add coupon validation`
- **Pull requests:** at least one reviewer, tests passing, no direct pushes to `main`
- **Boundaries:** a service must not read another service's database. Use APIs or events.
- **Naming:** services in `kebab-case`, tables in `snake_case`, API paths in plural nouns (`/appointments`)

## 12. Team and Contact

| Role | Name | Contact |
|---|---|---|
| Project Manager | safwan | zorrowtech@gmail.com |
| Tech Lead | safwan |  zorrowtech@gmail.com |
| Backend Team | suhaila thasni | #salon-backend (Slack) |

---

© 2026 zorrowtech. All rights reserved.
