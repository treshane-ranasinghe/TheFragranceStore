# SEO Plan (Google Traffic)

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

| Item | How it's done in Next.js |
|---|---|
| Server-rendered pages | Product, category, brand, and home pages are built on the server, so Google sees the full content |
| Page titles and descriptions | `generateMetadata()` on every page, e.g. *"Dior Sauvage EDP 100ml — Price in Sri Lanka | The Fragrance Store"* |
| Admin-editable SEO fields | Products, categories, and brands get optional `seo_title` and `seo_description`; if empty, they're built automatically from the name, brand, and size |
| Clean URLs | `/product/dior-sauvage-edp`, `/category/men`, `/brand/dior` (slugs from the product name, editable) |
| Rich results (price and stock in Google) | **Product** structured data (JSON-LD) on product pages: name, brand, image, price in LKR, availability. Plus **BreadcrumbList**, and **Organization** on the home page |
| Sitemap | `app/sitemap.ts` builds `sitemap.xml` from active products, categories, brands, **live promotions, and live bundles**, with last-updated dates |
| Robots | `app/robots.ts` allows the store; blocks `/admin`, `/cart`, `/checkout`, `/account`, `/order` |
| Duplicate pages | Canonical URLs; filtered/sorted shop pages point back to the main listing |
| Social sharing | Open Graph + Twitter image and text per product, so WhatsApp and Facebook links show the photo, name, and price |
| Images | `next/image`; alt text from product name + brand |
| Hidden / deleted products | Hidden → page returns 404 and drops from the sitemap. Slug changed → automatic 301 redirect from the old URL (`product_slug_redirects` table) |
| Promotions & bundles | Promotion and bundle pages are server-rendered and indexable, with their own SEO title/description. Product structured data shows the **sale price** with `priceValidUntil` = promotion end date. Ended promotions leave the sitemap but the page stays reachable |
| Content for search | Category and brand pages have an admin-editable intro paragraph (e.g. "Men's perfumes in Sri Lanka") |
| Google Search Console | Verify the domain, submit the sitemap, and watch indexing after launch |
| Local search | Google Business Profile for the store (outside the website, but links to it) |
