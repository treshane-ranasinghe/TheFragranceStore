# Checkout, Order Statuses & Stock Rules

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

Applies to both payment methods. Method-specific steps are in `05-bank-transfer.md` and `06-cash-on-delivery.md`.

This section explains exactly how Bank Transfer and Cash on Delivery (COD) work, step by step, including what happens when things go wrong. The important rules run **inside the database** (Supabase functions and scheduled jobs), not in the browser, so customers can't skip steps or change amounts.

## Checkout
1. **Login / sign up** (required to place an order; guest cart carries over)
2. **Delivery details:** full name, phone number (required, Sri Lankan format), address line 1 & 2, city, district, postal code, delivery notes
3. **Delivery method:** e.g. Standard delivery, Express delivery, Store pickup (set by admin, each with its own fee)
4. **Payment method:** Direct Bank Transfer or Cash on Delivery. COD only appears if the chosen delivery method allows it.
5. **Review & place order**

On success, the customer sees the **Order Received page** with the order number (e.g. `FS-2026-00042`).

## Order status and payment status

Every order has **two separate statuses**: one for the delivery journey and one for the money.

**Order status**

| Status | Meaning |
|---|---|
| `pending` | COD order placed; **waiting for the admin to confirm it**. Stock not reduced yet |
| `on_hold` | Bank transfer order placed; **waiting for the money to reach the store's account** (WooCommerce's "On hold"). Stock not reduced yet |
| `processing` | **Confirmed purchase**: paid (bank) or confirmed by admin (COD). Stock has been reduced; being packed |
| `shipped` | Handed to courier |
| `delivered` | Customer received it |
| `completed` | Delivered **and** paid. For COD this is when the cash reaches the store. Final state, as in WooCommerce |
| `returned` | Courier could not deliver or customer refused (COD); parcel coming back |
| `cancelled` | Cancelled by customer, admin, or automatically |

**Payment status**

| Status | Meaning |
|---|---|
| `unpaid` | No money received yet |
| `payment_notified` | Customer tapped "I've made the payment"; admin should check the bank account |
| `partially_paid` | Some money received; balance still due |
| `paid` | Full amount confirmed (bank transfer found in account, or COD cash received from courier) |
| `refund_pending` | Paid order was cancelled; money must be returned |
| `refunded` | Money returned to the customer |

## Allowed status changes

A database trigger blocks any change not in this table, so an order can never jump from `pending` straight to `delivered`, for example.

| From | Allowed to |
|---|---|
| `pending` | `processing` (only if stock is available), `cancelled` |
| `on_hold` | `processing` (only when payment status is `paid` and stock is available), `cancelled` |
| `processing` | `shipped`, `cancelled` |
| `shipped` | `delivered`, `returned` |
| `returned` | `cancelled` (after the parcel is back in stock) |
| `delivered` | `completed` (only when payment status is `paid`) |
| `completed` | — (final) |
| `cancelled` | — (final) |

Extra rules, matching WooCommerce:
- **Bank Transfer** orders start at `on_hold` and move to `processing` once paid. When delivered, they move to `completed` automatically, because they are already paid.
- **COD** orders start at `pending` and move to `processing` when the admin confirms them. When delivered, they stay `delivered` until the admin records the cash, then move to `completed`.
- **Stock is reduced at the moment an order enters `processing`**, for both payment methods (see **Stock rules summary** below).

## Placing an order (both payment methods)

When the customer clicks **Place order**, the website calls one database function, `place_order`. Everything below happens in a single step. If any check fails, nothing is saved and the customer sees a clear message.

1. **Stop double orders.** The checkout page creates a one-time `checkout_token`. If the same token arrives twice (double-click, slow network), the second request returns the first order instead of creating a new one.
2. **Check the cart.** It must not be empty. Every product must still be active.
3. **Check stock (no reduction).** Each item must have enough stock right now. If not, the order stops and the message says which item is short. Stock is **not** reduced here; it is reduced only when the order is confirmed (see below).
4. **Recalculate prices from the database.** Prices shown in the browser are ignored.
   - Subtotal = sum of (**effective price** × quantity). The effective price includes any live promotion; bundles use the bundle price (see `17-cms-promotions-bundles.md`)
   - Delivery fee = fee of the chosen delivery method, or **0** if the subtotal is at or above that method's free-delivery threshold
   - COD fee = COD fee from settings (COD orders only; 0 unless switched on ➕)
   - Total = subtotal + delivery fee + COD fee − discount
