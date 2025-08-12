# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Plane is an open-source project management tool for tracking issues, managing cycles (sprints), and building product roadmaps. It consists of:
- **Frontend Apps**: Next.js applications (web, admin, space, live)
- **Backend API**: Django REST framework API (apps/api)
- **Packages**: Shared TypeScript packages for UI components, utilities, and configurations

## Essential Commands

### Development
```bash
# Initial setup (required once)
./setup.sh

# Start Docker services (Redis, RabbitMQ, MinIO, PostgreSQL)
docker compose -f docker-compose-local.yml up

# Run all frontend apps in development
yarn dev

# Run specific app
yarn workspace web dev        # Main app on port 3000
yarn workspace admin dev      # Admin on port 3002
yarn workspace space dev      # Public spaces on port 3003
```

### Testing
```bash
# Backend (Django) tests
cd apps/api
python run_tests.py            # Run all tests
pytest -m smoke                 # Run smoke tests only
pytest plane/tests/            # Run specific test directory

# Frontend tests - no test scripts configured yet
```

### Code Quality
```bash
# Linting and formatting
yarn check                      # Run all checks (lint, types, format)
yarn fix                        # Fix all auto-fixable issues

# Individual checks
yarn check:lint                 # ESLint check
yarn check:types               # TypeScript type checking
yarn check:format              # Prettier format check

# Per-app checks (e.g., for web app)
yarn workspace web check:lint
yarn workspace web check:types
```

### Build & Production
```bash
# Build all apps
yarn build

# Build specific app
yarn workspace web build

# Clean build artifacts
yarn clean
```

## Architecture & Key Patterns

### Frontend Architecture (Next.js Apps)

**State Management**: MobX stores with React hooks
- Store instances in `store/` directories
- Use `observer` HOC for reactive components
- Access stores via custom hooks (e.g., `useProject()`, `useUser()`)

**Component Structure**:
- Components use TypeScript with explicit prop types
- Observer pattern for MobX reactivity
- Tailwind CSS for styling with `cn()` utility for conditional classes
- Icons from `lucide-react` package

**API Integration**:
- Services layer in `@plane/services` package
- Axios for HTTP requests
- SWR for data fetching and caching

### Backend Architecture (Django)

**Django Apps Structure**:
- `plane/app/` - Core application logic
- `plane/api/` - API endpoints
- `plane/authentication/` - Auth handlers
- `plane/bgtasks/` - Celery background tasks
- `plane/db/` - Database models

**API Patterns**:
- Django REST Framework for API views
- Serializers in `app/serializers/`
- ViewSets in `app/views/`
- URL routing in `app/urls/`

### Shared Packages

Located in `packages/`:
- `@plane/ui` - Reusable UI components
- `@plane/types` - TypeScript type definitions
- `@plane/constants` - Shared constants
- `@plane/services` - API service layer
- `@plane/utils` - Utility functions
- `@plane/i18n` - Internationalization

## Development Guidelines

### Frontend Conventions
- Use existing UI components from `@plane/ui` 
- Follow existing MobX store patterns
- Use TypeScript strictly - avoid `any` types
- Implement observer pattern for reactive components
- Use translation keys from `@plane/i18n` for user-facing text

### Backend Conventions
- Follow Django REST Framework patterns
- Write tests for new endpoints
- Use serializers for data validation
- Implement proper permission classes
- Add Celery tasks for long-running operations

### Database & Migrations
```bash
# Django migrations
cd apps/api
python manage.py makemigrations
python manage.py migrate
```

## Environment Configuration

Key environment files:
- `.env` - Main configuration
- `apps/web/.env` - Web app specific
- `apps/api/.env` - API specific
- `apps/admin/.env` - Admin panel specific
- `apps/space/.env` - Public spaces specific

Use `setup.sh` to initialize all `.env` files from examples.

## Key Technologies

- **Frontend**: Next.js 14, React 18, TypeScript, MobX, Tailwind CSS, SWR
- **Backend**: Django 4.2+, Django REST Framework, Celery, PostgreSQL 14+
- **Infrastructure**: Docker, Redis 6.2+, RabbitMQ, MinIO (S3-compatible storage)
- **Package Management**: Yarn workspaces, Turbo for monorepo

## Important Notes

- The project uses Yarn workspaces - always use `yarn` instead of `npm`
- Frontend apps run on specific ports: web (3000), admin (3002), space (3003), live (3004)
- God mode (instance admin) is accessible at `http://localhost:3001/god-mode/`
- All frontend code uses strict TypeScript - ensure proper typing
- MobX stores handle global state - avoid local state for shared data
- Use existing patterns for consistency across the codebase