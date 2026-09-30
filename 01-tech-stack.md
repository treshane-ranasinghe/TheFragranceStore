# Tech Stack, Project Structure & Quality Targets

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

## Tech Stack

| Part | Choice | Why |
|---|---|---|
| Framework | **Next.js 15 (App Router) + React + TypeScript** | Pages are rendered on the server so Google gets full product pages; built-in SEO tools (metadata, sitemap, image optimisation) |
| Styling | Tailwind CSS | Quick, consistent, responsive design |
| Routing | Next.js file-based routing (`app/` folder) | Each folder = a URL; clean, SEO-friendly addresses |
| Data on public pages | React **Server Components** + `@supabase/ssr` | Product and category pages are built on the server and cached |
| Data in interactive parts | TanStack Query + `@supabase/supabase-js` | Cart, account, and admin screens that update live in the browser |
| Mutations | Next.js **Server Actions** calling Supabase RPC functions | Checkout, admin edits; runs on the server |
| Forms | React Hook Form + Zod | Form handling and validation |
| Images | `next/image` with Supabase Storage | Automatic resizing, WebP/AVIF, lazy loading, which makes pages faster and helps Google rankings |
| Fonts | `next/font` (Cormorant Garamond + Inter) | Fonts served from your own domain, no layout jump |
| Database | Supabase Postgres | Relational data, SQL, security rules |
| Auth | Supabase Auth via `@supabase/ssr` (email + password, anonymous sign-in) | Login works on both server and browser (cookie-based) |
| File storage | Supabase Storage | Product images |
| Real-time | Supabase Realtime | Live cart activity and new-order alerts in admin |
| Server logic | Postgres functions (RPC) | Safe order placement (prices checked on the server) |
| Email (optional) | Resend (from a Server Action or Supabase Edge Function) | Order emails |
| Hosting | **Vercel** (Next.js), Supabase (backend) | Vercel is made by the Next.js team; free tier available |

### How pages are rendered

| Page type | How | Why |
|---|---|---|
| Home, shop, category, brand, product, info pages | **Server-rendered and cached** (Incremental Static Regeneration) | Fast, fully readable by Google |
| Cache refresh | When an admin saves a product, category, brand, banner, homepage section, promotion, or bundle, the Server Action calls `revalidatePath` / `revalidateTag`, so the live page updates within seconds | No stale prices or stock on Google-visible pages |
| Stock and price at checkout | Always read live from the database | Cached pages can never cause a wrong price at checkout |
| Scheduled content (banners, promotions, bundles) | A `pg_cron` job triggers page refresh when content starts or ends; CMS pages also re-check every 5 minutes | Sales start and end on time |
| Admin preview | Next.js **Draft Mode**, admins only | See unpublished or scheduled content before it goes live |
| Cart, checkout, account, admin | **Client-side** (browser), not indexed by Google | Personal and interactive; marked `noindex` |

## Project Structure (Next.js)

```
app/
  (store)/                     public shop — server-rendered, indexed by Google
    layout.tsx                 header, footer
    page.tsx                   home
    shop/page.tsx
    category/[slug]/page.tsx
    brand/[slug]/page.tsx
    product/[slug]/page.tsx    generateMetadata + Product JSON-LD
    promotions/[slug]/page.tsx
    bundles/page.tsx
    bundle/[slug]/page.tsx
    about/ contact/ delivery-policy/ returns-policy/ terms/ privacy/
  (shop-private)/              noindex
    cart/page.tsx
    checkout/page.tsx
    order/[orderNumber]/page.tsx   Order Received page
    account/...                    profile, addresses, orders
  (auth)/login, register, forgot-password
  admin/                       protected by middleware (admin role)
    layout.tsx                 sidebar
    page.tsx                   dashboard
    products/ categories/ brands/ orders/ customers/
    payments/bank/ payments/cod/
    cart-activity/
    settings/ (payments/bank-transfer, payments/cod, delivery)
    content/ (homepage, hero, announcements, media)
    promotions/ bundles/
  sitemap.ts
  robots.ts
  layout.tsx                   fonts, global styles
  globals.css
components/
  ui/                          buttons, inputs, modals, badges
  cms/                         HeroSlider, SectionRenderer (one component per section type), Countdown, AnnouncementBar
  store/                       ProductCard, CartDrawer, Filters, OrderedItems...
  admin/                       DataTable, ProductForm, OrderTimeline, ActivityFeed...
lib/
  supabase/server.ts           server client (cookies)
  supabase/client.ts           browser client
  supabase/middleware.ts       session refresh
  actions/                     Server Actions: checkout, admin CRUD, payments (+ revalidate)
  seo/                         metadata + JSON-LD helpers
  format.ts                    Rs. currency, dates, phone
middleware.ts                  protects /admin and /account
types/                         generated Supabase types
supabase/
  migrations/                  SQL for tables, RLS, functions, triggers, views, pg_cron
  seed.sql                     sample categories, brands, products, delivery methods
```

**Environment variables (Vercel):** `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `NEXT_PUBLIC_SITE_URL`, `RESEND_API_KEY` (optional), `REVALIDATE_SECRET` (for scheduled content refresh).

## Non-functional Requirements

- **Mobile-first:** most Sri Lankan shoppers will use phones.
- **Speed:** product pages load in under 2 seconds on 4G; images compressed and lazy-loaded.
- **SEO:** see `14-seo.md`.
- **Accessibility:** readable contrast, keyboard navigation, image alt text.
- **Reliability:** order placement is all-or-nothing (no half-created orders).
- **Backups:** Supabase daily backups (paid plan) or scheduled manual exports.
- **Currency format:** `Rs. 12,500.00`.
- **Phone validation:** Sri Lankan mobile numbers (`07XXXXXXXX` or `+947XXXXXXXX`).
- **Core Web Vitals** (Google's page-speed score): LCP < 2.5 s, CLS < 0.1, INP < 200 ms on mobile.
