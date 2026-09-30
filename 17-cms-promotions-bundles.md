# Content Management: Hero Banners, Homepage Sections, Promotions & Bundles

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

The admin panel includes a simple **content management system (CMS)**. The store owner can change what customers see, such as hero images, homepage sections, sales and product bundles, **without a developer**. Everything is edited under **Admin → Content**, **Admin → Promotions** and **Admin → Bundles**.

## Overview

| Feature | What it does | Example |
|---|---|---|
| **Hero banners** | The large images at the top of the homepage, as a slideshow. Each can be scheduled. | "Avurudu Sale — up to 30% off" banner from 1–20 April |
| **Announcement bar** | A thin strip at the very top of every page | "Free delivery on orders over Rs. 10,000" |
| **Homepage sections** | Build the homepage from blocks you can add, reorder, hide, and schedule | "New arrivals" row, "Gift sets" row, promo banner, brand logos |
| **Promotions** | Timed sales that automatically lower prices on chosen products, categories, or brands | 20% off all Dior, 10–15 Nov |
| **Bundles** | Several products sold together at one special price | "His & Hers Gift Set": Sauvage 100 ml + J'adore 50 ml for Rs. 55,000 (save Rs. 8,000) |
| **Media library** | One place for all uploaded banner and content images, reusable anywhere | — |

**Common to all content:**
- **Active** on/off switch
- Optional **start and end date/time**: content appears and disappears automatically
- **Preview** before publishing: the admin sees the page exactly as customers will, including scheduled items (Next.js Draft Mode)
- **Publish** refreshes the live pages within seconds (cache revalidation). Scheduled start and end times refresh the pages automatically (see *Scheduling* below).

---

## 1. Hero banners (homepage slideshow)

### Admin screen: Content → Hero banners
A list of banners with thumbnail, headline, status (*Live*, *Scheduled*, *Ended*, *Hidden*), dates, and **drag to reorder**.

### Banner fields

| Field | Required | Notes |
|---|---|---|
| Internal name | Yes | For the admin only, e.g. "Avurudu 2027 main" |
| **Desktop image** | Yes | Recommended 1920 × 800 px, JPG/PNG/WebP, max 5 MB |
| **Mobile image** | Recommended | 800 × 1000 px (portrait). Falls back to the desktop image, centre-cropped |
| Image alt text | Yes | Describes the image (for accessibility and Google) |
| Small top line (eyebrow) | No | e.g. "LIMITED TIME" (gold, uppercase) |
| Headline | No | e.g. "Avurudu Sale" |
| Sub-heading | No | e.g. "Up to 30% off selected fragrances" |
| Button 1 text + link | No | Link picker: a **promotion**, **bundle**, **product**, **category**, **brand**, or custom URL |
| Button 2 text + link | No | Optional second button (outline style) |
| Text position | — | Left / Centre / Right |
| Overlay | — | None / Light / Medium / Dark (Deep Forest tint, so text stays readable) |
| Text colour | — | Cream (on dark images) or Dark (on light images) |
| Start / end date-time | No | Blank = always on while active |
| Active | — | On/off |

### How it shows on the site
- Up to **5 live banners** rotate every 6 seconds. Rotation pauses on hover, can be swiped on phones, and has dot navigation. One banner = no rotation.
- The **first banner loads immediately** (priority image) for fast page speed; others load after.
- The whole banner is clickable if it has only one link.
- If no banner is live, a **default banner** (set in settings) is shown so the homepage is never empty.

---

## 2. Announcement bar
- Text (max ~90 characters), optional link, background colour (Dark Green / Gold / Beige), start/end, active.
- Several announcements can be scheduled; the newest live one is shown.
- Customers can close it; it stays closed for that visitor for 24 hours (browser storage).

---

## 3. Homepage sections (page builder)

### Admin screen: Content → Homepage
The homepage is a **list of sections** the admin can:
- **Add** from a menu of section types
- **Drag to reorder**
- **Edit**, **duplicate**, **hide / show**, **delete**
- **Schedule** (start/end), e.g. show a "Christmas gifts" row only in December
- **Preview** on desktop and phone before publishing

