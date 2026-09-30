# Admin Panel

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

Accessible at `/admin`. Only users with the `admin` role can open it.

Cart Activity Monitor is in `09-cart-activity-monitor.md`. Content (hero banners, homepage sections, announcement bar, media), Promotions, and Bundles are in `17-cms-promotions-bundles.md`.

## Dashboard
- Today's / this month's orders and revenue
- Orders waiting for action (bank payments to verify, new COD orders to pack, COD cash still with couriers, overdue bank transfers)
- Low-stock alerts
- **Stock warnings:** unconfirmed orders whose items no longer have enough stock to be confirmed
- Top products added to cart vs. top products actually ordered
- Live feed of latest cart activity

## Products (full CRUD)
- **List** with search, filters, stock level, **awaiting confirmation** quantity (sum of unconfirmed orders), active / hidden status
- **Create / Edit:** name, slug, brand, category, gender, concentration, description, scent notes, images (upload, reorder, delete), featured flag, active flag
- **Variants:** size (ml), SKU, price, compare-at price, stock quantity
- **Delete:** soft delete (hidden from store) if the product has past orders; hard delete otherwise
- Bulk actions: hide / show, mark featured

## Categories & Brands (CRUD)
- Name, slug, image, display order

## Orders
- List with filters by status, payment method, payment status, date
- Order details: customer, address, items, totals, payments received so far, balance due
- **Actions:**
  - **Record bank payment** (amount, date, bank reference); order becomes *Paid* when the full amount is received
  - Record part payment (shows the balance due to the customer)
  - Update order status (Processing → Shipped → Delivered → Completed)
  - Add courier name and tracking number when shipped
  - Mark COD cash received (order → Completed)
  - Cancel order (stock is returned automatically)
  - Internal notes (only admins can see these)
- Printable packing slip / invoice
- Real-time alert when a new order arrives

## Customers
- **List** of registered customers: name, phone, email, member since, total orders, total spent, last order date, refused-COD count. Search, sort, and export to CSV.
- **Customer profile (admin view):**
  - Profile details and saved addresses
  - **Summary:** total orders, total spent, average order value, first and last order date, completed / cancelled / returned counts
  - **Full purchase history:** every order with status, with links to the order page
  - **Products bought:** each product with quantity and last bought date
  - **Cart history:** current cart and past cart activity (from `cart_events`)
  - **Internal notes** about the customer (admin-only), e.g. "Prefers evening delivery"

## Store Settings
- **Payments → Direct Bank Transfer** (same fields as WooCommerce):
  - Enable / disable
  - **Title** shown at checkout (default "Direct bank transfer")
  - **Description** shown at checkout under the title
  - **Instructions** added to the Order Received page and emails
  - **Account details**: add, edit, delete, and reorder one or more bank accounts (account name, account number, bank name, branch, branch code, SWIFT optional)
  - Payment reminder on/off, auto-cancel unpaid on/off (off by default, like WooCommerce)
- **Payments → Cash on Delivery** (same fields as WooCommerce):
  - Enable / disable
  - **Title** shown at checkout (default "Cash on delivery")
  - **Description** shown at checkout under the title
  - **Instructions** added to the Order Received page and emails (default "Pay with cash upon delivery.")
  - **Enable for delivery methods**: choose which delivery methods offer COD (leave empty = all)
  - *Accept for virtual orders* is not needed; every product here is physical
  - ➕ COD fee, maximum COD order value, block after N refused parcels (all off by default)
  - Confirmation reminders: flag unconfirmed COD orders after `cod_confirm_hours` (default 48)
- **Delivery methods**: add, edit, reorder (name, description, fee, free-delivery threshold, active)
- Bank transfer payment deadline (hours)
- Abandoned cart threshold (hours)
- Store contact details (phone, WhatsApp, email)

## Admin users
- Promote an existing user to admin (only an existing admin can do this)
