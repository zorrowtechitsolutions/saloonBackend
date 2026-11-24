# Zorrow Salon – Backend

> Internal project of **Zorrow Tech**. Confidential. Do not share outside the organization.

A microservice-based backend for a **multi-salon platform**. Many salons join one shared system (similar to how shops sell on Amazon). Customers find salons, book appointments, pay, earn reward coins and leave reviews. Salon owners run their salon from an admin panel without installing, hosting or maintaining any software.

| | |
|---|---|
| **Project owner** | Zorrow Tech |
| **Status** | In development (Phase 3 – Documentation) |
| **Repository** | https://github.com/zorrowtechitsolutions/saloonBackend |
| **Last updated** | 2026-11-24 |

---

## 1. Overview

Zorrow Tech runs **one platform for all salons**. Each salon is a separate *tenant*: it sees only its own data, and customers can use one account across every salon on the platform.

Four groups use the system:

| Group | Role | What they do |
|---|---|---|
| **Zorrow team** | Super Admin | Approve and suspend salons, set commission, manage coin rewards, moderate reviews, view platform reports and settlements |
| **Salon owner** | Admin | Manage own salon profile, services, staff, offers, inventory and reports |
| **Salon employees** | Staff | Stylists and receptionists. Manage their schedule and appointments |
| **Customers** | User | Search salons, book, pay, earn and redeem coins, refer friends, leave reviews |

### Why salons join the platform

- No app to build, host, update or back up. Zorrow Tech maintains the platform for every salon.
- New customers come through the platform, and customers return because of reward coins.
- Coin rewards and referral rewards are paid by Zorrow, not by the salon.
- Salons pay only a commission on completed bookings (rate set by Zorrow, see `docs/business-rules.md`).

This repository contains the backend only. It exposes REST APIs through a single API gateway.

## 2. Key Features

- Multi-salon platform: salon registration, approval by Super Admin, public salon listing
- Role-based access control (Super Admin, Admin, Staff, User) with strict salon data isolation
- Appointment booking and stylist scheduling
- Service management per salon
- Invoicing and payments (Cash, Card, UPI)
- Offers, coupons and redemption tracking per salon
- **Referral and coins:** referrer earns 10 coins (₹10), the new customer earns 5 coins (₹5) after the new customer's first paid booking. Every paid booking earns 2% back in coins. 1 coin = ₹1. All values are configurable.
- Commission and monthly settlement between Zorrow and each salon
- Inventory, suppliers and purchase orders per salon
- SMS/Email notifications and reviews
- Salon-level and platform-level reports

## 3. Architecture Summary

The system uses **5 consolidated microservices** behind an API gateway, each owning its own database. Services communicate through the API gateway (synchronous) and events (asynchronous). Salon data is separated by a `salon_id` column on every salon-owned table.

| Service | Responsibility |
|---|---|
| `identity-user-service` | Users, roles, permissions, **salons (tenants)**, staff and customer profiles |
| `salon-operations-service` | Services, appointments, schedules |
| `billing-promotions-service` | Invoices, payments, offers, coupons, **referrals, coins, commission and settlements** |
| `inventory-analytics-service` | Products, stock levels, suppliers, purchase orders, then reports and metrics built from events |
| `engagement-service` | Notifications, templates, reviews, moderation |

Details: [docs/architecture.md](docs/architecture.md)

## 4. Tech Stack

| Layer | Technology |
|---|---|
| Language / Runtime | Node.js, TypeScript |
| Framework | Express.js |
| Database | PostgreSQL |
| ORM | Sequelize |
| Password Hashing | bcrypt |
| Real-time Communication | Socket.IO |
| Caching / Session Store | Redis |
| Hosting | DigitalOcean |
| Message Broker | RabbitMQ |
| API Gateway / Reverse Proxy | Nginx |
| Authentication | JWT |
| Version Control | Git, GitHub |
| Containerization | Docker, Docker Compose |
| File Storage | DigitalOcean Spaces (S3-compatible) |
| Testing | Postman |

