<img src="docs/logo.svg" width="80" />

# CoPaila Carbon Backend

CoPaila is a carbon-audit platform built for schools in Nepal. This is its NestJS API: it onboards schools through a multi-step registration flow, collects activity data (electricity, fuel, commuting, waste, and more) either through a web form or by scanning a paper OMR (optical mark recognition) answer sheet, and turns that data into a GHG Protocol-aligned emissions report with a confidence score. It also runs a lightweight gamification layer — XP, pets, lessons, and leaderboards — to get students engaged with their school's carbon footprint. This repo is API-only; it is consumed by a separate React frontend.

## Tech stack

![NestJS](https://img.shields.io/badge/NestJS-10-E0234E?logo=nestjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-5-2D3748?logo=prisma&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-4169E1?logo=postgresql&logoColor=white)
![Passport JWT](https://img.shields.io/badge/Passport-JWT_Auth-34E27A?logo=passport&logoColor=white)
![Swagger](https://img.shields.io/badge/Swagger-API_Docs-85EA2D?logo=swagger&logoColor=black)
![Python](https://img.shields.io/badge/Python-OMR_Scanner-3776AB?logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Container-2496ED?logo=docker&logoColor=white)

## Features

- **GHG Protocol-aligned carbon calculator** — Scope 1/2/3 emissions across 10 activity categories (electricity, generator/vehicle fuel, cooking fuel, refrigerants, commuting, paper, food, waste, water), with per-student intensity normalization and a letter grade (A+ to D).
- **Three-tier confidence model** — every data point is tagged Measured, Estimated, or Default, and the final result reports what percentage of total emissions came from each tier plus an overall confidence score.
- **OMR paper-form scanning** — schools without reliable internet can fill in a printed bubble sheet; the API spawns a Python OMR checker in-process, parses the results, and maps them straight into an audit submission (`POST /api/v1/omr/scan-and-submit`).
- **Multi-step school registration** — a 3-step draft flow (identity, location, operations) that expires unclaimed drafts, finalizing into a `School` + admin `User` on completion.
- **JWT authentication** — access/refresh token rotation, rate-limited login and registration endpoints, and a separate passwordless login for students (identified by school + class + roll number).
- **Student gamification** — XP, pet evolution stages, streaks, daily quests, a carbon-literacy lesson roadmap with quizzes, and class/school/global leaderboards.
- **Rule-based recommendations** — DB-editable, bilingual (English/Nepali) advice templates triggered by quest answers or audit results.
- **Configurable emission factors** — factors default from a config file but can be overridden per-key from the database without a redeploy.
- **Hardened by default** — Helmet, gzip compression, global validation pipes, rate limiting (`@nestjs/throttler`), and a global exception filter.
- **Swagger docs** — auto-generated OpenAPI docs at `/api/docs` in non-production environments.

## API overview

All routes are prefixed with `/api/v1`.

| Module | Base route | Purpose |
|---|---|---|
| `auth` | `/auth` | School/individual registration, login, student login, token refresh |
| `schools` | `/schools` | School records and admin management |
| `users` | `/users` | Staff user management |
| `students` | `/students` | Pet progress, quests, lessons, leaderboards |
| `carbon-calculator` | `/carbon-audits` | Submit activity data, run calculations, fetch results |
| `omr` | `/omr` | Scan a paper answer sheet and submit it as an audit |
| `lessons` | `/lessons` | Carbon-literacy lesson roadmap |
| `recommendations` | `/recommendations` | Rule-based advice templates |
| `emission-factors` | `/emission-factors` | View/override emission factor constants |

## Data model

Schools go through `SchoolRegistrationDraft` → `School`, which owns `User` accounts (staff and students) and `CarbonAudit` records. Each audit holds many `ActivityData` rows (one per category per period) and produces a single `CarbonResult` with the scope breakdown, confidence tiers, grade, and recommendations. Students get a lazily-created `StudentProfile` for gamification progress. See `prisma/schema.prisma` for the full model.

## Getting started

### Prerequisites
- Node.js 18+
- PostgreSQL database
- Python 3 environment (only required for the OMR scanning endpoints — see `OMR_INTEGRATION.md`)

### Setup

```bash
npm install
cp .env.example .env        # fill in DATABASE_URL, JWT secrets, etc.
npm run prisma:generate
npm run prisma:migrate:dev
npm run prisma:seed          # optional: seed lessons/recommendations/emission factors
```

### Run

```bash
npm run start:dev    # watch mode
npm run build
npm run start:prod    # run compiled build
```

### Other scripts

```bash
npm run lint
npm test
npm run test:cov
npm run prisma:studio    # inspect the database
```

Swagger docs are served at `/api/docs` when `NODE_ENV` is not `production`.

See `OMR_INTEGRATION.md` for details on the OMR scanning pipeline and the sheet-template generator.
