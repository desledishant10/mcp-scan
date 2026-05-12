# Disclosures

Outgoing coordinated-disclosure communications to maintainers of MCP servers where MCP-Scan identified vulnerabilities. Companion to [findings/](../findings/) — findings document what the scanner observed; disclosures document what was sent to maintainers and when.

## Why this directory is public

Open auditing means the disclosure timeline itself is part of the record. Anyone can see:

- What was reported
- When it was reported
- When the maintainer acknowledged / fixed
- When the disclosure went public

That's the difference between security work and security theater.

## Policy

- **Embargo window:** 90 days from notification before public disclosure, extended if a fix is in active development.
- **No exploit code published until embargo expires.** Reproduction *runbooks* are public from the start (since they're also defense documentation); concrete payloads, exfiltrated values, etc. are redacted in this directory until the maintainer is satisfied.
- **Each disclosure is one file**, named `YYYY-MM-DD-<short-slug>.md`. The file is append-only — updates go at the bottom under a timestamped heading, the original report is preserved verbatim.

## Entry format

```markdown
# <Short title>

**Filed:** YYYY-MM-DD
**Filed by:** <name + contact>
**Filed to:** <upstream URL + maintainer contact method>
**Affected:** <package(s) + version(s)>
**Embargo:** <YYYY-MM-DD>
**Status:** <draft | filed | acknowledged | patched | public>

## Body of the filed report
<verbatim text sent to the maintainer>

## Updates
### YYYY-MM-DD
<maintainer responses, status changes>
```
