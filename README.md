# Salon Management System – Backend

A microservice-based backend for managing salon operations: appointments, services, staff, customers, billing, offers, inventory, notifications, reviews and reports.

![Status](https://img.shields.io/badge/status-in%20development-yellow)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## Table of Contents
1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Architecture](#architecture)
4. [Tech Stack](#tech-stack)
5. [Project Structure](#project-structure)
6. [Getting Started](#getting-started)
7. [Environment Variables](#environment-variables)
8. [Running Tests](#running-tests)
9. [API Access](#api-access)
10. [Documentation](#documentation)
11. [Contributing](#contributing)
12. [License](#license)

---

## Overview
The Salon Management System provides a REST API for three types of users:

- **Admin:** manages branches, staff, services, offers and reports
- **Staff:** manages their schedule and appointments
- **Customer:** books appointments, pays invoices, redeems offers and leaves reviews

## Key Features
- Role-based authentication and authorization
- Appointment booking with stylist availability checks
- Service and branch management
- Invoicing and multiple payment methods (cash, card, UPI)
- Offers and coupon codes with redemption tracking
- Inventory and supplier management
- SMS/email notifications
- Customer reviews with admin moderation
- Revenue and performance reports

## Architecture
The system is split into 6 services behind an API gateway.

| Service | Responsibility |
|---|---|
| `identity-user-service` | Admins, staff, customers, roles and permissions |
| `salon-operations-service` | Appointments, services, branches, schedules |
| `billing-promotions-service` | Invoices, payments, offers, coupons |
| `inventory-service` | Products, stock, suppliers, purchase orders |
| `engagement-service` | Notifications, reviews and ratings |
| `analytics-service` | Reports and business metrics |

Services communicate through REST calls and asynchronous events (for example `appointment.completed`, `payment.received`).
See [architecture.md](docs/architecture.md) for details.

## Tech Stack
| Layer | Technology |
|---|---|
| Language / Runtime | Node.js 20 (TypeScript) |
| Framework | NestJS |
| Database | PostgreSQL (one database per service) |
| Messaging | RabbitMQ |
| API Gateway | Nginx / Kong |
| Containers | Docker, Docker Compose |
| Testing | Jest, Supertest |

## Project Structure
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
├── shared/
├── docs/
├── docker-compose.yml
└── .env.example
```

## Getting Started

### Prerequisites
- Node.js 20 or later
- Docker and Docker Compose
- Git

### Installation
```bash
# 1. Clone the repository
git clone https://github.com/<your-org>/salon-backend.git
cd salon-backend

# 2. Create your environment file
cp .env.example .env

# 3. Start all services and databases
docker-compose up --build

# 4. Run migrations and seed sample data
npm run migrate:all
npm run seed:all
```

### Verify the setup
```bash
curl http://localhost:8080/health
# Expected: {"status":"ok"}
```

## Environment Variables
| Variable | Description | Example |
|---|---|---|
| `JWT_SECRET` | Secret used to sign tokens | `change-me` |
| `JWT_EXPIRES_IN` | Token lifetime | `1d` |
| `DB_HOST` | Database host | `localhost` |
| `DB_PORT` | Database port | `5432` |
| `RABBITMQ_URL` | Message broker URL | `amqp://guest:guest@rabbitmq:5672` |
| `SMS_API_KEY` | SMS provider key | – |
| `SMTP_HOST` | Email server host | – |

> Never commit real secrets. Use `.env.example` for placeholders only.

## Running Tests
```bash
npm run test           # Unit tests
npm run test:e2e       # Integration tests
npm run test:cov       # Coverage report
```

## API Access
- **Base URL:** `http://localhost:8080/api/v1`
- **Interactive docs (Swagger):** `http://localhost:8080/api/docs`
- Full endpoint list: [api-documentation.md](docs/api-documentation.md)

## Documentation
| File | Purpose |
|---|---|
| [requirements.md](docs/requirements.md) | Functional and non-functional requirements |
| [architecture.md](docs/architecture.md) | Backend architecture |
| [modules.md](docs/modules.md) | Module descriptions |
| [database-design.md](docs/database-design.md) | Tables and relationships |
| [api-documentation.md](docs/api-documentation.md) | API endpoints |
| [business-rules.md](docs/business-rules.md) | Salon-specific rules |
| [development-plan.md](docs/development-plan.md) | Development phases and tasks |

## Contributing
1. Create a branch from `develop`: `feature/<module>-<short-description>`
2. Follow the commit style: `feat(appointments): add double-booking check`
3. Add or update tests for your change
4. Open a pull request and request a review

## License
Released under the [MIT License](LICENSE).
