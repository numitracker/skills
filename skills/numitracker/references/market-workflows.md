# Market workflow routing

Load this reference for product, offer, price, history, opportunity, and
watchlist tasks. Use the MCP tool schemas as the source of truth for arguments;
this file explains sequencing, interpretation, and output.

## Product resolution

1. Clarify identity fields that can change the canonical product: metal, form,
   weight, producer/mint, series, and (when relevant) year.
2. Use `list_product_attributes` when an exact Polish slug is unknown. Do not
   translate a display name into a guessed slug.
3. Use `search_products` broadly enough to find candidates, then reuse returned
   slugs to narrow. Pass `include_unavailable=true` when the product may have no
   current retail price (watchlist, alerts, specifications).
4. If more than one candidate remains plausible, present at most five with
   canonical slug and distinguishing fields. Wait for selection.
5. Use the canonical product slug returned by the server in every later call.

Do not use price ordering to resolve an ambiguous identity; the cheapest result
may be a different weight, year, alloy, or product series.

## Exact offer comparison

Ask for `PL`, `EU`, or `ALL` once and keep it consistent. `offers_country="PL"`
selects Polish dealers; `"EU"` selects dealers in NumiTracker's stored foreign
market (`dealer.country == "EU"`), not a geographic list of EU member states.
`"ALL"` imposes no dealer-market restriction. Prices, premium, spread, price
filters, and price ordering follow this selection. Do not describe EU results
as verified EU-member-state coverage.

For each selected product:

1. Retrieve product details to confirm identity and available price summaries.
2. Use `list_product_offers` for the required side:
   - `buy` = retail, dealer sells to customer, sort/interpret lowest first;
   - `sell` = buyback/skup, dealer buys from customer, highest first;
   - `both` when the user asks for spread or exit-side comparison.
3. Follow pagination while `meta.has_more` if claiming the market-wide best.
   With `both`, all retail rows come before all buyback rows in one paged list.
4. Retail rows include unavailable offers (`is_available=false`), still sorted
   by price; exclude them from a “can buy now” conclusion. Buyback rows are
   available offers only. Offer rows carry no refresh timestamp.

Recommended compact columns:

| Product | Retail: lowest | Retail dealer | Buyback: highest | Buyback dealer | Spread | Market / freshness |
|---|---:|---|---:|---|---:|---|

Omit buyback columns when the user asked only for retail. State missing buyback
coverage instead of displaying zero. With no prices in the selected market,
`best_buy_price`, `best_sell_price`, `over_spot`, and `spread` are null. Without
retail or known positive spot, premium is null; without either price side,
spread is null. Retail with known positive spot and no buyback still permits a
premium. Missing spot alone does not prevent a spread when both prices exist.
Never turn null into zero, a -100% premium, or an arbitrage signal. Details and
watchlist can retain active unpriced products. By default `search_products`
returns only products with a retail price in the selected market;
`include_unavailable=true` adds active unpriced products, which `min_price` and
every price-based ordering still drop. Its spread/buyback orderings also require
a buyback price.

## “Best” criteria

Translate the chosen criterion into an observable ranking:

- lowest retail price → minimum active `buy` price;
- lowest premium → server-provided `over_spot`, ascending;
- strongest buyback → maximum active `sell` price;
- tightest spread → server-provided `spread`, ascending;
- preferred dealer/country → apply that filter before ranking;
- product characteristics → compare specifications first and price second.

If a criterion combines several dimensions, ask for priority or weights. Do not
create hidden weights.

## Spot, premium, and spread

Use `get_spot_prices` for the latest indexed metal snapshot and its returned
timestamp (`created_at`). Use product fields for `over_spot` and `spread` where
the server supplies them. Both are percentages: `over_spot` is the retail price
over the product's weight × spot, and `spread` is
`(retail_price - buyback_price) / retail_price × 100%`.

Only derive when necessary and units are verified:

- premium percent = `(retail_price - metal_value) / metal_value × 100%`;
- absolute spread = `retail_price - buyback_price`;
- spread percent (when requested) = `(retail_price - buyback_price) /
  retail_price × 100%`.

Name the basis used. `get_spot_prices` values are PLN per troy ounce; a 10 g
product price cannot be compared directly with them without converting the
metal content. Missing per-metal spot/buy/sell snapshot prices and ratios with
missing operands are null. If there is no snapshot at all, `get_spot_prices`
returns a failed envelope; do not invent a zero-price snapshot or timestamp.

## Price history

Use `get_price_history` after resolving one canonical product.

- Ask for period and market if missing.
- `PL` and `EU` are valid history series. For `ALL`, fetch both and display them
  separately; do not blend them into one synthetic series.
- Preserve chronological order and returned currency.
- Each row is one day's lowest available retail and highest available buyback
  price for that market. A side with no offer that day is returned as null
  with a null dealer; report it as no observation, never as a zero price.
- Disclose `days` (1-365), page coverage, and gaps/no observations.
- Describe observed changes; do not extrapolate or forecast.

## Market opportunities

Use `list_market_opportunities` only when
`market_opportunities:read.active` is true. These are pre-computed indexed
signals, not recommendations or guaranteed arbitrage.

Each row has a `type`: `low_premium`, `price_drop`, `arbitrage`, or
`rare_opportunity`. Read `type` first, then only that type's fields: fields of
the other types are absent, not null. `score` contains exactly `total`,
`value`, `freshness`, and `market_context`; do not rely on extra score keys.
Monetary values are decimal strings and percentages are numbers. Treat nulls
within a row's own fields as unavailable, not zero.

Report opportunity type, score, relevant price/premium evidence, market and
freshness. The tool has no market filter; use each row's `dealer_country` and
`detected_at`. An empty result with a cold-cache warning means the signals are
not computed yet, not that none exist. Explain that “arbitrage” describes indexed retail/buyback records and
may not survive fees, availability, shipping, taxes, timing, or dealer checks.

## Watchlist

Use `list_watchlist` for reads and product identity checks. Inactive products
are excluded from both rows and totals, regardless of market; an active product
without prices can still appear. `add_to_watchlist` refuses inactive products.
A hidden inactive membership is not proof of deletion or freed quota: the add
limit counts stored memberships. Removal can still remove an inactive product
when its slug is known. For add/remove:

1. Resolve the canonical product slug.
2. Read current watchlist state to prevent an unintended target.
3. Preview the exact add/remove batch and obtain approval under `SKILL.md`.
4. Execute in order, stop on failure, then read the watchlist back and report
   the observed state.

Do not interpret watchlist membership as an investment holding or portfolio
allocation.

## No results and pagination

`list_product_offers` refuses an unknown or inactive product with
`success=false`, `isError: true`, `data.error.code="product_not_found"` and
summary `Product not found.`. This is different from a successful empty list
for an active product without matching offers. A missing snapshot from
`get_spot_prices` is a code-less refusal (`data=null`); preserve its summary.

When no data is returned, distinguish:

- no matching canonical product;
- product exists but has no offers for the chosen market/side;
- no history in the requested window;
- scope inactive;
- a filter slug that does not exist.

Never broaden `PL` to `ALL`, change weight/year, or drop dealer constraints
without telling the user. A partial page supports “best among these rows,” not
“best in NumiTracker.”
