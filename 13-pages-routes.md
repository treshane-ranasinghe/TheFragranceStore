# Pages & Routes

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

### Storefront
| Route | Page |
|---|---|
| `/` | Home |
| `/shop` | All products (filters, sort, search) |
| `/category/:slug` | Category listing |
| `/brand/:slug` | Brand listing |
| `/product/:slug` | Product details |
| `/promotions/:slug` | Promotion page (banner, countdown, products in the sale) |
| `/bundles`, `/bundle/:slug` | All bundles / bundle details |
| `/cart` | Cart |
| `/checkout` | Checkout |
| `/order/:orderNumber` | Order Received page (thank-you page) + bank transfer instructions |
| `/login`, `/register`, `/forgot-password` | Auth |
| `/account` | My Account overview (summary, orders in progress, recent purchases) |
| `/account/profile` | Profile details |
| `/account/orders`, `/account/orders/:orderNumber` | Full order history / order details (timeline, payment, buy again, invoice) |
| `/account/purchases` | Purchased products with Buy again |
| `/account/addresses` | Saved addresses |
| `/account/security` | Password, sessions, delete account |
| `/about`, `/contact`, `/delivery-policy`, `/returns-policy`, `/terms`, `/privacy` | Info pages |

### Admin
| Route | Page |
|---|---|
| `/admin` | Dashboard |
| `/admin/products`, `/admin/products/new`, `/admin/products/:id` | Products |
| `/admin/categories`, `/admin/brands` | Categories & brands |
| `/admin/orders`, `/admin/orders/:id` | Orders |
| `/admin/payments/bank` | Unpaid bank-transfer orders to match against the bank account |
| `/admin/payments/cod` | COD orders to confirm, and COD cash to collect |
| `/admin/cart-activity` | Cart Activity Monitor (feed, active, abandoned, insights) |
| `/admin/customers`, `/admin/customers/:id` | Customers and customer profile (full purchase history, products bought, cart history, notes) |
| `/admin/content/homepage` | Homepage section builder (add, reorder, schedule, preview) |
| `/admin/content/hero` | Hero banners |
| `/admin/content/announcements` | Announcement bar |
| `/admin/content/media` | Media library |
| `/admin/promotions`, `/admin/promotions/:id` | Promotions |
| `/admin/bundles`, `/admin/bundles/:id` | Bundles |
| `/admin/settings` | Store settings |
| `/admin/settings/payments/bank-transfer` | Direct Bank Transfer settings and bank accounts |
| `/admin/settings/payments/cod` | Cash on Delivery settings |
| `/admin/settings/delivery` | Delivery methods |
