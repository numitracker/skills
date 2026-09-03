# Security policy

## Supported versions

Security fixes are applied to the latest stable release. Consumers should pin a
release tag, verify `SHA256SUMS`, and upgrade when a security release is
published.

## Report a vulnerability

Use GitHub's **Security → Report a vulnerability** flow for this repository.
Do not open a public issue for a suspected API-key leak, unsafe write workflow,
prompt-injection path, permission bypass, or release-integrity problem.

Include the affected skill and version, reproduction steps, expected behavior,
and impact. Do not include a real NumiTracker API key, user prompt history, MCP
tool result, dealer data, or other personal information.

## Secret handling

NumiTracker API keys are shown once in the NumiTracker dashboard and configured
separately in the client. They must never appear in:

- installation prompts or URLs;
- this repository or a pull request;
- skill or plugin archives;
- screenshots, analytics, or public-page HTML;
- bug reports or eval fixtures.

Revoke an exposed key immediately in Dashboard → Integracje AI and create a new
least-privilege key.

## Trust boundary

Skills are instructions, not an authorization boundary. The NumiTracker server
enforces API-key scopes, the user's current subscription tier, dealer status,
ownership, and rate limits on every call. A skill must still preview writes and
obtain explicit approval because technically permitted actions can be
unintended.
