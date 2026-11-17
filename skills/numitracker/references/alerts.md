# Alert workflows

Load this reference for any alert create, enable, pause, or delete request. Read
with `list_alerts`; every mutation follows the exact-batch approval gate in
`SKILL.md`.

## Persistence is always explicit

Ask the user to choose:

- `one_time`: fires once, then deactivates;
- `recurring`: continues to evaluate after firing; a threshold fires again
  only after the measurement has left the threshold range and re-entered it.

Never infer persistence from “notify me,” “set an alert,” or a previous alert.
The preview must include the selected value. These are the only accepted
`execution_type` values; other strings are refused.

## Operators

- `lt`: below `value`; requires `value`, refuses `value2`;
- `gt`: above `value`; requires `value`, refuses `value2`;
- `eq`: equal to `value`; requires `value`, refuses `value2`;
- `between`: inclusive range using both `value` and `value2`, with
  `value <= value2`;
- `any`: any change; omit both `value` and `value2`, as non-null values are
  refused. It compares with the previous measurement, so the first evaluation
  never fires.

Omit thresholds an operator does not use; do not send zero as a placeholder.
Where the condition descriptions below say threshold(s), apply this operator
contract: `any` takes none, including availability, ratio, and TOP1 alerts.

Ask for units and currency when the threshold is monetary or percentage-based.
Do not guess whether `5` means PLN or percent. Price, premium, spread, and spot
thresholds round half up to two decimal places; ratio rounding is below.

Alerts take no market argument. Price, premium, spread, and availability
conditions use available offers from every dealer market (`PL` and `EU`
together), so do not present an alert as `PL`-only or `EU`-only.

## Call shape

`create_alert(name, alert_type, condition, execution_type)` takes one
`condition` object. Its `condition_type` selects the kind, and the object takes
only that kind's keys:

| `condition_type` | Keys besides `condition_type` |
|---|---|
| `product_buy_price`, `product_sell_price` | `operator`, `value`, `value2`; `product_slug` for one product, or `metal_type` with optional `product_form` and `weights` for a category |
| `premium`, `spread`, `availability` | `operator`, `value`, `value2`; `product_slug` (required) |
| `spot_price` | `operator`, `value`, `value2`; `product_slug` or `market_indicator` |
| `ratio` | `operator`, `value`, `value2`; `market_indicator` (required) |
| `cart_total_buy`, `cart_total_sell` | `operator`, `value`, `value2`; `cart_id` (required) |
| `lost_top1`, `gained_top1` | `product_slug` and `offer_type` (both required); `operator` is always `any`; no thresholds |

A non-null key of another kind, a missing required key, or an unknown
`condition_type` is an argument-validation error before the tool runs; the
message names the kind and the key. Keys of other kinds sent as `null` are
ignored. `weights` may also be sent as one slug or a JSON array string; it is
read in full.

## Supported alert types

### `product`

Targets one canonical product. Resolve `product_slug` first.

Valid conditions:

- `product_buy_price`: retail price (dealer sells to customer); requires
  `product_slug`, operator, and threshold(s).
- `product_sell_price`: buyback/skup price (dealer buys from customer); requires
  `product_slug`, operator, and threshold(s).

