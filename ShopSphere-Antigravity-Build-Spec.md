# ShopSphere — Frontend Build Spec for Antigravity
### Implement the Stitch designs in React and connect them to the live backend

> **Agent: read this whole file before writing code.** The visual design already exists (made in Google Stitch). Your job is to (1) turn those designs into a React app **without changing how they look**, and (2) wire every screen to the real ShopSphere API through the gateway. Do not invent endpoints, fields or routes. If something is ambiguous, choose the simplest option that matches this file and leave a `// NOTE:` comment.

---

## 0. Inputs you should have

| Input | Where | Purpose |
| --- | --- | --- |
| Stitch screens (HTML/CSS or Tailwind export, or screenshots) | Provided in the workspace / via Stitch MCP / pasted by the user | **Visual source of truth** |
| `api-contract.md` | Workspace | **Behavioural source of truth** for every request and response |
| `SRS.md` | Workspace | Required routes and UI-01…UI-08 |
| This file | Workspace | Build and integration rules |

**Priority when sources disagree:** the Stitch design wins on *look and layout*. `api-contract.md` wins on *data, endpoints, validation and status codes*. This file wins on *architecture*.

If the Stitch export is missing a screen listed in §4, build it in the same visual language (same tokens, components, spacing) and add `// NOTE: not in Stitch export`.

---

## 1. Tech stack (mandatory)

| Concern | Choice |
| --- | --- |
| Framework | React 18 + Vite + **TypeScript** |
| Routing | React Router v6 |
| HTTP | Axios, one shared instance |
| Styling | Tailwind CSS v3 with CSS-variable design tokens (§3) |
| Icons | lucide-react (or match the icon set used by the Stitch export) |
| Forms | react-hook-form + zod |
| Toasts | `sonner` or a small custom component |
| State | React Context only: Auth, Theme, Cart, Wishlist |
| Font | Whatever the Stitch design uses (default Inter) |

Dev server runs on **port 5173** (SRS Appendix A).

---

## 2. Turning Stitch output into React

Follow this process for every screen.

1. **Extract tokens first (once).** Read colours, font family, font sizes, radii, shadows and spacing from the Stitch export for *both* light and dark. Put them in `src/styles/tokens.css` as CSS variables (§3). Never scatter raw hex codes in components.
2. **Identify shared pieces** that repeat across screens and build them once in `src/components/`: Header, Footer, ProductCard, Button, Input, Select, Badge, Modal, Drawer, Tabs, Pagination, Skeleton, EmptyState, ErrorState, StarRating, QuantityStepper, Stepper, ThemeToggle.
3. **Convert each screen** into a page component under `src/pages/`. Keep the markup structure, class names and spacing from the Stitch export where practical, but:
   - replace hard-coded colours with token classes (`bg-surface`, `text-fg`, `border-line`, `bg-primary`…),
   - replace static sample text/images with live data,
   - replace `<a href>` with React Router `<Link>`,
   - make everything responsive (mobile, tablet, desktop),
   - add the loading / empty / error states in §6.
4. **Remove Stitch artefacts:** placeholder lorem ipsum, fake prices, fake names, inline `<script>` tags, external image hosts used only as mockups, duplicated headers/footers (use the shared Layout).
5. **Do not redesign.** No new colours, fonts, layouts or components unless a state is missing (e.g. an error banner). Missing states must reuse existing tokens and components.
6. **Verify visually** at 375 px, 768 px and 1440 px in light and dark mode against the Stitch screens.

### Palette fallback
If a token cannot be read from the Stitch export, use this teal palette (it is the one the designs were generated from):

```
LIGHT  bg #FAFAF9  surface #FFFFFF  surface-2 #F5F5F4  line #E7E5E4
       fg #1C1917  muted #78716C
       primary #0F766E  primary-hover #115E59  primary-fg #FFFFFF  primary-soft #F0FDFA
       accent #F59E0B  success #15803D  warning #B45309  danger #DC2626  info #0369A1
DARK   bg #0C0A09  surface #1C1917  surface-2 #292524  line #34302D
       fg #FAFAF9  muted #A8A29E
       primary #2DD4BF  primary-hover #5EEAD4  primary-fg #042F2E  primary-soft #0F2E2B
       accent #FBBF24  success #4ADE80  warning #FBBF24  danger #F87171  info #38BDF8
```

