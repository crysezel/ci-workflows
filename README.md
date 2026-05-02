# crysezel/ci-workflows

Centralized GitHub Actions reusable workflows for repos owned by `crysezel`.

## What's here

| File | Purpose |
|---|---|
| `.github/workflows/claude-review-reusable.yml` | Adversarial Claude PR review — called via `workflow_call` from consuming repos |
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
