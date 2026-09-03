# Key-free installer prompt for other agents

Copy the prompt below into an agent that can download files and install portable
Agent Skills. The prompt is intentionally version-pinned and contains no API key.

```text
Install the official NumiTracker skills release v1.0.0 from:
https://github.com/numitracker/skills/releases/download/v1.0.0/numitracker-skills-v1.0.0.zip

Before unpacking it, download:
https://github.com/numitracker/skills/releases/download/v1.0.0/SHA256SUMS

Verify the archive's SHA-256 checksum against the exact entry in SHA256SUMS.
Stop and report the mismatch if verification fails. If it matches, install the
`numitracker` and `numitracker-dealer` folders into the standard user-level
skills directory supported by this client. Do not request, read, copy, log, or
store any NumiTracker API key while installing the skills. Do not put a key in a
URL, prompt, archive, or repository file.

Then configure the MCP connection separately, using the client's secret or
environment-variable mechanism:
- endpoint: https://api.numitracker.com/mcp
- bearer-token variable: NUMITRACKER_API_KEY

Verify by calling `server_info`, then `capabilities`. Report the installed paths,
checksum result, and verification outcome.

If this client cannot install Agent Skills automatically, stop after checksum
verification and tell me how to manually copy each folder from the archive into
its documented user-level skills directory. Keep MCP configuration as a separate
manual step.
```

For clients without a standard skill directory, use `skills/numitracker/SKILL.md`
as task instructions and load only the referenced file needed for the current
workflow. Do not merge the consumer and dealer instructions into one prompt.