---

## 3. Theme (light and dark) — required

- Tokens live as CSS variables on `:root` (light) and `[data-theme="dark"]` (dark). Tailwind maps colours to them (`colors: { bg: 'var(--bg)', surface: 'var(--surface)', … }`).
- `ThemeContext` exposes `theme` and `toggle()`. First visit follows `prefers-color-scheme`; the user's choice persists in `localStorage` key `shopsphere.theme`.
- Add an inline script in `index.html` that sets `document.documentElement.dataset.theme` **before React mounts** (no flash).
- `ThemeToggle` (sun/moon) sits in the header of every layout, including admin.
- Dark mode is a designed theme (layered surfaces, thin borders), not an inversion. Keep text contrast at WCAG AA.
- Product images are never filtered in dark mode.

---

## 4. Routes (from SRS §5.1)

| Route | Access | Page |
| --- | --- | --- |
| `/` | Public | Home, featured products |
| `/products` | Public | Catalogue |
| `/products/:id` | Public | Product detail and reviews |
| `/login` | Public | Sign in |
| `/register` | Public | Create account |
| `/cart` | Customer | Cart |
| `/wishlist` | Customer | Wishlist |
| `/checkout` | Customer | Address and payment |
| `/orders` | Customer | Order history |
| `/orders/:id` | Customer | Order detail and tracking |
| `/profile` | Customer | Profile, addresses, password |
| `/admin/products` | Admin | Product and category management |
| `/admin/orders` | Admin | Order management |
| `*` | Public | Not found |

**Guards**
- `ProtectedRoute`: no session → `/login`, keeping `state.from` so the user returns afterwards (UI-03).
- `AdminRoute`: requires `role === "ADMIN"`; others see a "no access" state; guests go to `/login`.
- Logged-in users opening `/login` or `/register` are redirected (admin → `/admin/products`, customer → `/`).
- Admins do not shop: hide cart, wishlist, add-to-cart and heart controls for admins.
- Header, nav and footer appear on every page (UI-01).

---

## 5. Backend connection — make it seamless

### 5.1 How the app reaches the gateway
All traffic goes to the **API Gateway on port 8080**. No code may reference ports 8081–8088 (SRS constraint C-5).

Use a Vite **dev proxy** so there are no CORS problems in any environment (including GitHub Codespaces, whose forwarded origin is not `localhost:5173`):

`vite.config.ts`
```ts
export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      '/api': { target: process.env.VITE_GATEWAY_URL || 'http://localhost:8080', changeOrigin: true },
    },
  },
});
```

`.env.example`
```
# Leave VITE_API_BASE_URL empty in development: the Vite proxy forwards /api to the gateway.
# Set it only when deploying the frontend separately, e.g. http://localhost:8080
VITE_API_BASE_URL=
VITE_GATEWAY_URL=http://localhost:8080
```

`axiosClient.ts` uses `baseURL: import.meta.env.VITE_API_BASE_URL ?? ''`, so paths are written as `/api/...`. The backend also allows the Vite origin through the gateway's CORS config (FR-GW-06), so both modes work.

### 5.2 Axios client rules (UI-07, UI-08)
- **Request interceptor:** if a token exists, set `Authorization: Bearer <token>`. This is the *only* place a token is attached.
- **Response interceptor:** on `401` from anything except `POST /api/auth/login`, clear the session, toast "Your session has expired. Please sign in again.", and redirect to `/login` keeping the current location.
- Never put `userId` in a path or body: the backend derives the user from the token (`X-User-Id`).
- Never call any path containing `/internal/`; the gateway returns 404 for them.

