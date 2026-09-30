# Customer Profile & Purchase History

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

## Customer profile & purchase history

Every registered customer has a **profile** that keeps their **full purchase history for as long as the account exists**. Checkout requires an account (see `04-checkout-orders-stock.md`), so every order is linked to a profile. Guest carts are merged into the account at login.

### My Account overview (`/account`)
The first page after login:
- Greeting with the customer's name and **"Member since"** date
- **Summary cards:** total orders, total spent (completed orders only), orders in progress, last order date
- **Orders in progress:** up to 3 current orders with status badge and quick actions, e.g. bank details and "I've made the payment" for On-hold orders
- **Recent purchases:** last 4 products bought, each with a **Buy again** button
- Quick links: Profile, Order history, Purchased products, Addresses, Security

### Profile details (`/account/profile`)
| Field | Required | Notes |
|---|---|---|
| Full name | Yes | Pre-fills checkout |
| Mobile number | Yes | Sri Lankan format; used for delivery and COD |
| Email | Yes | Login email; changing it needs email confirmation |
| Date of birth | Optional | For birthday offers later |
| Profile photo | Optional | Stored in a private `avatars` bucket |
| Marketing emails | Optional | Opt-in checkbox (off by default) |

### Order history (`/account/orders`)
A complete list of **every order the customer has ever placed**: in progress, completed, cancelled, and returned. Nothing is hidden or deleted.
- Each row: order number, date, number of items, total, payment method, **status badge**
- **Filter** by status (All / In progress / Completed / Cancelled) and by year
- **Search** by order number or product name
- Newest first, 10 per page

### Order details (`/account/orders/[orderNumber]`)
- Items bought, with the **price paid at the time** (taken from `order_items`, so later price changes don't alter history), size, quantity, and line totals
- Subtotal, delivery fee, COD fee, total
- Delivery address and delivery method used
- **Status timeline** with dates: Placed → On hold / Processing → Shipped (courier + tracking) → Delivered → Completed
- **Payment section:** method, payment status, payments received so far, balance due. For On-hold bank orders: bank details and the "I've made the payment" button.
- **Actions:**
  - **Buy again**: adds all items from this order to the active cart at **today's price**, skipping anything out of stock or discontinued with a notice
  - **Download invoice (PDF)**
  - **Cancel order** (only while On hold, or Processing and not yet shipped)
  - **Contact us about this order** (WhatsApp link with the order number filled in)

### Purchased products (`/account/purchases`)
A list of **every product the customer has bought**, one row per product and size, built from completed and delivered orders:
- Product image, name, brand, size
- Times bought, total quantity, **last bought** date, last price paid
- **Buy again** button (current price shown)
- Handy for customers who re-buy the same fragrance

### Addresses (`/account/addresses`)
Add, edit, delete, and set a default. Saved addresses appear at checkout.

### Security & privacy (`/account/security`)
- Change password; log out of all devices
- **Delete my account:** personal details (name, phone, email, addresses) are erased, but the orders are **kept, anonymised**, for the store's accounting records. The customer is told this before confirming.
