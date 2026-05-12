# Terraform breaking change classification

A change is breaking if an existing caller updating the module ref would have their plan fail or infrastructure destroyed/recreated without an explicit migration step.

## Breaking changes

### Input variables (`variables.tf`)

| Change | Why breaking |
|---|---|
| Remove a required input (no `default`) | Caller's plan fails: "No value for required variable" |
| Rename a required input | Same as remove + add — caller's plan fails |
| Rename an optional input | Callers using the old name get "unsupported argument" |
| Change type incompatibly (e.g. `string` to `number`, `list` to `map`) | Caller's plan fails on type mismatch |
| Add a new required input (no `default`) | Caller's plan fails — must update every callsite |
| Remove a `default` from an optional input | Callers not setting it get plan failure |
| Add a validation block that tightens the allowed range | Callers with values outside the new range get plan failure |

#### Changed default value of an optional input (decision tree)

Changing a default on an existing optional input sits in the judgment zone — it does not always break, and it does not always pass. Walk this tree for each changed default:

1. **Does the new default force resource recreation?** Check the provider docs for the attribute's `ForceNew` flag. If yes → **BREAKING**. Callers not overriding the input get an unplanned destroy/recreate.
2. **Does the new default change a security boundary or IAM condition?** Examples: security group rule CIDR, IAM policy condition key, KMS key alias, S3 bucket policy principal. If yes → **BREAKING**. The change silently alters access controls for existing callers.
3. **Does the new default change instance sizing, engine type, or a cost-bearing attribute?** Examples: instance class, database engine version, storage type, throughput capacity. If yes → **BREAKING**. Callers who don't override get a plan showing a resource modification that affects performance or billing, which may be unexpected.
4. **Does the new default change a resource identifier or naming attribute?** Examples: `name`, `name_prefix`, `bucket`, `identifier`. If yes → **BREAKING**. These typically force recreation or break external references.
5. **Is the change purely expansion, cosmetic, or loosening?** Examples: increasing a timeout or retention period, changing a description, widening a tag value not used in IAM conditions, increasing a max capacity that callers set lower anyway. If yes → **NON-BREAKING**. Note it in the review for visibility but it does not trigger `!`.

When in doubt between steps 2-4, flag it as breaking and note the ambiguity. Callers can override the new default explicitly if they prefer the old behavior.

### Outputs (`outputs.tf`)

| Change | Why breaking |
|---|---|
| Remove an output | Callers referencing it get "Reference to undeclared output" |
| Rename an output | Same as remove + add |
| Add `sensitive = true` to an existing output | Callers consuming it in `module.X.Y` references get "Output is marked as sensitive" |

### Resources

| Change | Why breaking |
|---|---|
| Remove a resource | Existing infrastructure destroyed |
| Rename a resource (without `moved` block) | Shows as delete + create — destroys and recreates |
| Add `prevent_destroy = true` to an existing resource | Callers doing `terraform destroy` or recreating the resource hit an error |
| Change `count` or `for_each` on an existing resource | Resource addresses change, forces destroy/recreate of all instances |
| Change a resource attribute that forces recreation | The resource is destroyed and recreated — check provider docs for the `force_new` flag |

### Provider and version constraints

| Change | Why breaking |
|---|---|
| Tighten `required_providers` version constraint | Callers on the excluded versions can't init |
| Change `required_providers` source (e.g. `hashicorp/aws` to a fork) | Callers can't resolve the provider |
| Remove a provider alias | Callers using that alias get "provider configuration not present" |
| Require a new provider not already in callers' configs | Callers get "provider configuration not present" |

### Module interface (root `main.tf` or sub-module caller surface)

| Change | Why breaking |
|---|---|
| Remove a `module` block from the root | Callers lose that sub-module |
| Remove a `moved` block that was previously announced as deprecated | Callers who didn't migrate get resource destruction |

## Non-breaking changes

### Input variables

- Add a new optional input with a `default` value
- Deprecate an input (keep it working, add a warning)
- Add a validation block that only adds new allowed values (loosens)

### Outputs

- Add a new output
- Deprecate an output (keep it working)

### Resources

- Add a new resource or data source
- Add a resource attribute that does not force recreation
- Rename a resource **with a `moved` block** that migrates state

### Provider and version

- Loosen `required_providers` version constraint
- Add a new provider configuration block

### Other

- Documentation changes (README, comments, examples)
- Adding or modifying examples under `examples/`
- Test fixture changes under `tests/` or `test/`
- CI/CD config changes
- Adding `moved` blocks (they are migration helpers, not destructive)

## Classification rules per file type

| File location | Classification scope |
|---|---|
| `variables.tf` (root) | Breaking if it changes the root module's input contract |
| `outputs.tf` (root) | Breaking if it changes the root module's output contract |
| `main.tf` (root) | Breaking if resources are removed, renamed, or have `prevent_destroy` added |
| `modules/<name>/variables.tf` | Evaluate against that sub-module's contract, not the root |
| `modules/<name>/outputs.tf` | Evaluate against that sub-module's contract |
| `modules/<name>/*.tf` | Evaluate within that sub-module's scope |
| `examples/**/*.tf` | Never breaking — documentation only |
| `tests/**/*.tf`, `test/**/*.tf` | Never breaking — test fixtures only |

## What to flag vs ignore

Flag every breaking change with the file path, line number, and a one-line reason. Group by file for readability.

Ignore: formatting-only changes, comment changes, whitespace, `terraform-docs` auto-generated content, and changes confined entirely to file types the skill filters out (`.md`, `.yaml`, `.json`, CI configs).
