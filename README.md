<div align="center">

# actions-templates

**A collection of reusable GitHub Actions workflows and composite actions
for building, testing, scanning, pushing and releasing containerized applications.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub release](https://img.shields.io/github/v/release/francisjgarcia/actions-templates)](https://github.com/francisjgarcia/actions-templates/releases)
[![Last commit](https://img.shields.io/github/last-commit/francisjgarcia/actions-templates)](https://github.com/francisjgarcia/actions-templates/commits/main)
[![GitHub stars](https://img.shields.io/github/stars/francisjgarcia/actions-templates)](https://github.com/francisjgarcia/actions-templates/stargazers)
[![GitHub issues](https://img.shields.io/github/issues/francisjgarcia/actions-templates)](https://github.com/francisjgarcia/actions-templates/issues)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)

[Documentation](#documentation) · [Quick start](#quick-start) · [Workflows](#workflows) · [Contributing](CONTRIBUTING.md)

</div>

---

## Overview

`actions-templates` is a centralized library of reusable GitHub Actions workflows
and composite actions designed to standardize CI/CD pipelines across multiple repositories.

Instead of duplicating pipeline logic in every project, consumer repositories
call these templates with a few lines of YAML and get a full CI/CD pipeline
with automatic semantic versioning based on [Conventional Commits](https://www.conventionalcommits.org/).

### Key features

- **Automatic semantic versioning** — `feat:` bumps minor, `fix:` bumps patch, `feat!:` bumps major
- **Docker image build** with OCI labels and GitHub Actions cache
- **Vulnerability scanning** with Anchore and SARIF upload to GitHub Security tab
- **Automatic changelog** generation and GitHub releases
- **Multi-container deploy** support via JSON configuration
- **Language agnostic** — works with Python, Node.js, Go and any containerized app
- **Trunk Based Development** — single `main` branch, no `develop` branch needed

---

## Quick start

Add this file to your repository at `.github/workflows/cicd.yml`:
```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

permissions:
  contents: write
  packages: write
  security-events: write

jobs:
  setup:
    name: Setup environment
    if: github.event.head_commit.message != 'Initial commit'
    uses: francisjgarcia/actions-templates/.github/workflows/wf-setup.yml@main

  build:
    name: Build image
    needs: setup
    if: needs.setup.outputs.tag_version != ''
    uses: francisjgarcia/actions-templates/.github/workflows/wf-build.yml@main
    with:
      tag_version:       ${{ needs.setup.outputs.tag_version }}
      github_repository: ${{ github.repository }}
      author_name:       ${{ needs.setup.outputs.author_name }}
      author_email:      ${{ needs.setup.outputs.author_email }}
      repo_description:  ${{ needs.setup.outputs.repo_description }}
      created_at:        ${{ needs.setup.outputs.created_at }}

  tests:
    name: Test application
    needs: [setup, build]
    if: needs.setup.outputs.tag_version != ''
    uses: francisjgarcia/actions-templates/.github/workflows/wf-tests.yml@main
    with:
      tag_version:       ${{ needs.setup.outputs.tag_version }}
      github_repository: ${{ github.repository }}

  scan:
    name: Scan vulnerabilities
    needs: [setup, build]
    if: needs.setup.outputs.tag_version != ''
    uses: francisjgarcia/actions-templates/.github/workflows/wf-scan.yml@main
    with:
      tag_version:       ${{ needs.setup.outputs.tag_version }}
      github_repository: ${{ github.repository }}

  push:
    name: Push to registry
    needs: [setup, tests, scan]
    if: needs.setup.outputs.tag_version != ''
    uses: francisjgarcia/actions-templates/.github/workflows/wf-push.yml@main
    with:
      tag_version:       ${{ needs.setup.outputs.tag_version }}
      github_repository: ${{ github.repository }}

  release:
    name: Tag and release
    needs: [setup, push]
    if: needs.setup.outputs.tag_version != ''
    uses: francisjgarcia/actions-templates/.github/workflows/wf-release.yml@main
    with:
      tag_version:       ${{ needs.setup.outputs.tag_version }}
      bump_type:         ${{ needs.setup.outputs.bump_type }}
      github_repository: ${{ github.repository }}
      changelog_entries: ${{ needs.setup.outputs.changelog_entries }}
```

---

## Workflows

### `wf-setup.yml` — Setup environment

Calculates the next semantic version tag based on Conventional Commits,
and exposes repository metadata as outputs for downstream jobs.

| Output | Description |
| --- | --- |
| `tag_version` | Next semver tag (`v1.2.3`) or empty if no release needed |
| `bump_type` | `major`, `minor`, `patch` or `none` |
| `repo_name` | Repository name without org prefix |
| `repo_description` | Repository description from GitHub API |
| `author_name` | Name of the commit author |
| `author_email` | Email of the commit author |
| `created_at` | ISO timestamp of the run |
| `changelog_entries` | Commit list since last tag |

### `wf-build.yml` — Build Docker image

Builds a Docker image using `docker/build-push-action` with OCI labels,
GitHub Actions layer cache, and uploads the image tar as an artifact.

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Image tag |
| `github_repository` | ✅ | — | `org/repo` |
| `author_name` | ✅ | — | OCI label |
| `author_email` | ✅ | — | OCI label |
| `repo_description` | ❌ | `''` | OCI label |
| `created_at` | ✅ | — | OCI label |
| `dockerfile` | ❌ | `docker/Dockerfile` | Path to Dockerfile |
| `platform` | ❌ | `linux/amd64` | Target platform |

### `wf-tests.yml` — Run tests

Runs tests inside the built Docker image. Supports flake8, pytest and custom commands.

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Image tag |
| `github_repository` | ✅ | — | `org/repo` |
| `flake8_enabled` | ❌ | `false` | Enable flake8 linting |
| `pytest_enabled` | ❌ | `false` | Enable pytest |
| `test_command` | ❌ | `''` | Custom test command |

### `wf-scan.yml` — Vulnerability scan

Scans the Docker image with Anchore, uploads SARIF to the GitHub Security tab,
and generates a detailed summary.

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Image tag |
| `github_repository` | ✅ | — | `org/repo` |
| `fail_on_critical` | ❌ | `true` | Fail build on critical CVEs |
| `severity_cutoff` | ❌ | `critical` | Minimum severity to report |
| `only_fixed` | ❌ | `true` | Only report CVEs with a fix |

### `wf-push.yml` — Push to registry

Pushes the image to GitHub Container Registry (`ghcr.io`) with the semver tag and `latest`.

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Image tag |
| `github_repository` | ✅ | — | `org/repo` |

### `wf-release.yml` — Create release

Updates `CHANGELOG.md`, commits it, creates a git tag and a GitHub release.

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Release tag |
| `bump_type` | ✅ | — | `major`, `minor` or `patch` |
| `github_repository` | ✅ | — | `org/repo` |
| `changelog_entries` | ✅ | — | Formatted commit list |

### `wf-deploy.yml` — Deploy containers

Deploys one or more Docker containers on the runner host from a JSON configuration array.

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Image tag |
| `github_repository` | ✅ | — | `org/repo` |
| `containers` | ✅ | — | JSON array of container configs |

Container configuration schema:
```json
[
  {
    "name": "my-app",
    "image": "ghcr.io/org/repo:v1.0.0",
    "network": "proxy",
    "ports": ["8080:80"],
    "volumes": ["/data:/app/data"],
    "healthcheck_url": "http://localhost:8080/health",
    "memory_limit": "512m",
    "memory_reservation": "256m",
    "restart_policy": "always",
    "env_vars": {
      "ENV_VAR": "value"
    },
    "extra_networks": ["internal-network"]
  }
]
```

---

## Composite actions

### `actions/calculate-tag`

Analyzes all commits since the last tag and determines the next semver bump
following Conventional Commits specification.

| Bump | Trigger |
| --- | --- |
| `major` | `feat!:` or `BREAKING CHANGE:` footer |
| `minor` | `feat:` |
| `patch` | `fix:`, `perf:`, `refactor:` |
| `none` | `chore:`, `docs:`, `ci:`, `test:`, `style:` |

---

## Documentation

Detailed usage guides per language:

- [Python](docs/usage/python.md)
- [Node.js](docs/usage/nodejs.md)
- [Go](docs/usage/go.md)

Ready-to-use examples:

- [Single container](examples/single-container/cicd.yml)
- [Multi-container](examples/multi-container/cicd.yml)

---

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) first.

---

## License

[MIT](LICENSE) © [Francis J. García](https://www.francisjgarcia.es)
