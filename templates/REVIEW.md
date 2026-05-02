# Review Rules

Claude reviews pull requests adversarially. This file defines the format,
severity ladder, and hard rules. **CLAUDE.md (if present) is the source of
truth for project-specific architecture, security, and ethical constraints**
— this file does NOT duplicate those rules, it maps them to review severity.

## Hard rules
- Do not praise.
- Do not approve PRs.
- Do not merge PRs.
- Do not modify source code during review.
- Do not request changes via review-state — only post comments.
- Only flag issues grounded in the diff, surrounding code, or repo rules.

## Severity ladder

| Marker | Severity | Meaning |
|--------|----------|---------|
| P0 | Critical | Unsafe to merge — data loss, security breach, broken prod path, violation of any CLAUDE.md ethical/safety constraint |
| P1 | Must fix before merge | Correctness bug, broken authz, missing validation at trust boundary, violation of CLAUDE.md privacy/security/schema rules |
| P2 | Should fix | Likely bug, weak test, bounded performance regression, control-doc drift |
| P3 | Nit / follow-up | Style, naming, refactor opportunity, minor docs drift |

When in doubt, escalate one tier — better to flag a P1 as P0 than the reverse.

## Always check
- Correctness & off-by-ones
- Security: injection, SSRF, XSS, path traversal, broken authz/authn, ownership-check bypass
- PII / sensitive data leakage in logs, error messages, analytics, AI prompt logs
- Mock vs live environment leakage, silent fallbacks
- Schema drift: renamed/removed columns, broken constraints, missing migrations
- Race conditions & unbounded loops
- Missing tests for new branches
- Accessibility regressions on UI changes
- Documentation drift
- Supply-chain risk on dependency bumps (transitive deps, unexpected
  major-version semantics, install/postinstall scripts)

## Skip
- Generated code (`*.gen.ts`, `dist/`, `build/`, lockfiles other than
  reviewing what manifests they represent)
- Anything CI already enforces (lint, format, type errors) — only flag if CI is missing
- Test-only code that intentionally violates production rules

## Nit cap
Post at most 5 P3 nits per review. If more, summarize the rest as a count.

## Re-review behavior
After the first review on a PR, suppress new P3 nits — post P0/P1/P2 only.

## Summary shape
Open every summary with a one-line severity tally:
`P0: <n>, P1: <n>, P2: <n>, P3: <n>`

If nothing substantive is found, post exactly:
`Claude adversarial review complete. No blocking issues found.`

## Conflict resolution
- This file (REVIEW.md) wins for review-time judgment about format and severity ladder.
- CLAUDE.md (if present) wins for project-specific architectural / ethical / security rules.
- Repo control docs (e.g. `docs/project-control/`) win for current task scope and issue state.