### 5.3 Error normalisation
Backend errors share one envelope:
```json
{ "timestamp": "...", "status": 400, "error": "Validation Failed", "message": "One or more fields are invalid", "path": "/api/users/register", "fieldErrors": { "email": "must be a well-formed email address" } }
```
Normalise every failure to:
```ts
type ApiError = { status: number; message: string; fieldErrors?: Record<string, string> };
```
- `fieldErrors` → `setError` on the matching form field (UI-06).
- No response at all → "Can't reach the server. Please check that the backend is running and try again."
- `403` → "You don't have permission to do that." · `500` → "Something went wrong on our side. Please try again." · `404`, `409`, `400`, `402`, `503` → show the server `message`.
- Never show raw status codes, JSON or stack traces (UI-05).

### 5.4 Session
`POST /api/auth/login` → `{ token, userId, email, firstName, role, expiresIn }`. Store `token, userId, email, firstName, role, expiresAt (= now + expiresIn*1000)` in `localStorage` key `shopsphere.session`. On startup restore it; if expired, clear it. Schedule an auto-logout at expiry. Logout calls `POST /api/auth/logout` (ignore failure), then clears session, cart and wishlist state.

### 5.5 Formatting and pagination
- **Money:** JSON numbers. Format with `Intl.NumberFormat('en-IN', { style: 'currency', currency: 'INR' })`.
- **Dates:** ISO without timezone (`2026-09-01T14:32:10`); parse as local, display like `01 Sep 2026, 2:32 PM`.
- **Pagination:** query `page` (0-based) and `size` (default 12, max 100). Response: `{ content, page, size, totalElements, totalPages, last }`. UI shows 1-based pages.

### 5.6 API layer layout
One file per service in `src/api/`, each exporting typed functions. Pages and components never call Axios directly.

```
src/api/axiosClient.ts   auth.ts   users.ts   products.ts   categories.ts
                         cart.ts   wishlist.ts   orders.ts   payments.ts   reviews.ts
src/api/types.ts         // all DTO types below
```

---

## 6. Required states on every data screen
1. **Loading:** skeletons matching the layout (UI-04).
2. **Empty:** the Stitch empty-state design (icon, message, CTA).
3. **Error:** readable message plus "Try again" (UI-05).
Mutations: disable the trigger, show a spinner on the button, toast on success and on failure.

---

## 7. Endpoint inventory (the only calls allowed)

### Auth
| Method | Path | Body / notes |
| --- | --- | --- |
| POST | `/api/auth/login` | `{email,password}` → session. `401` → "Invalid email or password" |
| POST | `/api/auth/logout` | no body |
| POST | `/api/auth/change-password` | `{currentPassword,newPassword}`; `400` if current wrong |

### Users
| Method | Path | Notes |
| --- | --- | --- |
| POST | `/api/users/register` | `{firstName,lastName,email,password,phone}` → `201 {userId,firstName,lastName,email,phone}`; `409` duplicate email; `400` + `fieldErrors` |
| GET | `/api/users/me` | `{id,firstName,lastName,email,phone}` (key is **`id`**, not `userId`) |
| PUT | `/api/users/me` | **only** `firstName,lastName,phone`. Email is not updatable |
| GET | `/api/users/me/addresses` | array of address objects |
| POST | `/api/users/me/addresses` | `201`; body `{line1,line2?,city,state,pincode}` |
| PUT | `/api/users/me/addresses/{addressId}` | `200` |
| DELETE | `/api/users/me/addresses/{addressId}` | `204`; `400` if it is the only address |
| PATCH | `/api/users/me/addresses/{addressId}/default` | `200` |

Address object: `{addressId,line1,line2,city,state,pincode,isDefault}`. **Never send `isDefault`** on create or update. The first address becomes default automatically.

### Catalogue
| Method | Path | Notes |
| --- | --- | --- |
| GET | `/api/products` | params `keyword, categoryId, minPrice, maxPrice, inStock, sort(newest\|priceAsc\|priceDesc), page, size` |
| GET | `/api/products/{id}` | detail incl. `description, stockQuantity, categoryId, categoryName, createdAt` |
| POST / PUT / DELETE | `/api/products[/{id}]` | Admin. Body `{name,description,price,stockQuantity,imageUrl?,categoryId}` |
| GET | `/api/categories` | public |
| POST / PUT / DELETE | `/api/categories[/{id}]` | Admin |

