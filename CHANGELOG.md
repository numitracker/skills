# Changelog

All notable changes follow [Keep a Changelog](https://keepachangelog.com/) and
[Semantic Versioning](https://semver.org/).

## [Unreleased]

### Changed

- Align product spot alerts with product-selected metals and null stored
  indicators; omit ineffective market-alert metal filters.
- Enforce alert operator threshold shapes, the two execution modes, and the
  200-character name limit in guidance and evals.
- Clarify missing-spot premiums and snapshot/history nulls, plus active-only
  watchlist rows/totals, inactive-add refusals, and retained membership quotas.
- Document structured pending-offer refusal codes, required spot data and retry
  handling, acceptance validation, and ordinary repricing exceptions.
- Document changed-value profile limits, schema safety ceilings, widget/year
  bounds, own-domain websites, and location creation limits without blocking
  unchanged legacy values or unrelated edits.
- Add consumer and dealer evals for the updated validation and refusal paths.

- Use the dealer-prefixed tool names and separate location create/update/removal
  calls, with ownership, full-week hours, transaction, and retry guidance.
- Clarify the foreign-dealer EU market and null prices, premium, and spread.
- Document validated alert vocabularies and every ordered metal ratio with its
  aliases, precision, equality, and change-detection rules.
- Correct the per-key rate limits, search coverage, offer and history row
  semantics, spot units, percentage premium/spread, alert market scope and
  condition measurements, dealer price rounding, and primary-location rules.
- Expand consumer and dealer evals for these contracts and review refusals.

## [1.0.0]

### Added

- Stable consumer and dealer skills with progressive reference loading.
- Claude Code and Cursor plugin manifests with sensitive secret configuration.
- Codex, Claude Desktop, and portable-agent installation guidance.
- Release ZIPs, checksums, CI validation, security policy, and contribution rules.
- Expanded safety, permission, failure, pagination, freshness, and prompt-injection evals.

## [0.2.0]

### Added

- `numitracker-dealer` with read-only briefings, targeted repricing,
  pending-offer reviews, and profile update workflows.

## [0.1.0]

### Added

- `numitracker` with catalogue, offers, spot, history, opportunities, watchlist,
  and alerts workflows.

[1.0.0]: https://github.com/numitracker/skills/compare/v0.2.0...v1.0.0
[0.2.0]: https://github.com/numitracker/skills/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/numitracker/skills/releases/tag/v0.1.0
