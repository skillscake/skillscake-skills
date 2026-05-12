---
name: terraform-pr-review
description: Review Terraform module PRs for breaking changes and recommend the correct Conventional Commits prefix. Use when a PR touches .tf files in a shared Terraform module repo — in CI pipeline or manual review. Posts a structured review comment via gh.
---

## Workflow

### 1. Get PR context

```bash
gh pr view $PR_NUMBER --repo $REPO --json title,body,baseRefName,headRefName,files
```

### 2. Filter to relevant files

Only `.tf` files matter. Skip everything else:
- `.md`, `.yaml`, `.json`, `.hcl`, CI configs, lockfiles
- Files under `examples/` — these are documentation, not the module interface
- Files under `tests/` or `test/` — test fixtures

If no `.tf` files changed, post a review with `chore:` and stop.

### 3. Diff changed `.tf` files

For each `.tf` file that survived filtering, diff against the target branch:

```bash
gh pr diff $PR_NUMBER --repo $REPO -- <file>
```

### 4. Classify each change

Read [breaking-changes.md](references/breaking-changes.md) for the full classification guide. Apply every rule to each diff hunk. For changed default values on existing optional inputs — the judgment-zone case — walk the decision tree in that reference.

Core principle: a change is breaking if an existing caller updating the module ref would have their plan fail or infrastructure destroyed/recreated without an explicit migration.

**Sub-module rule**: When a file under `modules/<name>/` changes, evaluate from that sub-module's own interface — its `variables.tf`, `outputs.tf`, and resources. Do not flag root-only concerns against a sub-module.

**Examples-only changes**: If every changed `.tf` file is under `examples/`, the PR is non-breaking regardless of content. Output `docs:` or `chore:`.

### 5. Determine the prefix

| Change type | No breaking changes | Has breaking changes |
|---|---|---|
| New feature/adds capability | `feat:` | `feat!:` |
| Bug fix | `fix:` | `fix!:` |
| Docs/refactor/chore only | `chore:` or `docs:` | Rare — flag for manual review |

### 6. Post the review

```bash
gh pr review $PR_NUMBER --repo $REPO --body "$REVIEW_BODY"
```

Use `--request-changes` when breaking changes are found without the `!` prefix in the PR title.

### Review body template

```
## Conventional Commit Check

**Recommended prefix: `feat!:`**

### Breaking changes
- `variables.tf`: removed required input `instance_class` (line 42)
- `outputs.tf`: removed output `db_endpoint` (line 15)

### Non-breaking changes
- `main.tf`: added new `aws_db_parameter_group` resource
- `variables.tf`: added optional input `backup_retention_period` (default: 30)

### Ignored (non-tf)
- `README.md`
- `.github/workflows/ci.yaml`

---
reviewed by terraform-pr-review
```

## CI pipeline usage

In CI, the skill reads these environment variables:

| Variable | Source |
|---|---|
| `PR_NUMBER` | `${{ github.event.pull_request.number }}` |
| `REPO` | `${{ github.repository }}` |
| `GITHUB_TOKEN` | Already set by CI; `gh` uses it automatically |

The skill only reads the PR and posts a review. It must not modify commits, push, or merge.

## Gotchas

- **`force_new` is not always visible in the diff.** Adding a `name` or changing a resource key attribute forces recreation. Flag any attribute change that the provider docs mark as "Forces new resource."
- **Renamed resources show as delete + create.** This is always breaking unless the PR includes a `moved` block to migrate state.
- **`sensitive = true` added to an output** makes it unusable in downstream `output` references. Breaking for callers who consume that output.
- **`terraform-docs` changes are noise.** If the repo auto-generates docs, skip changes to generated sections.
- **`moved` blocks mitigate renames.** If a `moved` block maps the old address to the new one, the rename is non-breaking.
- **Provider version constraint changes:** tightening `required_providers` version constraints is breaking. Loosening them is not.
