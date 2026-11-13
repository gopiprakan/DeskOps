# DeskOps – Technical Design Document

**Project:** DeskOps – Service Desk Platform with an SLA Engine  
**Phase:** Phase 1 – Ticketing Core  
**Document ID:** LV-WD-DESKOPS-P1  
**Version:** 0.1 – Draft  
**Prepared by:** [Gopiprakan]  
**Date:** 13 November 2026  
**Status:** Pending mentor review and approval

---

## 1. Project Overview

DeskOps is a multi-tenant service desk platform designed to manage maintenance requests for facilities management companies.

Currently, maintenance requests may arrive through WhatsApp, phone calls and email. This makes it difficult to track requests, assignments and overdue work.

The purpose of DeskOps is to provide a centralized platform where residents can raise maintenance tickets and technicians can manage and resolve them. Every important ticket activity will be recorded.

## 2. Project Objectives

- Provide a centralized maintenance ticketing system.
- Allow residents to create and track maintenance requests.
- Allow staff to assign tickets to technicians.
- Maintain ticket status and activity history.
- Support multiple facilities organisations with tenant isolation.
- Provide real-time ticket updates.
- Support email-based ticket creation and attachments.

## 3. Project Scope

### 3.1 Included in Phase 1

- Organisation, property and unit management.
- User authentication and role-based access.
- Ticket creation and management.
- Ticket lifecycle and history.
- Technician assignment.
- Comments and attachments.
- Email-to-ticket integration.
- Real-time updates.
- Duplicate ticket merging (CR-1).
- Client-specific requirement (pending confirmation).

### 3.2 Out of Scope

- Maintenance-fee billing.
- Visitor management.
- Native mobile applications.

## 4. Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, TypeScript, Vite |
| UI styling | Tailwind CSS |
| Data fetching | TanStack Query |
| Forms and validation | React Hook Form, Zod |
| Backend | Node.js 22, TypeScript, Express |
| API documentation | Swagger / OpenAPI |
| Database | PostgreSQL |
| Database ORM | Prisma |
| Cache and queue support | Redis / Upstash |
| Background jobs | BullMQ |
| Real-time communication | Socket.io |
| File storage | Cloudflare R2 or AWS S3 |
| Testing | Vitest, Supertest, Playwright |
| Deployment | Docker, GitHub Actions, Render |
| Logging and monitoring | Pino, Sentry |

## 5. System Architecture

The application will use a frontend-backend architecture.

1. The React frontend provides interfaces for residents and staff.
2. The backend API handles authentication, validation, permissions and business logic.
3. Prisma communicates with the PostgreSQL database.
4. Middleware checks the user's organisation before accessing tenant-specific records.
5. Redis and BullMQ handle background jobs.
6. Socket.io sends ticket updates to other signed-in users.
7. S3-compatible storage handles attachments using presigned upload URLs.

### Request flow

User → React Frontend → Express API → Middleware → Service Layer → Repository Layer → PostgreSQL

Background jobs will be processed separately by BullMQ workers.

## 6. Initial Database Design

The following entities are proposed for the initial database design.

| Entity | Purpose |
|---|---|
| Organisation | Stores facilities company information |
| Property | Stores properties belonging to an organisation |
| Unit | Stores residential or office units |
| User | Stores user account information |
| TechnicianProfile | Stores technician skills and coverage |
| Ticket | Stores maintenance requests |
| TicketEvent | Stores ticket status changes and history |
| Comment | Stores internal and public comments |
| Attachment | Stores uploaded file information |
| Watcher | Stores users following a ticket |
| Assignment | Stores technician assignment information |

**Tenant isolation:** Every database table must carry `organisationId`. Database queries must be scoped through middleware, not by relying on developers to remember manual filtering.

Final relationships, constraints and indexes will be confirmed during implementation.

## 7. Ticket Lifecycle

Tickets must follow the required lifecycle:

`New → Triaged → Assigned → In Progress → On Hold → Resolved → Closed`

The backend will validate every status transition. Illegal transitions will be rejected.

Every transition will create a ticket history event containing:

- Ticket ID
- Previous status
- New status
- Actor
- Timestamp

Reopening will be supported within a configurable time window.

## 8. API Design

All APIs will use the version prefix `/api/v1`.

Proposed endpoints:

| Method | Endpoint | Purpose |
|---|---|---|
| POST | /api/v1/auth/login | User login |
| GET | /api/v1/tickets | List tickets |
| POST | /api/v1/tickets | Create a ticket |
| GET | /api/v1/tickets/:id | Get ticket details |
| PATCH | /api/v1/tickets/:id/status | Change ticket status |
| POST | /api/v1/tickets/:id/assign | Assign a technician |
| GET | /api/v1/tickets/:id/history | View ticket history |
| POST | /api/v1/tickets/:id/comments | Add a comment |
| POST | /api/v1/tickets/:id/attachments | Request an upload URL |

