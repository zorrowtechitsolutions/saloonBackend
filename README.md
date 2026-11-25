# Zorrow Salon – Backend

> Internal project of **Zorrow Tech**. Confidential. Do not share outside the organization.

A microservice-based backend for a **multi-salon platform**. Many salons join one shared system (similar to how shops sell on Amazon). Customers find salons, book appointments, pay, earn reward coins and leave reviews. Salon owners run their salon from an admin panel without installing, hosting or maintaining any software.

| | |
|---|---|
| **Project owner** | Zorrow Tech |
| **Status** | In development (Phase 3 – Documentation) |
| **Repository** | https://github.com/zorrowtechitsolutions/saloonBackend |
| **Last updated** | 2026-11-25 |

---

## 1. Overview

Zorrow Tech runs **one platform for all salons**. Each salon is a separate *tenant*: it sees only its own data, and customers can use one account across every salon on the platform.

Four groups use the system:

| Group | Role | What they do |
|---|---|---|
| **Zorrow team** | Super Admin | Manages subscription plans and payments, advertisement banners, platform inventory, coin and referral rules, and role permissions. **Views** everything else (salons, bookings, invoices, reviews, reports). Cannot approve, reject, suspend, edit or delete salons |
| **Salon owner** | Admin | Manages own salon profile, services, staff and offers. Views own reports and orders products from Zorrow inventory |
| **Salon employees** | Staff | Stylists and receptionists. Manage their schedule and appointments |
| **Customers** | User | Search salons, book, pay, earn and redeem coins, refer friends, leave reviews |

### How a salon joins

1. The owner registers as a normal user and logs in.
2. The owner completes the salon registration and becomes the salon's Admin.
3. The owner subscribes to the **Basic** monthly plan. Zorrow records the payment.
4. The salon goes live automatically. No one at Zorrow approves it.
5. If the subscription ends without renewal, the system marks the salon `expired` automatically. It becomes active again when the next payment is recorded.

### Why salons join the platform

- No app to build, host, update or back up. Zorrow Tech maintains the platform for every salon.
- New customers come through the platform, and customers return because of reward coins.
- Coin rewards and referral rewards are paid by Zorrow. Zorrow reimburses each salon monthly for the coins used at that salon.
- One simple monthly subscription (Basic plan). No commission on bookings.

This repository contains the backend only. It exposes REST APIs through a single API gateway.

## 2. Key Features

- Multi-salon platform: salon registration by any user, automatic activation by subscription, public salon listing
- Role-based access control (Super Admin, Admin, Staff, User) with strict salon data isolation
- Super Admin with limited write access: subscription, advertisements, inventory, coins, referrals and role permissions. View only for everything else
- Appointment booking and stylist scheduling
- Services managed by each salon
- Invoicing and payments (Cash, Card, UPI)
- Offers, coupons and redemption tracking per salon
- **Referral and coins:** the referrer earns 10 coins (₹10) and the new customer earns 5 coins (₹5) after the new customer's first paid booking. Every paid booking earns 2% back in coins. 1 coin = ₹1. All values are configurable by the Super Admin
- Monthly coin reimbursement statements for salons
- Platform inventory managed by Zorrow, with product ordering by salons
- Advertisement banners shown to customers
- SMS/Email notifications and reviews
- Salon-level and platform-level reports

## 3. Architecture Summary

The system uses **5 consolidated microservices** behind an API gateway, each owning its own database. Services communicate through the API gateway (synchronous) and events (asynchronous). Salon data is separated by a `salon_id` column on every salon-owned table.

| Service | Responsibility |
|---|---|
| `identity-user-service` | Users, roles and role permissions, **salons (tenants)**, staff and customer profiles |
| `salon-operations-service` | Services, appointments, schedules |
| `billing-promotions-service` | Invoices, payments, offers, coupons, referrals, coins, **subscriptions, coin reimbursements** |
| `inventory-analytics-service` | Platform inventory (products, stock, suppliers, orders), then reports and metrics built from events |
| `engagement-service` | Notifications, templates, reviews, **advertisement banners** |

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

Seed the first Super Admin account and the Basic plan with the seed scripts in each service. Super Admin accounts cannot be created through the API.

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

Coin rewards, subscription prices and similar business values are **not** environment variables. They are stored in the database and changed by a Super Admin through the API.

## 8. Running Tests

APIs are tested with Postman. Import the collection from `docs/postman/` and run it against your local environment.

All new APIs must be tested in Postman and pass before merging. Every salon-scoped endpoint must also have a test proving that an Admin or Staff user from salon A cannot read or change salon B's data. Every Super Admin write route outside the allow-list must be tested to return `403`.

## 9. API Access

- Base URL (local): `http://localhost:3000/api/v1`
- All requests go through the API gateway.
- Protected endpoints need the header `Authorization: Bearer <token>`.

Full endpoint list: [docs/api-documentation.md](docs/api-documentation.md)

## 10. Project Documentation

| File | Purpose |
|---|---|
| [requirements.md](docs/requirements.md) | Business model, functional and non-functional requirements |
| [architecture.md](docs/architecture.md) | Backend architecture, multi-salon design, Super Admin access, events |
| [modules.md](docs/modules.md) | Module descriptions |
| [database-design.md](docs/database-design.md) | Tables and relationships |
| [api-documentation.md](docs/api-documentation.md) | API endpoints |
| [business-rules.md](docs/business-rules.md) | Platform, salon, subscription, coin and inventory rules |
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
- **Super Admin access:** the Super Admin has write access only to subscriptions, advertisements, inventory, coins, referrals and role permissions. Every other Super Admin write is denied by default. A new Super Admin write route must be added to the allow-list in `docs/architecture.md` and to `docs/business-rules.md` in the same pull request.
- **Naming:** services in `kebab-case`, tables in `snake_case`, API paths in plural nouns (`/appointments`)

## 12. Team and Contact

| Role | Name | Contact |
|---|---|---|
| Project Manager / Tech Lead | Safwan | zorrowtech@gmail.com |
| Backend Developer | Suhaila Thasni | suhailathasni0@gmail.com |
| Backend Team Channel | - | #salon (Pumble) |

---

© 2026 zorrowtech. All rights reserved.
