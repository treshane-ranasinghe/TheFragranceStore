# Direct Bank Transfer

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

## Summary

**Direct Bank Transfer** (works the same way as WooCommerce's "Direct Bank Transfer / BACS")
- At checkout the customer sees the method's title and a short description, e.g. *"Make your payment directly into our bank account. Please use your Order ID as the payment reference. Your order will not be shipped until the funds have cleared in our account."*
- The order is created **On hold**. Stock is **not** reduced until the admin confirms the payment has arrived. The Order Received page and the order email show the instructions, the store's bank account(s), the exact amount, and the order number to use as the reference.
- **No slip upload.** The customer just transfers the money. They can optionally tap **"I've made the payment"** so the admin knows to check.
- The admin checks the store's bank account, finds the transfer by order number or amount, and marks the order **Paid** (full logic in below).
- If no payment arrives within **48 hours** (configurable), the order is flagged as overdue for follow-up or cancellation.

## Direct Bank Transfer logic (WooCommerce model)

This follows WooCommerce's **Direct Bank Transfer (BACS)** gateway. It is an **offline** payment method: the website never handles the money. It records the order, puts it **On hold**, and tells the customer how to pay. The customer pays from their own bank. The store owner checks the bank account and moves the order to **Processing** once the money has arrived. **Customers do not upload slips.**

A few small extras are added for Sri Lanka. They are marked ➕ and can be switched off in settings.

### Step 1 — Checkout
- If Direct Bank Transfer is enabled, it appears as a payment option using the **Title** from settings.
- When selected, the **Description** from settings shows under it.
- The customer clicks **Place order**. `place_order` (`04-checkout-orders-stock.md`) runs:
  - Order status → `on_hold`, payment status → `unpaid`
  - Stock is **not reduced yet**. It is reduced when the payment is confirmed (Step 4).
  - Cart items are locked to the order and stay in the cart (`03-cart.md`)
  - `payment_due_at` = now + `bank_payment_hours` (default 48) ➕

### Step 2 — Order Received page (thank-you page)
Shown immediately after placing the order:
1. "Thank you. Your order has been received."
2. Order summary: order number, date, total, payment method
3. The **Instructions** text from settings
4. **"Our bank details"**: every bank account from settings, in the saved order. Each shows the bank name, branch, account name and account number, with a **copy** button ➕
5. The **exact amount** and a highlighted note: *"Use **FS-2026-00042** as the payment reference."*
6. Deadline: "Please pay by 1 Oct, 6:30 PM" ➕
7. Optional **"I've made the payment"** button ➕. It needs no file; it just alerts the admin.

The same instructions and bank details also appear:
- in the **"Order on hold" email** to the customer, placed **before the order table**, as in WooCommerce
- on the order page in **My Orders**
- on the order's card in the cart's **Ordered items** section (`03-cart.md`)

### Step 3 — Customer pays outside the website
The customer transfers the money using their bank app, online banking, CDM/ATM, or a branch, with the order number as the reference or remark. Nothing happens on the website at this step unless they tap "I've made the payment" ➕ (payment status → `payment_notified`, admin alerted).

### Step 4 — Admin checks the bank account and moves the order to Processing
The admin opens **Admin → Orders**, filtered to **On hold**, or **Admin → Payments → Bank transfers**. Orders where the customer tapped "I've made the payment" appear at the top.

The admin checks online banking or bank SMS alerts. The rule, as in WooCommerce: **only continue once the money has cleared in the account, never on the customer's word alone.** Transfers are matched by:
1. the order number in the reference or remark (best)
2. otherwise the exact amount plus the customer's name and date
3. if still unclear, a call or WhatsApp to the customer

Then the admin chooses:

| Admin action | What the system does |
|---|---|
| **Mark as paid** (full amount) | Adds a row to `order_payments` (amount, date, bank reference, admin). Payment → `paid`, `paid_at` saved. Then `confirm_order` runs: **stock is reduced** and the order moves **`on_hold` → `processing`**. Customer receives the **"Order processing"** email ("Payment received — we're preparing your order"). Cart badge → *Preparing your order*. |
| **Record part payment** ➕ | Adds a row to `order_payments`. Payment → `partially_paid`. Order **stays `on_hold`**. Customer sees "Received Rs. X, balance Rs. Y due". |
| **Overpayment** | Marked as paid; the extra amount is noted on the order for refund or store credit. |
| **Not received yet** (after "I've made the payment") ➕ | Payment → `unpaid`, deadline extended by `bank_recheck_hours` (default 24). Customer emailed to double-check or contact the store. |
| **Cancel** | Order → `cancelled` (no stock change, since stock was never reduced), items released back to the customer's cart. Customer emailed. |

**If stock ran out before the payment arrived:** the payment is still recorded, because the money is in the account, but the order **stays On hold** with a red *"Paid — out of stock"* flag. The admin either adds stock and clicks **Confirm order**, or cancels the order and refunds (Step 6). The customer is contacted either way.

After this point the order follows the normal flow: `processing` → `shipped` → `delivered` → `completed` (automatic, since it's already paid).

### Step 5 — If the customer never pays
WooCommerce leaves unpaid On-hold orders waiting until the owner cancels them. This spec does the same by default, with optional help ➕ from an hourly `pg_cron` job:

| Situation | Action | Default |
|---|---|---|
| Unpaid, 24 h before the deadline | Reminder email (once) | On |
| Unpaid, past `payment_due_at` | **Overdue** tag in admin | On |
| Unpaid, past deadline + `bank_grace_hours` (24 h) | Auto-cancel, release items to cart, email customer (no stock change) | **Off** (admin cancels by hand) |
| `payment_notified` or `partially_paid` | Never auto-cancelled | — |

### Step 6 — Refunds
If a paid or part-paid order is cancelled, payment → `refund_pending`. The admin transfers the money back from the bank, then records the refund reference and date. Payment → `refunded`, and the order is set to `cancelled`.

```
Direct Bank Transfer flow (WooCommerce model)

Checkout ──Place order──▶ ON HOLD / unpaid
                              │   (Order Received page + "on hold" email
                              │    show instructions & bank accounts)
                              │
           customer pays at their bank (outside the site)
           (optional ➕ "I've made the payment" → payment_notified)
                              │
                 admin sees money in bank account
                              │
             ┌────────────────┼──────────────────┐
         full amount      part amount ➕       never paid
             ▼                ▼                   ▼
   PROCESSING / paid    ON HOLD / part-paid   CANCELLED (items back
   STOCK REDUCED HERE   (balance shown)       in cart; stock untouched)
   ("processing" email)
             ▼
   SHIPPED ──▶ DELIVERED
```
