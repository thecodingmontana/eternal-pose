# 🧭 Eternal Pose

> A self-hosted One Piece manga reader. Always pointing to the next chapter.

Eternal Pose scrapes chapter data from 1piecemanga.com, stores page images in Postgres, and serves them through a Hono REST API to a Next.js reader frontend — built as a Turborepo monorepo with Drizzle ORM.

---

## Stack

| Layer    | Tech                                                                         |
| -------- | ---------------------------------------------------------------------------- |
| Monorepo | [Turborepo](https://turbo.build)                                             |
| Frontend | [Next.js](https://nextjs.org)                                                |
| API      | [Hono](https://hono.dev)                                                     |
| Database | [Postgres](https://postgresql.org) + [Drizzle ORM](https://orm.drizzle.team) |
| Scraper  | Node.js + Cheerio + Axios                                                    |

---

## Project Structure

```
eternal-pose/
├── apps/
│   ├── scraper/        # Scrapes chapter + page image URLs
│   ├── api/            # Hono REST API
│   └── web/            # Next.js reader UI
└── packages/
    └── db/             # Drizzle schema + shared DB client
```

---

## Getting Started

### Prerequisites

- Node.js 20+
- pnpm 9+ — `npm i -g pnpm`
- A Postgres instance ([Supabase](https://supabase.com) free tier works great)

### Setup

```bash
# 1. Install dependencies
pnpm install

# 2. Copy env and fill in your DATABASE_URL
cp .env.example .env

# 3. Push schema to Postgres
pnpm db:push

# 4. Test the scraper (dry run — no DB writes)
pnpm --filter @onepiece/scraper scrape -- --chapters 5 --dry-run

# 5. Scrape for real
pnpm --filter @onepiece/scraper scrape
```

---

## Scraper

```bash
# Scrape all chapters
pnpm scrape

# Scrape a specific range
pnpm --filter @onepiece/scraper scrape -- --start 1 --end 100

# Dry run (no DB writes)
pnpm --filter @onepiece/scraper scrape -- --dry-run

# Adjust concurrency (default: 2)
pnpm --filter @onepiece/scraper scrape -- --concurrency 3
```

The scraper is **idempotent** — safe to re-run anytime to pick up new chapters without duplicating data. It cross-references MangaDex to verify each chapter exists before saving.

---

## API

```bash
pnpm --filter @onepiece/api dev
# Running on http://localhost:4000
```

### Endpoints

| Method | Path                            | Description                      |
| ------ | ------------------------------- | -------------------------------- |
| GET    | `/health`                       | Health check                     |
| GET    | `/api/chapters`                 | List all chapters (paginated)    |
| GET    | `/api/chapters?page=2&limit=50` | Paginated                        |
| GET    | `/api/chapters?order=desc`      | Latest first                     |
| GET    | `/api/chapters?search=arlong`   | Search by name                   |
| GET    | `/api/chapters/:number`         | Single chapter metadata          |
| GET    | `/api/chapters/:number/pages`   | All page image URLs (for reader) |
| GET    | `/api/chapters/meta/latest`     | Latest chapter available         |

---

## Database Schema

### `chapters`

| Column            | Type        | Notes                          |
| ----------------- | ----------- | ------------------------------ |
| id                | serial PK   |                                |
| chapter_number    | varchar     | `"1"`, `"1101.5"`              |
| name              | text        | Chapter title                  |
| slug              | text        | URL slug from source           |
| source_url        | text        | Scraped from                   |
| cover_image       | text        | First page URL                 |
| cover_images      | jsonb       | First 1–3 page URLs            |
| page_count        | int         | Total pages scraped            |
| mangadex_verified | bool        | Cross-referenced with MangaDex |
| published_at      | timestamptz |                                |

### `pages`

| Column      | Type      | Notes          |
| ----------- | --------- | -------------- |
| id          | serial PK |                |
| chapter_id  | int FK    | Cascade delete |
| page_number | int       | 1-based        |
| image_url   | text      | CDN image URL  |
| alt_text    | text      |                |

---

## Roadmap

- [x] Scraper — chapter list + page images
- [x] Hono API
- [x] Drizzle schema
- [ ] Next.js reader UI
- [ ] Auto-scrape new chapters (cron)
- [ ] Image proxy (avoid CDN CORS issues)

---

## Disclaimer

This project is for personal, non-commercial use. One Piece is created by Eiichiro Oda and published by Shueisha. Please support the official release.