### Section types

| Section type | Settings |
|---|---|
| **Hero slideshow** | Uses the hero banners above (only one allowed, always at the top) |
| **Product row / grid** | Title, sub-title, layout (slider row or grid), number of products (4–12), and **product source**: hand-picked products, a category, a brand, a promotion, *New arrivals*, *Best sellers*, *On sale*, or *Featured*. Optional "View all" link |
| **Promotion banner** | Wide image + headline + button, linked to a promotion. Optional **countdown timer** to the promotion's end |
| **Bundle showcase** | Title, and either hand-picked bundles or "all live bundles" |
| **Category tiles** | Pick categories and their order; uses each category's image |
| **Brand strip** | Pick brands; shows their logos in a row |
| **Image + text** | Image left or right, headline, paragraph, button (e.g. "Our story", "Why buy from us") |
| **Text block** | Heading + formatted text (bold, links, lists) |
| **Trust badges** | Up to 4 icons + short text, e.g. "100% original", "Island-wide delivery", "Cash on delivery" |

Every section also has: **background** (Cream / Beige / Dark Green), **top and bottom spacing** (Small / Medium / Large), **active**, and **start/end**.

### Default homepage (set up at launch)
Hero slideshow → Trust badges → New arrivals → Promotion banner (if a promotion is live) → Shop by category → Best sellers → Bundle showcase → Brand strip → Image + text ("Our story").

---

## 4. Promotions (sales)

### Admin screen: Promotions
List with name, discount, what it applies to, dates, status (*Live*, *Scheduled*, *Ended*, *Draft*), and number of products affected.

### Promotion fields

| Field | Notes |
|---|---|
| Name | e.g. "Avurudu Sale 2027" |
| URL slug | e.g. `avurudu-sale` → page `/promotions/avurudu-sale` |
| Description | Shown on the promotion page |
| Banner image | For the promotion page and promotion-banner sections (desktop + mobile) |
| **Discount type** | **Percentage off** (e.g. 20%) or **Fixed amount off** per item (e.g. Rs. 2,000) |
| **Applies to** | Any mix of: whole store, categories, brands, specific products, specific sizes (variants) |
| **Exclusions** | Products or brands to leave out |
| **Badge text** | Shown on product cards, e.g. "Avurudu −20%" |
| Start / end date-time | Required |
| Show countdown | On/off: "Ends in 2d 5h" on product pages and banners |
| SEO title / description | For the promotion page |
| Active | On/off (Draft until switched on) |

### Pricing rules
- A promoted item shows the **sale price**, the **original price struck through**, and the **badge**.
- **Prices are never lower than zero.** Fixed discounts larger than the price are blocked when saving.
- **No stacking:** if a product is in two live promotions, the customer gets the **one that gives the lowest price**. Discounts are not added together.
- **Manual "compare-at" prices** (set on the product) still work outside promotions. When a promotion applies, the promotion price wins if it is lower.
- Promotions apply to **single products only**. Bundles have their own fixed price (section 5), unless the admin ticks *"Include bundles"* on the promotion.
- **Prices are calculated in the database** by one function, `effective_price(variant_id)`, used everywhere: product pages, cart, and `place_order`. What the customer sees and what they pay always match.
- **Checkout uses the price at the moment the order is placed.** If a promotion ends while a customer is in checkout, the cart shows *"Price updated: promotion ended"* before they can place the order.
- Each order line saves the **original price, discount, promotion used, and final price**, so order history and reports always show what was paid and why.

### Promotion page (`/promotions/[slug]`)
- Banner, title, description, countdown (optional), and a grid of every product in the promotion, with the normal filters and sort.
- Indexed by Google while live (see `14-seo.md`).
- After it ends: shows *"This offer has ended"* and links to current offers. The page stays reachable (no broken links) but drops from the sitemap.

---

## 5. Bundles (product sets)

