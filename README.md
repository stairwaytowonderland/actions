# :twisted_rightwards_arrows: Shared Actions and Workflows

Shared actions and workflows for use by this organization.

## Project structure

> [!NOTE]
> `tree -a -F -L 3 -I '.git|.vscode' --gitignore --dirsfirst .`

```none
./
├── .github/
│   ├── actions/
│   │   ├── conventional-commit-action/
│   │   ├── issue-create-action/
│   │   ├── labels-create-action/
│   │   ├── labels-sync-action/
│   │   ├── merge-commit-action/
│   │   ├── parse-issue-csv-action/
│   │   ├── terraform-apply/
│   │   └── terraform-plan/
│   ├── workflows/
│   │   ├── ci-package-update.yaml
│   │   ├── ci.yaml
│   │   ├── conventional-commit.yaml
│   │   ├── create-labels.yaml
│   │   ├── import-csv-issues.yaml
│   │   ├── pre-commit.yaml
│   │   ├── publish.yaml
│   │   ├── release.yaml
│   │   ├── repository-created.yaml
│   │   ├── sync-labels.yaml
│   │   └── terraform-deploy.yaml
│   └── dependabot.yml
├── .editorconfig
├── .gitignore
├── .markdownlint.json
├── .pre-commit-config.yaml
├── .prettierignore
├── .prettierrc
├── .releaserc
├── CHANGELOG.md
├── LICENSE
└── README.md
```

## GitHub Actions

### 1. `labels-create-action`

**Description:**
Ensures all needed labels exist in the repository.

**Inputs:**

| Name              | Description                                                                | Required | Default                                                                                                                                                                                                                                                                                                                               |
| ----------------- | -------------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `effort-labels`   | JSON string array of effort label objects (name, color, description)       | No       | `[{ "name": "effort:high", "color": "FA2251", "description": "High effort" }, { "name": "effort:medium", "color": "BFD4F2", "description": "Medium effort" }, { "name": "effort:low", "color": "ADFCC0", "description": "Low effort" }`                                                                                               |
| `impact-labels`   | JSON string array of impact label objects (name, color, description)       | No       | `[{ "name": "impact:high", "color": "E5ABC7", "description": "High impact" }, { "name": "impact:low", "color": "C2E0C6", "description": "Low impact" }]`                                                                                                                                                                              |
| `priority-labels` | JSON string array of priority label objects (name, color, description)     | No       | `[{ "name": "priority:high", "color": "FF2BBF", "description": "High priority" }, { "name": "priority:low", "color": "BAF165", "description": "Low priority" }]`                                                                                                                                                                      |
| `other-labels`    | JSON string array of other (misc) label objects (name, color, description) | No       | `[{ "name": "poc", "color": "0FD40A", "description": "Proof of concept" }, { "name": "task", "color": "0052CC", "description": "General task" }, { "name": "security", "color": "FBCA04", "description": "Something is a security risk" }, { "name": "uncategorized", "color": "6302AB", "description": "Needs to be categorized" }]` |
| `effort-map`      | JSON string object mapping effort values to normalized label names         | No       | `{ "high": "high", "larger": "high", "large": "high", "medium": "medium", "low": "low", "smaller": "low", "small": "low" }`                                                                                                                                                                                                           |
| `impact-map`      | JSON string object mapping impact values to normalized label names         | No       | `{ "higher": "high", "high": "high", "medium": "medium", "lower": "low", "low": "low" }`                                                                                                                                                                                                                                              |
| `priority-map`    | JSON string object mapping priority values to normalized label names       | No       | `{ "higher": "high", "high": "high", "medium": "medium", "lower": "low", "low": "low" }`                                                                                                                                                                                                                                              |
| `matrix`          | JSON string matrix of issues to process (from parse-csv output)            | No       | `''`                                                                                                                                                                                                                                                                                                                                  |
| `types`           | JSON string array of types in order of processing                          | No       | `["bug","feature","documentation","task"]`                                                                                                                                                                                                                                                                                            |
| `dry-run`         | Dry run (no changes made)                                                  | No       | `''`                                                                                                                                                                                                                                                                                                                                  |
| `update-existing` | Update existing labels                                                     | No       | `''`                                                                                                                                                                                                                                                                                                                                  |
| `github-token`    | GitHub token for authentication                                            | No       | `${{ github.token }}`                                                                                                                                                                                                                                                                                                                 |

**Outputs:**

