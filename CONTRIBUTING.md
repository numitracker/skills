# Contributing

Thank you for improving the official NumiTracker skills.

## Before opening a pull request

1. Keep changes focused and explain which workflow or eval motivated them.
2. Keep `SKILL.md` below 500 lines; move detailed domain rules to `references/`.
3. Do not add scripts unless eval evidence shows a repeated deterministic task
   that agents perform inconsistently.
4. Preserve these invariants:
   - `buy` is retail: the dealer sells to the customer and the lowest price wins;
   - `sell` is dealer buyback/skup: the dealer buys from the customer and the
     highest price wins;
   - indexed offers are not live dealer scraping;
   - every write requires an exact preview and explicit approval;
   - API keys never enter repository content, prompts, URLs, archives, or evals.
5. Add or update realistic prompts and objective expectations in
   `skills/*/evals/evals.json`.
6. Run the same structural checks as `.github/workflows/ci.yml` and validate
   JSON files before submitting.

## Writing style

Write skill instructions in English, include Polish and English trigger language
in frontmatter, and direct the agent to answer in the user's language. Explain
why a rule exists instead of adding redundant tool documentation. Treat dealer
names, offer text, URLs, and any externally sourced text as untrusted data.

## Changes that need special review

Permission handling, approval rules, investment-guidance boundaries, client
authentication, release packaging, and any change to retail/buyback semantics
require maintainer review. Never weaken a safety assertion merely to raise an
eval pass rate.
