# Dealer profile updates

Load this reference before `dealer_update_profile`, `dealer_set_location`, or
`dealer_remove_location`. All three require `dealer.profile:write`.

## Read before write

Call `dealer_get_profile` immediately before drafting a change. Use its current
values, location IDs, completeness, and update notes. Omitted top-level fields
are unchanged, so send only intended fields, except where a provided nested
list replaces its complete current value.

Editable top-level fields include contact details, website, descriptions,
established year, metals, shipping/payment methods and info, feature toggles,
featured-offers configuration. Locations use the two separate location tools. Dealer name, `is_verified`, and
`is_trusted` are admin-managed and cannot be edited here.

A changed website must be a valid HTTP or HTTPS URL on the dealer's own current
domain. Only path, query, or scheme changes are supported here; a domain change
requires staff. Credentials, embedded or encoded whitespace/control characters,
backslashes, invalid ports, and trailing-dot hosts are refused. An exactly
unchanged stored website is preserved without domain revalidation; omit it for
unrelated edits.

To clear the established year, use `clear_established_year=true`. Do not send it
with a new `established_year` value. For every other field, `null` means
omitted; send `""` to clear a profile text field. Website cannot be cleared with
an empty string. A call with no fields is refused.

## Validate changed values

Apply these business limits to changed values:

- `description`: at most 2,000 characters;
- `metals`, `shipping_methods`, `payment_methods`, `features`: at most 50 entries
  per list;
- shipping/payment `label`: at most 100 characters;
- each metal string and each feature `id`, `label`, and `description`: at most
  200 characters;
- `featured_offers_config.manual_product_ids`: at most 500 IDs when that ID list
  changes, even if other configuration fields change independently.

Unchanged stored descriptions/lists are allowed above these business limits.
Changing a list validates the complete resulting list, including retained entries;
an unchanged entry is not exempt inside a changed list. Do not truncate legacy
values to save an unrelated edit. Omit unchanged fields where possible. Input
schemas still enforce types and safety ceilings: 20,000 characters for these
text values, 500 entries per profile list, and 5,000 manual product IDs.

Every supplied `featured_offers_config` must have `products_per_row` from 1 to 6
and `number_of_rows` from 1 to 10, inclusive. Every supplied non-null
`established_year` must be from 1800 to the current calendar year, inclusive.
These schema bounds apply even when the supplied value matches stored data.
Other schema text limits are: `phone` 20, `email` 254, `short_description` 200,
`shipping_info` 500, and `website` 200 characters.

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

## Create, update, or remove one location

Creating a location is refused when the profile already has 50 locations.
Existing branches can still be updated or removed at or above that count; an
unrelated profile edit does not require deleting legacy locations. The 50-branch
cap is not a blanket block on saving the profile.

Use `dealer_set_location(location={...})` for one branch:

- Omit `location.id` or send `null` to create a location.
- Supply the ID from `dealer_get_profile` to update that owned location.
- Include `address`, `city`, `postal_code`, `country`, `lat`, `lng`, and
  `is_primary` for both creation and update; an ID and name alone are insufficient.
- Omit `name` or send `null` to preserve it on update; `""` clears it.
- Other locations remain unchanged, except that `is_primary=true` removes the
  primary flag from any other location. Omission never deletes a location.

Use `dealer_remove_location(location_id=...)` to remove exactly one owned
location. Unknown, foreign, and already removed IDs are refused. Never guess an
ID. Removing the primary location promotes the remaining location with the
lowest ID, if any. `dealer_update_profile` accepts profile fields only; do not send `locations`
or `remove_location_ids` to it.

Each call is atomic, but a profile change and several location calls are not one
transaction. A later failure does not undo earlier successes. Preview the exact
ordered calls and stop at the first failure. Do not compensate automatically.

A repeated create without an ID creates a second location. A repeated removal
is refused, not reported as an idempotent success. After a timeout or ambiguous
result, read the profile before proposing any retry; do not recreate a branch or
repeat a removal blindly. Both location tools are declared one-shot.

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
be or offer to preserve the current entries from `dealer_get_profile`; do not
write until the full resulting week is known.

## Preview format

Show:

1. exact top-level PATCH fields;
2. before → after scalar/object changes;
3. complete before → after replacement lists;
4. each location action as create/update/delete with ID where applicable;
5. the complete seven-day opening-hours object when changing hours.

The preview must be sufficient to reconstruct the exact tool call. After
approval, execute only those calls in order. `dealer_update_profile` returns
`updated_fields` and `profile`; `dealer_set_location` returns `location`
(including its saved ID) and `profile`; `dealer_remove_location` returns
`location_id`, `removed=true`, and `profile`. Compare each response with its
preview, then use `dealer_get_profile` when any state remains ambiguous. Report
completed, failed, and unattempted calls separately.