All request inputs will be validated using Zod. APIs will use a consistent error response format, cursor pagination and idempotency keys for create operations.

OpenAPI documentation will be available at `/docs`.

## 9. Roles and Permissions

The following is a proposed initial permission model, subject to mentor review.

| Role | Main permissions |
|---|---|
| Resident | Create tickets, view own tickets, add public comments |
| Technician | View assigned tickets, update permitted statuses, add comments |
| Admin / Staff | Manage tickets, assign technicians, view and manage organisation records |

Internal comments must only be visible to staff. Public comments may be visible to residents.

Access will be enforced by backend authorization checks, not only by hiding frontend controls.

## 10. Functional Requirements Coverage

| Requirement | Design approach |
|---|---|
| FR-1.01 Organisation model | Organisation, property, unit and user entities |
| FR-1.02 Ticket lifecycle | Backend state-machine validation |
| FR-1.03 Ticket history | TicketEvent records |
| FR-1.04 Attachments | Presigned uploads, file checks and thumbnail worker |
| FR-1.05 Email-to-ticket | Inbound email webhook or compatible mock endpoint |
| FR-1.06 Real-time updates | Socket.io |
| FR-1.07 Comments | Internal and public comment visibility |
| FR-1.08 Client-specific rule | Pending mentor confirmation |

## 11. Non-Functional Requirements

- **NFR-01 Tenancy:** Every table has `organisationId`; tenant queries are middleware-scoped. Cross-tenant access must return 404.
- **NFR-02 Access control:** Maintain a role-permission matrix and test permitted and denied access.
- **NFR-03 Data integrity:** Use transactions, constraints or locking where needed. Test 20 parallel requests at the critical write.
- **NFR-04 API standard:** Use `/api/v1`, Zod, consistent errors, cursor pagination and idempotency keys.
- **NFR-05 Performance:** Target p95 below 300 ms for list/detail APIs at 50 virtual users and Lighthouse 90+ on key pages. Load testing applies from Phase 2.
- **NFR-06 Reliability:** Use retry with backoff, a dead-letter queue, an admin view, `/health` and `/ready`.
- **NFR-07 Security:** Hash passwords with Argon2 or bcrypt; use refresh-token rotation, Helmet, CORS allow-list and file checks. Never commit secrets.
- **NFR-08 Observability:** Use structured JSON logs with request IDs and Sentry on frontend and backend.
- **NFR-09 Audit:** Record who changed business-critical data and when.
- **NFR-10 Testing:** Target 70% service coverage, Playwright for the main journey and tests in CI.
- **NFR-11 Documentation:** Maintain the design document, ADRs, OpenAPI, runbook and release notes.

## 12. Change Request CR-1

**Requirement:** Merge duplicate tickets.

The duplicate ticket will be closed, and its comments and watchers will be moved to the original ticket.

An impact estimate will be prepared covering affected data, API, interface, testing, effort and risks when the change request is issued.

## 13. Testing Strategy

The following testing approach is proposed:

- Unit tests using Vitest.
- API tests using Supertest.
- Frontend journey tests using Playwright.
- Tenant isolation test between two organisations.
- Permission tests for protected routes.
- Concurrency test using 20 parallel assignment requests.
- Automated tests for the client-specific rule once confirmed.
- CI execution through GitHub Actions.

## 14. Deployment Strategy

The application will use Docker and docker-compose for its development environment.

GitHub Actions will run the automated checks. Render will be used for staging and production deployment.

Environment variables will be managed outside the repository. An `.env.example` file will document the required configuration without exposing secrets.

## 15. Risks and Open Questions

| Question / Risk | Status |
|---|---|
| Individual or team assignment | Pending mentor confirmation |
| Assigned product module | Pending mentor confirmation |
| GitHub repository | Not yet provided |
| Client-specific rule (FR-1.08) | Pending mentor confirmation |
| Final database relationships | To be reviewed |
| External email provider | Mock endpoint available as fallback |

## 16. Implementation Milestones

| Milestone | Official due date |
|---|---|
| Technical design approved | 14 November 2026 |
| Walking skeleton on staging | 16 November 2026 |
| Core flows complete on staging | 20 November 2026 |
| CR-1 issued | 21 November 2026 |
| Submission opens | 2 December 2026 |
| Final v1.0.0 submission | 7 December 2026, 11:59 PM IST |

## 17. Approval

**Prepared by:** [Gopiprakan] 
**Reviewed by:** [Shaurya Pandey]  
**Approval status:** Pending  
**Review date:** [To be confirmed]

