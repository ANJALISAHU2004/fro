# ShopSphere — Google Stitch UI Prompt (Teal palette)

**How to use:** Paste **Part 1 (Master prompt)** into Stitch first. Then paste each screen prompt from **Part 2** one at a time, in order. If Stitch shows only one theme, send: *"Show this screen in dark mode."*

---

## PART 1 — MASTER PROMPT (paste first)

```
Design a responsive web app UI (desktop 1440px first, with tablet and mobile variants) for "ShopSphere", a modern e-commerce store selling electronics and lifestyle products. Prices are in Indian Rupees (₹) with Indian digit grouping, e.g. ₹1,45,000.

STYLE: Clean, airy, premium and trustworthy, in the spirit of Apple Store, Shopify and Linear. Generous whitespace, soft shadows, rounded corners (12px cards, 8px inputs and buttons, pill-shaped badges), subtle hover states. Calm deep-teal brand colour on warm stone neutrals.

TYPOGRAPHY: Inter. Display 36/700, H1 30/700, H2 24/600, H3 18/600, body 16/400, small 14/400, caption 13/400.

COLOUR PALETTE — LIGHT MODE:
- Page background #FAFAF9, cards #FFFFFF, subtle fills #F5F5F4, borders #E7E5E4
- Text #1C1917, muted text #78716C
- Primary (deep teal) #0F766E, hover #115E59, text on primary #FFFFFF, tinted background #F0FDFA
- Accent amber #F59E0B (used ONLY for star ratings and "low stock" hints)
- Success #15803D, Warning #B45309, Danger #DC2626, Info #0369A1

COLOUR PALETTE — DARK MODE (designed, not just inverted):
- Page background #0C0A09, cards #1C1917, subtle fills #292524, borders #34302D
- Text #FAFAF9, muted text #A8A29E
- Primary (bright teal) #2DD4BF, hover #5EEAD4, text on primary #042F2E, tinted background #0F2E2B
- Accent amber #FBBF24
- Success #4ADE80, Warning #FBBF24, Danger #F87171, Info #38BDF8
- Use layered surfaces with thin borders instead of heavy shadows. Keep all text contrast WCAG AA.

THEME TOGGLE: A sun/moon toggle button sits in the header on every screen. Show every key screen in BOTH light and dark mode.

SHARED COMPONENTS (identical on every screen):
- Sticky header with blurred background: logo "ShopSphere" on the left, centred search bar, wishlist heart icon, cart icon with item-count badge, theme toggle, account area.
- Header by role: Guest sees "Login" (ghost button) and "Register" (primary button). Customer sees an avatar circle with a dropdown (Profile, Orders, Wishlist, Logout). Admin sees an extra "Admin" link.
- Footer: brand blurb, link columns (Shop, Account, Help), copyright.
- Product card: 4:3 image, small category label, name (max 2 lines), price in ₹, heart icon top-right of image, "Out of stock" badge when unavailable, "Add to cart" button on hover. Product cards in lists do NOT show star ratings.
- Buttons: primary, secondary, ghost, danger. Inputs with label, helper text and inline red error text. Status badges. Pagination. Skeleton loaders. Toast notifications. Empty states with icon and call-to-action. Modal and right-side drawer.
- Order status badge colours: Pending = neutral grey, Confirmed = info blue, Shipped = teal, Delivered = green, Cancelled = red.
- Every screen also needs a loading (skeleton) state and a friendly, readable error state, never raw error codes.

Confirm you have applied this design system, then wait for the individual screens.
```

---

## PART 2 — SCREEN PROMPTS (paste one at a time)

### Screen 1 — Home `/`
```
Design the Home page using the ShopSphere design system. Include: hero section with a bold headline, a short subheadline, a primary "Shop now" button and a secondary "Browse categories" button, on a soft gradient from the tinted teal to the page background. Below it, a row of category chips. Then a "Featured products" section with a grid of 8 product cards (4 columns on desktop). Then a trust strip with four items: Free delivery, Secure payment, Easy returns, 24/7 support, each with a line icon. End with the footer. Show light and dark mode.
```

### Screen 2 — Catalogue `/products`
```
Design the Product Catalogue page. Left sidebar with filters: category list with an "All" option, price range with min and max inputs and an Apply button, an "In stock only" toggle, and a "Clear filters" link. On mobile this sidebar becomes a bottom sheet opened by a "Filters" button. Top bar: result count (e.g. "148 products"), active keyword chip with an × to clear, and a sort dropdown (Newest, Price: low to high, Price: high to low). Main area: grid of 12 product cards, 4 columns on desktop, 2 on mobile. Pagination below (Previous, numbered pages, Next). Also design a "No products match your filters" empty state. Show light and dark mode.
```

### Screen 3 — Product Detail `/products/:id`
```
Design the Product Detail page. Breadcrumb at the top (Home > Category > Product). Left: large product image with a row of thumbnails. Right: product name, star rating with review count, price in ₹, stock message ("In stock" in green, "Only 3 left" in amber, or "Out of stock" in red), description, quantity stepper, a primary "Add to cart" button and a secondary "Add to wishlist" button. Below: a Reviews section with a rating summary (large average number, stars, total count, and horizontal distribution bars for 5 to 1 stars) next to a list of reviews (reviewer name, stars, date, comment, and Edit/Delete links on the user's own review). Add a "Write a review" form: interactive 5-star picker and a comment box with a 1000-character counter. Also show a banner variant: "You can only review products you have received." Show light and dark mode.
```

