# GitHub Copilot instructions

## Project type

This is a GitHub Actions template library. It contains only YAML workflow files
and composite action definitions. There is no application code, no package.json,
no requirements.txt, and no runtime.

---

## Core concepts to understand

### Reusable workflows (`wf-*.yml`)
Called by consumer repositories via `workflow_call`. They appear in
`.github/workflows/` and are the primary deliverable of this repo.

### Composite actions (`actions/*/action.yml`)
Reusable step sequences called with `uses:` inside other workflows.
Currently only `actions/calculate-tag/action.yml` exists.

### Consumer pattern
A consumer repository has a minimal `.github/workflows/cicd.yml` that calls
these workflows. When suggesting changes, always think about how the change
affects consumers.

---

## Pipeline order

When working on this repo, understand the dependency chain:

```
setup → build → tests ──→ push → release → deploy
              → scan  ──↗
```

`wf-setup` is the gate — if it outputs an empty `tag_version`, all other jobs
skip automatically via `if: inputs.tag_version != ''`.

---

## Commit message format (required)

All commits must follow Conventional Commits. The type determines the semver bump:

| Commit | Result |
| --- | --- |
| `feat!:` or `BREAKING CHANGE:` | Major release |
| `feat:` | Minor release |
| `fix:` `perf:` `refactor:` | Patch release |
| `docs:` `chore:` `ci:` `test:` | No release |

---

## Strict rules for suggestions

### Never suggest
- Adding `on: push` or `on: schedule` to `wf-*.yml` files
- Declaring `GITHUB_TOKEN` in `workflow_call` secrets blocks
- Using `secrets.*` inside `with:` input blocks
- Using `actions/checkout@main` or any `@latest` action ref
- Direct interpolation of `${{ github.event.*.title }}` in `run:` scripts
- Editing `CHANGELOG.md` directly
- Adding a `development` branch or `gitflow` patterns

### Always suggest
- Pinned version tags for actions (`@v4`, `@v3.6.1`)
- `env:` blocks to pass external values to bash scripts
- `if: inputs.tag_version != ''` guard on every job in reusable workflows
- `retention-days: 1` on docker image artifacts
- Step summaries (`$GITHUB_STEP_SUMMARY`) on every job

---

## YAML formatting

- Indent: 2 spaces
- No `---` document start
- Max line length: 180 characters
- Align values in `with:` blocks for readability:

```yaml
with:
  tag_version:       ${{ needs.setup.outputs.tag_version }}
  github_repository: ${{ github.repository }}
  author_name:       ${{ needs.setup.outputs.author_name }}
```

---

## When modifying a reusable workflow

If you add, rename, or remove an input/output, also update:
1. `README.md` — the inputs/outputs table for that workflow
2. `docs/usage/*.md` — if the consumer calling pattern changes
3. `examples/*/cicd.yml` — if new required inputs are added

---

## Secret handling pattern

Sensitive container env vars go in a single `APP_ENV_VARS` JSON secret:

```yaml
# In consumer cicd.yml — correct pattern
secrets:
  APP_ENV_VARS: ${{ secrets.APP_ENV_VARS }}

# Inside wf-deploy.yml — merged at deploy time via jq
```

Non-sensitive config goes in `vars.*` and can be interpolated in `with:` blocks.

---

## Local vs remote workflow refs

This repo's own workflows use local refs:
```yaml
uses: ./.github/workflows/wf-setup.yml
```

Consumer repositories use remote refs:
```yaml
uses: francisjgarcia/actions-templates/.github/workflows/wf-setup.yml@main
```

Never suggest remote refs inside this repository's own workflow files.

---

## Language

- All code, YAML, variables, comments, documentation: **English**
- The maintainer communicates in **Spanish** — respond accordingly if asked
