# Cart (incl. Ordered Items & Guest Carts)

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

## Cart
- List of items, quantity change, remove item
- Subtotal, delivery fee, COD fee (if applicable), total
- Cart is saved in the database, so it survives page refresh and device changes (after login)
- **Bundles** are one cart line (bundle name and price) with their items listed underneath
- Promoted items show the sale price and promotion badge
- Warns if an item's price changed (e.g. a promotion ended) or went out of stock since it was added
- **Ordered items stay in the cart.** After checkout, the items don't disappear. They move to an **"Ordered items"** section of the cart, grouped by order number, with a status badge (e.g. *Awaiting payment*, *Payment received*, *Shipped*). These items are locked: they can't be edited, removed, or checked out again. Full logic in below.
- The cart icon count only includes items the customer can still check out, not ordered items.

## Guest tracking with server rendering
The anonymous guest session (see User Roles in `README.md`) is created **only when a visitor first adds something to the cart**, not on page load. This stops Google's crawler and other bots from creating thousands of fake guest users. It also keeps public pages cacheable.

## Ordered items stay in the cart

After checkout, items are **not deleted** from the cart. They stay visible so the customer can always see what they ordered and what's happening to it.

**How it works**
- Every cart item has a `state`:
  - `active` — normal item. It can be edited or removed, and it is included in the next checkout.
  - `ordered` — part of a placed order (`order_id` set). It is locked: no edits, no removal, never included in a new checkout.
- `place_order` switches all `active` items to `ordered` and links them to the new order (`04-checkout-orders-stock.md` step 9).
- The same product can be in both states at once, e.g. one bottle already ordered and another added afterwards. The rule "one row per product" applies to `active` items only.

**What the customer sees in the cart**

| Section | Contents |
|---|---|
| **Your cart** | `active` items, totals, and **Checkout** button |
| **Ordered items** | One card per order: order number, items, total, status badge, and a **View order** link. For On-hold bank orders the card also shows the instructions, bank accounts, amount due, deadline, and the "I've made the payment" button. |

**Status badge shown on the ordered items**

| Order / payment state | Badge |
|---|---|
| Bank, `on_hold` + `unpaid` | On hold: awaiting payment |
| Bank, `on_hold` + `payment_notified` | Checking your payment |
| Bank, `on_hold` + `partially_paid` | Balance due Rs. X |
| COD, `pending` | Awaiting confirmation |
| `processing` | Preparing your order (COD: "Pay Rs. X on delivery") |
| `shipped` | On the way (with tracking number) |

**When ordered items leave the cart**

| Event | What happens to the ordered items |
|---|---|
| Order `delivered` | Removed from the cart after `cart_keep_days_after_delivery` (default 3 days), so the customer sees "Delivered" briefly. The order stays in "My Orders" permanently. |
| Order `cancelled` (by customer, admin, or auto-cancel) | Switched back to `active`, so the customer can simply check out again. If an item is now out of stock or hidden, it is removed with a notice. If the same product is already in the active cart, the quantities are merged. |
| Order `returned` (COD refused) | Removed from the cart when the order is restocked and cancelled (not put back, since the customer refused it). |

**Cart Activity Monitor**
- Checking out and releasing items are logged in `cart_events` as `checked_out`, `released` (back to cart after cancel), and `completed` (removed after delivery).
- Active carts and abandoned carts count only `active` items, so placed orders aren't mistaken for abandoned carts.
