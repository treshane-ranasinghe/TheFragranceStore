# The Fragrance Store — E-commerce Website Specification

**Version:** 2.0 (Draft — content management, promotions & bundles added)
**Date:** 29 September 2026
**Stack:** Next.js (React) + Supabase
**Market:** Sri Lanka (prices in LKR)

---

## Overview

The Fragrance Store is an online shop for perfumes and fragrances. Customers browse products, add them to a cart, and place orders. **No online card payments are taken.** Customers pay in one of two ways:

1. **Bank Transfer** — the customer transfers money to the store's bank account using the order number as the reference. The admin checks the bank account and marks the order as paid. **Customers do not upload payment slips.**
2. **Cash on Delivery (COD)** — the customer pays the courier in cash when the order arrives.

A private **Admin Panel** lets the store owner manage products, process orders, confirm payments, and **see who is adding what to their cart, in real time**.

### In simple terms

- **Next.js** builds the screens people see and click (the website). It is built on React, but it also prepares each page on a server before sending it. That means Google receives complete product pages it can read and rank, which helps the store get search traffic.
- **Supabase** is the back office: it stores the data (products, orders), handles logins, stores images, and enforces who is allowed to see or change what.

---

## Scope

### In scope
- Public storefront (catalogue, search, filters, product pages)
- Shopping cart
- Checkout with Bank Transfer or Cash on Delivery
- Customer accounts and order history
- Admin panel: product CRUD, category CRUD, order management, payment confirmation, cart activity monitoring, customer list, store settings
- Mobile-friendly design
- Content management: editable hero banners, homepage sections, promotions, and bundles

### Out of scope (this version)
- Online card / wallet payments (PayHere, Stripe, etc.)
- Coupon / discount codes (promotions are automatic in this version)
- Multi-vendor marketplace
- Courier API integration (tracking numbers are entered manually)
- Mobile apps (the website works on phones instead)
- Multiple languages (English only; can be added later)

---

## User Roles

| Role | Who | What they can do |
|---|---|---|
| **Visitor** | Anyone browsing, not logged in | Browse products, add to cart (tracked anonymously) |
| **Customer** | Registered user | Everything a visitor can + place orders, view own orders and ordered items in their cart, manage addresses |
| **Admin** | Store owner / staff | Everything in the Admin Panel. Cannot be created from the public sign-up page. |

**How visitors are tracked:** Supabase "anonymous sign-in" gives every visitor a temporary ID the moment they first add something to their cart. Their cart is saved against that ID. When they sign up or log in at checkout, the cart moves to their real account. This means the admin can see cart activity from **both** guests and registered customers.

---

## Spec files

| # | File | What it covers |
|---|---|---|
| 01 | [Tech stack](01-tech-stack.md) | Next.js + Supabase choices, how pages render, folder structure, speed and quality targets |
| 02 | [Storefront](02-storefront.md) | Home, product listing, filters, search, product details |
| 03 | [Cart](03-cart.md) | Cart behaviour, guest carts, ordered items staying in the cart |
| 04 | [Checkout, orders & stock](04-checkout-orders-stock.md) | Checkout steps, order and payment statuses, placing an order, cancellations, stock rules |
| 05 | [Direct Bank Transfer](05-bank-transfer.md) | WooCommerce-style bank transfer: On hold → paid → Processing |
| 06 | [Cash on Delivery](06-cash-on-delivery.md) | COD: Pending → confirmed → delivered → cash received → Completed |
| 07 | [Customer profile](07-customer-profile.md) | My Account, order history, purchased products, buy again, addresses, security |
| 08 | [Admin panel](08-admin-panel.md) | Dashboard, products, categories, brands, orders, customers, settings, admin users |
| 09 | [Cart activity monitor](09-cart-activity-monitor.md) | Who adds what to cart: live feed, active and abandoned carts, product insights |
| 10 | [Notifications](10-notifications.md) | Customer emails and admin alerts |
| 11 | [Database](11-database.md) | Tables, views, server functions, storage buckets |
| 12 | [Security](12-security.md) | Row Level Security, admin protection, keys |
| 13 | [Pages & routes](13-pages-routes.md) | Every URL in the store and admin |
| 14 | [SEO](14-seo.md) | Google traffic plan: metadata, structured data, sitemap |
| 15 | [Design & theme](15-design-theme.md) | Colours, fonts, components, Tailwind setup |
| 16 | [Build plan & checklist](16-build-plan-checklist.md) | Build phases and acceptance checklist |
| 17 | [Content, promotions & bundles](17-cms-promotions-bundles.md) | CMS: editable hero banners, announcement bar, homepage section builder, timed promotions, product bundles, media library |

---

## Open Decisions

These need confirming before build. Defaults used in this spec are shown in brackets.

1. **Database setup:** use an existing Supabase project, create a new one, or provide SQL only? *[not decided]*
2. **Guest shopping:** can guests add to cart before signing up? *[Yes — tracked anonymously; login required only at checkout]*
3. **Delivery methods and fees:** which methods (Standard, Express, Pickup) and prices? *[Admin-managed list]*
4. **COD extras:** COD fee, maximum order value, refused-parcel block? *[All off by default; switch on if needed]*
5. **Bank transfer deadline:** *[48 hours, then flagged for follow-up]*
6. **Returns policy:** fragrances are often non-returnable once opened — confirm policy wording.
7. **Emails in version 1:** *[Optional; in-panel alerts are included regardless]*
8. **Domain name and brand assets:** logo and final fonts. *[Colours confirmed — see `15-design-theme.md`; fonts are a recommendation until confirmed]*
