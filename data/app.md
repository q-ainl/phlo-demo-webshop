# Couture webshop demo

L'Atelier Corseterie: a bilingual-plus storefront (en/fr/es/de) for corsetry
supplies with a schema-driven Phlo CMS admin, all on one SQLite file
(`data/shop.db`).

## The two faces

One app, dispatched in `app.phlo`:

- **Shop**: every request that is not admin. Routed manually through
  `%shop->route()` (shop.phlo); the shop has NO route nodes on purpose, so
  `app::route()` never runs for it and a stray `/products` can never reach a
  CMS list on the shop hostname.
- **Admin**: hostname starting with `demo.admin.`, listening port 8081, or
  `PHLO_ADMIN=1`. Optionally guarded by HTTP basic auth from `[dashboard]` in
  `data/creds.ini`. `app::route()` here dispatches only CMS routes (plus
  `CMS.list.sort.phlo`). CMS links are root-relative, which is why the admin
  lives on its own root instead of under /admin; `/admin` on the shop renders
  a signpost (page.admin.phlo).

## Routes (shop, all via shop.phlo)

- `/` home in the cookie/default language (no redirect), `/{lang}` home
- `/{lang}/{products}[/colors][/materials][/offset]` catalogue with filters
  (section slugs are localised per language, see the props in shop.phlo)
- `/{lang}/{category}/{slug}[/{subslug}][/offset]`
- `/{lang}/{product}/{slug}` detail with variants and stock
- `/{lang}/{cart|checkout|about|contact|materials|tutorials}`
- `/cart` page, `/cart/{variantId}` async add, `/cart/update/{id}/{n}` async
- `/checkout/success` POST target: stores the order, renders confirmation
- `/files|images|thumbs/{token}/{filename}` media (CMS image-field layout)

## Data model (modules/, all extend entity.phlo -> SQLite data/shop.db)

category (tree + sale flag) / product / variant / variant_stock /
product_image / order / order_variant (FK column is `orderId`; `order` is a
reserved column name in the ORM) and per-language `*_translations` tables.
English is the master on the record itself; fr/es/de live in translations.
`entity::syncTranslations` keeps them in sync on save and auto-translates via
OpenAI when a key is configured in creds.ini (otherwise it copies and never
overwrites hand-edited rows).

## Translations

Interface strings use `{en: ...}` / `en()`; fr/es/de are pre-seeded into
`langs/*.ini` by seed.phlo, so no OpenAI key is needed at runtime.

## Seeding

`php www/app.php seed::run` rebuilds the schema, the catalogue, the imagery
(GD-generated monogram cards into data/uploads/), a year of demo orders and
the langs/*.ini files. The result is committed, so a clone is populated.

## TODO

- [ ] Real product photography would beat the generated monogram cards.
