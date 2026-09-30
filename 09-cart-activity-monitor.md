# Cart Activity Monitor (Admin)

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

This answers "who is adding what to their cart".

- **Live Activity Feed** — every add, remove, and quantity change as it happens: time, customer (name/email, or "Guest #1234"), product, size, quantity, action.
- **Active Carts** — everyone who currently has items in their cart (not-yet-ordered items only; ordered items are shown under Orders): customer, number of items, cart value, last activity time.
- **Abandoned Carts** — carts with not-yet-ordered items and no order for more than 24 hours (configurable). Shows the customer's contact details (if registered) so staff can follow up.
- **Product Insights** — for each product: times added to cart, times removed, times ordered, and cart-to-order conversion rate.
- Filters by date range, product, and customer type (guest / registered).
- Export to CSV.
- Bundles are tracked like products (labelled "Bundle"), and Product Insights include bundles.
