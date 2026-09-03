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
   slugs to narrow.
4. If more than one candidate remains plausible, present at most five with
   canonical slug and distinguishing fields. Wait for selection.
5. Use the canonical product slug returned by the server in every later call.

Do not use price ordering to resolve an ambiguous identity; the cheapest result
may be a different weight, year, alloy, or product series.

## Exact offer comparison

Ask for `PL`, `EU`, or `ALL` once and keep it consistent.

For each selected product:

1. Retrieve product details to confirm identity and available price summaries.
2. Use `list_product_offers` for the required side:
   - `buy` = retail, dealer sells to customer, sort/interpret lowest first;
   - `sell` = buyback/skup, dealer buys from customer, highest first;
   - `both` when the user asks for spread or exit-side comparison.
3. Follow pagination while `meta.has_more` if claiming the market-wide best.
4. Exclude unavailable retail offers from a “can buy now” conclusion even if
   they appear in historical or descriptive fields.

Recommended compact columns:

| Product | Retail: lowest | Retail dealer | Buyback: highest | Buyback dealer | Spread | Market / freshness |
|---|---:|---|---:|---|---:|---|

Omit buyback columns when the user asked only for retail. State missing buyback
coverage instead of displaying zero.

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
timestamp. Use product/offer fields for `over_spot` and `spread` where the
server supplies them.

Only derive when necessary and units are verified:

- premium percent = `(retail_price - metal_value) / metal_value × 100%`;
- absolute spread = `retail_price - buyback_price`;
- spread percent (when requested) = `(retail_price - buyback_price) /
  retail_price × 100%`.

Name the basis used. A 1 oz product price cannot be compared directly with a
per-gram spot value without converting the fine-metal content.

## Price history

Use `get_price_history` after resolving one canonical product.

- Ask for period and market if missing.
- `PL` and `EU` are valid history series. For `ALL`, fetch both and display them
  separately; do not blend them into one synthetic series.
- Preserve chronological order and returned currency.
- Disclose `days`, page coverage, and gaps/no observations.
- Describe observed changes; do not extrapolate or forecast.

## Market opportunities

Use `list_market_opportunities` only when
`market_opportunities:read.active` is true. These are pre-computed indexed
signals, not recommendations or guaranteed arbitrage.

Report opportunity type, score, relevant price/premium evidence, market and
freshness. Explain that “arbitrage” describes indexed retail/buyback records and
may not survive fees, availability, shipping, taxes, timing, or dealer checks.

## Watchlist

Use `list_watchlist` for reads and product identity checks. For add/remove:

1. Resolve the canonical product slug.
2. Read current watchlist state to prevent an unintended target.
3. Preview the exact add/remove batch and obtain approval under `SKILL.md`.
4. Execute in order, stop on failure, then read the watchlist back and report
   the observed state.

Do not interpret watchlist membership as an investment holding or portfolio
allocation.

## No results and pagination

When no data is returned, distinguish:

- no matching canonical product;
- product exists but has no offers for the chosen market/side;
- no history in the requested window;
- scope inactive;
- a filter slug that does not exist.

Never broaden `PL` to `ALL`, change weight/year, or drop dealer constraints
without telling the user. A partial page supports “best among these rows,” not
“best in NumiTracker.”
