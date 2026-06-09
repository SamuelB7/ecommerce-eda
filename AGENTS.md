# Repository Guidelines

## Project Structure & Module Organization

This repository is an event-driven e-commerce system built as independent NestJS microservices. Root files hold orchestration and docs: `docker-compose.yml`, `README.md`, and `docs/arquitetura-e-stack.md`. Service code lives in `auth-service/`, `orders-service/`, `inventory-service/`, `shipping-service/`, and `notification-service/`. Each service owns its own `src/`, `Dockerfile`, `package.json`, `package-lock.json`, `nest-cli.json`, and `tsconfig.json`. `auth-service` also owns Prisma files in `auth-service/prisma/`.

Keep domain logic inside the owning service. Do not introduce shared business packages. Shared contracts should stay limited to schemas, event contracts, or small technical helpers.

## Build, Test, and Development Commands

Run the full stack from the repository root:

```bash
docker compose up --build
```

Run one service locally from its folder:

```bash
cd auth-service
npm install
npm run start:dev
```

Build a service with `npm run build`. In `auth-service`, use `npm run prisma:generate`, `npm run prisma:migrate:dev`, and `npm run prisma:migrate:deploy` for Prisma client generation and migrations. Health checks are at `/health`; Swagger is at `/docs` on ports `3001` through `3005`.

## Coding Style & Naming Conventions

Use TypeScript and NestJS conventions. Keep controllers thin, put business rules in services, and isolate persistence in repositories. File names use kebab-case with Nest suffixes, such as `auth.controller.ts`, `auth.service.ts`, `access-token.guard.ts`, and `signup.dto.ts`. Classes use PascalCase; variables, functions, and DTO properties use camelCase.

## Testing Guidelines

No test scripts are currently defined in service `package.json` files. When adding tests, use Nest's `@nestjs/testing` package already present in dev dependencies. Put unit tests beside code as `*.spec.ts` or under a service-local `test/` folder for integration tests. Cover controllers, service rules, repositories, Kafka consumers, and idempotency behavior when changed.

## Commit & Pull Request Guidelines

Recent commits use short imperative messages, sometimes with a Conventional Commit prefix: `update docs`, `chore: add microservices as submodules`. Prefer concise messages like `chore: update docker compose` or `feat(auth): add refresh token rotation`.

Pull requests should describe services touched, behavior changed, commands run, and any database or Kafka topic impact. Include Swagger examples for API changes. Link related issues when available.

## Security & Configuration Tips

Local secrets in `docker-compose.yml` are development-only. Do not commit production credentials. Keep each service database private. Version Kafka topics as `<domain>.<aggregate>.<event>.v1`, and update service READMEs when exposed events or ports change.

## Agent-Specific Instructions

Automation agents in this workspace should prefix shell commands with `rtk`, for example `rtk npm run build`, to match the local command proxy setup.
