# Astro Shopify storefront

A server-rendered headless storefront starter built with Astro, Svelte, and the Shopify Storefront API. It includes a product page, a persistent cart drawer, and a small Tailwind UI design system.

![Storefront preview](https://user-images.githubusercontent.com/10447155/214480671-8380f410-cbfb-4f53-a6bb-5c744073e2f2.jpg)

## Features

- Server-side rendering on Vercel
- Product listing and product detail pages by handle
- Shopify cart creation, line item updates, and checkout handoff
- Cart persisted in `localStorage` and restored on the next visit
- Product image gallery, breadcrumbs, price, stock state, and recommendations
- Product accordions and a review section
- Zod validation for every Storefront API response
- Responsive layout built with Tailwind UI patterns
- Custom 404 page

## Stack

| Layer | Choice |
| --- | --- |
| Framework | Astro 2 with `output: "server"` |
| UI islands | Svelte 3 |
| Styling | Tailwind CSS 3 |
| Commerce | Shopify Storefront API, `@shopify/hydrogen-react` |
| Cart state | `nanostores` and `@nanostores/persistent` |
| Validation | Zod |
| Deployment | Vercel adapter |

## Requirements

- Node.js 18 or newer
- A Shopify store
- The [Shopify Headless channel](https://apps.shopify.com/headless) installed on that store
- A test product with at least one image

## Quick start

```bash
pnpm install
pnpm dev
```

The dev server runs at <http://localhost:3000>. Create a `.env` file first with the values below.

```dotenv
PUBLIC_SHOPIFY_SHOP=your-store.myshopify.com
PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN=your-public-storefront-token
PRIVATE_SHOPIFY_STOREFRONT_ACCESS_TOKEN=your-private-storefront-token
```

`PUBLIC_` variables are exposed to the browser. The private token is only read during server-side requests, so never expose it in client code or commit `.env`.

The dev server runs at <http://localhost:3000>.

## Shopify setup

1. Open the [Shopify admin](https://accounts.shopify.com/store-login) and install the Headless channel.
2. Create a storefront.
3. Copy the public and private Storefront API access tokens into `.env`.
4. Confirm the Storefront API scopes include:
   - `unauthenticated_read_product_listings`
   - `unauthenticated_read_product_inventory`
5. Add at least one product with an image, then restart the dev server.

If you get `401` responses, check that you used the Storefront API tokens rather than Admin API tokens, and that the product is available to the storefront.

## Environment variables

| Variable | Purpose |
| --- | --- |
| `PUBLIC_SHOPIFY_SHOP` | Shopify store domain, for example `my-store.myshopify.com` |
| `PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN` | Public Storefront API token used in browser requests |
| `PRIVATE_SHOPIFY_STOREFRONT_ACCESS_TOKEN` | Private Storefront API token used in server requests |

The Storefront API version is set in `src/utils/config.ts` and currently points at `2023-01`.

## Scripts

| Command | Action |
| --- | --- |
| `pnpm dev` | Start the dev server on port 3000 |
| `pnpm build` | Build the production site into `dist/` |
| `pnpm preview` | Preview the production build |
| `pnpm astro ...` | Run Astro CLI commands |

The repo contains both `package-lock.json` and `pnpm-lock.yaml`. Pick one package manager and delete the other lockfile before shipping.

## Project structure

```text
/
├── public/
├── src/
│   ├── components/          # Astro and Svelte UI, cart, product sections
│   ├── layouts/             # Base and 404 layouts
│   ├── pages/
│   │   ├── index.astro      # Product listing
│   │   └── products/[...handle].astro
│   ├── stores/cart.ts       # Persistent cart store and cart mutations
│   ├── styles/global.css
│   └── utils/
│       ├── config.ts        # Shopify env and API version
│       ├── graphql.ts       # Storefront queries and mutations
│       ├── schemas.ts       # Zod response schemas
│       └── shopify.ts       # Storefront API client
├── astro.config.mjs
└── package.json
```

## How it works

`src/utils/shopify.ts` is the only place that talks to Shopify. It posts GraphQL operations from `graphql.ts`, parses the response with Zod schemas, and returns typed data to Astro pages and Svelte islands.

Requests are split by environment:

- Server requests use `PRIVATE_SHOPIFY_STOREFRONT_ACCESS_TOKEN` and forward the buyer IP.
- Browser requests use `PUBLIC_SHOPIFY_STOREFRONT_ACCESS_TOKEN`.

The cart lives in `src/stores/cart.ts` and is persisted under the `cart` key. On the first request of a session, the store fetches the cart again to confirm Shopify has not expired it.

## Deployment

The project uses the Vercel serverless adapter.

```bash
pnpm build
```

Set the three Shopify variables in the Vercel project, then deploy. If you deploy somewhere else, replace `@astrojs/vercel/serverless` in `astro.config.mjs` with the adapter for that platform and keep `output: "server"`.

## Notes and limits

- The Storefront API version is pinned to `2023-01`, and Astro 2 is several major versions behind current. Update both before using this as a production base.
- The homepage calls `getProducts` without a limit, and the function defaults to one product. Pass a larger `limit` when you want a full catalog grid.
- The starter covers products and carts only. Collections, search, customer accounts, discounts, and inventory alerts are not implemented.
- Product reviews are presentational and do not submit anywhere.
- Checkout is handed off to Shopify through `cart.checkoutUrl`.

## License

MIT. See [LICENSE](LICENSE).