## 5. Project Structure

```text
saloonBackend/
├── services/
│   ├── api-gateway/
│   ├── identity-user-service/
│   ├── salon-operations-service/
│   ├── billing-promotions-service/
│   ├── inventory-analytics-service/
│   └── engagement-service/
├── docs/
│   ├── requirements.md
│   ├── architecture.md
│   ├── modules.md
│   ├── database-design.md
│   ├── api-documentation.md
│   ├── business-rules.md
│   ├── development-plan.md
│   └── postman/               # Postman collections
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

- Node.js 20 or higher
- npm (installed together with Node.js)
- Docker and Docker Compose
- Git
- Postman (optional, for testing APIs)

### Setup

```bash
# 1. Clone the repository
git clone https://github.com/zorrowtechitsolutions/saloonBackend.git
cd saloonBackend

# 2. Create your environment file
cp .env.example .env

# 3. Start all services
docker-compose up --build

# 4. Run database migrations (one per service, in a second terminal)
for s in identity-user-service salon-operations-service billing-promotions-service \
         inventory-analytics-service engagement-service; do
  docker-compose exec "$s" npx sequelize-cli db:migrate
done
```

The migration loop assumes the Docker Compose service names match the folder names under `services/`. Adjust them if your compose file uses different names.

### Verify

Open `http://localhost:3000/health`. All services should return `OK`.

## 7. Environment Variables

| Variable | Description |
|---|---|
| `DB_URL_*` | Database connection string for each service |
| `JWT_SECRET` | Secret used to sign tokens |
| `REDIS_URL` | Redis connection string |
| `MESSAGE_BROKER_URL` | Event broker connection |
| `SMS_API_KEY` / `EMAIL_API_KEY` | Notification providers |
| `SPACES_KEY` / `SPACES_SECRET` / `SPACES_BUCKET` / `SPACES_ENDPOINT` | DigitalOcean Spaces file storage |

Never commit real secrets. Use `.env.example` for placeholders only.

Coin rewards, commission and similar business values are **not** environment variables. They are stored in the database and changed by a Super Admin through the API.

## 8. Running Tests

APIs are tested with Postman. Import the collection from `docs/postman/` and run it against your local environment.

All new APIs must be tested in Postman and pass before merging. Every salon-scoped endpoint must also have a test proving that an Admin or Staff user from salon A cannot read or change salon B's data.

## 9. API Access

- Base URL (local): `http://localhost:3000/api/v1`
- All requests go through the API gateway.
- Protected endpoints need the header `Authorization: Bearer <token>`.

Full endpoint list: [docs/api-documentation.md](docs/api-documentation.md)

## 10. Project Documentation

| File | Purpose |
|---|---|
| [requirements.md](docs/requirements.md) | Business model, functional and non-functional requirements |
| [architecture.md](docs/architecture.md) | Backend architecture, multi-salon design, events |
| [modules.md](docs/modules.md) | Module descriptions |
| [database-design.md](docs/database-design.md) | Tables and relationships |
| [api-documentation.md](docs/api-documentation.md) | API endpoints |
| [business-rules.md](docs/business-rules.md) | Platform, salon, coin and commission rules |
| [development-plan.md](docs/development-plan.md) | Development phases, tasks and open decisions |

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
- **Tenancy:** every salon-owned table has `salon_id`. For Admin and Staff, the salon always comes from the JWT, never from the request body.
- **Naming:** services in `kebab-case`, tables in `snake_case`, API paths in plural nouns (`/appointments`)

## 12. Team and Contact

| Role | Name | Contact |
|---|---|---|
| Project Manager / Tech Lead | Safwan | zorrowtech@gmail.com |
| Backend Developer | Suhaila Thasni | suhailathasni0@gmail.com |
| Backend Team Channel | - | #salon (Pumble) |

---

© 2026 zorrowtech. All rights reserved.
