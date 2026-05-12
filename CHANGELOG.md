# Changelog

All notable changes to MCP-Scan. Format roughly follows [Keep a Changelog](https://keepachangelog.com/); the project is alpha so changes are not yet versioned with semver discipline.

## [Unreleased] — main branch

### Added

- `mcp-scan-audit` — one-shot CLI that pip-installs a package, captures its tools/list, runs the analyzer and classifier, and prints a human-readable report. Replaces the previous three-command quickstart in the README.
- Analyzer rule **MCP-S-004** — flags tools whose `annotations.readOnlyHint: true` or `destructiveHint: false` contradicts write-indicating verbs in the name or description.
- Analyzer rule **MCP-S-008** — heuristic SQLi detection from captured `tools/list`; flags query-typed parameters without parameterized-query mention.
- Analyzer rule **MCP-S-009** — heuristic SSRF detection from captured `tools/list`; static counterpart to the dynamic MCP-D-003 probe. Fires on `mcp-server-fetch` and `mcp-server-http-request`.
- Dynamic scenario **MCP-D-007** — cloud-metadata-exfiltration scenario with strict oracle (only fires on JSON-shape metadata field names; designed for EC2 audit verification).
- `disclosures/` directory with append-only audit-trail records of outgoing coordinated-disclosure communications. First entry covers the fetch + http-request SSRF disclosure.
- `findings/` directory entries for: D-003 SSRF on mcp-server-fetch (vulnerability, demonstrated on EC2 + disclosed as [modelcontextprotocol/servers#4143](https://github.com/modelcontextprotocol/servers/issues/4143)); D-003 SSRF on mcp-server-http-request (vulnerability, email-disclosed); D-001/D-006 defense observations against Claude Opus 4.7; D-002 defense observations against mcp-server-git and mcp-server-aidd; S-003 informational on mcp-server-time; aidd multi-rule informational.
- `docs/audit-runbook-ec2-ssrf-verification.md` — step-by-step runbook from AWS account creation through EC2 reproduction, evidence capture, and teardown.
- `docs/blog-draft-2026-08-10-mcp-ssrf-disclosure.md` — embargo-day blog draft (publication scheduled for 2026-08-10).
- `SECURITY.md` and `CONTRIBUTING.md`.
- Calibration corpus growth: 5 → 10 labeled targets, 33 → 81 tools. Stable per spec.
- Calibration-driven lexicon improvements (each commit-annotated with the corpus evidence that drove it).

### Changed

- README rewritten to feature real findings + one-command quickstart instead of planning-document framing.
- Scaffolded ground-truth files now include `labeled: false` so the eval skips drafts by default.

### Fixed

- `\blists?\b` lexicon pattern false-positive on Python type annotations (`Optional[List[str]] - Tags`); now uses `(?<!\[)\blists?\b(?![\[\(])` to exclude generic-type contexts.
- D-002 scenario YAML had an embedded null byte (`%00` escape-sequence smuggling); replaced with literal `%00` characters.

### Security

- Coordinated disclosure filed for class-wide SSRF in `mcp-server-fetch` (Anthropic reference) and `mcp-server-http-request` (community). Embargo expires 2026-08-10.

---

## [0.1.0a0] — 2026-05-10

### Added

- Initial scaffolding for analyzer, harness, classifier, calibration, and scenarios packages.
- 6 analyzer rules (S-001, S-002, S-003, S-005, S-006, S-007).
- 6 dynamic scenarios (D-001 through D-006).
- Capability classifier with Layer 1 (lexical) detection across 8 capability tags and 8 parameter roles.
- HTTP canary server for dynamic-scenario SSRF probes.
- Proxy-mode harness with stub and Anthropic agent drivers.
- Mock MCP server for plumbing tests.
- 76 tests across analyzer, classifier, harness, and calibration packages.
- Initial calibration corpus of 5 labeled targets (3 verified by capture, 2 best-effort from public docs).