| Name             | Description                                                         |
| ---------------- | ------------------------------------------------------------------- |
| `created-labels` | JSON array of needed label names that were created or already exist |

### 2. `labels-sync-action`

**Description:**
Syncs a standardized set of labels across all active organization repositories

**Inputs:**

| Name              | Description                                                                                                                   | Required | Default               |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------- | -------- | --------------------- |
| `dry-run`         | Dry run (no changes made)                                                                                                     | No       | `''`                  |
| `update-existing` | Update existing labels                                                                                                        | No       | `''`                  |
| `github-token`    | GitHub token for authentication                                                                                               | No       | `${{ github.token }}` |
| `repos`           | Specific repositories to sync (comma-separated, e.g. "repo1,repo2"). If not provided, all active repositories will be synced. | No       | `''`                  |
| `repos-ignore`    | Names or patterns (regex) of repositories to ignore (e.g. `/^.github.*$/`).                                                   | No       | `''`                  |

**Outputs:**

| Name            | Description                                                         |
| --------------- | ------------------------------------------------------------------- |
| `target-labels` | JSON array of target label names that were created or already exist |

### 3. `parse-issue-csv-action`

**Description:**
Parses a CSV file of issues and outputs a grouped matrix by type.

**Inputs:**

| Name                | Description                                  | Required | Default                                    |
| ------------------- | -------------------------------------------- | -------- | ------------------------------------------ |
| `csv-path`          | Path to the CSV file (relative to repo root) | Yes      | `''`                                       |
| `types`             | JSON array of types in order of processing   | No       | `["bug","feature","documentation","task"]` |
| `status-if-missing` | Default status to assign if missing          | No       | `new`                                      |

**Outputs:**

| Name     | Description                                                       |
| -------- | ----------------------------------------------------------------- |
| `matrix` | Grouped matrix object by type (bug, feature, documentation, task) |
| `types`  | Array of types in the order they were processed                   |

### 4. `issue-create-action`

**Description:**
Creates a GitHub issue from a CSV row and appends to the job summary.

**Inputs:**

| Name                          | Description                                                          | Required | Default                                                                                                                     |
| ----------------------------- | -------------------------------------------------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------- |
| `allow-duplicates`            | Allow creating duplicate issues                                      | No       | `true`                                                                                                                      |
| `allow-closed-duplicates`     | Allow creating duplicate issues for closed issues                    | No       | `''`                                                                                                                        |
| `allow-unrecognized-status`   | Allow unrecognized status values (fallback to `new`)                 | No       | `true`                                                                                                                      |
| `fail-on-unrecognized-status` | Fail action if an unrecognized status value is encountered           | No       | `''`                                                                                                                        |
| `batch-json`                  | JSON array of issue rows for batch creation                          | No       | `[]`                                                                                                                        |
| `title`                       | Issue title (conflicts with `batch-json`)                            | No       | `''`                                                                                                                        |
| `priority`                    | Priority value (conflicts with `batch-json`)                         | No       | `''`                                                                                                                        |
| `priority-field-id`           | Field ID for organization Priority field                             | No       | `''`                                                                                                                        |
| `priority-map`                | JSON string object mapping priority values to normalized label names | No       | `{ "higher": "high", "high": "high", "lower": "low", "low": "low" }`                                                        |
| `effort`                      | Effort value (conflicts with `batch-json`)                           | No       | `''`                                                                                                                        |
| `effort-field-id`             | Field ID for organization Effort field                               | No       | `''`                                                                                                                        |
| `effort-map`                  | JSON string object mapping effort values to normalized label names   | No       | `{ "high": "high", "larger": "high", "large": "high", "medium": "medium", "low": "low", "smaller": "low", "small": "low" }` |
| `description`                 | Issue description (conflicts with `batch-json`)                      | No       | `''`                                                                                                                        |
| `status`                      | Issue status (conflicts with `batch-json`)                           | No       | `''`                                                                                                                        |
| `completed`                   | Completed flag (conflicts with `batch-json`)                         | No       | `''`                                                                                                                        |
| `dry-run`                     | Dry run                                                              | No       | `''`                                                                                                                        |
| `github-token`                | GitHub token for authentication                                      | No       | `${{ github.token }}`                                                                                                       |

**Outputs:**

| Name         | Description                                                     |
| ------------ | --------------------------------------------------------------- |
| `issue-url`  | URL of the created issue (empty if dry run)                     |
| `issue-urls` | JSON array of created issue URLs (batch only, empty if dry run) |

