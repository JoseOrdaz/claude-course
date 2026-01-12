# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
# Initial setup (install deps, generate Prisma client, run migrations)
npm run setup

# Development server with Turbopack
npm run dev

# Production build
npm run build

# Linting
npm run lint

# Run tests
npm test

# Run a single test file
npx vitest run src/lib/__tests__/file-system.test.ts

# Reset database
npm run db:reset

# Development server in background (logs to logs.txt)
npm run dev:daemon
```

Note: The project works without `ANTHROPIC_API_KEY` using a mock provider that returns static responses.

## Architecture Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in chat, and Claude generates code that renders in a live preview.

### Core Concepts

**Virtual File System**: Components are generated into an in-memory file system (`VirtualFileSystem` class in `src/lib/file-system.ts`), not written to disk. This allows safe preview and editing without touching the actual filesystem.

**AI Tool Integration**: The chat API (`src/app/api/chat/route.ts`) uses Vercel AI SDK with two custom tools:
- `str_replace_editor`: Creates, views, and modifies files (view, create, str_replace, insert commands)
- `file_manager`: Renames and deletes files

**File System Context**: React context (`src/lib/contexts/file-system-context.tsx`) manages the virtual file system state on the client side, handling tool calls from the AI to update files in real-time.

### Key Directories

- `src/app/` - Next.js App Router pages and API routes
- `src/components/` - React components (chat, editor, preview, auth, ui)
- `src/lib/` - Core utilities (file system, auth, Prisma, AI tools, prompts)
- `src/actions/` - Server actions for project CRUD
- `prisma/` - Database schema (SQLite)

### Generated Component Rules

The AI generates components following these constraints (defined in `src/lib/prompts/generation.tsx`):
- Every project must have a root `/App.jsx` as the entry point
- Components use Tailwind CSS for styling (not inline styles)
- Non-library imports use the `@/` alias (e.g., `@/components/Calculator`)
- No HTML files - `App.jsx` is the sole entry point

### Preview System

The preview (`src/components/preview/PreviewFrame.tsx`) uses `@babel/standalone` to transpile JSX in the browser and renders components in an iframe with Tailwind CSS support.

### Data Flow

```
User chat input → POST /api/chat → Claude AI (with tools)
    → Tool calls update VirtualFileSystem → SSE streaming response
    → FileSystemContext handles tool results → PreviewFrame re-renders
    → On completion: save to Prisma DB (if authenticated)
```

### Authentication

JWT-based auth using `jose` library with HTTP-only cookies. Middleware (`src/middleware.ts`) protects `/api/projects` and `/api/filesystem` routes. Anonymous users can use the app without signup but projects aren't persisted.

## Database

The database schema is defined in `prisma/schema.prisma`. Reference it anytime you need to understand the structure of data stored in the database.

## Code Style

- Use comments sparingly. Only comment complex code.
