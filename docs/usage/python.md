# Python usage guide

This guide shows how to use `actions-templates` in a Python project
packaged as a Docker container.

## Prerequisites

- A `docker/Dockerfile` in your repository
- Python source code under `src/`
- Tests under `tests/` (optional)

## Recommended repository structure

```
my-python-app/
├── .github/
│   └── workflows/
│       └── cicd.yml
├── docker/
│   └── Dockerfile
├── src/
│   ├── app.py
│   └── requirements.txt
├── tests/
│   └── test_app.py
```

## Dockerfile example
```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY src/ .

EXPOSE 5000

HEALTHCHECK --interval=1h --timeout=10s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1

CMD ["python", "app.py"]
```

## cicd.yml
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
      flake8_enabled:    true
      pytest_enabled:    true

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
