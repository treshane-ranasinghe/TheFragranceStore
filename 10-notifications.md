# Notifications (Emails & Admin Alerts)

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

| Event | Customer | Admin |
|---|---|---|
| Order placed | Email with order summary (bank transfer: "Order on hold" email with instructions + bank accounts) | Real-time alert in panel (+ optional email) |
| "I've made the payment" tapped | — | Real-time alert |
| Payment recorded / not found | Email | — |
| Order shipped | Email with courier + tracking number | — |
| Order cancelled | Email | — |

Emails are optional for version 1 (Edge Function + Resend). In-panel alerts use Supabase Realtime.

## Payment and order events in detail

| Event | Customer | Admin |
|---|---|---|
| Order placed (bank) | **"Order on hold" email**: instructions + bank accounts before the order table, amount, reference, deadline | Real-time alert |
| Bank payment marked as paid | **"Order processing" email** | — |
| Order placed (COD) | **"Order received" email**: COD instructions before the order table, amount to keep ready, "we'll confirm shortly" | Real-time alert: "COD order to confirm" |
| COD order confirmed | **"Order confirmed" email** | — |
| COD cash recorded | **"Order completed" email** | — |
| Customer taps "I've made the payment" | — | Real-time alert |
| Payment recorded (full / part) | Email with amount received and any balance | — |
| Payment not found | Email asking customer to check or contact the store | — |
| Payment reminder / auto-cancel | Email | Shown on dashboard |
| Shipped | Courier + tracking + amount to pay (COD) | — |
| COD refused | — | Flag on customer |
| Courier cash overdue | — | Dashboard alert |
