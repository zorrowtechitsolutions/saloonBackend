# Zorrow Salon – Backend

> Internal project of **Zorrow Tech**. Confidential. Do not share outside the organization.

A microservice-based SaaS backend for a **multi-salon platform**. Multiple salons join one shared cloud system. Customers discover salons, book appointments, pay at the counter, earn reward coins, and leave reviews. Salon owners manage their business without installing, hosting, or maintaining any software.

| | |
|---|---|
| **Project Owner** | Zorrow Tech |
| **Status** | In Development (Phase 3 – Documentation & Design Architecture) |
| **Repository** | https://github.com/zorrowtechitsolutions/saloonBackend |
| **Last Updated** | 2026-11-24 |

---

## 1. Overview & Business Model

Zorrow Tech operates a centralized **SaaS platform for salons**. Each salon operates as an isolated tenant: it accesses strictly its own data, while customers enjoy a single account across all salons on the platform.

### Platform Revenue Streams
1. **Monthly Salon Subscriptions:** Salons pay a monthly subscription fee based on tier limits (staff limit, booking quota, feature access).
2. **Platform Advertisements:** Salons purchase sponsored ad slots (Hero Carousel Banners, Featured Search Listings, Category Spotlights) managed centrally by Zorrow.

### User Roles & Governance

| Role | Scope | What They Do |
|---|---|---|
| **Super Admin** | Platform-Wide | Controls **only**: Monthly Subscriptions, Advertisements, Master Services Catalog, Master Inventory Catalog, Coins & Referrals, and Global Role Permissions. Has **view-only** access to salons (cannot approve or suspend salons). |
| **Admin** | Single Salon (`salon_id`) | Salon owner. Selects services from master catalog and sets local prices; manages local stock from master inventory; hires staff and schedules; manages bookings, local offers, and pays subscriptions. |
| **Staff** | Single Salon (`salon_id`) | Stylists and receptionists. Manage daily appointments, working schedules, and walk-in counter payment collection. |
| **User** | Global (All Salons) | Customer. Searches salons, books appointments, pays, earns and redeems coins across all salons, refers friends, and leaves reviews. |

---

## 2. Key Features

- **Multi-Tenant SaaS:** Self-service salon registration (salons start immediately active; Super Admin has view-only oversight). Strict `salon_id` tenant isolation.
- **Monthly Subscription Billing:** Tiered plans (Basic, Pro, Enterprise) configured by Super Admin; automated 30-day billing cycle and 7-day grace period.
- **Platform Advertisements:** Super Admin defines banner slots and daily/weekly rates; Salon Admins book targeted promotions.
- **Master Services Catalog:** Super Admin standardizes service taxonomy (categories, baseline durations); Salons choose active services and set local prices.
- **Master Inventory Catalog:** Super Admin manages standard brands, SKUs, and authorized suppliers; Salons track shelf stock and raise purchase orders.
- **Cross-Salon Coins & Referrals:** Referrer gets 10 coins, new customer gets 5 coins upon first paid booking (min. ₹200). 2% back in coins on all paid bookings. 1 coin = ₹1. Coins redeemable at any salon up to 20% of invoice subtotal.
- **Appointments & POS:** Real-time stylist slot calculation, booking, rescheduling, no-show blocking (after 3 no-shows), and counter payments (Cash, Card, UPI).
- **Engagement & Real-Time Alerts:** SMS/Email notifications, Socket.IO live booking alerts, and verified appointment reviews.

---

## 3. Architecture Summary

The system is organized into **5 consolidated microservices** behind an **Nginx API Gateway**, with each service owning an independent PostgreSQL database. Services communicate via internal REST APIs (synchronous) and **RabbitMQ** events (asynchronous via Transactional Outbox pattern).