List item: `{id,name,price,imageUrl,categoryName,inStock,averageRating:null,reviewCount:null}`. **Ratings are null in lists: never render ratings on cards.**

### Cart
| Method | Path | Notes |
| --- | --- | --- |
| GET | `/api/cart` | `{cartId,items[{itemId,productId,productName,unitPrice,quantity,lineTotal,imageUrl,available}],itemCount,subtotal}` |
| POST | `/api/cart/items` | `{productId,quantity}` → `200` with full cart; adds to existing line |
| PUT | `/api/cart/items/{itemId}` | `{quantity}` |
| DELETE | `/api/cart/items/{itemId}` | remove one |
| DELETE | `/api/cart` | empty |

### Wishlist
| Method | Path | Notes |
| --- | --- | --- |
| GET | `/api/wishlist` | `{items[{itemId,productId,productName,price,imageUrl,inStock,addedAt}],itemCount}` |
| POST | `/api/wishlist/items` | `{productId}`; `409` if already present (treat as success) |
| DELETE | `/api/wishlist/items/{itemId}` | |
| POST | `/api/wishlist/items/{itemId}/move-to-cart` | on failure the item stays |

### Orders and payments
| Method | Path | Notes |
| --- | --- | --- |
| POST | `/api/orders` | `{addressId,paymentMethod}`; method ∈ `CARD, UPI, NET_BANKING, CASH_ON_DELIVERY` |
| GET | `/api/orders` | paginated summaries `{orderId,status,totalAmount,itemCount,placedAt}`, newest first |
| GET | `/api/orders/{id}` | `{orderId,status,totalAmount,placedAt,paymentStatus,transactionRef,deliveryAddress{line1,city,state,pincode},items[{productId,productName,unitPrice,quantity,lineTotal}]}` |
| PATCH | `/api/orders/{id}/cancel` | `400` if SHIPPED, DELIVERED or CANCELLED |
| GET | `/api/orders/admin/all` | Admin, paginated |
| PATCH | `/api/orders/admin/{id}/status` | Admin, `{status}` |
| GET | `/api/payments/order/{orderId}` | `{paymentId,orderId,amount,paymentMethod,status,transactionRef,processedAt,failureReason?}` |

Order status values: `PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED`. A freshly placed order comes back `CONFIRMED`.

### Reviews
| Method | Path | Notes |
| --- | --- | --- |
| GET | `/api/reviews/product/{productId}` | public: `{productId,averageRating,reviewCount,reviews[{reviewId,userId,userName,rating,comment,createdAt}]}` |
| GET | `/api/reviews/product/{productId}/summary` | public |
| POST | `/api/reviews` | `{productId,rating(1-5),comment?(≤1000)}`; `403` if not purchased/delivered; `409` if already reviewed |
| PUT / DELETE | `/api/reviews/{id}` | own review only |
| GET | `/api/reviews/me` | own reviews |

---

## 8. Screen-by-screen wiring

For each screen, keep the Stitch layout and wire the data and behaviour below.

**Home `/`** — `GET /api/categories` for chips (click → `/products?categoryId=`); `GET /api/products?sort=newest&size=8` for featured cards.

**Catalogue `/products`** — All filter state lives in the **URL query string**. Debounce keyword 400 ms; any filter change resets `page` to 0. Sort dropdown maps to `sort`. Price inputs validate `min ≤ max`. "In stock only" → `inStock=true`. Empty state "No products match your filters" with Clear filters. Hero search in the header navigates to `/products?keyword=`.

**Product detail `/products/:id`** — `GET /api/products/{id}` then, **separately**, `GET /api/reviews/product/{id}` (own skeleton and error state so a reviews failure never hides the product). Stock message: `0` → Out of stock (buttons disabled); `1–5` → "Only N left" (warning); else In stock. Quantity stepper max = `stockQuantity`. Rating distribution bars are computed client-side from the reviews list. Review form only for customers; guests see "Log in to write a review". Handle `403` as an info banner ("You can only review products you have received") and `409` ("You've already reviewed this product"). Edit/Delete show only when `review.userId === session.userId`. `404` → "Product not found". Guests pressing Add to cart or the heart go to `/login` with `state.from`.

