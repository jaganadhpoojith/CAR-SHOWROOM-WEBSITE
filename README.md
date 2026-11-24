
# Elite Motors — Premium Car Showroom

> **Drive your dream. Experience the extraordinary.**

Elite Motors is a multi-page car showroom web app with a luxury black, white and gold design. Customers can browse inventory, compare cars, calculate EMI, book test drives and send enquiries. Staff manage everything from an admin dashboard.

The whole app is one self-contained file, `elite-motors.html`, published as a Claude Artifact.

- **Live page:** https://claude.ai/artifact/KtMb4zWrYV3p9nSNcQuADo
- **Source:** `elite-motors.html`

---

## Features

### Customer-facing
| Area | What it does |
|---|---|
| Homepage | Announcement bar, sticky nav, hero, featured vehicles, latest arrivals, luxury/sports/supercar collections, browse by brand, why choose us, offers, testimonials, showroom and contact section, newsletter, footer |
| Inventory | 14 demo cars across 7 categories (Luxury, Sports, SUV, Sedan, Hatchback, Electric, Supercar) and 13 brands. Search, brand/category/price filters, sort by popularity, newest or price |
| Car details | Gallery (exterior and interior), specs, colours, features, description, EMI calculator, similar vehicles, Book / Enquire / Compare / Save buttons |
| Test drive booking | Validated form (name, phone, email, car, date, time, showroom). Saved to the database |
| Enquiries | General enquiry, callback request, availability enquiry, financing enquiry, WhatsApp button, call button |
| Compare | Up to 3 cars side by side |
| Wishlist | Save favourite cars (stored in the browser) |
| Finance | Financing information and a standalone EMI calculator |
| Theme | Dark/light toggle, follows the system preference by default |
| SEO | Title, meta description, Open Graph tags, JSON-LD `AutoDealer` data, per-car page titles |

### Admin dashboard (`#admin`)
- Stats: total vehicles, available, sold, enquiries, test drive bookings, revenue from sold cars
- Add, edit price, change status (Available / Reserved / Sold) and delete vehicles
- Load the demo inventory into the database with one click
- View bookings, change their status (new / confirmed / completed / cancelled) and delete them
- View and delete customer enquiries

### Honesty rules
- A booking or enquiry confirmation appears **only after the database save succeeds**.
- If the database is unavailable, the user sees an error and nothing is shown as submitted.
- No payment flow exists, and none is faked.

---

## Pages and routes

Navigation uses URL hashes, so every page is linkable and the back button works.

| Route | Page |
|---|---|
| `#home` | Homepage |
| `#cars` | Inventory |
| `#car-<id>` | Vehicle details (e.g. `#car-porsche-911`) |
| `#compare` | Comparison table |
| `#wishlist` | Saved cars |
| `#finance` | Financing and EMI |
| `#book` | Test drive booking |
| `#contact` | Enquiry forms |
| `#admin` | Admin dashboard (owner and editors only) |

---

## Technology

| Layer | Choice |
|---|---|
| Frontend | Vanilla HTML, CSS and JavaScript in a single file |
| Styling | CSS variables for theming, glassmorphism, CSS animations |
| Routing | Hash-based, in about 20 lines of JS |
| Backend | Artifact runtime `db` capability (document database) and `user` capability (permissions) |
| Images | Inline SVG renders (remote images are blocked on published pages) |

**Why not React, Next.js, Tailwind or Supabase?** Published Artifact pages must be self-contained and can only load scripts from a short list of hosts, so a build pipeline is not available. See *Migrating to Next.js + Supabase* below if you want that stack.

---

## Data model

The database has three collections.

### `cars`
| Field | Type | Example |
|---|---|---|
| `brand` | string | `Porsche` |
| `model` | string | `911 Carrera S` |
| `year` | number | `2025` |
| `cat` | string | `Sports` |
| `price` | number (INR) | `24200000` |
| `fuel` | string | `Petrol` |
| `trans` | string | `8-speed PDK` |
| `mileage` | string | `9 kmpl` |
| `engine` | string | `3.0L flat-6 · 443 hp` |
| `status` | string | `Available`, `Reserved` or `Sold` |
| `pop` | number | `98` (popularity score used for sorting) |
| `hue` | number | `40` (colour of the generated illustration; `0` = graphite) |

