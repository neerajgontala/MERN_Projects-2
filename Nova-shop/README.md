# Nova Shop

A React storefront built on a live REST API, with product browsing, a debounced search-as-you-type bar, product detail pages, and a persistent cart backed by Redux Toolkit.

**Stack:** React 19 · Redux Toolkit · React Router · Tailwind CSS · Vite

---

## Features

- **Product catalog** — fetches and renders a responsive product grid (2→5 columns across breakpoints).
- **Live search with suggestions** — queries the search endpoint as you type and renders a dropdown of matching products with thumbnails, each linking straight to its detail page.
- **Product detail pages (PDP)** — routed by product ID, showing imagery, pricing, and computed discount.
- **Cart** — add items, track per-item quantities, and view a line-item breakdown with discounted pricing.

## Design decisions worth calling out

A few things here were deliberate rather than incidental:

**Debounced search.** The search bar doesn't fire a request on every keystroke. A `setTimeout` held in a `useRef` is cleared and re-armed on each change, so the API is only hit once the user pauses for 500ms. Storing the timer in a ref (not state) avoids re-rendering the component just to track a pending timeout.

```js
useEffect(() => {
  if (timerId.current) clearTimeout(timerId.current);
  timerId.current = setTimeout(() => getApiData(), 500);
}, [searchQuery]);
```

**Redux store as a fetch cache.** The product grid checks the store before fetching — if `screenProducts` is already populated, the request is skipped entirely. Navigating back to the home screen therefore costs zero network calls.

**PDP reads from the store first, network second.** When you click into a product, the detail page looks for it in the already-fetched list and only falls back to a per-ID API call if it isn't there (e.g. on a cold page load or a deep link). Same data, one fewer round trip in the common case.

**Cart stored as a keyed object, not an array.** The cart is `{ [productId]: { quantity } }` rather than a list of product objects. That gives O(1) lookups when incrementing quantity, avoids duplicating product data already in the store, and keeps the cart state small. The cart screen rehydrates full product details by joining those keys against `screenProducts` at render time.

## Project structure

```
src/
├── App.jsx                 # routes: /, /products/:productId, /cart
├── store.js                # Redux store
├── ProductSlice.js         # product + cart state and reducers
├── Screen/
│   ├── Home.jsx
│   ├── Pdp.jsx             # product detail page
│   └── Cart.jsx
└── Components/
    ├── Navbar.jsx
    ├── SearchBar.jsx       # debounced search + suggestions dropdown
    ├── ProductGrid.jsx     # catalog fetch + grid
    ├── ProductCard.jsx
    └── PdpComponent.jsx
```

## Running locally

```bash
npm install
npm run dev
```

Then open the URL Vite prints (default `http://localhost:5173`).

```bash
npm run build     # production build
npm run preview   # preview the production build
npm run lint      # eslint
```

## Data source

Product data comes from [DummyJSON](https://dummyjson.com), a free placeholder REST API — no API key or environment configuration needed.

## Known limitations

Worth being upfront about the current state:

- Cart state is in-memory only; a page refresh clears it. Persisting to `localStorage` (or a real backend) is the obvious next step.
- Quantity can be incremented but not decremented, and items can't yet be removed from the cart.
- No loading skeletons or error states on failed fetches — a request failure currently renders an empty grid.
- No tests.