**Login `/login`** — On success store the session, then admin → `/admin/products`, customer → `state.from` or `/`. `401` → banner "Invalid email or password".

**Register `/register`** — Client validation mirrors the server: first/last name 2–50; valid email; phone exactly 10 digits; password 8–64 with uppercase, lowercase, digit and special char (live checklist); confirm password (client only). `409` → error on email field; `400` → map `fieldErrors`. Success → toast and `/login` with email prefilled.

**Cart `/cart`** — Quantity changes call `PUT /api/cart/items/{itemId}` (debounced ~400 ms, optimistic with rollback). Rows with `available === false` are greyed with "No longer available", and **Proceed to checkout is disabled** with the hint "Remove unavailable items to continue". Clear cart uses a confirm modal. `CartContext` refreshes the header badge after every mutation and on login.

**Wishlist `/wishlist`** — Move to cart (disabled when `inStock === false`) → `POST .../move-to-cart`; on success remove the card, refresh cart badge and toast; on failure keep the card. `WishlistContext` holds the set of wishlisted productIds so hearts are filled across the app; toggling adds (POST) or removes (DELETE with the matching `itemId`).

**Checkout `/checkout`** — Redirect to `/cart` if empty or any item unavailable. Addresses from `GET /api/users/me/addresses` (default preselected). Inline add-address form: `line1`, `line2?`, `city`, `state`, `pincode` (exactly 6 digits). Payment method cards: Card, UPI, Net Banking, Cash on Delivery. **No card-number inputs** (payment is simulated). Place order → `POST /api/orders`, prevent double submit:
- `201` → refresh cart badge, go to `/orders/:orderId` with banner "Order placed. Thank you!" and the transaction ref.
- `402` → red banner "Payment failed. Your order has not been placed." Stay on page.
- `400` → show server message (empty cart, invalid address, stock shortfall) with a link to the cart.
- `503` → server message with Retry.
(Code comment only: the simulated gateway fails any total ending in `.99`.)

**Orders `/orders`** — Paginated cards with status badges (Pending grey, Confirmed info, Shipped primary, Delivered success, Cancelled danger).

**Order detail `/orders/:id`** — `GET /api/orders/{id}`, plus `GET /api/payments/order/{id}` in a separate request (if it fails, show a small inline message and keep the page). Stepper Confirmed → Shipped → Delivered; `CANCELLED` replaces it with a red banner. **Cancel order** only for `PENDING` or `CONFIRMED` (confirm modal → `PATCH .../cancel`, toast "Order cancelled."). `DELIVERED` shows "Write a review" links to `/products/:productId`. `403`/`404` → "Order not found".

**Profile `/profile`** — Tabs Profile / Addresses / Security. Email is read-only (disabled, lock icon, tooltip). Saving updates the stored `firstName` so the header refreshes. Addresses: add, edit, delete (confirm), set default; show the server message on the "only address" 400. Security: change password; `400` → "Current password is incorrect" on that field.

**Admin Products `/admin/products`** — Tabs Products / Categories. Table + side drawer form. Validation: name 3–150; description required max 2000; price > 0; stockQuantity integer ≥ 0; category required; imageUrl optional valid URL with preview. Inline "Adjust stock" calls `PUT /api/products/{id}` with existing fields plus the new `stockQuantity`. Stock ≤ 5 amber, 0 red. Map `fieldErrors` onto the drawer form.

**Admin Orders `/admin/orders`** — `GET /api/orders/admin/all` (paginated). The response shape for admin rows is not fully specified in the contract: render defensively (`customerName ?? userId ?? '—'`). Status dropdown offers **only valid next steps** and is disabled for terminal states:
```
PENDING   → CONFIRMED, CANCELLED
CONFIRMED → SHIPPED, CANCELLED
SHIPPED   → DELIVERED
DELIVERED, CANCELLED → (none)
```
Confirm modal → `PATCH /api/orders/admin/{id}/status`. Update the row on success; show the server message on `400`.

