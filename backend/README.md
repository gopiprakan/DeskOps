# Backend

The Phase 1 API will be implemented here using Node.js 22, TypeScript and Express or NestJS. The framework choice must be recorded in the technical design/ADR before implementation.

Organise the API into layers:
- Routes/controllers: HTTP input and response handling.
- Services: business rules and transactions.
- Repositories: tenant-scoped database access.
- Middleware: authentication, organisation context, permissions and request IDs.
- Workers: BullMQ background processing.

Required foundations:
- PostgreSQL and Prisma migrations.
- Zod request validation.
- Versioned routes under /api/v1.
- OpenAPI documentation at /docs.
- Uniform error response format.
- /health and /ready endpoints.

Tenant identity must come from authenticated server-side context. Never trust an organisationId supplied by the browser as authorization.
