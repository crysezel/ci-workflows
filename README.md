# crysezel/ci-workflows

Centralized GitHub Actions reusable workflows for repos owned by `crysezel`.

## What's here

| File | Purpose |
|---|---|
| `.github/workflows/claude-review-reusable.yml` | Adversarial Claude PR review — called via `workflow_call` from consuming repos |
| `.github/workflows/review-gate-reusable.yml` | Merge gate: proves Codex review actually happened, and was clean, against the current head |
| `templates/REVIEW.md` | Generic adversarial-review rulebook to copy into a consuming repo's root |

## How a consuming repo opts in

1. Add this stub to the consuming repo at `.github/workflows/claude-review.yml`:

   ```yaml
   name: Claude Adversarial PR Review

   on:
     pull_request:
       types: [opened, synchronize, reopened, ready_for_review]

   jobs:
     review:
       uses: crysezel/ci-workflows/.github/workflows/claude-review-reusable.yml@main
       secrets:
         CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
   ```

2. Copy `templates/REVIEW.md` to the consuming repo's root as `REVIEW.md`.

3. Set the OAuth secret in the consuming repo:

   ```bash
   gh secret set CLAUDE_CODE_OAUTH_TOKEN -R crysezel/<repo>
   # paste the token from `claude setup-token`
   ```

4. Open a draft PR with the stub + REVIEW.md. After merge, every PR gets reviewed.

## Review Gate — how a consuming repo opts in

Add this stub at `.github/workflows/review-gate.yml`:

```yaml
name: Review Gate

on:
  pull_request:
    types: [opened, reopened, ready_for_review, synchronize, edited]
    branches: [main]

jobs:
  review-gate:
    # REQUIRED. These repos have default_workflow_permissions=read, and a called
    # workflow that declares its own top-level `permissions:` fails the run at
    # startup (startup_failure, no jobs, no annotation). The caller must grant
    # them here instead.
    permissions:
      contents: read
      pull-requests: read
    uses: crysezel/ci-workflows/.github/workflows/review-gate-reusable.yml@main
    with:
      policy_doc: "docs/review-gate.md"
      sensitive_paths: |
        ^\.github/
        ^supabase/(migrations|functions)/
```

Put the receipt in the PR description as a raw HTML comment, **not** inside a code fence - fenced blocks are stripped so documentation examples are not mistaken for real receipts:

```
<!-- review-gate
status: CLEAN
head_sha: <current PR head SHA>
verdict: <one line, e.g. "no blockers after 2 fix rounds">
-->
```

`CLEAN` is only accepted when Codex has actually reviewed **that exact head SHA**
and has **no unresolved blocking threads** on it — so writing `CLEAN` by hand
does not pass. Push a fix and the receipt goes stale, forcing re-review.

When Codex is genuinely unavailable, use `status: OVERRIDE` with `reason:` and
`approved_by:` — **and** comment `review-gate-override <head_sha>` on the PR from
an allowlisted account. The receipt's `approved_by` must name that same account.

### Inputs

| Input | Default | Purpose |
|---|---|---|
| `sensitive_paths` | `.github/`, `AGENTS/CLAUDE/REVIEW.md`, supabase migrations/functions, auth-ish filenames | Newline-separated EREs. A match puts the PR in the strict lane where **P2 also blocks**. Over-match rather than under-match. |
| `allowed_approvers` | `crysezel` | Comma-separated logins that may authorize an OVERRIDE. |
| `policy_doc` | generic text | Cited in error messages. |
| `codex_bot_login` | `chatgpt-codex-connector` | REST appends `[bot]`, GraphQL does not — matched by prefix. |
| `codex_required` | `true` | Set `false` only where Codex review is not enabled yet. CLEAN then rests on the receipt alone and proves nothing; the run warns loudly. |

### Severity lanes

| Severity | Routine change | Sensitive change |
|---|---|---|
| P0 / P1 | blocks | blocks |
| P2 | advisory | **blocks** |
| P3 | advisory | advisory |

Sensitivity is derived from the PR's changed files, never self-declared, so a
session cannot relabel a migration PR as routine.

### Prerequisites

- **Codex review must be enabled on the repo.** Without it, no `CLEAN` receipt
  can ever be satisfied and the gate is permanently red. Either enable Codex, or
  start with `codex_required: false` and tighten later.
- A `request-codex-review.yml` (or equivalent) that asks Codex to review.

### Known ceiling

In a repo where one account is author, approver and thread-resolver, no check
that account controls can bind it. This gate makes a bypass **explicit,
deliberate and auditable — not impossible.** It is a discipline aid, not a
security boundary. For a real boundary you need a second identity.

## Versioning

The stub references `@main`. For continuous delivery (you want every consumer
to pick up changes immediately), keep `@main`. For pinned consumers, switch
to `@v1.0.0` style tags in the stub.

To update review behavior for ALL consumers:
- Edit `.github/workflows/claude-review-reusable.yml`
- (Optional) tag a new release if consumers pin tags

## Cost / quota

- OAuth route uses your Anthropic Max plan quota
- For 11 repos with active PR streams, monitor weekly token usage at
  https://console.anthropic.com — switch consumers to `ANTHROPIC_API_KEY`
  if Max quota becomes a bottleneck

## Excluded from rollout (intentionally)

- `crysezel/solas-dashboard` — gated on PR #205 (test fixes) and PR #172 (SHA-pin) clearing first
- `crysezel/work-connector` — has its own standalone copy (can migrate to stub later)

## Related

- Anthropic Claude Code action: https://github.com/anthropics/claude-code-action
- GitHub reusable workflow docs: https://docs.github.com/en/actions/using-workflows/reusing-workflows
