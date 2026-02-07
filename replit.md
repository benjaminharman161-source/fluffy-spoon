# LuaPilot - Roblox Studio AI Assistant

## Overview

LuaPilot is a full-stack web application that serves as an AI-powered assistant for Roblox game developers. It provides three core features:

1. **AI Chat Assistant** - A conversational interface for Lua scripting help, powered by OpenAI (via Replit AI Integrations), with streaming responses (SSE) and optional image/vision support
2. **Resource Library** - A curated, searchable collection of Roblox development resources (plugins, models, tutorials)
3. **Dashboard** - A landing page showcasing recent chats and featured resources

The app uses a monorepo structure with a React frontend, Express backend, and PostgreSQL database.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Directory Structure
- `client/` - React frontend (Vite-based SPA)
- `server/` - Express backend API
- `shared/` - Shared types, schemas, and route definitions used by both client and server
- `server/replit_integrations/` - Modular AI integration modules (chat, image, audio, batch)
- `migrations/` - Drizzle-generated database migrations

### Frontend Architecture
- **Framework**: React with TypeScript, bundled by Vite
- **Routing**: Wouter (lightweight client-side router) with three pages: Home (`/`), Chat (`/chat`), Resources (`/resources`)
- **State Management**: TanStack React Query for server state; local React state for UI
- **UI Components**: shadcn/ui component library (new-york style) built on Radix UI primitives with Tailwind CSS
- **Styling**: Tailwind CSS with CSS variables for theming (dark mode, Roblox Studio-inspired blue/dark gray palette). Custom fonts: Inter (body), Outfit (display), JetBrains Mono (code)
- **Animations**: Framer Motion for page transitions and chat bubble animations
- **Code Highlighting**: PrismJS with Lua language support for displaying code blocks in chat
- **Path Aliases**: `@/` maps to `client/src/`, `@shared/` maps to `shared/`, `@assets/` maps to `attached_assets/`

### Backend Architecture
- **Framework**: Express 5 on Node.js with TypeScript (run via tsx in dev, esbuild bundle in production)
- **API Pattern**: RESTful JSON API under `/api/` prefix
- **AI Integration**: OpenAI SDK configured with Replit AI Integrations environment variables (`AI_INTEGRATIONS_OPENAI_API_KEY`, `AI_INTEGRATIONS_OPENAI_BASE_URL`)
- **Chat Streaming**: Server-Sent Events (SSE) for streaming AI responses to the client
- **Image Generation**: OpenAI `gpt-image-1` model for image generation via `/api/generate-image`
- **Audio/Voice**: Optional voice chat support with PCM16 audio streaming, WebM recording, ffmpeg conversion
- **Modular Integrations**: AI features are organized in `server/replit_integrations/` as separate modules (chat, image, audio, batch) each with their own routes, storage, and client files

### Database
- **Database**: PostgreSQL (required, connection via `DATABASE_URL` environment variable)
- **ORM**: Drizzle ORM with `drizzle-zod` for schema-to-validation integration
- **Schema Location**: `shared/schema.ts` and `shared/models/chat.ts`
- **Tables**:
  - `resources` - id, title, description, url, type, category, imageUrl
  - `conversations` - id, title, createdAt
  - `messages` - id, conversationId (FK to conversations), role, content, imageUrl, createdAt
- **Schema Push**: Use `npm run db:push` (drizzle-kit push) to sync schema to database
- **Seeding**: The server auto-seeds the `resources` table on first startup if empty

### Shared Layer
- `shared/schema.ts` - Drizzle table definitions and Zod insert schemas (re-exports chat models)
- `shared/models/chat.ts` - Conversation and message table definitions
- `shared/routes.ts` - API route path constants and Zod response schemas, used by both client and server for type safety

### Build System
- **Dev**: `npm run dev` runs the server with tsx, Vite dev server is set up as middleware with HMR
- **Production Build**: `npm run build` runs a custom build script (`script/build.ts`) that builds the client with Vite and bundles the server with esbuild. Specific dependencies are bundled (allowlisted) to reduce cold start times
- **Production Start**: `npm start` runs the compiled `dist/index.cjs`

### Key API Routes
- `GET /api/resources` - List all resources
- `GET /api/resources/:id` - Get single resource
- `GET /api/conversations` - List all conversations
- `GET /api/conversations/:id` - Get conversation with messages
- `POST /api/conversations` - Create new conversation
- `DELETE /api/conversations/:id` - Delete conversation
- `POST /api/conversations/:id/messages` - Send message and get AI streaming response (SSE)
- `POST /api/generate-image` - Generate image from prompt

## External Dependencies

### Required Services
- **PostgreSQL Database** - Connection string via `DATABASE_URL` environment variable. Required for all data storage
- **Replit AI Integrations (OpenAI)** - Configured via `AI_INTEGRATIONS_OPENAI_API_KEY` and `AI_INTEGRATIONS_OPENAI_BASE_URL` environment variables. Powers the chat assistant, image generation, and optional voice features

### Key NPM Packages
- **Frontend**: React, Vite, wouter, @tanstack/react-query, framer-motion, prismjs, shadcn/ui (Radix UI + Tailwind), lucide-react
- **Backend**: Express 5, OpenAI SDK, drizzle-orm, pg (node-postgres), connect-pg-simple
- **Shared**: Zod, drizzle-zod
- **Build**: esbuild, tsx, TypeScript

### Optional Services
- **ffmpeg** - Required only if using voice/audio features (converts audio formats to WAV for speech-to-text)