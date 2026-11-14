# DeskOps — Service Desk Platform with an SLA Engine

Phase 1: Ticketing Core · LogicVeda Web Development Internship

> This repository is the independent implementation for the DeskOps Phase 1 requirements. Do not commit secrets, local `.env` files, production data, or confidential programme documents.

## Project status

- Phase: 1 — Ticketing Core
- Status: Foundation and technical design
- Technical design: [docs/technical-design.md](docs/technical-design.md)
- Client-specific rule (FR-1.08): Pending confirmation in the LogicVeda Team Workspace
- Assigned module: Pending mentor confirmation

## Required technology standard

| Layer | Technology |
|---|---|
| Frontend | React 18, TypeScript, Vite, TanStack Query, React Hook Form, Zod, Tailwind CSS |
| Backend | Node.js 22, TypeScript, Express or NestJS, Zod, Swagger/OpenAPI |
| Data | PostgreSQL, Prisma migrations, Redis (Upstash) |
| Jobs and real time | BullMQ workers, Socket.io |
| File storage | S3-compatible storage (Cloudflare R2 or AWS S3), presigned uploads |
| Delivery | Docker, docker-compose, GitHub Actions, Render |
| Quality | Vitest, Supertest, Playwright, k6, OWASP ZAP baseline, pino, Sentry |

Use the required stack. A deviation requires the mentor's written approval and an Architecture Decision Record (ADR).

## Repository layout

```text
DeskOps/
├── frontend/                 # React application (to be scaffolded)
├── backend/                  # API: routes, services, repositories
├── docs/
│   ├── adr/                  # One-page architecture decision records
│   ├── retros/               # Retrospective notes
│   └── technical-design.md   # Phase 1 technical design
├── infra/                    # Deployment and infrastructure notes
├── .github/
│   ├── workflows/            # CI workflows
│   └── pull_request_template.md
├── .env.example
├── .nvmrc
├── docker-compose.yml
├── .gitignore
├── CHANGELOG.md
└── README.md
```

## Prerequisites

Install:
1. Git
2. Node.js 22
3. Docker Desktop
4. A GitHub account with access to this repository

Verify:

```bash
node --version
git --version
docker --version
docker compose version
```

Node.js should report version 22.x.

## Local setup

1. Clone your repository:

```bash
git clone https://github.com/gopiprakan/DeskOps.git
cd DeskOps
```

2. Use Node.js 22 (when nvm is installed):

```bash
nvm install 22
nvm use 22
```

3. Create your local environment file:

Windows Command Prompt:
```bat
copy .env.example .env
```

PowerShell:
```powershell
Copy-Item .env.example .env
```

4. Start local infrastructure:

```bash
docker compose up -d
docker compose ps
```

The initial compose file is intended only for local development. Do not use its development credentials in staging or production.

## Current implementation status

The repository currently contains the technical design document. Frontend, backend, database schema, CI and deployment must be implemented and verified incrementally. Do not treat planned components as completed features.

## Engineering practices

- Use small, understandable commits with Conventional Commit messages.
- Use feature branches and pull requests; do not bypass repository protections.
- Run tests before opening a pull request.
- Make database changes through Prisma migrations.
- Never commit `.env`, credentials, tokens or secrets.
- Commit `.env.example` with safe placeholder values.
- Use labelled sample data only; never use production data in development.
- Record significant architecture decisions in `docs/adr/`.
- Explain and understand every change before committing it.

## Phase 1 acceptance focus

- Tenant isolation: organisation A cannot retrieve organisation B's records; return 404.
- Assignment safety: 20 concurrent assignment requests result in exactly one assignment.
- Full ticket lifecycle works in production.
- CR-1 duplicate ticket merge is implemented and demonstrated.
- The assigned client-specific rule is implemented and covered by an automated test.

## Documentation

- [Technical Design](docs/technical-design.md)
- [Architecture Decision Records](docs/adr/)
- [Retrospectives](docs/retros/)

## Release

The planned Phase 1 release is `v1.0.0`. Create the release tag only after the release acceptance criteria and required CI checks pass. See the programme requirements and mentor instructions for the submission process.
