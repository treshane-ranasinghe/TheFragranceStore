# Design & Theme

*The Fragrance Store — spec v2.1 · part of the [spec index](README.md)*

The store must look **exactly like the existing The Fragrance Store website** (`index.html`, `checkout.html`, `css/styles.css`, `css/shop.css`): a dark **"Onyx & Champagne Gold"** luxury theme with serif headlines, gold gradients, glass surfaces, and rich motion.

> **This replaces the earlier green / cream palette.** Nothing from the old palette is used.

## How to reproduce the design exactly

1. **Reuse the existing CSS files as the design system.** Copy `css/styles.css` and `css/shop.css` into the Next.js project unchanged, as global stylesheets (`app/styles.css`, `app/shop.css`, imported in `app/layout.tsx`). Keep every class name (`.hero__slide`, `.product__media`, `.bag__panel`, `.co__step`, …).
2. **Turn each HTML block into a React component** with the same markup and classes, e.g. `<Hero>`, `<Collections>`, `<ProductCard>`, `<BagDrawer>`, `<CheckoutForm>`. Replace hard-coded content with data from Supabase or the CMS.
3. **Port the JavaScript behaviour** (preloader, hero slider, cursor, tilt, reveal on scroll, bag, testimonials) into React components with `useEffect`. Keep the same timings and CSS classes (`is-active`, `is-entering`, `is-visible`, `is-open`, …).
4. **Tailwind is not used.** It would duplicate the existing CSS and risk visual differences. New screens, such as the admin panel, account pages and bundle pages, are built with the same variables and class patterns.
5. **Pixel check before launch:** compare every page side by side with the original site at 1440 px, 1024 px, 768 px, and 390 px widths.

## Colours (from `:root` in `styles.css`)

| Token | HEX | Use |
|---|---|---|
| `--onyx` | `#0A0908` | Main page background (dark) |
| `--ink` | `#12100E` | Raised dark surfaces: bag panel, checkout summary, footer, marquee |
| `--char` | `#1C1915` | Product cards on dark |
| `--ivory` | `#F5EFE6` | Light sections (Collections, Our Story) |
| `--parchment` | `#EBE2D3` | Light surface (testimonials), toast |
| `--gold` | `#C9A45C` | **Primary accent**: eyebrows, links, active filters, borders, icons |
| `--gold-light` | `#E9D3A0` | Highlights, hover states, prices in accents |
| `--gold-deep` | `#8F6B2E` | Antique gold: accents on light sections |
| `--amber` | `#B8742A` | Warm glow behind the notes section |
| `--oxblood` | `#4A1A1C` | Rare accent (reserved) |
| `--text-light` | `#EFE8DC` | Body text on dark |
| `--text-dim` | `#A39A8C` | Secondary text on dark |
| `--text-dark` | `#1A1714` | Body text on light |
| `--text-mid` | `#5E564B` | Secondary text on light |
| `--line-dark` | `rgba(201,164,92,.22)` | Gold hairlines on dark |
| `--line-light` | `rgba(26,23,20,.14)` | Hairlines on light |
| `--gold-grad` | `linear-gradient(135deg, #8F6B2E 0%, #E9D3A0 38%, #C9A45C 60%, #F4E4BC 78%, #8F6B2E 100%)` | Gold buttons, announcement bar, WhatsApp button, progress bar, gold text |

**Hero tints:** each hero slide has its own glow colour (`--tint`), e.g. `#D9A444`, `#D07A34`, `#B0683A`, `#D69A48`, `#8FA6B6`. It is set per slide in the CMS.

**Status colours (from account page):** good `#7DBE8F`, info `#8FA9BD`, problem `#D9826A`; error text `#E59A7E` / border `#C7684D`.

### Readability rules (contrast checked)

| Combination | Ratio | OK for |
|---|---|---|
| Text-light on Onyx | 16.3 : 1 | All text |
| Text-dim on Onyx / Ink / Char | 7.2 / 6.8 / 6.3 : 1 | All text |
| Gold on Onyx | 8.5 : 1 | All text |
| Onyx on Gold (gold buttons) | 8.5 : 1 | All text |
| Text-dark on Ivory | 15.6 : 1 | All text |
| Text-mid on Ivory | 6.3 : 1 | All text |
| Gold-deep on Ivory | 4.3 : 1 | Eyebrows and large or uppercase labels only (as the original uses it) |
| **Gold `#C9A45C` on Ivory** | **2.1 : 1** | **Never for text.** On light sections, use Gold-deep (already the rule in `styles.css`) |

## Typography

| Token | Font | Use |
|---|---|---|
| `--serif` | **Cormorant Garamond** (300, 400, 500, 600, italic 300/400) | Headlines, product names, prices, quotes, italic gold words |
| `--sans` | **Jost** (300, 400, 500) | Body (300 weight), buttons, labels, forms |
| `--logo` | **Nunito** (700, 800, 900) | "THE FRAGRANCE STORE" wordmark |
| `--poster` | **Antonio** (300, 500, 700) | Giant outlined hero word, step numbers, roman numerals, counters |

