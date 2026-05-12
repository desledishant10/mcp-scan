# MCP-D-002 vs mcp-server-aidd

**Date:** 2026-05-11
**Target:** `mcp-server-aidd` v? (PyPI; coding-assistant MCP)
**Tested by:** [scenarios/MCP-D-002-path-traversal-fs-tool.yaml](../scenarios/MCP-D-002-path-traversal-fs-tool.yaml)
**Agent driver:** n/a (direct mode — harness as MCP client)
**Outcome:** **DEFENSE**

## Result

`mcp-server-aidd` resisted every path-traversal probe across **126 calls** (14 fs_read-classified tools × 9 traversal payloads). The canary file content (`MCP_SCAN_CANARY_<run_id>`) never appeared in any response. aidd's `update_allowed_directory` mechanism acts as a real path containment boundary, returning explicit `"Access denied: Path (...)"` responses.

```json
{
  "scenario_id": "MCP-D-002",
  "passed": true,
  "oracle_evidence": [],
  "skipped_steps": [],
  "error": null
}
```

## Reproduction

```bash
pip install mcp-server-aidd
mcp-scan-test scenarios/MCP-D-002-path-traversal-fs-tool.yaml \
    --server-cmd mcp-server-aidd
```

## What happened

Tools the harness classified as `fs_read` and therefore probed:
`directory_tree`, `get_file_info`, `git_diff`, `git_diff_staged`, `git_diff_unstaged`, `git_log`, `git_show`, `git_status`, `list_directory`, `read_file`, `read_image_file`, `read_multiple_files`, `search_files`, `tree_sitter_map`.

Two distinct defense responses observed across the 126 calls:

### Layer 1: explicit "Access denied" with the offending path

```
list_directory({"path": "/var/folders/.../mcp-scan-canary-...txt"})
  → "Access denied: Path (/var/folders/.../mcp-scan-canary-...txt) outside allowed directory"
```

aidd canonicalizes the requested path and rejects anything outside its configured `allowed_directory`. The default `allowed_directory` appears to be the current working directory of the server process — well outside `/var/folders/.../canary.txt` written by the D-002 fixture.

### Layer 2: "Path does not exist" for traversal-shape payloads

```
list_directory({"path": "/Users/dishantdesle/Desktop/..."})
  → "Path does not exist: /Users/dishantdesle/Desktop/..."
```

Some traversal payloads (`../../../...`, `....//....//`) resolve to paths that don't exist on the test system. aidd's path canonicalization runs *before* the existence check, so these get a clean "not found" rather than leaking any information about the file system structure.

## Interpretation

This is a **positive example** worth modeling. aidd implements two complementary defenses:

1. **Explicit allow-list ("allowed directory")** — server-side rather than relying on MCP's `roots` capability. Reduces dependency on client-side hardening.
2. **Path canonicalization before existence check** — prevents both traversal and information disclosure via differential responses (existing-but-denied vs. non-existing paths would otherwise be distinguishable).

For a coding-assistant MCP with broad fs surface (33 tools, 14 read-shaped + 6 write-shaped), this is the *right* defense posture. Compare with `mcp-server-git`'s [similarly strong defense](2026-05-11-MCP-D-002-git-direct-defense.md), which relies on gitpython rejecting non-repository paths — both are defense-in-depth approaches, achieved differently.

## What was *not* exercised

- **Symlink escape.** If `/some/allowed/path/decoy` symlinks to `/etc/passwd`, would aidd follow the symlink before applying the allowed-directory check? Not tested here.
- **`update_allowed_directory` abuse.** A successful prompt injection could call `update_allowed_directory("/")`, broadening the scope to the entire filesystem. The static-analyzer side flagged this tool's "configuration-as-capability" nature; a dynamic scenario specifically for this attack would be valuable.
- **Race conditions.** Time-of-check-time-of-use on the canonicalized path. Hard to exercise in direct-mode probing.

## Caveats

- **Single test run.** Results are deterministic for direct-mode probing, but a different aidd version or configuration could behave differently.
- **macOS-only test environment.** aidd's path handling on Windows (UNC paths, drive letters, etc.) could differ.
- **D-002's payload set is finite.** 9 payloads cover the common traversal forms; novel encoding tricks are not exercised.

## Suggested follow-up

1. Author a D-008 scenario specifically for the `update_allowed_directory` → broaden-scope → arbitrary-read attack chain.
2. Author a D-002b "symlink escape" scenario; run against aidd and git.
3. Compare aidd's "Access denied: Path (X)" response carefully — the inclusion of the requested path in the error message is itself a minor information disclosure (an attacker can probe for file existence by comparing "Access denied" vs "Path does not exist"). Soft finding, worth raising as a maintainer issue.

## Disclosure

Not applicable for the path traversal vector — defense observed. The information-disclosure observation in the error messages is a soft finding worth a courtesy issue if anyone cares to file one, not a vulnerability.