A **bundle** is several products sold together as **one item at one price**, like a gift set.

### Admin screen: Bundles
List with image, name, items, bundle price, "you save", availability, dates, status.

### Bundle fields

| Field | Notes |
|---|---|
| Name, URL slug | e.g. "His & Hers Gift Set" → `/bundle/his-and-hers-gift-set` |
| Description | Rich text |
| Images | Bundle photo(s); if none, a collage of the item images is generated |
| **Items** | Pick product sizes (variants) and quantity each, e.g. Sauvage EDP 100 ml × 1, J'adore EDP 50 ml × 1 |
| **Bundle price** | Fixed price, e.g. Rs. 55,000 |
| "You save" | Calculated automatically: (sum of the items' current prices) − bundle price. Shown as "Save Rs. 8,000" |
| Category | Optional, e.g. "Gift Sets", so it appears in category pages |
| Max per order | Optional, e.g. 2 |
| Start / end date-time | Optional (for limited-time sets) |
| SEO title / description | For Google |
| Active | On/off |

Saving is blocked if the bundle price is **higher** than buying the items separately. A warning appears if the saving is under 5%.

### Stock (no separate bundle stock)
- A bundle has **no stock of its own**. Its availability is calculated from its items: *how many complete sets can be made from current stock.*
  Example: Sauvage has 6, J'adore has 3 → **3 bundles** available.
- If any item is out of stock or hidden, the bundle shows **"Out of stock"**.
- Following the stock rules in `04-checkout-orders-stock.md`, **stock of each item is reduced when the order is confirmed** (bank payment recorded / COD confirmed), and returned on cancellation if it had been reduced.
- Checkout checks the **combined** need. If a customer has the bundle *and* a loose Sauvage in the cart, both are counted against Sauvage's stock.

### Bundles in the cart, orders, and history
- A bundle is **one line in the cart**, showing the bundle name and price, with the items listed underneath.
- In the order, the bundle is saved as **one main line** (bundle name and price paid) plus **item lines** underneath (each product and size, for packing and stock). Item lines have no separate price.
- Packing slips and invoices list the bundle and its contents.
- **Buy again** re-adds the bundle if it is still live; otherwise it offers the individual items.
- The Cart Activity Monitor logs bundle add/remove like any product, labelled "Bundle".

### Bundle pages
- `/bundles`: all live bundles
- `/bundle/[slug]`: images, price, "Save Rs. X", item list (each linking to its product page), availability, **Add to cart**
- Product pages show **"Also available in these sets"** when the product is part of a live bundle.

---

## 6. Media library
- **Content → Media**: all uploaded content images in a grid, with upload (drag and drop, multiple), search by name, alt text, size, and "where used".
- Any image field in hero banners, sections, promotions, or bundles can **upload new** or **choose from the library**.
- Deleting an image that is in use is blocked, with a list of where it's used.
- Stored in the `cms-media` bucket; served through `next/image` (auto-resized, WebP/AVIF).

---

## 7. Scheduling and page refresh
- Content with start/end times is filtered by the database (`now()` between start and end), so customers only ever receive live content.
- Public pages are cached for speed (see `01-tech-stack.md`). To make scheduled changes appear on time:
  - A `pg_cron` job runs **every 5 minutes**. When any banner, section, announcement, promotion, or bundle **starts or ends**, it calls a secure Next.js revalidation endpoint, which refreshes the affected pages.
  - Pages that show CMS content also expire from cache every 5 minutes as a safety net.
- **Publishing** or **saving** in admin refreshes the affected pages immediately.

---

## 8. Admin menu additions

| Menu | Pages |
|---|---|
| **Content** | Homepage (section builder), Hero banners, Announcement bar, Media library |
| **Promotions** | List, create/edit, preview promotion page |
| **Bundles** | List, create/edit, preview bundle page |

Dashboard additions: **live promotions** (with end dates), **scheduled content starting in the next 7 days**, and **bundles out of stock** (because an item ran out).
