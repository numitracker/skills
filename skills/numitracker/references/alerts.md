# Alert workflows

Load this reference for any alert create, enable, pause, or delete request. Read
with `list_alerts`; every mutation follows the exact-batch approval gate in
`SKILL.md`.

## Persistence is always explicit

Ask the user to choose:

- `one_time` — fires once, then deactivates;
- `recurring` — continues to evaluate after firing.

Never infer persistence from “notify me,” “set an alert,” or a previous alert.
The preview must include the selected value.

## Operators

- `lt`: below `value`;
- `gt`: above `value`;
- `eq`: equal to `value`;
- `between`: inclusive range using both `value` and `value2`, with
  `value <= value2`;
- `any`: any change; use where the condition supports change events.

Ask for units and currency when the threshold is monetary or percentage-based.
Do not guess whether `5` means PLN or percent.

## Supported alert types

### `product`

Targets one canonical product. Resolve `product_slug` first.

Valid conditions:

- `product_buy_price`: retail price (dealer sells to customer); requires
  `product_slug`, operator, and threshold(s).
- `product_sell_price`: buyback/skup price (dealer buys from customer); requires
  `product_slug`, operator, and threshold(s).
- `premium`: premium over spot for one product; requires `product_slug`,
  operator, and percentage threshold(s).
- `spread`: retail-to-buyback spread for one product; requires `product_slug`,
  operator, and threshold(s) with verified units.
- `availability`: availability change for one product; requires `product_slug`;
  normally use `any` unless the tool schema/evaluator indicates another valid
  comparison.
- `spot_price`: spot condition attached to the product; requires
  `product_slug`, operator, and threshold(s).

### `product_general`

Targets a product category. Valid conditions are `product_buy_price` (retail)
and `product_sell_price` (buyback). Provide:

- `metal_type` as a Polish slug;
- `product_form` when the user wants to restrict form;
- `weights` when the user wants specific weights;
- operator and threshold(s).

Confirm whether omitted form/weights intentionally mean all forms/weights.

### `market`

Valid conditions:

- `spot_price`: provide `metal_type`, a valid `market_indicator`, operator, and
  threshold(s). Indicators include `spot_gold`, `spot_silver`,
  `spot_platinum`, and `spot_palladium`.
- `ratio`: provide `market_indicator=ratio_au_ag`, operator, and threshold(s).

Use current `get_spot_prices` data to show context before previewing the alert,
but do not change the threshold based on that context without user instruction.

### `cart`

Valid conditions:

- `cart_total_buy`: requires the user's numeric `cart_id` for a retail/buy cart;
- `cart_total_sell`: requires `cart_id` for a buyback/sell cart.

The referenced cart must belong to the user and its direction must match. If no
active tool can resolve the cart ID, explain that the user must supply it or use
the dashboard; never guess an ID.

### `lost_top1` and `gained_top1`

Dealer-account position alerts:

- `alert_type` and `condition_type` must match exactly;
- use `operator=any`;
- require a canonical `product_slug`;
- require `offer_type=buy` for retail or `offer_type=sell` for buyback.

Confirm direction because the ranking is opposite: retail lowest wins, buyback
highest wins.

## Create checklist and preview

Gather and show:

1. human-readable `name`;
2. `alert_type` and valid `condition_type`;
3. target product/category/cart/indicator;
4. direction (`buy` retail or `sell` buyback) where relevant;
5. `operator`, `value`, `value2`, units, and currency;
6. `execution_type` (`one_time` or `recurring`).

Example preview:

| # | Action | Target | Condition | Persistence |
|---|---|---|---|---|
| 1 | `create_alert` | `krugerrand-1-oz-p123` | retail `product_buy_price` `lt 12,000 PLN` | `one_time` |

Then ask for explicit approval. After success, report the returned alert ID and
read it back with `list_alerts`.

## Existing alert changes

Use `list_alerts` to resolve IDs and current state before previewing:

- `set_alert_active(alert_id, true)` enables;
- `set_alert_active(alert_id, false)` pauses;
- `delete_alert(alert_id)` permanently removes the alert and its conditions.

Deletion is goal-state/idempotent, but still requires approval. A request to
“replace” an alert is two writes: preview deletion and creation together, then
stop if deletion fails rather than creating an unintended duplicate.