A `product` alert refuses a price condition without `product_slug` with code
`invalid_alert_condition` ("Product alerts require a product on price
conditions."); use `product_general` for a category.
- `premium`: premium over spot for one product; requires `product_slug`,
  operator, and percentage threshold(s).
- `spread`: retail-to-buyback spread for one product, as a percentage of the
  retail price; requires `product_slug`, operator, and percentage threshold(s).
- `availability`: availability change for one product; requires `product_slug`.
  The measurement is `1` when the product has at least one available retail
  offer and `0` otherwise; normally use `any`.
- `spot_price`: with `product_slug`, measures the product's metal value in PLN
  (its weight × the metal's spot price, with matching weight units). The product's
  metal selects the spot price. Omit `market_indicator`: it is not required,
  and any supplied indicator is discarded and stored as null. Provide operator
  and threshold(s) according to the operator contract.

### `product_general`

Targets a product category. Valid conditions are `product_buy_price` (retail)
and `product_sell_price` (buyback). Provide:

- `metal_type` as a current `MetalType.slug` (required);
- `product_form` as a current `MetalFormType.slug` when restricting form;
- `weights` as current `Weight.slug` values when restricting weights;
- operator and threshold(s).

Do not send `product_slug`: a `product_general` alert refuses a product, and a
missing `metal_type`, with code `invalid_alert_condition`. Confirm whether
omitted form/weights intentionally mean all forms/weights.

### `market`

Valid conditions:

- `spot_price`: provide a valid `market_indicator`, operator, and threshold(s)
  in PLN per troy ounce. The indicator alone selects the metal.
  The complete spot vocabulary is `spot_gold`, `spot_silver`, `spot_platinum`,
  and `spot_palladium`.
- `ratio`: provide an ordered-pair `market_indicator` from the rules below,
  an operator, and threshold(s) unless using `any`.

Do not send `metal_type` for market alerts: `spot_price` and `ratio`
conditions have no such key, so a non-null `metal_type` is an argument error.

Use current `get_spot_prices` data to show context before previewing the alert,
but do not change the threshold based on that context without user instruction.

### Validated vocabularies

Use `list_product_attributes` to resolve exact current slugs for `metal_type`,
`product_form`, and every element of `weights`. Display labels and guessed
hyphenation are not valid substitutes. Unknown slugs, empty scalar slugs, and
any invalid weight refuse the whole creation before alert, condition, or
activity rows are written. Unknown indicators are also refused,
except that product `spot_price` discards the indicator before validating it.
`None`/JSON `null` means absent; `weights=[]` means no weight restriction. At most 1,000
weights are accepted; oversized lists are refused rather than truncated.

A market `spot_price` condition requires one of `spot_gold`, `spot_silver`,
`spot_platinum`, `spot_palladium`. A `ratio` condition requires a ratio indicator;
spot and ratio indicators are not interchangeable. Use the full canonical
indicator in the preview and inspect the failed envelope's summary on refusal.

### Ratio indicators and comparisons

Canonical form: `ratio_<numerator>_<denominator>`. The value is the numerator's
spot price divided by the denominator's spot price in the same snapshot.
All ordered pairs of two different metals are supported:

| Numerator | Allowed canonical indicators |
|---|---|
| Gold | `ratio_zloto_srebro`, `ratio_zloto_platyna`, `ratio_zloto_pallad` |
| Silver | `ratio_srebro_zloto`, `ratio_srebro_platyna`, `ratio_srebro_pallad` |
| Platinum | `ratio_platyna_zloto`, `ratio_platyna_srebro`, `ratio_platyna_pallad` |
| Palladium | `ratio_pallad_zloto`, `ratio_pallad_srebro`, `ratio_pallad_platyna` |

Lowercase symbol aliases are accepted in either position, including mixed
symbol/slug pairs: `au` → `zloto`, `ag` → `srebro`, `pt` → `platyna`, `pd` →
`pallad`. For example, `ratio_au_ag` and `ratio_zloto_ag` normalize to
`ratio_zloto_srebro` before storage. Reversing the pair changes the ratio;
`ratio_srebro_zloto` is not the gold/silver special case. Same-metal pairs,
unknown metals, malformed indicators, and uppercase aliases are invalid.

- Gold/silver (`ratio_zloto_srebro`, including aliases) uses two decimal
  places. `eq` matches only when `abs(current - target) < 0.01`; `any` detects
  an exact change between the two-place measurements.
- Every other pair uses six decimal places for storage/display and for `lt`,
  `gt`, and inclusive `between`. `eq` matches only when the absolute difference
  is strictly less than one unit in the target's fourth significant figure.
  For target `0.0137`, the tolerance is `0.00001`: `0.013694` matches;
  `0.013690` and `0.013710` do not. A zero target uses `0.000001` tolerance.
- For those other pairs, `any` rounds each measurement half up to four
  significant figures, then detects a difference. This is a different rule
  from `eq`; do not promise a notification for every sixth-place change.
- Submitted thresholds round half up to two places for gold/silver and six
  for other pairs. Show the stored threshold in the preview; for example,
  `80.125` becomes `80.13`, and `0.0136945` becomes `0.013695`.
- Missing snapshots or nonpositive, missing, or nonfinite operands yield no
  ratio measurement and cannot trigger a threshold alert.

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

1. human-readable `name` of at most 200 characters (`maxLength: 200` in the input schema);
2. `alert_type` and valid `condition_type`;
3. target product/category/cart/indicator;
4. direction (`buy` retail or `sell` buyback) where relevant;
5. `operator`, `value`, `value2`, units, and currency;
6. `execution_type` (`one_time` or `recurring`).

Every non-null `value` and `value2` must be finite: NaN and positive/negative
infinity are invalid. An overlong name, a non-finite threshold, or a condition
key that does not belong to its kind is an MCP argument-validation error before
`create_alert` executes, not a failed tool envelope. Do not assume `data.error.code` is available for these errors.
An unknown or inactive `product_slug` instead returns `success=false`,
`isError: true`, code `product_not_found` and summary `Product not found.`.
Other domain refusals carry their code in `data.error.code`; read it with the
summary and do not retry or change the proposed alert silently.

Example preview:

| # | Action | Target | Condition | Persistence |
|---|---|---|---|---|
| 1 | `create_alert` | `krugerrand-1-oz-p123` | `condition={condition_type: product_buy_price, product_slug: krugerrand-1-oz-p123, operator: lt, value: 12000}` (retail, PLN) | `one_time` |

Then ask for explicit approval. After success, report the returned alert ID and
read it back with `list_alerts`.

## Existing alert changes

Use `list_alerts` to resolve IDs and current state before previewing. Each
listed condition carries only the keys of its `condition_type`: cart conditions
include `cart_id`, and TOP1 conditions have no `value` or `value2`.

- `set_alert_active(alert_id=..., active=true)` enables;
- `set_alert_active(alert_id=..., active=false)` pauses;
- `delete_alert(alert_id)` permanently removes the alert and its conditions.

When `set_alert_active` changes the state, it also resets `trigger_count` to 0
and `last_triggered_at` to null, and logs `alert_activated` or `alert_paused`.
Include that reset in the preview. Requesting the current state is a no-op:
trigger history is preserved and no activation/pause activity is logged.
Missing or foreign IDs are refused with `success=false`, `isError: true`,
`data.error.code="not_found"` and summary `The requested resource was not found.`.
Do not describe that refusal as a successful pause or enable.

Deletion is goal-state/idempotent, but still requires approval. A request to
“replace” an alert is two writes: preview deletion and creation together, then
stop if deletion fails rather than creating an unintended duplicate.
