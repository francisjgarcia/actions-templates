# Conventions

## Language

| Context | Language |
| --- | --- |
| Code, YAML, variables, comments | English |
| Documentation (README, guides) | English |
| Commit messages and PR titles | English |
| Conversation with maintainer | Spanish |

---

## Conventional Commits

All commit messages and PR titles must follow this exact format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

### Types and semver impact

| Type | Impact | Use for |
| --- | --- | --- |
| `feat!` | MAJOR | Breaking change in workflow inputs/outputs/behavior |
| `feat` | MINOR | New workflow, new action, new optional or required input |
| `fix` | PATCH | Bug fix in workflow or action logic |
| `perf` | PATCH | Performance improvement (e.g. adding cache) |
| `refactor` | PATCH | Code cleanup without behavior change |
| `docs` | none | README, guides, comments |
| `chore` | none | Dependency updates, maintenance |
| `ci` | none | Changes to CI configuration |
| `test` | none | Adding or fixing tests |
| `style` | none | Formatting, whitespace |

### Breaking changes

Use `!` after the type or add a `BREAKING CHANGE:` footer:

```
feat!: rename input tag_version to version in all reusable workflows

BREAKING CHANGE: the input `tag_version` has been renamed to `version`.
Update all consumer cicd.yml files.
```

### Scope

Optional. Use the affected workflow or action name without the `wf-` prefix:

```
fix(deploy): correct healthcheck flag quoting in bash array
feat(scan): add severity_cutoff input to control minimum CVE level
docs(setup): clarify tag_version empty behavior in README
```

### Examples — correct

```
feat: add wf-notify-slack reusable workflow
fix(calculate-tag): handle repos with zero tags on first run
perf(build): enable github actions layer cache with type=gha
refactor(deploy): replace string flags with bash array for docker run
docs: update python usage guide with flake8 configuration example
chore: update softprops/action-gh-release to v2.1.0
ci: remove redundant push trigger from lint workflow
```

### Examples — incorrect

```
Added new workflow          # no type
feat: Added new workflow    # description should not start with capital
FIX: broken deploy          # type must be lowercase
feat : add slack            # no space before colon
update stuff                # no type, vague description
```

---

## YAML style

All YAML files follow these rules (enforced by yamllint):

```yaml
# Indentation: 2 spaces
# No document start (---)
# Max line length: 180 characters
# No trailing whitespace
# Final newline required

# Colons: alignment allowed for readability
with:
  tag_version:       ${{ needs.setup.outputs.tag_version }}
  github_repository: ${{ github.repository }}
  author_name:       ${{ needs.setup.outputs.author_name }}
```

### GitHub Actions expressions

Pass external values through `env:` blocks — never interpolate directly in scripts:

```yaml
# Correct
- name: Use external value
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: echo "$PR_TITLE"

# Incorrect — security risk (script injection)
- name: Use external value
  run: echo "${{ github.event.pull_request.title }}"
```

### Action version pinning

Always use pinned version tags for third-party actions:

```yaml
# Correct
uses: actions/checkout@v4
uses: docker/build-push-action@v6.7.0
uses: anchore/scan-action@v3

# Incorrect
uses: actions/checkout@main
uses: docker/build-push-action@latest
```

---

## Reusable workflow conventions

### Naming

- Reusable workflows: `wf-<purpose>.yml` (e.g. `wf-build.yml`, `wf-deploy.yml`)
- Caller workflows in this repo: `<purpose>.yml` (e.g. `cicd.yml`, `lint.yml`)
- Composite actions: directory name describes purpose (`calculate-tag`, `setup-node`)

### Required structure for all `wf-*.yml`

```yaml
name: <Name> (Reusable)

on:
  workflow_call:
    inputs:
      tag_version:
        required: true
        type: string
      # ... more inputs

jobs:
  <job_name>:
    runs-on: ubuntu-latest
    environment: production
    if: inputs.tag_version != ''   # skip if no release needed
    permissions:
      contents: read               # minimal permissions
```

### Input defaults

All optional inputs must have explicit `default:` values:

```yaml
inputs:
  dockerfile:
    required: false
    type: string
    default: docker/Dockerfile
  platform:
    required: false
    type: string
    default: 'linux/amd64'
```

### `if` guards

Every job in every `wf-*.yml` must guard on `tag_version`:

```yaml
if: inputs.tag_version != ''
```

Every job in `cicd.yml` of consumer repos must mirror this guard:

```yaml
if: needs.setup.outputs.tag_version != ''
```

### Secrets

```yaml
# Never declare GITHUB_TOKEN in workflow_call — it is automatic
# Only declare secrets that consumers must explicitly pass:
secrets:
  APP_ENV_VARS:
    required: false
    description: 'JSON object with sensitive env vars'
```

---

## Docker conventions

### Container configuration

Consumer repos pass container config as a JSON array to `wf-deploy.yml`:

```json
[
  {
    "name": "my-app",
    "image": "ghcr.io/org/repo:v1.0.0",
    "network": "proxy",
    "healthcheck_url": "http://localhost:5000/health",
    "memory_limit": "512m",
    "memory_reservation": "256m",
    "restart_policy": "always",
    "ports": ["8080:80"],
    "volumes": ["/data:/app/data"],
    "extra_networks": ["internal"],
    "env_vars": {
      "NON_SENSITIVE_VAR": "value"
    }
  }
]
```

### Sensitive env vars

All sensitive variables (API keys, passwords, tokens) go in `APP_ENV_VARS`:

```json
{
  "API_KEY": "secret",
  "DB_PASSWORD": "secret"
}
```

Never put secrets inside the `containers` JSON — that value appears in logs.

### Dockerfile location

Always at `docker/Dockerfile` within the consumer repository.

---

## Branch and PR conventions

### Branch naming

```
<type>/<short-description>
```

Examples:
```
feat/add-wf-notify-slack
fix/deploy-healthcheck-quoting
docs/update-python-guide
chore/update-anchore-action
```

### PR title

Must match the conventional commit format — this becomes the squash commit message
on `main` and directly drives the semver automation.

```
feat(deploy): add dns configuration support for containers
```

### Merge strategy

Squash merge only. One PR = one commit on `main`.

---

## Step summaries

Every job must write a `$GITHUB_STEP_SUMMARY` section. Use tables where appropriate:

```yaml
- name: Build summary
  run: |
    echo "## Build summary" >> $GITHUB_STEP_SUMMARY
    echo "| Input | Value |" >> $GITHUB_STEP_SUMMARY
    echo "| --- | --- |" >> $GITHUB_STEP_SUMMARY
    echo "| Image | \`ghcr.io/...:$TAG\` |" >> $GITHUB_STEP_SUMMARY
```
