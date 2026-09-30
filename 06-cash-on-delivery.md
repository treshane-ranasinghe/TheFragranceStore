# Cash on Delivery

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

## Summary

- At checkout the customer sees the method's title and description, e.g. *"Pay with cash when your order is delivered."*
- The order is created as **Pending (awaiting confirmation)**. The admin confirms it, usually after a quick call, and only then does it move to **Processing** and reduce stock.
- The Order Received page and email show the COD instructions and the exact cash amount to have ready.
- The courier collects the cash. The order becomes **Completed** only when the admin records that the cash has reached the store (full logic in below).

## Cash on Delivery logic (WooCommerce model)

This follows WooCommerce's **Cash on Delivery** gateway. Like bank transfer, it is an **offline** payment method: the website takes the order without any money, the courier collects cash at the door, and the store marks the order **Completed** once that cash has been received.

**Change from WooCommerce:** WooCommerce sends COD orders straight to Processing. Here, COD orders first wait as **Pending** until the admin **confirms** them. Only then do they move to Processing and **reduce stock**, so stock always reflects confirmed purchases only.

A few extras useful in Sri Lanka are marked ➕. They are all **off by default**, so the basic flow is exactly WooCommerce's.

### When COD is offered
COD appears at checkout only when:

| Check | Setting | Default |
|---|---|---|
| COD is enabled | `cod_enabled` | On |
| The chosen delivery method allows COD (WooCommerce's "Enable for shipping methods") | `delivery_methods.cod_allowed` | All methods allowed |
| ➕ Order total is not above the COD limit | `cod_max_order_value` | Off (no limit) |
| ➕ Customer has not refused too many COD parcels | `cod_max_failed` | Off |

The same checks run again inside `place_order`, so COD can't be forced through by editing the page. If COD is hidden, a short note explains why (e.g. "Cash on Delivery isn't available for Express delivery") and Direct Bank Transfer is offered instead.

### Step 1 — Checkout
- COD appears with its **Title**; the **Description** shows when it is selected.
- The customer clicks **Place order**. `place_order` (`04-checkout-orders-stock.md`) runs:
  - Order status → **`pending`**, payment status → `unpaid`
  - Stock is **not reduced yet**
  - Cart items are locked to the order and stay in the cart (`03-cart.md`)
  - Admin gets a real-time "New COD order to confirm" alert

### Step 2 — Order Received page and email
- "Thank you. Your order has been received."
- Order summary: order number, items, total, payment method "Cash on delivery"
- The **Instructions** text from settings, e.g. "Pay with cash upon delivery."
- A highlighted line: **"Please keep Rs. 12,850.00 ready for the courier."**
- A note: *"We'll confirm your order shortly, usually by phone."*
- The same content in the **"Order received"** email, with the instructions placed **before the order table**, as in WooCommerce. A **"Order confirmed"** email follows when the admin confirms it.

### Step 3 — Pack and ship
| Admin action | What the system does | Order / Payment |
|---|---|---|
| Packs the order | — | `processing` / `unpaid` |
| **Mark as shipped** | Admin enters courier name and tracking number. The **cash to collect** is set to the order total automatically. Customer emailed with tracking and the amount to pay. | `shipped` / `unpaid` |

### Step 4 — Delivery
| Outcome | Admin action | Order / Payment |
|---|---|---|
| Delivered, cash collected by courier | **Mark as delivered** | `delivered` / `unpaid` |
| Refused / not reachable / wrong address | **Mark delivery failed** with the reason. ➕ Customer's refused-parcel count +1. | `returned` / `unpaid` |
| Returned parcel back at the store | **Restock**: stock goes back up unless marked damaged. Order cancelled. | `cancelled` / `unpaid` |

### Step 5 — Cash reaches the store → Completed
WooCommerce's rule: **only set a COD order to Completed once the payment has actually been collected.** Couriers usually pay stores in batches, often weekly and minus their fees, so this happens later than delivery.

| Admin action | What the system does | Order / Payment |
|---|---|---|
| **Record COD cash received** (single order, or several at once from the courier's payment report) | Adds a row to `order_payments` (amount, date, courier remittance reference). Payment → `paid`. Order → **`completed`**. | `completed` / `paid` |

The admin page **Payments → COD cash to collect** lists every `delivered` + `unpaid` COD order with: order number, courier, delivery date, amount, and days waiting (red after `cod_remit_alert_days`, default 7). It also shows a **total owed per courier**. The admin can tick several orders and record one courier payment against all of them. If the amount differs from the total due, e.g. a courier fee deduction, a note is required.

### Confirming a COD order (stock is reduced here)
New COD orders appear in **Payments → COD orders to confirm**, oldest first.
- The admin may call the customer to check the order and address. Calls are optional but recommended, and each attempt can be logged (`confirmed` / `no_answer` / `wrong_number` / `cancel_requested`).
- **Confirm order** → `confirm_order` runs: stock is checked and **reduced**, the order moves `pending` → `processing`, and the customer gets the **"Order confirmed"** email. Then it continues with Step 3 (pack and ship).
- If stock is no longer enough, Confirm is blocked with a message. The admin can add stock, or cancel the order and tell the customer.
- After `cod_max_call_attempts` (3) or `cod_confirm_hours` (48) without confirmation, the order is flagged for the admin to cancel. Items are released back to the cart; no stock change is needed.

```
Cash on Delivery flow (WooCommerce model)

Checkout ──Place order──▶ PENDING / unpaid   (stock NOT reduced)
                              │  (Order Received page + "order received" email:
                              │   "Pay with cash upon delivery", amount to keep ready)
                              │
                    admin confirms (call optional) ──no answer / cancel──▶ CANCELLED
                              ▼
                   PROCESSING / unpaid   STOCK REDUCED HERE
                              ▼
                     SHIPPED / unpaid   (courier + tracking, cash to collect)
                              │
              ┌───────────────┴────────────────┐
         delivered                       refused / failed
              ▼                                ▼
     DELIVERED / unpaid                 RETURNED ──restock──▶ CANCELLED
   (cash is with the courier)
              │
   courier pays the store
              ▼
     COMPLETED / paid

```