5. **Check the payment method is allowed** (COD rules in `06-cash-on-delivery.md`).
6. **Create the order** with status `on_hold` (Bank Transfer) or `pending` (COD), payment `unpaid`, a new order number (`FS-2026-00042`), and a copy of the delivery address.
7. **Copy each cart item into `order_items`**, with the name, size, original price, discount, promotion used, and final price at that moment (a bundle becomes one main line plus its item lines), so later price changes don't affect this order.
8. **Do not reduce stock yet.** The admin panel shows these quantities as "awaiting confirmation" so the owner can see demand.
9. **Lock the cart items to the order** instead of deleting them: each item gets the new `order_id` and the state `ordered` (see `03-cart.md`). Write the first line of `order_status_history`.
10. **Set the bank payment deadline** (Bank Transfer only): `payment_due_at` = now + `bank_payment_hours` (default 48).
11. Return the order number. The website opens the confirmation page.

```
place_order(checkout_token, address, delivery_method_id, payment_method, note):
  if order exists with checkout_token → return that order
  cart = cart_items for current user where state = 'active'
  if cart empty → error "Your cart is empty"
  for each item:
    if product inactive → error "<name> is no longer available"
    if stock_qty < quantity → error "Only <n> left of <name>"
  -- bundles expand into their items for the stock check (combined with loose items)
  subtotal = Σ(effective_price(item) × quantity)   -- promotion or bundle price
  method = delivery_methods[delivery_method_id] (must be active)
  delivery_fee = subtotal >= method.free_over ? 0 : method.fee
  cod_fee = payment_method = 'cod' ? settings.cod_fee : 0
  total = subtotal + delivery_fee + cod_fee - discount
  if payment_method = 'cod' → check_cod_eligibility(user, address, total)
  status = payment_method = 'bank_transfer' ? 'on_hold' : 'pending'
  insert order (status, payment_status 'unpaid', ...)
  insert order_items (snapshot of each item)
  -- stock is NOT reduced here (see confirm_order, Stock rules summary)
  update cart_items set state = 'ordered', order_id = new_order.id
    where user_id = current user and state = 'active'
  insert order_status_history (status, 'Order placed')
  if payment_method = 'bank_transfer' → payment_due_at = now() + bank_payment_hours
  return order_number
```

## Cancellation rules (both methods)

| Who | When allowed | Stock | Money |
|---|---|---|---|
| Customer | Bank: while `on_hold`. COD: while `pending` | Nothing to return (not yet reduced) | Bank: if any payment was recorded, becomes `refund_pending` |
| Admin | Any time before `shipped` | Returned only if the order was already `processing` | Bank + paid → `refund_pending` |
| Admin | After `shipped` | Use **Returned** instead, then restock | As above |
| System | Unpaid bank transfer past deadline + grace (if auto-cancel is on) | Nothing to return | None taken |

Every cancellation needs a reason, saved in `order_status_history`. When an order is cancelled, its items are released back to the customer's normal cart (`03-cart.md`).

## Stock rules summary
- **Stock reduces only for confirmed purchases**: at the moment an order moves into `processing`.
  - **Bank Transfer:** when the admin marks the payment as received (`on_hold` → `processing`)
  - **Cash on Delivery:** when the admin confirms the order (`pending` → `processing`)
- Placing an order does **not** reduce stock. Checkout only checks that enough stock exists at that moment.
- **Confirming** re-checks stock with the variant rows locked, so two admins can't confirm the last bottle twice. If there isn't enough, confirmation is blocked.
- Because unconfirmed orders don't hold stock, two customers can order the last bottle. **Whoever is confirmed first gets it.** The other order is flagged "Out of stock — action needed" for the admin to restock or cancel. The admin panel shows **awaiting confirmation** quantities per product so this can be spotted early.
- The order records `stock_reduced_at`. **Return** stock on cancellation **only if** `stock_reduced_at` is set, i.e. the order was already `processing`. Cancelling a `pending` or `on_hold` order needs no stock change.
- **Return** stock for `returned` orders only when the admin clicks **Restock**, after checking the parcel is undamaged. The admin can also choose "Damaged — do not restock".
- Stock can never go below 0 (database rule).
- **Bundles** have no stock of their own. Confirming a bundle order reduces the stock of each item in the bundle.
