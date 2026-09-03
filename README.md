# Official NumiTracker skills

Portable AI-agent skills for comparing Polish precious-metals prices with the
[NumiTracker MCP server](https://api.numitracker.com/mcp). The repository is the
canonical, MIT-licensed source for:

- `numitracker` — catalogue search, retail and buyback offers, spot prices,
  history, market opportunities, watchlists, and alerts;
- `numitracker-dealer` — dealer briefings, targeted repricing, pending-offer
  review, competition analysis, and profile maintenance.

The skills guide an agent through the workflow and safety rules. They do not
contain price data, scrape dealer sites, or bypass NumiTracker plan and API-key
permissions.

## Requirements

- A NumiTracker Pro or Max account. Free accounts do not have MCP scopes.
- A scoped API key created in
  [Dashboard → Integracje AI](https://numitracker.com/dashboard/integracje-ai).
- An MCP-capable agent with skill support.

The public endpoint is:

```text
https://api.numitracker.com/mcp
```

Keep the API key separate from the skill. It must never be added to this
repository, an installation URL, a prompt, or a skill archive.

## Install

### Claude Code

This repository is a self-hosted Claude Code marketplace. Add it and install the
plugin:

```text
/plugin marketplace add numitracker/skills
/plugin install numitracker@numitracker-skills
```

During plugin configuration, paste the API key into the masked **NumiTracker API
key** field. Claude Code stores it as sensitive plugin configuration; it is not
part of the checked-in `.mcp.json`.

### Cursor

Import `https://github.com/numitracker/skills` as a GitHub plugin repository, or
clone it into `~/.cursor/plugins/local/numitracker` for local use. In
**Plugins → Configure**, set `NUMITRACKER_API_KEY`. The Cursor manifest refers to
that secret only by variable name.

### Codex

Install one or both standalone skill folders from this repository with
`$skill-installer`, then add this to `~/.codex/config.toml`:

```toml
[mcp_servers.numitracker]
url = "https://api.numitracker.com/mcp"
bearer_token_env_var = "NUMITRACKER_API_KEY"
```

Set `NUMITRACKER_API_KEY` in the environment that starts Codex. The
`bearer_token_env_var` field is the current key-safe configuration described by
the [official OpenAI Codex MCP documentation](https://developers.openai.com/codex/mcp/).

### Claude Desktop

Download the versioned skill ZIP from Releases and install it as a local skill.
Configure the MCP connection separately with a local HTTP-to-stdio bridge. The
bridge template in [`clients/claude-desktop.json`](clients/claude-desktop.json)
contains a placeholder to replace only in your local Desktop configuration; the
downloaded skill never contains the key.

### Other agents

Use the version-pinned, checksum-verifying prompt in
[`install/other-agents.md`](install/other-agents.md). It downloads a release
archive without credentials and keeps MCP authentication as a separate manual
step. A manual fallback is included for agents that cannot install skills.

## Verify

After configuring the MCP connection:

1. Call `server_info` to verify transport connectivity.
2. Call `capabilities` to verify the key, current plan, and active scopes.
3. Ask a read-only question, for example: “Porównaj ceny 1 oz Krugerranda na
   rynku PL.”

Every offer is the latest record indexed by NumiTracker, not a live dealer
scrape. Agents should report the returned market and freshness metadata and say
when no refresh timestamp is available.

## Polish / Po polsku

Oficjalne skille NumiTracker łączą agenta AI z katalogiem produktów, ofertami,
historią cen, alertami i narzędziami dla dealerów. Klucz utworzysz w
[panelu Integracje AI](https://numitracker.com/dashboard/integracje-ai). Najpierw
zainstaluj skill, a klucz skonfiguruj osobno jako sekret klienta — nigdy nie
wklejaj go do repozytorium ani promptu instalacyjnego. Agent odpowiada w języku
użytkownika i przed każdą zmianą pokazuje pełny podgląd operacji do zatwierdzenia.

## Versioning and releases

Releases follow semantic versioning:

- `v0.1.0` introduced the consumer skill;
- `v0.2.0` added the dealer skill;
- `v1.0.0` stabilized both skills and the cross-client install surface.

Each GitHub Release contains separate skill ZIPs, a combined bundle, and
`SHA256SUMS`. Do not install an asset whose checksum does not match.

## Development

Read [CONTRIBUTING.md](CONTRIBUTING.md). Pull requests must keep each `SKILL.md`
under 500 lines, preserve the retail/buyback direction rules, add or update
evals, and pass the repository validation workflow.

Security issues belong in GitHub private vulnerability reporting, not a public
issue. See [SECURITY.md](SECURITY.md).