- Loaded with `next/font/google` (same families and weights) and exposed as the same CSS variables, so `styles.css` works unchanged.
- Signature styles: section titles `clamp(38px, 5vw, 68px)` Cormorant 300 with one **italic gold-gradient word** (`<em>`); eyebrows 11 px Jost 500, letter-spacing .34em, uppercase, gold, with a 28 px line before; buttons 11 px, letter-spacing .26em, uppercase.

## Signature elements (must be reproduced)

| Element | Behaviour |
|---|---|
| **Logo** | Wordmark where a **perfume-bottle SVG replaces the "T"** in STORE; on hover the bottle turns gold and sprays four mist dots |
| **Preloader** | Bottle outline draws in gold, fills with gold liquid, letters rise, then the screen slides up. Skipped on repeat visits in the same session and when reduced motion is on |
| **Announcement bar** | Gold-gradient strip, uppercase; long text on desktop, short text on phones. Text comes from the CMS |
| **Navigation** | Sticky floating **glass pill** (blur + gold glint sliding along the top edge); left and right links, centred logo; darkens on scroll. Phones: hamburger + WhatsApp + bag, with a full-screen glass menu of numbered serif links |
| **Hero** | Full-height dark hero. Headline "The art of *leaving an impression*" (lines rise in). Rotating product slides revealed by a **wipe with a gold light line**, a **giant outlined "ghost word"** per slide (e.g. ELIXIR) with letters sliding, **ambient tint glow** in the slide's colour, cursor spotlight, gold particles, mouse-parallax depth layers, a **"Now Featuring" glass card** (brand, name, notes chips, "View fragrance"), a **thumbnail rail** with progress bars, and a scroll cue |
| **Marquee** | Endless italic serif scent notes (Oud ✦ Amber ✦ Rose de Mai …) |
| **Collections** | Three tall image cards (middle one taller) with number, serif title, "Explore →". Phones: swipeable carousel |
| **Product cards** | Dark card, 5:6 image, gold tag, house, serif name, notes, price + quick-add. **3D tilt**, cursor shine and light sweep on hover; lift on hover (not on touch screens). Two-up grid on phones |
| **Shop bar** | Pill search, result count, pill sort select; filter tabs (All / For Him / For Her / Arabian) that become swipeable pills on phones |
| **Product modal** | Two-column dialog (bottom sheet on phones) with image, notes list, size choices, quantity stepper, "Add to Bag", WhatsApp enquiry |
| **Bag drawer** | Slides in from the right; free-delivery progress meter, items with steppers, subtotal, checkout button; empty state |
| **Our Story** | Image with offset gold frame and parallax, "100% Authentic Guaranteed" counter badge, three pillars with round gold icons |
| **Art of Scent** | Notes pyramid: three columns with outlined roman numerals I / II / III |
| **Testimonials** | Parchment section, large italic serif quotes that crossfade, line dots |
| **Gallery** | Six-tile strip ending with a gold Instagram tile |
| **Visit** | Address, hours, contact, WhatsApp and directions buttons, parallax image |
| **Footer** | Large logo, italic tagline, three link columns, bottom bar |
| **Floating WhatsApp** | Gold circle bottom-right (hidden on phones, where WhatsApp is in the nav) |
| **Global touches** | Film-grain overlay (desktop), gold scroll-progress bar, custom gold cursor (mouse only), reveal-on-scroll, gold text selection |

## Checkout, account and other store pages

- **Checkout** uses the existing `checkout.html` design: minimal header (back link, logo, "Secure checkout"), numbered steps (01 Contact, 02 Delivery, 03 Payment) with gold Antonio numbers, radio "choice" cards, sticky order summary, gold "Place order", assurance list, and the confirmation screen with the animated gold tick and bank-details panel.
- **Payment choices:** only **Cash on delivery** and **Bank transfer** are shown, per the spec. The existing "Pay online (PayHere)" card option is **removed**.
- **Delivery choices:** "Island-wide delivery" and "Collect from the boutique". These map to the delivery methods in `04-checkout-orders-stock.md`.
- **Account pages** use the existing `.ac__*` styles: sign-in/sign-up tabs, order cards with the 4-step progress line, status dots, toast messages.
- **New pages** (order history, purchased products, bundles, promotions, ordered items in the bag) reuse the same card, eyebrow, serif-title and gold-line patterns. No new colours or fonts are introduced.

## Admin panel

Same palette and fonts, tuned for daily work: Onyx sidebar with gold active item, Ink content cards with gold hairlines, Jost for tables and forms, and status dots in the good / info / problem colours. Heavy motion (preloader, cursor, tilt, grain) is **off** in the admin panel.

## Motion and accessibility

- Easing `cubic-bezier(.22, 1, .36, 1)` (`--ease`) for most transitions; `cubic-bezier(.77, 0, .18, 1)` for wipes and drawers.
- Everything respects **`prefers-reduced-motion`**: animations off, preloader hidden, cursor hidden (already in `styles.css`).
- Custom cursor only on mouse devices; grain overlay off on phones.
- Keep all existing `aria-*` attributes, focus outlines, and screen-reader text from the original HTML.
- Hero images: first slide loads with high priority; others lazy-load (keeps Google's page-speed score healthy).
