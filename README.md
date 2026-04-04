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
- **Automatic changelog** generation and GitHub releases with AI-generated summary
- **Multi-container deploy** support via JSON configuration
- **PR title validation** enforcing Conventional Commits format
- **Workflow linting** with actionlint and yamllint
- **Language agnostic** — works with Python, Node.js, Go and any containerized app
- **Trunk Based Development** — single `main` branch, no `develop` branch needed

---

## Quick start

Copy one of the ready-to-use examples into your repository at `.github/workflows/`:

| Example | Description |
| --- | --- |
| [single-container/cicd.yml](examples/single-container/cicd.yml) | One container deployment |
| [multi-container/cicd.yml](examples/multi-container/cicd.yml) | Multiple containers deployment |

For language-specific guides see the [Documentation](#documentation) section below.

---

## Workflows

### `wf-setup.yml` — Setup environment

Calculates the next semantic version tag based on Conventional Commits,
and exposes repository metadata as outputs for downstream jobs.

> Source: [.github/workflows/wf-setup.yml](.github/workflows/wf-setup.yml)

| Output | Description |
| --- | --- |
| `tag_version` | Next semver tag (`v1.2.3`) or empty if no release needed |
| `bump_type` | `major`, `minor`, `patch` or `none` |
| `repo_name` | Repository name without org prefix |
| `repo_description` | Repository description from GitHub API |
| `author_name` | Name of the commit author |
| `author_email` | Email of the commit author |
| `created_at` | ISO timestamp of the run |
| `changelog_entries` | Grouped commit list since last tag |

---

### `wf-build.yml` — Build Docker image

Builds a Docker image using `docker/build-push-action` with OCI labels,
GitHub Actions layer cache, and uploads the image tar as an artifact.

> Source: [.github/workflows/wf-build.yml](.github/workflows/wf-build.yml)

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

---

### `wf-tests.yml` — Run tests

Runs tests inside the built Docker image. Supports flake8, pytest and custom commands.

> Source: [.github/workflows/wf-tests.yml](.github/workflows/wf-tests.yml)

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Image tag |
| `github_repository` | ✅ | — | `org/repo` |
| `flake8_enabled` | ❌ | `false` | Enable flake8 linting |
| `pytest_enabled` | ❌ | `false` | Enable pytest |
| `test_command` | ❌ | `''` | Custom test command |

---

### `wf-scan.yml` — Vulnerability scan

Scans the Docker image with Anchore, uploads SARIF to the GitHub Security tab,
and generates a detailed summary.

> Source: [.github/workflows/wf-scan.yml](.github/workflows/wf-scan.yml)

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Image tag |
| `github_repository` | ✅ | — | `org/repo` |
| `fail_on_critical` | ❌ | `true` | Fail build on critical CVEs |
| `severity_cutoff` | ❌ | `critical` | Minimum severity to report |
| `only_fixed` | ❌ | `true` | Only report CVEs with a fix |

---

### `wf-push.yml` — Push to registry

Pushes the image to GitHub Container Registry (`ghcr.io`) with the semver tag and `latest`.

> Source: [.github/workflows/wf-push.yml](.github/workflows/wf-push.yml)

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Image tag |
| `github_repository` | ✅ | — | `org/repo` |

---

### `wf-release.yml` — Create release

Updates `CHANGELOG.md`, commits it, creates a git tag and a GitHub release
with an AI-generated summary and a grouped commit changelog.

> Source: [.github/workflows/wf-release.yml](.github/workflows/wf-release.yml)

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `tag_version` | ✅ | — | Release tag |
| `bump_type` | ✅ | — | `major`, `minor` or `patch` |
| `github_repository` | ✅ | — | `org/repo` |
| `changelog_entries` | ✅ | — | Formatted commit list |

---

### `wf-deploy.yml` — Deploy containers

Deploys one or more Docker containers on the runner host from a JSON configuration array.

> Source: [.github/workflows/wf-deploy.yml](.github/workflows/wf-deploy.yml)

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

### `wf-lint.yml` — Lint workflows

Runs actionlint and yamllint against the repository workflows.

> Source: [.github/workflows/wf-lint.yml](.github/workflows/wf-lint.yml)

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `actionlint_paths` | ❌ | `.github/workflows/` | Paths for actionlint |
| `yamllint_paths` | ❌ | `.github/workflows/` | Paths for yamllint |
| `yamllint_max_line_length` | ❌ | `180` | Max line length |

---

### `wf-pr-validation.yml` — PR title validation

Validates that the PR title follows the Conventional Commits format.

> Source: [.github/workflows/wf-pr-validation.yml](.github/workflows/wf-pr-validation.yml)

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `allowed_types` | ❌ | `feat\|fix\|docs\|chore\|...` | Pipe-separated allowed types |
| `max_title_length` | ❌ | `100` | Max description length |

---

## Composite actions

### `actions/calculate-tag`

Analyzes all commits since the last tag and determines the next semver bump
following Conventional Commits specification.

> Source: [actions/calculate-tag/action.yml](actions/calculate-tag/action.yml)

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
