# Database Design (Supabase / Postgres)

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

### Tables

**`profiles`** — one per user (linked to Supabase Auth)
| Column | Type | Notes |
|---|---|---|
| id | uuid (PK) | = `auth.users.id` |
| full_name | text | |
| phone | text | |
| role | text | `customer` (default) or `admin` |
| is_anonymous | boolean | guest visitor |
| date_of_birth | date | optional |
| avatar_path | text | optional, private `avatars` bucket |
| marketing_opt_in | boolean | default false |
| admin_note | text | admin only (hidden from the customer by RLS column rules / separate admin view) |
| deleted_at | timestamptz | set when the customer deletes their account (details erased, orders kept) |
| created_at | timestamptz | "Member since" |

**`addresses`**
| id | uuid PK | user_id → profiles | full_name | phone | line1 | line2 | city | district | postal_code | is_default | created_at |

**`categories`**
| id | uuid PK | name | slug (unique) | image_url | sort_order | is_active |

**`brands`**
| id | uuid PK | name | slug (unique) | logo_url | is_active |

**`products`**
| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| name, slug | text | slug unique |
| brand_id | uuid → brands | |
| category_id | uuid → categories | |
| gender | text | men / women / unisex |
| concentration | text | EDP / EDT / Parfum / Body Mist / Other |
| description | text | |
| top_notes, middle_notes, base_notes | text[] | |
| is_featured | boolean | |
| is_active | boolean | hidden from store if false |
| deleted_at | timestamptz | soft delete |
| created_by | uuid → profiles | |
| created_at, updated_at | timestamptz | |

**`product_variants`**
| id | uuid PK | product_id → products | size_ml int | sku (unique) | price numeric(10,2) | compare_at_price numeric(10,2) | stock_qty int (≥ 0) | is_active |

**`product_images`**
| id | uuid PK | product_id → products | storage_path | alt_text | sort_order |

**`cart_items`** — each cart's contents, including ordered items (see `03-cart.md`)
| id | uuid PK | user_id → profiles | variant_id → product_variants (null for a bundle) | bundle_id → bundles (null for a product) | quantity int (> 0) | state (`active` / `ordered`) | order_id → orders (null while active) | added_at | updated_at |
Unique on (user_id, variant_id) **where state = 'active'** (partial unique index).

**`cart_events`** — permanent log of every cart action (powers the Cart Activity Monitor)
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| user_id | uuid → profiles | guest or registered |
| variant_id | uuid → product_variants | |
| action | text | `add`, `remove`, `update_qty`, `checked_out`, `released`, `completed` |
| quantity_before, quantity_after | int | |
| price_at_time | numeric | |
| created_at | timestamptz | |

Written automatically by a database trigger on `cart_items`, so the website can't skip or fake it.

**`orders`**
| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| order_number | text unique | `FS-YYYY-00001` |
| user_id | uuid → profiles | |
| status | text | see `04-checkout-orders-stock.md` |
| payment_method | text | `bank_transfer` or `cod` |
| payment_status | text | see `04-checkout-orders-stock.md` |
| subtotal, delivery_fee, cod_fee, discount, total | numeric(10,2) | |
| shipping_name, shipping_phone, shipping_line1, shipping_line2, shipping_city, shipping_district, shipping_postal_code | text | copied from address at checkout |
| customer_note | text | |
| admin_note | text | admin only |
| courier_name, tracking_number | text | |
| payment_due_at | timestamptz | bank transfer deadline |
| paid_at, shipped_at, delivered_at, cancelled_at | timestamptz | |
| created_at, updated_at | timestamptz | |

