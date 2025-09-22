# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Manager
This project uses `pnpm` as the package manager.

## Development Commands
- `pnpm dev` - Start development server
- `pnpm build` - Create production build
- `pnpm preview` - Preview production build
- `pnpm check` - Run type checking with svelte-check
- `pnpm check:watch` - Run type checking in watch mode

## Database Commands
The project uses Drizzle ORM with Turso/libSQL:
- `pnpm db:push` - Push schema changes to database
- `pnpm db:generate` - Generate migration files
- `pnpm db:migrate` - Apply migrations
- `pnpm db:studio` - Open Drizzle Studio database GUI

## Architecture
- **Framework**: SvelteKit 5 with TypeScript
- **Styling**: TailwindCSS 4
- **Database**: Turso (libSQL) with Drizzle ORM
- **Database Schema**: Located at `src/lib/server/db/schema.ts`
- **Database Client**: Configured in `src/lib/server/db/index.ts`

## Environment Variables
Required environment variables (see `.env.example`):
- `DATABASE_URL` - Turso database URL or local file path
- `DATABASE_AUTH_TOKEN` - Required for production/remote databases

## Key Files
- `drizzle.config.ts` - Drizzle configuration pointing to schema
- `src/lib/server/db/` - Database setup and schema
- `src/routes/` - SvelteKit pages and layouts