```text
                 Client Apps (Web / Mobile)
                             │
                             ▼
                 ┌──────────────────────┐
                 │ API Gateway (Nginx)  │  ◄── Rate Limiting, SSL, X-Request-ID
                 └──────────┬───────────┘
       ┌──────────┬─────────┼──────────┬──────────────┐
       ▼          ▼         ▼          ▼              ▼
  identity-   salon-     billing-   inventory-    engagement-
  user        operations promotions analytics     service
  service     service    service    service
       │          │         │          │              │
       ▼          ▼         ▼          ▼              ▼
    DB (PG)    DB (PG)   DB (PG)    DB (PG)        DB (PG)

         All services publish and consume events through RabbitMQ
              Redis is used for caching and sessions
```

| Microservice | Responsibilities | Super Admin Controlled Areas |
|---|---|---|
| `identity-user-service` | Users, authentication, customer & staff profiles, salon directory. | **Roles & Permissions (RBAC)**. *Super Admin has view-only access to salons.* |
| `salon-operations-service` | Stylist schedules, slots, appointments, no-show blocks. | **Master Services Catalog** (categories, standard services, durations). |
| `billing-promotions-service` | Counter POS invoicing, salon offers/coupons. | **Monthly Subscriptions**, **Advertisements**, and **Coins & Referrals Engine**. |
| `inventory-analytics-service` | Salon shelf stock, purchase orders, salon & platform reports. | **Master Product Catalog** & **Master Suppliers Catalog**. Platform metrics. |
| `engagement-service` | Multi-channel notifications (SMS/Email), Socket.IO live alerts, reviews. | Notification templates. |

---

## 4. Tech Stack

| Layer | Technology |
|---|---|
| Language / Runtime | Node.js (v20+), TypeScript (Strict Mode) |
| Framework | Express.js |
| Relational Database | PostgreSQL (1 independent DB per microservice) |
| ORM | Sequelize |
| Password Hashing | bcrypt |
| Message Broker | RabbitMQ (with Transactional Outbox pattern) |
| Cache & Session Store | Redis |
| Real-time Communication | Socket.IO |
| API Gateway | Nginx |
| Object Storage | DigitalOcean Spaces (S3-compatible) |
| Infrastructure | Docker, Docker Compose, DigitalOcean |

---

## 5. Project Documentation

| Document | Purpose |
|---|---|
| [requirements.md](docs/requirements.md) | Business model, SaaS subscription mechanics, and functional requirements |
| [business-rules.md](docs/business-rules.md) | In-depth business logic, governance, coin rules, and access constraints |
| [architecture.md](docs/architecture.md) | Microservice boundaries, multi-tenancy, outbox pattern, and event catalog |
| [modules.md](docs/modules.md) | Module taxonomy and ownership across microservices |
| [database-design.md](docs/database-design.md) | Database schemas, table columns, indexes, and relationship constraints |
| [api-documentation.md](docs/api-documentation.md) | Complete REST API endpoints, access controls, request envelopes, and status codes |
| [development-plan.md](docs/development-plan.md) | Engineering roadmap, phase breakdown, open decisions, and risk register |

---

## 6. Development Guidelines

- **Tenant Isolation:** Every salon-owned record must include `salon_id`. For Admin and Staff users, `req.salonId` is extracted strictly from the verified JWT, never from user input.
- **Super Admin Boundary:** Super Admin controls **only** Subscriptions, Advertisements, Master Services, Master Inventory, Coins & Referrals, and Roles & Permissions. Super Admin has **view-only** access to salons.
- **Transactional Outbox:** Never publish RabbitMQ events outside the database transaction. Always write the event to the `outbox` table in the same commit.
- **Idempotency:** All payment, booking, and coin-redemption endpoints must enforce `Idempotency-Key` headers.
- **Health Checks:** Every service must provide `/health/live` and `/health/ready` endpoints.
- **Branches:** `feature/*` → `main` → `demo` → `production`. Direct pushes to `main`, `demo`, or `production` are prohibited.

---

## 7. Team & Contact

| Role | Name | Contact |
|---|---|---|
| Project Manager / Tech Lead | Safwan | zorrowtech@gmail.com |
| Backend Developer | Suhaila Thasni | suhailathasni0@gmail.com |
| Backend Team Channel | - | #salon (Pumble) |

---

© 2026 Zorrow Tech. All rights reserved.
