# Key-free installer prompts for other agents

Start with the consumer skill. Install the dealer add-on only when dealer
functionality is required. Both prompts resolve the latest stable release from
NumiTracker and contain no API key.

## Consumer skill

Copy this prompt into an agent that can download files and install portable
Agent Skills:

```text
Fetch the current official NumiTracker skills metadata from:
https://numitracker.com/api/skills/latest

Read the `skills.numitracker.installPrompt` value from the JSON response and
follow it exactly. Use only the version, download URL, and SHA-256 checksum from
that response. Stop if the endpoint is unavailable, its response is invalid, or
the downloaded archive does not match the published checksum. Do not install
the dealer add-on unless I separately request it.
```

## Dealer add-on

For a Max account linked to an active dealer, copy this prompt after installing
the consumer skill:

```text
Fetch the current official NumiTracker skills metadata from:
https://numitracker.com/api/skills/latest

Read the `skills["numitracker-dealer"].installPrompt` value from the JSON
response and follow it exactly. Use only the version, download URL, and SHA-256
checksum from that response. Stop if the endpoint is unavailable, its response
is invalid, or the downloaded archive does not match the published checksum.
The resolved instructions must first check for the base `numitracker` skill. If
it is missing, accept the recommended installation of both skills; if you do not
want the base skill installed, cancel the dealer installation.
```

For clients without a standard skill directory, use `skills/numitracker/SKILL.md`
as task instructions. Use `skills/numitracker-dealer/SKILL.md` additionally only
for dealer workflows. Load only the referenced file needed for the current
workflow, and do not merge the two skills into one prompt.
