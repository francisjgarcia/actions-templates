# Architecture

## Overview

`actions-templates` centralizes CI/CD logic for all of Francis J. García's
containerized repositories. Consumer repositories contain only a minimal caller
workflow; all pipeline logic lives here.

```
┌─────────────────────────────────────┐
│          Consumer repository         │
│   .github/workflows/cicd.yml        │
│   (minimal, ~80 lines)              │
└────────────────┬────────────────────┘
                 │ calls via workflow_call
                 ▼
┌─────────────────────────────────────┐
│         actions-templates            │
│   .github/workflows/wf-*.yml        │
│   actions/calculate-tag/            │
└─────────────────────────────────────┘
```

---

## Pipeline flow

### Trigger: push to `main` in a consumer repository

```
setup
  │
  ├── calculate-tag (composite action)
  │     reads commits since last tag
  │     applies conventional commits semver rules
  │     outputs: tag_version, bump_type, changelog_entries
  │
  │   if tag_version == '' → all jobs skip (chore/docs/ci commits)
  │
  ├── build
  │     docker buildx with layer cache (type=gha)
  │     OCI labels from setup outputs
  │     uploads docker-image.tar artifact (retention: 1 day)
  │
  ├── tests (parallel with scan)
  │     downloads artifact
  │     runs inside built image via docker run --rm
  │     supports: flake8, pytest, custom command
  │
  ├── scan (parallel with tests)
  │     downloads artifact
  │     Anchore scan → SARIF format
  │     uploads to GitHub Security tab (code-scanning)
  │     second pass with table format for fail-build logic
  │
  ├── push (after tests AND scan pass)
  │     ghcr.io login with GITHUB_TOKEN
  │     pushes semver tag + latest
  │
  ├── release (after push)
  │     generates AI summary via GitHub API
  │     updates CHANGELOG.md (grouped by commit type)
  │     commits via GitHub App token (bypasses branch protection)
  │     creates GitHub release with AI summary + commit changelog
  │
  └── deploy (after push AND release)
        receives container config as JSON array
        receives secrets as APP_ENV_VARS JSON
        creates docker networks if missing
        pulls images, removes old containers, runs new ones
        connects containers to extra networks
        cleans old images
```

### Trigger: pull request to `main`

```
lint
  ├── actionlint   → validates GitHub Actions syntax and expressions
  └── yamllint     → validates YAML formatting

pr-validation
  └── checks PR title matches conventional commit format
      blocks merge if invalid
```

---

## Reusable workflows

| Workflow | Purpose | Key inputs |
| --- | --- | --- |
| `wf-setup.yml` | Semver calculation + metadata | — |
| `wf-build.yml` | Docker image build | `tag_version`, `dockerfile`, `platform` |
| `wf-tests.yml` | Run tests in image | `flake8_enabled`, `pytest_enabled`, `test_command` |
| `wf-scan.yml` | CVE vulnerability scan | `severity_cutoff`, `fail_on_critical`, `only_fixed` |
| `wf-push.yml` | Push to ghcr.io | `tag_version` |
| `wf-release.yml` | Changelog + GitHub release | `tag_version`, `bump_type`, `changelog_entries` |
| `wf-deploy.yml` | Multi-container deploy | `containers` (JSON), `APP_ENV_VARS` (secret) |
| `wf-lint.yml` | actionlint + yamllint | `yamllint_paths` |
| `wf-pr-validation.yml` | PR title validation | `allowed_types`, `max_title_length` |

---

## Composite actions

| Action | Purpose |
| --- | --- |
| `actions/calculate-tag` | Reads git log, classifies commits, computes next semver tag and grouped changelog |

---

## Semver automation

The `calculate-tag` action processes all commits since the last `v*` tag.
The highest bump in the commit range wins.

```
BREAKING CHANGE / feat! → MAJOR (x.0.0) resets minor and patch
feat                    → MINOR (0.x.0) resets patch
fix / perf / refactor   → PATCH (0.0.x)
docs / chore / ci / test → none (pipeline exits after setup)
```

Changelog entries are grouped by type in the release body:
- ⚠️ Breaking changes
- ✨ New features
- 🐛 Bug fixes
- ⚡ Performance improvements
- ♻️ Refactors
- 🔧 Other changes

---

## Secret and variable model

GitHub Actions has a fundamental limitation: `secrets.*` cannot be interpolated
inside `with:` input blocks of reusable workflow calls. Only `vars.*` can.

This repo solves it with a convention:

```
vars.*         → non-sensitive config (URLs, network names, memory limits)
                 interpolated in with: blocks of the caller cicd.yml

APP_ENV_VARS   → single JSON secret containing all sensitive env vars
                 passed via secrets: block to wf-deploy.yml
                 merged into container env at deploy time
```

Example `APP_ENV_VARS` value in consumer repo:
```json
{
  "API_KEY": "abc123",
  "DB_PASSWORD": "secret",
  "EXTERNAL_TOKEN": "xyz"
}
```

---

## Branch strategy

**Trunk Based Development** — single `main` branch.

```
main (protected)
  ↑
  └── short-lived feature branches
        merged via squash PR
        PR title becomes the single commit message on main
        → drives semver automation
```

Branch protection rules on `main`:
- PR required (no direct push for humans)
- `Lint / Lint workflows` must pass
- `Lint / Lint YAML` must pass
- `PR validation / Validate PR title` must pass
- GitHub App (`actions-templates-bot`) has bypass to push `CHANGELOG.md`

---

## GitHub App

A custom GitHub App (`actions-templates-bot`) is used by `wf-release.yml`
to commit and push `CHANGELOG.md` directly to `main` bypassing branch protection.

This is necessary because the release job needs to update the changelog file
on `main` as part of the release process, but `GITHUB_TOKEN` cannot bypass
branch protection rules.

The app is added to the bypass list of the `main` ruleset.

---

## This repo's own pipeline

`actions-templates` uses its own reusable workflows with local `./` references:

```yaml
uses: ./.github/workflows/wf-setup.yml    # local ref
uses: ./.github/workflows/wf-release.yml  # local ref
```

Consumer repositories use remote refs:
```yaml
uses: francisjgarcia/actions-templates/.github/workflows/wf-setup.yml@main
```

The repo's own pipeline only runs `setup` and `release` — no Docker build,
no tests, no scan, no deploy. This repo contains only YAML, not application code.
