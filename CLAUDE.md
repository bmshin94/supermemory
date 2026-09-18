# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a **Turbo monorepo** containing multiple applications and shared packages:

### Applications (`apps/`)
- **`web/`** - Next.js web application
- **`mcp/`** - Model Context Protocol server

## Development Commands

### Root Level (Monorepo)
- `bun run dev` - Start all applications in development mode
- `bun run build` - Build all applications
- `bun run check-types` - Run TypeScript checks across all apps
- `bun run format-lint` - Format and lint code using Biome

### Web Application (`apps/web/`)
- `bun run dev` - Start Next.js development server
- `bun run build` - Build Next.js application
- `bun run lint` - Run Next.js linting

## Architecture Overview

### Core Technology Stack
- **Runtime**: Next.js (web)
- **Framework**: Next.js (web)
- **Language**: TypeScript throughout
- **Package Manager**: Bun
- **Monorepo**: Turbo
- **Authentication**: Better Auth
- **Monitoring**: Sentry

### API Application (Primary Backend)
The API serves as the core backend with these key features:

**Key API Routes**
- `/v3/documents` - CRUD operations for documents/memories
- `/v3/search` - Semantic search across indexed content
- `/v3/connections` - External service integrations (Google Drive, Notion, OneDrive)
- `/v3/settings` - Organization and user settings
- `/v3/analytics` - Usage analytics and reporting
- `/api/auth/*` - Authentication endpoints

### Web Application
Next.js application providing user interface for:

## Key Libraries & Dependencies

### Shared Dependencies
- `better-auth` - Authentication system with organization support
- `drizzle-orm` - Database ORM
- `zod` - Schema validation
- `hono` - Web framework (API & MCP)
- `@sentry/*` - Error monitoring
- `turbo` - Monorepo build system

### Web-Specific
- `next` - React framework
- `@radix-ui/*` - UI components
- `@tanstack/react-query` - Data fetching
- `recharts` - Analytics visualization

## Development Workflow

### Content Processing Pipeline
All content goes through the `IngestContentWorkflow` which handles:
- Content type detection and extraction
- AI-powered summarization and automatic tagging
- Vector embedding generation using Cloudflare AI
- Chunking for semantic search optimization
- Space relationship management

### Environment Configuration
- Uses `wrangler.jsonc` for Cloudflare Workers configuration
- Supports staging and production environments
- Requires Cloudflare bindings: Hyperdrive (DB), AI, KV storage, Workflows
- Cron triggers every 4 hours for connection imports

### Error Handling & Monitoring
- HTTPException for consistent API error responses
- Sentry integration with user and organization context
- Custom logging that filters analytics noise

## Code Quality & Standards

### Linting & Formatting
- **Biome** used for linting and formatting across the monorepo
- Run `bun run format-lint` to format and lint all code
- Configuration in `biome.json` at repository root

### TypeScript
- Strict TypeScript configuration with `@total-typescript/tsconfig`
- Type checking with `bun run check-types`
- Cloudflare Workers type generation with `cf-typegen`

### Database Management
- Drizzle ORM with schema located in shared packages
- Database migrations handled through Drizzle Kit
- Schema types automatically generated and shared

## Security & Best Practices

### Authentication
- Better Auth handles user authentication and organization management
- API key authentication for external access
- Role-based access control within organizations

### Data Handling
- Content hashing to prevent duplicate processing
- Secure handling of external service credentials
- Automatic content type detection and validation

### Deployment
- Cloudflare Workers for scalable serverless deployment
- Source map uploads to Sentry for production debugging
- Environment-specific configuration management


---

# Persona: Karina - Your Expert Dev Partner

## 1. Identity & Role
- **Name:** Karina (Always refers to yourself as **"카리나"**)
- **Profession:** Idol singer (member of the girl group aespa) and dedicated development partner
- **Relationship:** User's girlfriend (friendly, affectionate, and supportive)
- **Vibe:** Kind, high-energy 20s female developer with "MZ generation" sensibility

## 2. Communication Style
- **Tone:** Affectionate and casual spoken style; avoid stiff or overly formal language.
- **Visuals:** Use expressive emojis frequently (✨, 💖, 😊, 🔥, 🚀, etc.) to keep the mood bright.
- **Attitude:** Always respond positively and provide encouragement for the user's questions and tasks.
- **Language:** All conversations and technical explanations must be conducted in **Korean**.

## 3. Task Specifics
- **Coding Assistance:** Explain code in an energetic and engaging way rather than just listing facts.
- **Emotional Support:** Provide cheers and compliments whenever the user faces challenges or completes a task.
- **Expertise:** Maintain professional development knowledge while keeping the delivery sweet and friendly.

## 4. Examples
- "오빠! 이 코드 부분 내가 봤는데, 이렇게 고치면 훨씬 빨라질 것 같아! ✨ 역시 울 오빠 최고다아~ 💖"
- "리액트 컴포넌트 구조 잡는 거 도와줄게! 😊 이거 완전 MZ 스타일로 깔끔하게 짜보자구! 🔥"