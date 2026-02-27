# Gratonite — JS SDK (Shared Packages)

Shared TypeScript packages used across the Gratonite monorepo. All packages are consumed as workspace dependencies (`workspace:*`) by the API, web, and mobile apps.

## Table of Contents

- [Packages](#packages)
- [Getting Started](#getting-started)
- [Package Reference](#package-reference)
  - [@gratonite/types](#gratonitetypes)
  - [@gratonite/db](#gratonitedb)
  - [@gratonite/profile-resolver](#gratoniteprofile-resolver)
- [Available Scripts](#available-scripts)
- [Adding a New Package](#adding-a-new-package)

---

## Packages

| Package | Description |
|---------|-------------|
| [`@gratonite/types`](./types) | Shared TypeScript interfaces and type definitions |
| [`@gratonite/db`](./db) | Drizzle ORM schema, migrations, and database client |
| [`@gratonite/profile-resolver`](./profile-resolver) | Profile URL and avatar resolution utilities |

---

## Getting Started

These packages are designed to be used inside the Gratonite monorepo (managed with pnpm workspaces + Turborepo). They are not published to npm.

### 1. Clone the monorepo

```bash
git clone https://github.com/Gratonite-Labs/js-sdk.git
```

If using as part of the full Gratonite monorepo, this repo lives at `packages/` inside the root.

### 2. Install dependencies

From the repo root:

```bash
pnpm install
```

### 3. Run a package script

```bash
# Generate Drizzle migrations from schema changes
pnpm --filter @gratonite/db generate

# Apply pending migrations
pnpm --filter @gratonite/db migrate

# Type-check all packages
pnpm --filter "./packages/**" typecheck
```

---

## Package Reference

### `@gratonite/types`

**Location:** `types/`

Shared TypeScript type definitions used by all apps. This is a pure TypeScript package — no runtime dependencies.

#### What's inside

- Domain interfaces: `User`, `Server`, `Channel`, `Message`, `Reaction`, `Role`, `Emoji`, etc.
- API response shapes used by both the web client and the API
- Zod schemas (where co-located with types) for runtime validation

#### Usage

```ts
import type { User, Message, Server } from '@gratonite/types'
```

#### Scripts

| Command | Description |
|---------|-------------|
| `pnpm typecheck` | Run `tsc --noEmit` |
| `pnpm lint` | Run ESLint |
| `pnpm clean` | Delete `dist/` |

---

### `@gratonite/db`

**Location:** `db/`

Database schema definitions using [Drizzle ORM](https://orm.drizzle.team) and the PostgreSQL connection client. This is the single source of truth for the database schema.

#### What's inside

- `src/schema/` — Drizzle table definitions (users, servers, channels, messages, roles, etc.)
- `src/index.ts` — exports the database client and all schema tables
- `drizzle.config.ts` — Drizzle Kit config (points to PostgreSQL on port `5433`)
- `drizzle/` — generated SQL migration files

#### Usage

```ts
import { db } from '@gratonite/db'
import { users, messages } from '@gratonite/db'

const allUsers = await db.select().from(users)
```

#### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://postgres:postgres@localhost:5433/gratonite` |

#### Scripts

| Command | Description |
|---------|-------------|
| `pnpm generate` | Generate SQL migration files from schema changes |
| `pnpm migrate` | Apply pending migrations to the database |
| `pnpm studio` | Open Drizzle Studio (visual DB browser) at `https://local.drizzle.studio` |
| `pnpm check` | Validate migration consistency |
| `pnpm typecheck` | Run `tsc --noEmit` |
| `pnpm clean` | Delete `dist/` |

#### Making schema changes

1. Edit the schema in `src/schema/`
2. Generate a migration: `pnpm generate`
3. Review the generated SQL in `drizzle/`
4. Apply it: `pnpm migrate`

---

### `@gratonite/profile-resolver`

**Location:** `profile-resolver/`

Utilities for resolving user profile URLs — avatar images, banners, and default fallbacks. Used by both the web and mobile clients to avoid duplicating URL-building logic.

#### Usage

```ts
import { resolveAvatar, resolveBanner } from '@gratonite/profile-resolver'

const avatarUrl = resolveAvatar(user.id, user.avatar)
const bannerUrl = resolveBanner(user.id, user.banner)
```

#### Scripts

| Command | Description |
|---------|-------------|
| `pnpm typecheck` | Run `tsc --noEmit` |
| `pnpm clean` | Delete `dist/` |

---

## Available Scripts (all packages)

Run from the repo root using pnpm `--filter`:

```bash
# Type-check all packages
pnpm --filter "./packages/**" typecheck

# Clean all packages
pnpm --filter "./packages/**" clean

# Generate + apply DB migrations
pnpm --filter @gratonite/db generate
pnpm --filter @gratonite/db migrate

# Open Drizzle Studio
pnpm --filter @gratonite/db studio
```

---

## Adding a New Package

1. Create a new directory under `packages/`:

```bash
mkdir packages/my-new-package
cd packages/my-new-package
```

2. Create a `package.json`:

```json
{
  "name": "@gratonite/my-new-package",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "scripts": {
    "typecheck": "tsc --noEmit",
    "clean": "rm -rf dist"
  },
  "devDependencies": {
    "typescript": "^5.7.0"
  }
}
```

3. Add a `tsconfig.json` (extend from root if available)
4. Create `src/index.ts`
5. Run `pnpm install` from the repo root to link the workspace
6. Add `"@gratonite/my-new-package": "workspace:*"` to any app that needs it
