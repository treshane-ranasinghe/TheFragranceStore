# Storefront (Shop Pages)

*The Fragrance Store — spec v2.0 · part of the [spec index](README.md)*

## Home page
The homepage is **built by the admin** from sections (see `17-cms-promotions-bundles.md`):
- Announcement bar (top of every page)
- **Hero slideshow**: admin-uploaded banner images with headline and buttons, schedulable for promotions
- Rows such as New arrivals, Best sellers, On sale, Featured, a category or brand
- Promotion banners (with optional countdown), bundle showcase, category tiles, brand logos, image + text, trust badges

## Product listing
- Grid of products with image, name, brand, size, price, and "Add to cart"
- **Filters:** category, brand, gender, concentration (EDP / EDT / Parfum / Body Mist), size, price range
- **Sort:** newest, price low–high, price high–low, best selling
- **Search** by name, brand, or scent notes
- "Out of stock" badge; add-to-cart disabled when stock is 0
- Sale price, struck-through original price, and promotion badge on promoted products
- Bundles appear in their category (e.g. Gift Sets) alongside products

## Product details page
- Image gallery
- Name, brand, price (and "was" price + promotion badge if on sale; optional countdown)
- Size variants (e.g. 50 ml / 100 ml), each with its own price and stock
- Concentration, gender, scent notes (top / middle / base), description
- Quantity selector + Add to cart
- Related products
- "Also available in these sets" (bundles that include this product)

## Promotion and bundle pages
- `/promotions/[slug]`: promotion banner, description, countdown, all products in the sale
- `/bundles` and `/bundle/[slug]`: gift sets and bundles with "You save" and item list
- Full details in `17-cms-promotions-bundles.md`
