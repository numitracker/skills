# Dealer profile updates

Load this reference before `update_dealer_profile`.

## Read before write

Call `get_dealer_profile` immediately before drafting a change. Use its current
values, location IDs, completeness, and update notes. Omitted top-level fields
are unchanged, so send only intended fields — except where a provided nested
list replaces its complete current value.

Editable top-level fields include contact details, website, descriptions,
established year, metals, shipping/payment methods and info, feature toggles,
featured-offers configuration, and locations. Dealer name, `is_verified`, and
`is_trusted` are admin-managed and cannot be edited here.

To clear the established year, use `clear_established_year=true`. Do not send it
with a new `established_year` value.

## Whole-list replacement fields

When provided, these values represent the whole resulting list rather than an
item-level merge:

- `metals`;
- `shipping_methods`;
- `payment_methods`;
- `features`.

Read the current list, apply the user's intended change locally, and preview the
entire replacement. An empty list intentionally clears the field. Do not send a
partial list as shorthand for “add this item.” Treat
`featured_offers_config` as one complete configuration object when provided.

## Method entry objects

Every shipping or payment method is an object:

```json
{"label": "Przelew bankowy", "enabled": true, "custom": false}
```

- `label`: display text;
- `enabled`: whether the method is currently offered;
- `custom`: `true` for a dealer-authored label, `false` for a standard option.

To keep a custom method but turn it off, retain the object with
`enabled=false`. Removing it from the replacement list deletes it and loses the
custom label.

## Location merge and deletion

Locations differ from the whole-list fields:

- item with `id` → update that dealer-owned location;
- item without `id` → create a location;
- existing location omitted from `locations` → keep it unchanged;
- `remove_location_ids` → the only deletion mechanism.

Never delete by omission. Never guess an ID. A foreign or unknown ID fails the
whole call, so resolve every target from the fresh profile read.

For an updated/created location, include all fields required by the tool schema:
address, city, postal code, country, latitude, longitude, and `is_primary`.
Omitting `name` on update keeps it; an empty string clears it.

## Opening hours replace a full week

Each location owns its opening hours. When `opening_hours` is provided for a
location, it replaces that location's whole week.

- Send all seven keys: `monday`, `tuesday`, `wednesday`, `thursday`, `friday`,
  `saturday`, `sunday`.
- Each day is `{ "open": "HH:MM", "close": "HH:MM", "closed": bool }`.
- A partial week is rejected; do not fill missing days from assumptions.
- Omit `opening_hours` or send `null` to keep stored hours unchanged.
- Send `{}` only when the user explicitly wants to clear all hours.

If the user supplies hours for only some days, ask what the other days should
be or offer to preserve the current entries from `get_dealer_profile`; do not
write until the full resulting week is known.

## Preview format

Show:

1. exact top-level PATCH fields;
2. before → after scalar/object changes;
3. complete before → after replacement lists;
4. each location action as create/update/delete with ID where applicable;
5. the complete seven-day opening-hours object when changing hours.

The preview must be sufficient to reconstruct the exact tool call. After
approval, call only that patch. The write response returns `updated_fields` and
a fresh profile; compare both with the preview, then call `get_dealer_profile`
again when any state remains ambiguous.
