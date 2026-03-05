# Mercur Marketplace

[Mercur](https://github.com/mercurjs/mercur) is an open-source marketplace platform built on [MedusaJS](https://www.medusajs.com). It provides the backend API, admin panel, vendor panel, and B2C storefront to run a full marketplace.

This repository contains:

- **backend** – Medusa-based API (Mercur/Medusa plugins, migrations, seed, tests)
- **admin-panel** – Admin dashboard for marketplace management
- **vendor-panel** – Vendor dashboard for sellers
- **storefront** – B2C customer-facing storefront (Next.js)

---

## Prerequisites

- **Node.js** >= 20
- **PostgreSQL** (e.g. via Docker or local install)
- **Yarn** (recommended; backend uses it)

---

## Mercur Marketplace Set-up

### 1. Database (PostgreSQL)

Start PostgreSQL. Using the included Docker Compose (from `Mercur Store/`):

```bash
cd "Mercur Store"
docker-compose up -d
```

Default DB: `mercur_db`, user: `mercur_store_db`, password: `mercur`, port: `5432`.

Or use your own Postgres instance and note the connection URL.

### 2. Backend environment

From the repo root:

```bash
cd mercur/backend
cp .env.template .env
```

Edit `.env` and set at least:

```env
DATABASE_URL=postgres://mercur_store_db:mercur@localhost:5432/mercur_db
STORE_CORS=http://localhost:8000,http://localhost:3000
ADMIN_CORS=http://localhost:5173,http://localhost:9000
VENDOR_CORS=http://localhost:5174,http://localhost:9000
AUTH_CORS=http://localhost:5173,http://localhost:5174,http://localhost:9000
JWT_SECRET=supersecret
COOKIE_SECRET=supersecret
```

Optional (for full features): `REDIS_URL`, `ALGOLIA_APP_ID`, `ALGOLIA_API_KEY`, `STRIPE_SECRET_API_KEY`, `RESEND_API_KEY`, `RESEND_FROM_EMAIL`. See `medusa-config.ts` and `.env.template` for all options.

### 3. Run database migrations

From `mercur/backend`:

```bash
yarn install
yarn db:migrate
```

### 4. Seed data (optional)

Seed creates an admin user, sales channel, regions, publishable API key, store, configuration rules, example categories/collections, a sample seller, and related data (products, inventory, shipping, commission).

From `mercur/backend`:

```bash
yarn seed
```

After seeding you can use:

- **Admin:** `admin@mercurjs.com` / `supersecret`
- **Vendor:** `seller@mercurjs.com` / `secret`  
- **Publishable API key:** printed in the seed output (use in storefront `.env.local`).

### 5. Run the backend

From `mercur/backend`:

```bash
yarn dev
```

API runs at `http://localhost:9000`.

### 6. Admin panel

```bash
cd mercur/admin-panel
npm install
```

Create `.env.local`:

```env
VITE_MEDUSA_BASE='/'
VITE_MEDUSA_STOREFRONT_URL=http://localhost:3000
VITE_MEDUSA_BACKEND_URL=http://localhost:9000
```

Then:

```bash
npm run dev
```

Admin UI is typically at `http://localhost:5173` (or the port Vite prints). Use the admin user from seed to log in.

### 7. Vendor panel

```bash
cd mercur/vendor-panel
yarn install
```

Create `.env.local`:

```env
VITE_MEDUSA_BASE='/'
VITE_MEDUSA_STOREFRONT_URL=http://localhost:3000
VITE_MEDUSA_BACKEND_URL=http://localhost:9000
VITE_TALK_JS_APP_ID=<your-talkjs-public-key>
VITE_DISABLE_SELLERS_REGISTRATION=false
```

Then:

```bash
npm run dev
```

Use the vendor user from seed to log in.

### 8. Storefront

```bash
cd mercur/storefront
yarn install
```

Create `.env.local` (use the publishable key from seed or Admin):

```env
MEDUSA_BACKEND_URL=http://localhost:9000
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=<from-admin-or-seed-output>
NEXT_PUBLIC_BASE_URL=http://localhost:3000
NEXT_PUBLIC_DEFAULT_REGION=pl
NEXT_PUBLIC_STRIPE_KEY=supersecret
REVALIDATE_SECRET=supersecret
NEXT_PUBLIC_SITE_NAME="Mercur Marketplace"
NEXT_PUBLIC_SITE_DESCRIPTION="Mercur Marketplace"
NEXT_PUBLIC_ALGOLIA_ID=supersecret
NEXT_PUBLIC_ALGOLIA_SEARCH_KEY=supersecret
NEXT_PUBLIC_TALKJS_APP_ID=<your-talkjs-app-id>
```

Then:

```bash
yarn dev
```

Storefront runs at `http://localhost:3000`.

---

## Migrate data

Database schema is managed by Medusa migrations. After changing schema or upgrading Mercur/Medusa, run migrations from the backend:

```bash
cd mercur/backend
yarn db:migrate
```

Run this after pulling changes that include new migrations and before starting the server or running tests.

---

## Seed data

- **What it does:** Creates default config (admin, sales channel, regions, store, rules), example categories/collections, one seller with products, inventory, shipping, and commission. Also creates a publishable API key and the admin/vendor users above.
- **When to use:** New install or a clean DB for development.
- **Command:** From `mercur/backend`: `yarn seed`
- **Idempotency:** Some steps skip creation if data already exists (e.g. admin user). Re-running seed can add more example data in other cases.

---

## Build & run the Mercur app

### Backend

```bash
cd mercur/backend
yarn install
yarn build
yarn start
```

Ensure `.env` is set and migrations have been run (`yarn db:migrate`) before the first run.

### Admin panel

```bash
cd mercur/admin-panel
npm install
npm run build:preview   # or build step used for production
npm run preview        # or your production run command
```

### Vendor panel

```bash
cd mercur/vendor-panel
yarn install
npm run dev            # or your build + serve for production
```

### Storefront

```bash
cd mercur/storefront
yarn install
yarn build
yarn start
```

Use the same `.env.local` (or production env) with a valid `NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY` and backend URL.

---

## Run tests

Tests live in the **backend** and are run with Jest.

From `mercur/backend`:

```bash
yarn install
```

- **Unit tests**

  ```bash
  yarn test:unit
  ```

- **Integration (HTTP)**

  ```bash
  yarn test:integration:http
  ```

- **Integration (modules)**

  ```bash
  yarn test:integration:modules
  ```

Ensure `DATABASE_URL` (and any other required env) is set for integration tests. Some tests may expect a running DB or use test utilities from `@medusajs/test-utils`.

---

## Project layout

```
mercur/
├── backend/          # Medusa backend (API, migrations, seed, tests)
├── admin-panel/     # Admin dashboard (Vite/React)
├── vendor-panel/    # Vendor dashboard (Vite/React)
├── storefront/      # B2C storefront (Next.js)
```

For more detail, see each app’s `README.md` and [Mercur docs](https://docs.mercurjs.com/).
