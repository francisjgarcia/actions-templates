# Contributing to actions-templates

Thank you for your interest in contributing! This document explains how to
propose changes, report issues, and submit pull requests.

## Table of contents

- [Contributing to actions-templates](#contributing-to-actions-templates)
  - [Table of contents](#table-of-contents)
  - [Code of Conduct](#code-of-conduct)
  - [Getting started](#getting-started)
  - [How to contribute](#how-to-contribute)
  - [Conventional Commits](#conventional-commits)

---

## Code of Conduct

By participating in this project you agree to abide by the [Code of Conduct](CODE_OF_CONDUCT.md).

---

## Getting started

1. Fork the repository
2. Clone your fork:
```bash
   git clone https://github.com/YOUR_USERNAME/actions-templates.git
   cd actions-templates
```
3. Create a branch for your change (see [Branch strategy](#branch-strategy))

---

## How to contribute

- **Bug reports** → open a [Bug report issue](https://github.com/francisjgarcia/actions-templates/issues/new?template=0_bug_report.yml)
- **Feature requests** → open a [Feature request issue](https://github.com/francisjgarcia/actions-templates/issues/new?template=1_feature_request.yml)
- **Documentation fixes** → open a [Documentation update issue](https://github.com/francisjgarcia/actions-templates/issues/new?template=5_documentation_update.yml)
- **Questions** → use [GitHub Discussions](https://github.com/francisjgarcia/actions-templates/discussions)

---

## Conventional Commits

This project uses [Conventional Commits](https://www.conventionalcommits.org/) for all commit messages.
The format is:

```
<type>[optional scope]: <description>
[optional body]
[optional footer]
```

| Type | When to use |
| --- | --- |
| `feat` | A new workflow, action or capability |
| `fix` | A bug fix in a workflow or action |
| `perf` | A performance improvement |
| `refactor` | Code change that is not a fix or feature |
| `docs` | Documentation only changes |
| `test` | Adding or updating tests |
| `chore` | Maintenance tasks, dependency updates |
| `ci` | Changes to CI/CD configuration |

**Breaking changes** must append `!` after the type and include a `BREAKING CHANGE:` footer:

```
feat!: rename input tag_version to version
BREAKING CHANGE: the input parameter tag_version has been renamed to version
in all reusable workflows. Update your caller workflows accordingly.
```

> **Tip:** You can optionally install [commitlint](https://commitlint.js.org/) locally
> to validate your commit messages automatically:
> ```bash
> npm install --save-dev @commitlint/cli @commitlint/config-conventional
> echo "export default { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js
> npx husky init
> echo "npx commitlint --edit \$1" > .husky/commit-msg
> ```

---

## Branch strategy

This project follows **Trunk Based Development**. All changes go directly to `main` via short-lived branches:

```
main
└── fix/calculate-tag-empty-repo
└── feat/wf-notify-slack
└── docs/update-python-usage
```

Branch naming convention:

```
<type>/<short-description>
```

---

## Pull request process

1. Ensure your branch is up to date with `main`
2. Open a PR using the [pull request template](.github/PULL_REQUEST_TEMPLATE.md)
3. Link the related issue with `Closes #issue_number`
4. Wait for review from [@francisjgarcia](https://github.com/francisjgarcia)
5. Address any requested changes
6. Once approved, the PR will be squash-merged into `main`

---

## Local validation

Before opening a PR, validate your workflow syntax locally:
```bash
# Install actionlint
brew install actionlint       # macOS
sudo snap install actionlint  # Linux

# Run against all workflows
actionlint .github/workflows/*.yml
```

Also validate YAML syntax:
```bash
pip install yamllint
yamllint .github/workflows/
yamllint actions/
```