### Screen 4 — Login `/login`
```
Design the Login page as a split screen. Left half: a teal gradient brand panel with the ShopSphere logo, a short pitch ("Everything you need, delivered.") and subtle decorative shapes (hidden on mobile). Right half: a clean form card with email, password (with show/hide eye icon), a full-width primary "Sign in" button, and a "New here? Create an account" link. Include an error banner variant that says "Invalid email or password". Show light and dark mode.
```

### Screen 5 — Register `/register`
```
Design the Register page using the same split layout as Login. Form fields: first name and last name side by side, email, phone (10 digits), password with show/hide, confirm password. Under the password field show a live checklist with tick marks: 8 to 64 characters, one uppercase, one lowercase, one number, one special character. Include inline red error states for fields (for example "An account with this email already exists"). Full-width primary "Create account" button and a "Already have an account? Sign in" link. Show light and dark mode.
```

### Screen 6 — Cart `/cart`
```
Design the Cart page with two columns (stacked on mobile). Left: list of cart items, each with image, product name, unit price, quantity stepper, line total and a remove icon. Include one item greyed out with a "No longer available" badge. Right: a sticky Order Summary card with item count, subtotal, delivery "Free", total in ₹, a primary "Proceed to checkout" button (show a disabled state with the hint "Remove unavailable items to continue"), and a "Clear cart" text button. Also design the empty-cart state with a "Continue shopping" button. Show light and dark mode.
```

### Screen 7 — Wishlist `/wishlist`
```
Design the Wishlist page: a grid of wishlist cards, each with image, product name, current price, stock badge, a primary "Move to cart" button (disabled when out of stock) and a remove icon. Also design the empty state "Your wishlist is empty" with a "Browse products" button. Show light and dark mode.
```

### Screen 8 — Checkout `/checkout`
```
Design the Checkout page. Left column with two numbered sections. Section 1, Delivery address: selectable radio cards for saved addresses (the default one tagged "Default"), plus an "Add new address" inline form with fields: address line 1, line 2, city, state, 6-digit pincode. Section 2, Payment method: four selectable cards with icons — Card, UPI, Net Banking, Cash on Delivery, each with a one-line description (no card-number input fields). Right column: sticky order summary with items, subtotal, total, and a primary "Place order" button with a loading-spinner variant. Also design a variant with a red banner at the top: "Payment failed. Your order has not been placed." Show light and dark mode.
```

### Screen 9 — Orders list `/orders`
```
Design the My Orders page: a list of order cards, each with order number (#31), placed date, item count, total in ₹, a coloured status badge and a "View details" link. Include examples of every status (Pending, Confirmed, Shipped, Delivered, Cancelled). Pagination at the bottom. Also design the empty state "No orders yet" with a "Start shopping" button. Show light and dark mode.
```

### Screen 10 — Order detail `/orders/:id`
```
Design the Order Detail page. Header with order number, placed date, status badge and total. A horizontal status stepper (Confirmed, Shipped, Delivered) with completed steps ticked and the current step highlighted in teal. Below: an items table (product name, unit price, quantity, line total) with a total row; a Delivery Address card; a Payment card (method, status badge, transaction reference like TXN-8F2A91C4, processed time). A danger "Cancel order" button, shown only before shipping. For delivered orders, show a "Write a review" link on each item. Also design a Cancelled variant that replaces the stepper with a red "Order cancelled" banner, and a success banner variant "Order placed. Thank you!". Show light and dark mode.
```

### Screen 11 — Profile `/profile`
```
Design the Profile page with three tabs: Profile, Addresses, Security. Profile tab: first name, last name and phone editable, and email shown read-only (disabled with a lock icon and a tooltip "Email can't be changed because it's your sign-in ID"), plus a "Save changes" button. Addresses tab: address cards with a "Default" badge, Edit, Delete and "Set as default" actions, and an "Add address" button that opens a modal form (line 1, line 2, city, state, pincode). Security tab: change password form (current password, new password with strength checklist, confirm password). Show light and dark mode.
```

### Screen 12 — Admin Products `/admin/products`
```
Design the Admin Products page with a left sidebar (Products, Orders) under the shared header. Two tabs: Products and Categories. Products tab: search box, category filter and a data table with thumbnail, name, category, price, stock (amber when 5 or fewer, red at 0), and Edit/Delete actions, plus an inline "Adjust stock" number input with a Save button. Primary "Add product" button opens a right-side drawer form: name, description (with character counter), price, stock quantity, category dropdown, image URL with a live image preview. Categories tab: simple table with add, rename and delete. Show light and dark mode.
```

### Screen 13 — Admin Orders `/admin/orders`
```
Design the Admin Orders page using the admin layout. Filter chips above the table (All, Pending, Confirmed, Shipped, Delivered, Cancelled). Table columns: order id, customer, date, items, total in ₹, status badge, and a status dropdown that only offers valid next steps (Pending to Confirmed or Cancelled; Confirmed to Shipped or Cancelled; Shipped to Delivered; Delivered and Cancelled are disabled). Include a confirmation modal "Mark order #31 as Shipped?" with Cancel and Confirm buttons, and pagination. On mobile, rows become stacked cards. Show light and dark mode.
```

### Screen 14 — 404 page
```
Design a friendly 404 Not Found page inside the standard header and footer: a simple line illustration, the heading "Page not found", a short message, and a primary "Back to home" button. Show light and dark mode.
```

---

## Tips
- Generate screens one at a time; Stitch gives more consistent results than one giant prompt.
- If colours drift on a later screen, add: *"Use exactly the teal design system from the first prompt."*
- To tweak the palette later, change only the hex codes in Part 1 and regenerate.