**Not found `*`** — Stitch 404 design inside the standard layout.

---

## 9. Cross-cutting rules
1. Guests who press an auth-gated action are sent to `/login` with `state.from`.
2. Header cart badge = `itemCount` from `GET /api/cart`; hide when 0; refresh after login, cart mutations and checkout.
3. Toasts for every mutation, success and failure.
4. Validate on blur and submit; focus the first invalid field; show server `fieldErrors` against the right field.
5. Confirm every destructive action (delete address/product/review, clear cart, cancel order).
6. Set `document.title` per page ("Cart · ShopSphere"); scroll to top on navigation.
7. Images: lazy load, `alt` from product name, placeholder on missing/failed `imageUrl`.
8. Accessibility: semantic HTML, visible focus rings, `aria-label` on icon buttons, modal focus trap and Esc, respect `prefers-reduced-motion`.
9. No secrets or hard-coded hosts in components; all requests go through `src/api/*`.

---

## 10. Running against the real backend

The backend is a Spring Boot microservice system. Before testing the frontend, it must be running. From the backend repo root (Codespace or local):

```bash
docker compose up -d            # MySQL
./scripts/start-all.sh          # builds, starts Eureka, gateway and services
```

Then in the frontend folder:

```bash
npm install
cp .env.example .env
npm run dev                     # http://localhost:5173
```

Quick checks before blaming the UI:
```bash
curl -i http://localhost:8080/api/categories                 # expect 200 JSON
curl -i http://localhost:8080/api/products?size=2            # expect 200 paginated JSON
curl -i http://localhost:8080/api/cart/internal/7            # expect 404 (internal routes blocked)
```

Notes:
- Registration only creates **CUSTOMER** accounts. An **ADMIN** account must already exist in the seeded data. Do not hard-code credentials in the frontend. Ask the user for them or read them from the backend seed.
- To demo the payment failure path, an order total ending in `.99` returns `402`.
- If the browser shows a network/CORS error while using `VITE_API_BASE_URL`, clear that variable and rely on the Vite proxy.

---

## 11. Build order
1. Scaffold Vite + React + TS + Tailwind; extract tokens from Stitch; theme script and toggle.
2. Shared components and layouts (Header, Footer, Layout, AdminLayout, guards).
3. Axios client, error normaliser, `src/api/*`, `AuthContext`, session handling.
4. Login and Register.
5. Home, Catalogue, Product detail with reviews.
6. Cart and Wishlist (contexts and pages).
7. Checkout, Orders, Order detail.
8. Profile.
9. Admin Products and Admin Orders.
10. 404, responsive pass, accessibility pass, light/dark pass on every page, compare to Stitch screens.
11. `README.md` with setup steps from §10.

---

## 12. Acceptance checklist
- [ ] All 14 routes render; guards redirect correctly (UI-03).
- [ ] Every screen matches its Stitch design at 375 / 768 / 1440 px in **light and dark**.
- [ ] Theme toggle persists, follows system on first visit, no flash on load.
- [ ] Every request goes through the gateway (proxy or `VITE_API_BASE_URL`); no ports 8081–8088, no `/internal/` calls.
- [ ] JWT attached only by the Axios request interceptor; 401 clears session and redirects (UI-07, UI-08).
- [ ] Every async action shows a loading state (UI-04); every backend error is readable (UI-05); field errors appear on the right field (UI-06).
- [ ] No mock or placeholder data remains from the Stitch export.
- [ ] Product cards show no ratings; detail page loads reviews in a second request.
- [ ] Cart blocks checkout when any item is unavailable.
- [ ] Checkout distinguishes 201, 400, 402 and 503.
- [ ] Cancel appears only for PENDING/CONFIRMED; admin status dropdown offers only valid transitions.
- [ ] Profile email is read-only; default address changes only through the PATCH action; `isDefault` never sent on create/update.
- [ ] `npm run build` passes with no TypeScript or lint errors.