### `bookings`
`name`, `phone`, `email`, `car`, `date`, `time`, `showroom`, `status`, `ts` (timestamp)

### `enquiries`
`type`, `name`, `phone`, `email`, `message`, `status`, `ts`. Newsletter signups are stored here with `type: "Newsletter"`.

### Browser-only data
- Wishlist and theme choice use `localStorage` (private to each browser).
- The compare list lives in memory only.

---

## Access rules

| Collection | Read | Write |
|---|---|---|
| `cars` | Anyone with view access | Admin (owner and editors) |
| `bookings` | Contributors and up | Contributors and up |
| `enquiries` | Contributors and up | Contributors and up |

**Important:**
1. For customers to submit forms, the Artifact must be shared so that they have **Contributor** access. Viewers cannot write.
2. Because of how the access rules work, Contributors can also *read* other customers' bookings and enquiries. Treat this as a prototype. Do not collect sensitive personal data in production without per-customer privacy (see Roadmap).
3. The admin dashboard checks edit rights. Only the owner and editors can open it, and the database enforces the same rule on writes to `cars`.

---

## Getting started

1. Open the live page and sign in on claude.ai.
2. Go to `#admin`. The first time, the inventory is empty. Click **Load demo inventory into database**.
3. Share the Artifact with customers as Contributors so forms can save.
4. Add real vehicles with the **Add vehicle** form and delete the demo ones.

If the database is empty or unreachable, the site falls back to built-in demo vehicles so it still renders.

---

## Configuration before going live

These constants are at the top of the `<script>` in `elite-motors.html`:

```js
const WA  = '910000000000';   // WhatsApp number: country code + number, no "+"
const TEL = '+910000000000';  // Phone number for the Call button
const MAP = 'https://www.google.com/maps/search/?api=1&query=Banjara+Hills+Hyderabad';
```

Also replace the placeholder showroom address, hours and city list in the markup (search for `Banjara Hills`). The sample testimonials (`REV`) and demo vehicles (`D`) are illustrative, so replace them with real data.

---

## Customising

| To change | Where |
|---|---|
| Colours | CSS variables at the top of `<style>` (`--gold`, `--bg`, `--fg`, `--card`) |
| Brands | `BR` array |
| Categories | `CATS` array |
| Demo vehicles | `D` array (compact rows, mapped into objects by `DEMO`) |
| Test drive time slots | The `<select name="time">` in `book()` |
| EMI defaults | `emi()` and `ec()` (rate 9.5%, 5 years, 20% down) |

---

## Known limitations

- Car images are generated SVG illustrations, not photographs, and there is no image upload yet.
- Testimonial and homepage banner management are not in the admin dashboard.
- The Google Map opens in a new tab, not embedded.
- Revenue is the sum of prices of cars marked Sold, not real financial data.
- Admin edits reload the admin view, so avoid typing in the admin while many changes arrive at once.
- Bookings and enquiries are visible to all Contributors (see *Access rules*).
- There are no email or SMS notifications. Staff must check the admin dashboard.

---

## Roadmap

1. Real photography with an upload field in the admin (Artifact `assets` capability).
2. Per-customer privacy for bookings and enquiries.
3. Testimonial and banner management.
4. Booking and enquiry notifications to staff.
5. Embedded map, vehicle video and a 360° viewer.

---

## Migrating to Next.js + Supabase

If you need the full production stack, the app maps over directly:

| This project | Production equivalent |
|---|---|
| Hash routes | Next.js App Router (`/cars`, `/cars/[id]`, `/book`, `/admin`) |
| `cars`, `bookings`, `enquiries` | Supabase tables with the same columns |
| `canEdit()` admin check | Supabase Auth plus Row Level Security policies |
| Inline-SVG art | `next/image` with Supabase Storage |
| CSS variables | Tailwind theme tokens |
| Template-string views | React components (`CarCard`, `FilterBar`, `EmiCalculator`, `BookingForm`, …) |

Suggested row-level security: public read on `cars`; public insert only on `bookings` and `enquiries`; read, update and delete restricted to authenticated admins.

---

## Licence and notice

Demo vehicle names, specifications, prices, reviews and showroom details are illustrative only. Brand names belong to their respective owners and are used here as demonstration content.
