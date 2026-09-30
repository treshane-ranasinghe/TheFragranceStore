# Security (Row Level Security & Access)

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

Every table has Row Level Security switched on. Summary of the rules:

| Table | Public / Guest | Customer | Admin |
|---|---|---|---|
| products, variants, images, categories, brands | Read active items only | Read active items only | Full access |
| cart_items | Own cart only (can't edit `ordered` items) | Own cart only (can't edit `ordered` items) | Read all |
| cart_events | Insert via trigger only | Insert via trigger only | Read all |
| orders, order_items | — | Read own; create only via `place_order` | Full access |
| order_payments | — | Read own (via their orders) | Full access |
| addresses, profiles | Own only | Own only (can't change `role` or `admin_note`) | Read all |
| my_* views | — | Own rows only | — |
| store_settings | Read public fields (payment titles, descriptions, instructions, fees) | Same | Full access |
| bank_accounts | Read active accounts | Same | Full access |
| delivery_methods | Read active methods | Same | Full access |
| hero_banners, announcements, home_sections, promotions, promotion_targets, bundles, bundle_items, bundle_images | Read **live** items only (active and within start/end dates) | Same | Full access |
| media_assets | Read | Read | Full access |

Other rules:
- Customers can never change `role`, prices, totals, or order status directly.
- The Next.js **revalidation endpoint** used by scheduled content requires a secret token (`REVALIDATE_SECRET`) and only refreshes pages; it cannot change data.
- **Preview (Draft Mode)** is only available to logged-in admins.
- The first admin is set manually in the Supabase dashboard; after that, admins promote others from the panel.
- Supabase secret keys are never sent to the browser. Only the public "anon" key uses the `NEXT_PUBLIC_` prefix. The service-role key, if ever needed, stays in server-only code on Vercel.
- **Next.js middleware** checks the login cookie on every `/admin` and `/account` request. `/admin` also requires the `admin` role; everyone else is redirected before the page loads. The database security rules still apply underneath.
