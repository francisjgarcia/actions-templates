# Go usage guide

This guide shows how to use `actions-templates` in a Go project
packaged as a Docker container.

## Prerequisites

- A `docker/Dockerfile` in your repository
- Go source code under `src/`

## Recommended repository structure

```
my-go-app/
├── .github/
│   └── workflows/
│       └── cicd.yml
├── docker/
│   └── Dockerfile
├── src/
│   ├── main.go
│   └── go.mod
├── tests/
│   └── main_test.go
```

## Dockerfile example
```dockerfile
FROM golang:1.23-alpine AS builder

WORKDIR /build

COPY src/go.mod src/go.sum ./
RUN go mod download

COPY src/ .
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

FROM alpine:3.21

WORKDIR /app

COPY --from=builder /build/app .

EXPOSE 8080

HEALTHCHECK --interval=1h --timeout=10s --start-period=30s --retries=3 \
  CMD wget -qO- http://localhost:8080/health || exit 1

CMD ["./app"]
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
      test_command:      "go test ./..."

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
