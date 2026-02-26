# Folkeep

> 🇧🇷 [Leia em Português](./README.pt-BR.md)

A B2B people analytics platform for managing and analyzing workforce data, focused on headcount, turnover, and organizational distribution reports.

---

## Overview

A multitenant system where companies manage their employees and extract strategic HR insights — without the complexity of a full HRIS. The goal is to turn structured people data into actionable reports.

---

## Stack

| Layer | Technology |
|---|---|
| Backend | Laravel 12+ (REST API) |
| Frontend | React + TypeScript (decoupled SPA) |
| Relational Database | PostgreSQL |
| Logs / Events | MongoDB |
| Authentication | Laravel Sanctum (stateless token) |

---

## Architecture

```
┌─────────────────────┐        HTTP / JSON        ┌──────────────────────┐
│                     │  ──────────────────────►  │                      │
│   React + TypeScript│                           │    Laravel API        │
│   (SPA)             │  ◄──────────────────────  │    (REST)             │
│                     │                           │                       │
└─────────────────────┘                           └──────────┬────────────┘
                                                             │
                                              ┌──────────────┴──────────────┐
                                              │                             │
                                     ┌────────▼────────┐        ┌──────────▼───────┐
                                     │   PostgreSQL     │        │     MongoDB       │
                                     │  (relational     │        │  (event logs /    │
                                     │    data)         │        │   history)        │
                                     └─────────────────┘        └──────────────────┘
```

The frontend is an independent project that consumes the API via Sanctum token. There is no coupling via Inertia or Blade — API contracts are the only interface between the two projects.

---

## Planned Features

**MVP**
- Token-based authentication with multitenancy support
- Employee registration with department, role, contract type, and hire date
- Employee event history (promotions, role changes, salary updates) stored in MongoDB
- Headcount report by department
- Monthly/quarterly turnover report
- Salary distribution by band and level

**Future Expansions**
- Real-time notifications (Laravel Reverb)
- PDF report export
- Document upload with S3
- CI/CD with GitHub Actions
- Diversity and inclusion metrics

---

## Key Technical Decisions

**Employee history with Slowly Changing Dimensions**
Employees change roles, salaries, and departments over time. To enable accurate historical reports, changes are recorded with `valid_from` / `valid_to` columns in PostgreSQL — not just the current state.

**MongoDB for events**
Every relevant action in the system (hire, promotion, termination, salary change) generates a flexible-schema event document in MongoDB, indexed by `tenant_id`, `employee_id`, and `timestamp`.

**Schema-based multitenancy**
Each company (tenant) has its own PostgreSQL schema, isolating data without the overhead of separate databases.

**Versioned API**
All endpoints follow the `/api/v1/` prefix with a standardized response envelope:
```json
{
  "data": {},
  "meta": {},
  "errors": []
}
```

---

## Project Structure

```
people-analytics/
├── api/          # Laravel — REST API
└── web/          # React + TypeScript — SPA
```

---

## Running Locally

> Prerequisites: Docker, Node.js 20+, PHP 8.2+, Composer

```bash
# Clone the repository
git clone https://github.com/your-username/people-analytics.git
cd people-analytics

# Backend
cd api
cp .env.example .env
composer install
php artisan key:generate
php artisan migrate --seed

# Frontend
cd ../web
cp .env.example .env.local
npm install
npm run dev
```

---

## Status

🚧 Under development — MVP in progress.
