# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Terralist is a private Terraform registry for providers and modules following HashiCorp protocols. It allows secure distribution of confidential Terraform modules and providers with a web-based management interface.

**Tech Stack:**
- Backend: Go 1.25 with Gin web framework, GORM ORM
- Frontend: Svelte 4 + Vite + TypeScript + Tailwind CSS
- Database: SQLite, MySQL, or PostgreSQL
- Storage: AWS S3, Azure Blob, Google Cloud Storage, Local, or Proxy mode

## Common Commands

```bash
# Install dependencies
task tidy

# Build everything (frontend + backend)
task build

# Build components separately
task build:frontend
task build:backend              # Release mode
task build:backend -- --debug   # Debug mode

# Linting
task lint                       # Run all linters
task lint:frontend -- --fix     # Fix frontend issues
task lint:backend -- --fix      # Fix backend issues

# Testing
task test:unit                  # Run unit tests
task test:unit -- --summary     # With formatted output
task test:e2e                   # E2E tests (requires running server)
task test:generate-mocks        # Generate mock files (run before tests if mocks are missing)
task test:prepare-database      # Create test SQLite database

# Run the server
./terralist server --port=5758 --database-type=sqlite --database-path=test.db
```

## Architecture

### Backend Structure (`internal/server/`)

The backend follows a layered architecture:

```
Controllers (HTTP handlers) → Services (business logic) → Repositories (data access) → Database
```

- **controllers/**: HTTP endpoint handlers (login, module, provider, authority, artifact, probe)
- **services/**: Business logic layer
- **repositories/**: GORM-based data access
- **models/**: DTOs and domain models
- **handlers/**: Middleware (auth, logging)

### Key Packages (`pkg/`)

- **auth/**: OAuth providers (GitHub, GitLab, Bitbucket, OIDC) + JWT handling
- **database/**: Database engine abstraction with factory pattern
- **storage/**: Storage backends (S3, Azure, GCS, Local) with factory pattern
- **session/**: Cookie-based session management
- **rbac/**: Casbin-based role-based access control
- **api/**: REST API routing utilities

### Frontend (`web/`)

Svelte SPA with:
- **pages/**: Route components (Login, Dashboard, Artifact, Settings)
- **components/**: Reusable UI components
- **api/**: Axios-based API clients

The frontend builds to `web/dist/` and is embedded into the Go binary via `web/embed.go`.

## Key Patterns

- **Factory Pattern**: Used for database, storage, auth, and session providers
- **Dependency Injection**: Services receive dependencies via constructors
- **Embedded Frontend**: Static assets compiled into the Go binary

## Testing

Unit tests use Go's testing package with Mockery-generated mocks. Run `task test:generate-mocks` if you see mock-related errors.

E2E tests use Venom (YAML-based REST API testing) in `e2e/suites/`. These require a running Terralist server.

## Configuration

The server accepts configuration via CLI flags, environment variables (`TERRALIST_*`), or YAML config file. Key flags are defined in `pkg/api/flags.go`.