**`order_items`** — snapshot of what was bought (so later price changes don't alter old orders)
| id | uuid PK | order_id → orders | variant_id → product_variants | bundle_id → bundles | parent_item_id → order_items (item lines of a bundle) | product_name | brand_name | size_ml | original_unit_price | discount_amount | promotion_id → promotions | unit_price | quantity | line_total |

**`product_slug_redirects`** — keeps old links working after a product is renamed (301 redirect)
| id | old_slug (unique) | product_id → products | created_at |

**SEO fields** added to `products`, `categories`, `brands`: `seo_title`, `seo_description`; plus `intro_text` on `categories` and `brands`.

**`delivery_methods`** — delivery options at checkout; `cod_allowed` is WooCommerce's "Enable for shipping methods"
| id | uuid PK | name | description | fee numeric(10,2) | free_over numeric(10,2) | cod_allowed boolean | sort_order | is_active |

**`bank_accounts`** — store bank accounts shown for Direct Bank Transfer (multiple, reorderable, like WooCommerce)
| id | uuid PK | account_name | account_number | bank_name | branch | branch_code | swift | sort_order | is_active |

**`order_payments`** — every payment the admin records (bank transfer or COD cash)
| id | uuid PK | order_id → orders | amount numeric(10,2) | method | bank_reference | received_on | note | recorded_by | created_at |

**`order_status_history`**
| id | bigint PK | order_id → orders | from_status | to_status | changed_by | note | created_at |

**`store_settings`** — single row
| bacs_enabled | bacs_title | bacs_description | bacs_instructions | cod_title | cod_description | cod_instructions | cod_enabled | cod_fee | cod_max_order_value | bank_payment_hours | abandoned_cart_hours | cart_keep_days_after_delivery | store_phone | store_whatsapp | store_email | default_hero_media_id |

### Database views

**For customers** (each only returns the logged-in customer's own rows):
- `my_order_summary` — total orders, total spent (completed), orders in progress, first and last order date
- `my_order_history` — every order with item count, total, payment method, status (feeds the Order history page)
- `my_purchased_products` — one row per product and size bought: times bought, total quantity, last bought date, last price paid, current price, in stock

**For the admin panel:**
- `admin_customer_stats` — the same summary for every customer (order counts by status, total spent, average order value, first and last order)
- `admin_active_carts` — carts with items, customer, item count, value, last activity
- `admin_abandoned_carts` — active carts idle longer than `abandoned_cart_hours`
- `admin_product_cart_stats` — adds, removes, orders, conversion rate per product
- `admin_sales_summary` — revenue and order counts by day

### Content, promotion & bundle tables (see `17-cms-promotions-bundles.md`)

**`media_assets`** — id, storage_path, file_name, alt_text, width, height, size_bytes, uploaded_by, created_at

**`hero_banners`** — id, internal_name, desktop_media_id, mobile_media_id, alt_text, eyebrow, headline, subheading, button1_text, button1_link_type (`promotion` / `bundle` / `product` / `category` / `brand` / `url`), button1_link_target, button2_text, button2_link_type, button2_link_target, text_position, overlay, text_colour, sort_order, starts_at, ends_at, is_active, updated_at

**`announcements`** — id, text, link_url, background, starts_at, ends_at, is_active

**`home_sections`** — id, type (`hero` / `product_row` / `promo_banner` / `bundle_showcase` / `category_tiles` / `brand_strip` / `image_text` / `text` / `trust_badges`), title, subtitle, config (jsonb, checked by a Zod schema per type), background, spacing, sort_order, starts_at, ends_at, is_active, updated_at

**`promotions`** — id, name, slug (unique), description, desktop_media_id, mobile_media_id, discount_type (`percent` / `amount`), discount_value, badge_text, show_countdown, include_bundles, seo_title, seo_description, starts_at, ends_at (required), is_active, created_at

**`promotion_targets`** — id, promotion_id, target_type (`all` / `category` / `brand` / `product` / `variant`), target_id, is_exclusion

**`bundles`** — id, name, slug (unique), description, category_id, price numeric(10,2), max_per_order, seo_title, seo_description, starts_at, ends_at, is_active, created_at, updated_at

**`bundle_items`** — id, bundle_id, variant_id, quantity (> 0)

**`bundle_images`** — id, bundle_id, storage_path, alt_text, sort_order

**Content functions and views**
- `effective_price(variant_id)` — lowest of: normal price, and each live promotion's price for that variant (no stacking)
- `bundle_availability(bundle_id)` — complete sets that can be made from current stock (0 if any item is out of stock or hidden)
- `live_home_sections`, `live_hero_banners`, `live_announcement`, `live_promotions`, `live_bundles` — views returning only active items within their dates
- `cms_schedule_tick()` — `pg_cron` every 5 minutes; finds content that started or ended since the last run and calls the Next.js revalidation endpoint

Cart rule: each `cart_items` row has **exactly one** of `variant_id` or `bundle_id`. The unique active-row rule applies per variant and per bundle.

### Server-side functions (RPC)
| Function | What it does |
|---|---|
| `is_admin()` | Returns true if the current user is an admin (used by security rules) |
| `place_order(address, payment_method, note)` | Reads the user's cart, **re-checks prices and stock from the database** (so nobody can change prices in the browser), calculates fees, creates the order and items (stock is **not** reduced), locks the cart items to the order (they stay visible in the cart), and returns the order number — all in one step |
| `cancel_order(order_id, reason)` | Cancels, returns stock **only if it was reduced** (`stock_reduced_at` set), and releases the items back to the customer's active cart (customer: only while `pending`; admin: any time before delivery) |
| `update_order_status(order_id, new_status, note)` | Admin only; logs history |
| `record_bank_payment(order_id, amount, received_on, bank_reference, note)` | Admin only; adds a payment. Full amount → `paid`, then calls `confirm_order` (stock reduced, `on_hold` → `processing`); if stock is short the order stays On hold, flagged. Part amount → `partially_paid`, stays `on_hold` |
| `mark_payment_not_found(order_id, note)` | Admin only; `payment_notified` → `unpaid`, extends deadline |
| `merge_guest_cart(guest_user_id)` | Moves a guest cart into the logged-in account |
| `buy_again(order_id)` | Adds the order's items to the customer's active cart at current prices; returns which items were skipped (out of stock / discontinued) |
| `delete_my_account()` | Erases personal details, anonymises the profile, keeps orders for records, signs the user out |
| `notify_payment(order_id, note)` | Customer's "I've made the payment" button; sets `payment_notified` |
| `confirm_order(order_id)` | Admin only (also called by `record_bank_payment` on full payment). Locks the variant rows, checks stock, **reduces stock**, sets `stock_reduced_at`, moves `pending`/`on_hold` → `processing`, logs history, sends the confirmed/processing email. Fails with a clear message if stock is short |
| `log_cod_call(order_id, result, note)` | Admin only; records a confirmation call attempt |
| `mark_shipped(order_id, courier, tracking)` | Admin only; sets COD amount to collect |
| `mark_delivery_failed(order_id, reason)` | Admin only; order → `returned`, customer failed-COD count +1 |
| `restock_order(order_id, damaged)` | Admin only; returns stock unless damaged, order → `cancelled` |
| `mark_delivered(order_id)` | Admin only; `shipped` → `delivered`. Bank orders (already paid) move on to `completed` automatically |
| `record_cod_remittance(order_ids[], courier, reference, amount, date)` | Admin only; marks delivered COD orders `paid` → `completed`, in one batch |
| `record_refund(order_id, reference, date)` | Admin only; `refund_pending` → `refunded` |
| `process_overdue_orders()` | Runs hourly via `pg_cron`: reminders, overdue flags, auto-cancel (`05-bank-transfer.md` step 4) |

### Storage buckets
| Bucket | Access |
|---|---|
| `product-images` | Public read; only admins can upload / delete |
| `cms-media` | Public read; only admins can upload / delete (hero banners, section images, promotion and bundle images) |
| `avatars` | Private. Each customer can upload / view / delete only their own photo; admins can view |

## Extra database fields this logic needs
These add to the tables in below.

| Table | New fields |
|---|---|
| `orders` | `checkout_token` (unique), `amount_received` (running total), `payment_notified_at`, `payment_customer_note`, `is_overdue`, `reminder_sent_at`, `delivery_method_id`, `cod_amount_to_collect`, `completed_at`, `stock_reduced_at`, `cod_confirmed_at`, `cod_call_attempts`, `failure_reason`, `cod_remittance_id`, `refund_reference`, `refunded_at`, `restocked_at` |
| `cart_items` | `state` (`active` / `ordered`), `order_id` (see `03-cart.md`) |
| `profiles` | `failed_cod_count` |
| `store_settings` | `bank_recheck_hours`, `bank_grace_hours`, `auto_cancel_unpaid`, `cod_max_failed` (default off), `cod_max_call_attempts`, `cod_confirm_hours`, `cod_remit_alert_days` |
| **New:** `delivery_methods` | `id`, `name`, `description`, `fee`, `free_over`, `cod_allowed` (default true), `sort_order`, `is_active` |
| **New:** `bank_accounts` | `id`, `account_name`, `account_number`, `bank_name`, `branch`, `branch_code`, `swift` (optional), `sort_order`, `is_active` |
| **New:** `order_payments` | `id`, `order_id`, `amount`, `method` (`bank_transfer` / `cod`), `bank_reference`, `received_on`, `note`, `recorded_by`, `created_at` |
| **New:** `cod_call_logs` | `id`, `order_id`, `called_by`, `result` (`confirmed` / `no_answer` / `wrong_number` / `cancel_requested`), `note`, `created_at` |
| **New:** `cod_remittances` | `id`, `courier_name`, `reference`, `amount_received`, `received_on`, `note`, `recorded_by`, `created_at` |
