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
│       ├── cicd.yml
│       ├── lint.yml
│       └── pr-validation.yml
├── docker/
│   └── Dockerfile
├── src/
│   ├── main.go
│   └── go.mod
└── tests/
    └── main_test.go
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

## Workflow files

Use the ready-to-use example as your starting point and adjust it to your needs:

- **[examples/single-container/cicd.yml](../../examples/single-container/cicd.yml)** — full pipeline for a single container
- **[examples/multi-container/cicd.yml](../../examples/multi-container/cicd.yml)** — full pipeline for multiple containers

For Go repositories, use `test_command` in the `tests` job:

```yaml
  tests:
    name: Test application
    needs: [setup, build]
    if: needs.setup.outputs.tag_version != ''
    uses: francisjgarcia/actions-templates/.github/workflows/wf-tests.yml@main
    with:
      tag_version:       ${{ needs.setup.outputs.tag_version }}
      github_repository: ${{ github.repository }}
      test_command:      "go test ./..."
```

## Lint and PR validation

Add the following workflow files to your repository to enable linting and PR title validation.
These files do not need to be modified — they call the reusable workflows directly:

- **[.github/workflows/wf-lint.yml](../../.github/workflows/wf-lint.yml)**
- **[.github/workflows/wf-pr-validation.yml](../../.github/workflows/wf-pr-validation.yml)**

```yaml
# .github/workflows/lint.yml
name: Lint

on:
  pull_request:
    branches:
      - main
  push:
    branches:
      - main

jobs:
  lint:
    name: Lint workflows
    uses: francisjgarcia/actions-templates/.github/workflows/wf-lint.yml@main
    with:
      actionlint_paths: '.github/workflows/'
      yamllint_paths: '.github/workflows/'
```

```yaml
# .github/workflows/pr-validation.yml
name: PR validation

on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - edited
      - synchronize
      - reopened

jobs:
  pr-validation:
    name: Validate PR title
    uses: francisjgarcia/actions-templates/.github/workflows/wf-pr-validation.yml@main
```

## Available workflow inputs

For the full list of inputs and outputs of each reusable workflow, refer to the
[README](../../README.md#workflows) or directly to the workflow source files
under [.github/workflows/](../../.github/workflows/).
