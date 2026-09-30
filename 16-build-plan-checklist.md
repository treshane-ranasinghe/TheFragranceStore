# Build Plan & Acceptance Checklist

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

## Build Plan

| Phase | Deliverables |
|---|---|
| **1. Foundation** | Supabase project, database tables, security rules, storage buckets, Next.js project setup with brand theme (see `15-design-theme.md`), Supabase SSR auth + middleware (incl. anonymous guests on first add-to-cart) |
| **2. Admin – catalogue** | Admin login, product / variant / image CRUD, categories, brands |
| **3. Storefront** | Server-rendered home, shop, filters, search, product page, with cache refresh on admin edits |
| **4. Cart & tracking** | Database cart, guest-to-account merge, `cart_events` trigger |
| **5. Checkout & orders** | Checkout, `place_order`, Direct Bank Transfer (On hold → Processing), Order Received page and emails, Cash on Delivery (Processing → Completed), delivery methods, ordered items in cart, order pages |
| **5b. Customer profiles** | Account overview, profile, order history + details, purchased products, buy again, invoice PDF, addresses, security / delete account |
| **6. Admin – operations** | Order management, bank payment recording, status updates, settings |
| **6b. Content, promotions & bundles** | Media library, hero banners, announcement bar, homepage section builder with preview, promotions with `effective_price`, bundles (pages, cart, orders, stock), scheduled refresh |
| **7. Cart Activity Monitor** | Live feed, active carts, abandoned carts, product insights, CSV export |
| **8. SEO, polish & launch** | Metadata, JSON-LD, sitemap, robots, redirects (`14-seo.md`); emails; policy pages; speed check (Core Web Vitals); deploy to Vercel; custom domain; Google Search Console + sitemap submission |

---

## Acceptance Checklist

- [ ] Visitor can browse, search, filter, and add to cart without an account
- [ ] Guest cart carries over after login / sign up
- [ ] Checkout offers only Bank Transfer and Cash on Delivery
- [ ] Order total is calculated on the server and can't be altered from the browser
- [ ] Bank transfer orders are created **On hold**, with stock **not** reduced
- [ ] Order Received page and "on hold" email show the instructions, all active bank accounts, amount, and order-number reference; no slip upload anywhere
- [ ] Admin can manage bank transfer title, description, instructions, and multiple bank accounts
- [ ] Marking a bank order as paid moves it On hold → Processing and sends the processing email
- [ ] Admin can record full or part bank payments; customer sees the result
- [ ] Ordered items stay in the cart, locked, with a live status badge
- [ ] Cancelled orders put their items back into the customer's cart
- [ ] Stock drops **only when an order is confirmed** (bank: payment recorded; COD: admin confirms), and returns on cancellation only if it had dropped
- [ ] Confirming is blocked when stock is short; paid-but-out-of-stock bank orders are flagged
- [ ] Admin sees "awaiting confirmation" quantities per product
- [ ] Unpaid bank orders get a reminder, are flagged overdue, and auto-cancel after the grace period (when enabled)
- [ ] COD orders are created **Pending**; admin Confirm moves them to Processing and reduces stock
- [ ] COD only shows for delivery methods that allow it (and within ➕ limits if switched on)
- [ ] Order Received page and "processing" email show the COD instructions and amount to keep ready
- [ ] Delivered COD orders stay "unpaid" until courier cash is recorded, then move to **Completed**
- [ ] Delivered bank-transfer orders move to **Completed** automatically
- [ ] Invalid status jumps (e.g. pending → delivered) are blocked by the database
- [ ] Admin can create, view, edit, and delete products, variants, images, categories, and brands
- [ ] Admin sees cart adds / removes live, with customer identity
- [ ] Abandoned carts and product cart-to-order stats are visible and exportable
- [ ] Customers cannot see other customers' carts, orders, or payments
- [ ] Non-admins cannot open any `/admin` page or call admin functions
- [ ] Site works well on a phone
- [ ] Every order is linked to a customer profile; order history shows all past orders (including cancelled/returned) with prices as paid at the time
- [ ] Account overview shows total orders, total spent, and orders in progress
- [ ] Purchased products list works, and Buy again adds items at today's price, skipping unavailable ones
- [ ] Customers can download an invoice PDF for any order
- [ ] Deleting an account erases personal details but keeps anonymised orders
- [ ] Admin can view any customer's full purchase history
- [ ] Product pages show full content with JavaScript turned off (server-rendered)
- [ ] Each product page has a unique title, description, Open Graph image, and valid Product structured data (passes Google's Rich Results Test)
- [ ] `sitemap.xml` lists all active products, categories, and brands; `robots.txt` blocks private pages
- [ ] Editing a product in admin updates the live page within seconds
- [ ] Bots browsing the site do not create guest users
- [ ] Mobile Core Web Vitals pass (LCP < 2.5 s, CLS < 0.1, INP < 200 ms)
- [ ] Colours, fonts, and buttons follow `15-design-theme.md`; no gold text on cream backgrounds
- [ ] Admin can upload, edit, reorder, schedule, and preview hero banners (desktop + mobile images); changes appear on the live homepage within seconds
- [ ] Admin can add, reorder, hide, schedule, and preview homepage sections of every type
- [ ] A scheduled banner or promotion appears and disappears within 5 minutes of its start/end time without anyone publishing
- [ ] Promotions lower prices on the chosen products/categories/brands, show badge and "was" price, never stack, and the checkout total matches the displayed price
- [ ] Order lines record original price, discount, and promotion used
- [ ] Bundles show "You save", correct availability from item stock, add to cart as one line, and reduce each item's stock on confirmation
- [ ] Promotion and bundle pages are in the sitemap while live