### 5. `conventional-commit-action`

**Description:**
Validates commit messages against the Conventional Commits specification.

**Inputs:**

| Name                           | Description                                      | Required | Default       |
| ------------------------------ | ------------------------------------------------ | -------- | ------------- |
| `base-ref-from-default-branch` | Use repository default branch as base ref        | No       | `true`        |
| `base-ref`                     | Base reference to compare against                | No       | `origin/main` |
| `fail-on-error`                | Fail action if invalid commit messages are found | No       | `true`        |
| `types`                        | Comma-separated allowed commit types             | No       | `''`          |
| `scopes`                       | Comma-separated allowed scopes (optional)        | No       | `''`          |
| `fetch-depth`                  | Git fetch depth used to compute comparison range | No       | `100`         |

**Outputs:**

| Name              | Description                                        |
| ----------------- | -------------------------------------------------- |
| `valid`           | Whether all commits are valid conventional commits |
| `invalid-commits` | Multiline list of invalid commit messages          |
| `commit-count`    | Total number of commits checked                    |

### 6. `merge-commit-action`

**Description:**
Checks whether the current commit is a merge commit.

**Outputs:**

| Name       | Description                                              |
| ---------- | -------------------------------------------------------- |
| `is_merge` | `true` if commit has multiple parents, otherwise `false` |

### 7. `terraform-plan`

**Description:**
Reusable composite action for Terraform plan.

**Inputs:**

| Name                | Description                          | Required | Default |
| ------------------- | ------------------------------------ | -------- | ------- |
| `working-directory` | Directory to run Terraform in        | No       | `''`    |
| `custom-directory`  | Custom directory to run Terraform in | No       | `''`    |
| `extra-init-args`   | Extra arguments for Terraform init   | No       | `''`    |
| `extra-plan-args`   | Extra arguments for Terraform plan   | No       | `''`    |

### 8. `terraform-apply`

**Description:**
Reusable composite action for Terraform apply.

**Inputs:**

| Name                | Description                          | Required | Default |
| ------------------- | ------------------------------------ | -------- | ------- |
| `working-directory` | Directory to run Terraform in        | No       | `''`    |
| `custom-directory`  | Custom directory to run Terraform in | No       | `''`    |
| `extra-init-args`   | Extra arguments for Terraform init   | No       | `''`    |
| `extra-apply-args`  | Extra arguments for Terraform apply  | No       | `''`    |

## Workflows

### 1. `ci-package-update`

**Description:**
Checks for changes to package.json and package-lock.json and triggers the publish workflow if needed. Can be called
directly or as a reusable workflow.

**Triggers:**

| Event           | Details                                                                       |
| --------------- | ----------------------------------------------------------------------------- |
| `push`          | Runs on `main` branch when `package.json` or `package-lock.json` changes      |
| `workflow_call` | Can be called from other workflows with custom file paths and ignored authors |

**Inputs:**

| Name             | Description                                                                                            | Required | Type   | Default                                         |
| ---------------- | ------------------------------------------------------------------------------------------------------ | -------- | ------ | ----------------------------------------------- |
| `paths`          | Newline-delimited list of file paths to check for changes (e.g. package.json and package-lock.json)    | No       | string | `package.json`<br>`package-lock.json`           |
| `ignore-authors` | Newline-delimited list of commit authors to ignore (e.g. semantic-release-bot and github-actions[bot]) | No       | string | `github-actions[bot]`<br>`semantic-release-bot` |

**Secrets:**

| Name           | Description                                         |
| -------------- | --------------------------------------------------- |
| `github-token` | Optional token with repo scope for checking commits |

### 2. `ci`

**Description:**
Main orchestration workflow for release and publish operations of this repo.

**Triggers:**

| Event               | Details                                                                                               |
| ------------------- | ----------------------------------------------------------------------------------------------------- |
| `push`              | Runs on `main` (excluding `CHANGELOG.md`, `package.json`, `package-lock.json`) and tags matching `v*` |
| `workflow_dispatch` | Manual run with `fail-on-no-release` and `publish` flags                                              |

**Behavior:**

- Runs `release.yaml` on `main` pushes unless publish mode is requested.
- Runs `publish.yaml` for `v*` tags or manual runs with `publish: true`.
- Updates major version tags via `nowactions/update-majorver@v1` after publish.
