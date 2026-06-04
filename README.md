# Shared Actions and Workflows

Shared actions and workflows for use by the organization.

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
│   │   ├── merge-commit-action/
│   │   ├── parse-issue-csv-action/
│   │   ├── terraform-apply/
│   │   └── terraform-plan/
│   ├── workflows/
│   │   ├── ci-package-update.yaml
│   │   ├── ci.yaml
│   │   ├── conventional-commit.yaml
│   │   ├── import-csv-issues.yaml
│   │   ├── pre-commit.yaml
│   │   ├── publish.yaml
│   │   ├── release.yaml
│   │   └── terraform-deploy.yml
│   └── dependabot.yml
├── .editorconfig
├── .gitignore
├── .markdownlint.json
├── .pre-commit-config.yaml
├── .prettierignore
├── .prettierrc
├── .releaserc
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── package-lock.json
├── package.json
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
| `impact-map`      | JSON string object mapping impact values to normalized label names         | No       | `{ "higher": "high", "high": "high", "lower": "low", "low": "low" }`                                                                                                                                                                                                                                                                  |
| `priority-map`    | JSON string object mapping priority values to normalized label names       | No       | `{ "higher": "high", "high": "high", "lower": "low", "low": "low" }`                                                                                                                                                                                                                                                                  |
| `matrix`          | JSON string matrix of issues to process (from parse-csv output)            | No       | `''`                                                                                                                                                                                                                                                                                                                                  |
| `dry-run`         | Dry run (no changes made)                                                  | No       | `''`                                                                                                                                                                                                                                                                                                                                  |
| `update-existing` | Update existing labels                                                     | No       | `''`                                                                                                                                                                                                                                                                                                                                  |
| `github-token`    | GitHub token for authentication                                            | No       | `${{ github.token }}`                                                                                                                                                                                                                                                                                                                 |

**Outputs:**

| Name             | Description                                                         |
| ---------------- | ------------------------------------------------------------------- |
| `created-labels` | JSON array of needed label names that were created or already exist |

### 2. `parse-issue-csv-action`

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

### 3. `issue-create-action`

**Description:**
Creates a GitHub issue from a CSV row and appends to the job summary.

**Inputs:**

| Name                          | Description                                                | Required | Default               |
| ----------------------------- | ---------------------------------------------------------- | -------- | --------------------- |
| `allow-duplicates`            | Allow creating duplicate issues                            | No       | `true`                |
| `allow-closed-duplicates`     | Allow creating duplicate issues for closed issues          | No       | `''`                  |
| `allow-unrecognized-status`   | Allow unrecognized status values (fallback to `new`)       | No       | `true`                |
| `fail-on-unrecognized-status` | Fail action if an unrecognized status value is encountered | No       | `''`                  |
| `batch-json`                  | JSON array of issue rows for batch creation                | No       | `[]`                  |
| `title`                       | Issue title (conflicts with `batch-json`)                  | No       | `''`                  |
| `priority`                    | Priority value (conflicts with `batch-json`)               | No       | `''`                  |
| `effort`                      | Effort value (conflicts with `batch-json`)                 | No       | `''`                  |
| `description`                 | Issue description (conflicts with `batch-json`)            | No       | `''`                  |
| `status`                      | Issue status (conflicts with `batch-json`)                 | No       | `''`                  |
| `completed`                   | Completed flag (conflicts with `batch-json`)               | No       | `''`                  |
| `dry-run`                     | Dry run                                                    | No       | `''`                  |
| `github-token`                | GitHub token for authentication                            | No       | `${{ github.token }}` |

**Outputs:**

| Name         | Description                                                     |
| ------------ | --------------------------------------------------------------- |
| `issue-url`  | URL of the created issue (empty if dry run)                     |
| `issue-urls` | JSON array of created issue URLs (batch only, empty if dry run) |

### 4. `conventional-commit-action`

**Description:**
Validates commit messages against the Conventional Commits specification.

**Inputs:**

| Name                           | Description                                      | Required | Type    | Default       |
| ------------------------------ | ------------------------------------------------ | -------- | ------- | ------------- |
| `base-ref-from-default-branch` | Use repository default branch as base ref        | No       | boolean | `true`        |
| `base-ref`                     | Base reference to compare against                | No       | string  | `origin/main` |
| `fail-on-error`                | Fail action if invalid commit messages are found | No       | boolean | `true`        |
| `types`                        | Comma-separated allowed commit types             | No       | string  | `''`          |
| `scopes`                       | Comma-separated allowed scopes (optional)        | No       | string  | `''`          |
| `fetch-depth`                  | Git fetch depth used to compute comparison range | No       | number  | `100`         |

**Outputs:**

| Name              | Description                                        |
| ----------------- | -------------------------------------------------- |
| `valid`           | Whether all commits are valid conventional commits |
| `invalid-commits` | Multiline list of invalid commit messages          |
| `commit-count`    | Total number of commits checked                    |

### 5. `merge-commit-action`

**Description:**
Checks whether the current commit is a merge commit.

**Outputs:**

| Name       | Description                                              |
| ---------- | -------------------------------------------------------- |
| `is_merge` | `true` if commit has multiple parents, otherwise `false` |

### 6. `terraform-plan`

**Description:**
Reusable composite action for Terraform plan.

**Inputs:**

| Name                | Description                          | Required | Type   | Default |
| ------------------- | ------------------------------------ | -------- | ------ | ------- |
| `working-directory` | Directory to run Terraform in        | No       | string | `''`    |
| `custom-directory`  | Custom directory to run Terraform in | No       | string | `''`    |
| `extra-init-args`   | Extra arguments for Terraform init   | No       | string | `''`    |
| `extra-plan-args`   | Extra arguments for Terraform plan   | No       | string | `''`    |

### 7. `terraform-apply`

**Description:**
Reusable composite action for Terraform apply.

**Inputs:**

| Name                | Description                          | Required | Type   | Default |
| ------------------- | ------------------------------------ | -------- | ------ | ------- |
| `working-directory` | Directory to run Terraform in        | No       | string | `''`    |
| `custom-directory`  | Custom directory to run Terraform in | No       | string | `''`    |
| `extra-init-args`   | Extra arguments for Terraform init   | No       | string | `''`    |
| `extra-apply-args`  | Extra arguments for Terraform apply  | No       | string | `''`    |

## Workflows

### `ci-package-update`

**Description:**
Checks for changes to package.json and package-lock.json and triggers the publish workflow if needed. Can be called
directly or as a reusable workflow.

**Triggers:**

| Event           | Details                                                                       |
| --------------- | ----------------------------------------------------------------------------- |
| `push`          | Runs on `main` branch when `package.json` or `package-lock.json` changes      |
| `workflow_call` | Can be called from other workflows with custom file paths and ignored authors |

**Inputs:**

| Name             | Description                                                                                            | Required | Type   | Default                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------ | -------- | ------ | --------------------------------------------- |
| `paths`          | Newline-delimited list of file paths to check for changes (e.g. package.json and package-lock.json)    | No       | string | `package.json<br>package-lock.json`           |
| `ignore-authors` | Newline-delimited list of commit authors to ignore (e.g. semantic-release-bot and github-actions[bot]) | No       | string | `github-actions[bot]<br>semantic-release-bot` |

**Secrets:**

| Name           | Description                                         |
| -------------- | --------------------------------------------------- |
| `github-token` | Optional token with repo scope for checking commits |

### `ci`

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

## Reusable Workflows (`workflow_call`)

### `conventional-commit`

**Description:**
Reusable workflow to validate commit messages in pull requests or caller-defined refs.

**Inputs:**

| Name         | Description                                       | Required | Type   | Default           |
| ------------ | ------------------------------------------------- | -------- | ------ | ----------------- |
| `ref`        | Git ref to check                                  | No       | string | `refs/heads/main` |
| `action-ref` | Ref of this shared actions repository to checkout | No       | string | `main`            |

**Outputs:**

| Name                         | Description                                  |
| ---------------------------- | -------------------------------------------- |
| `conventional-commits-valid` | `true` when all checked commits are valid    |
| `commit-count`               | Number of commits validated                  |
| `is-merge`                   | Whether the checked commit is a merge commit |

### `import-csv-issues`

**Description:**
Reusable workflow to import issues from a CSV file.

**Usage Example:**

```yaml
uses: stairwaytowonderland/actions/.github/workflows/import-csv-issues.yaml@main
with:
 csv-path: path/to/issues.csv
 dry-run: false
 batch: true
 allow-duplicates: false
 allow-closed-duplicates: false
 ref: main
 action-ref: main
 node-version: 24
 max-parallel: 5
secrets:
 github-token: ${{ secrets.GH_PAT }}
```

**Inputs:**

| Name                      | Description                                                                     | Required | Type    | Default        |
| ------------------------- | ------------------------------------------------------------------------------- | -------- | ------- | -------------- |
| `secret-token-name`       | Name of the repository secret containing the GitHub token (e.g. `GH_PAT`)       | No       | string  | `GITHUB_TOKEN` |
| `ref`                     | Git ref of the shared actions repo to use (e.g. `main` or a tag/commit)         | No       | string  | `main`         |
| `action-ref`              | Git ref of the shared actions repo to use (e.g. `main` or a tag/commit)         | No       | string  | `main`         |
| `node-version`            | Node.js version to use for parsing CSV (e.g. `18`, `20`, `24`)                  | No       | string  | `24`           |
| `dry-run`                 | Dry run (no issues created)                                                     | No       | boolean | `true`         |
| `max-parallel`            | Maximum number of parallel issue creations (only applies when `batch` is false) | No       | number  | `5`            |
| `batch`                   | Create issues in batches (by type)                                              | No       | boolean | `true`         |
| `allow-duplicates`        | Allow creating duplicate issues                                                 | No       | boolean | `false`        |
| `allow-closed-duplicates` | Allow creating duplicate issues for closed issues                               | No       | boolean | `false`        |
| `csv-path`                | Path (relative) to the CSV file                                                 | Yes      | string  | `TODO.csv`     |

**Secrets:**

| Name           | Description                                                                     |
| -------------- | ------------------------------------------------------------------------------- |
| `github-token` | Optional token with permissions to create issues (falls back to `GITHUB_TOKEN`) |

### `pre-commit`

**Description:**
Reusable workflow to run pre-commit checks.

**Usage Example:**

```yaml
uses: stairwaytowonderland/actions/.github/workflows/pre-commit.yaml@main
with:
 config: .pre-commit-config.yaml
```

**Inputs:**

| Name     | Description                        | Required | Type   | Default                   |
| -------- | ---------------------------------- | -------- | ------ | ------------------------- |
| `config` | Path to the pre-commit config file | No       | string | `.pre-commit-config.yaml` |

### `release`

**Description:**
Runs semantic-release and exposes release metadata for downstream workflows.

**Usage Example:**

```yaml
uses: stairwaytowonderland/actions/.github/workflows/release.yaml@main
with:
 fail-on-no-release: true
 ref: refs/heads/main
secrets:
 github-token: ${{ secrets.GH_PAT_CREATE_RELEASE }}
```

**Inputs:**

| Name                 | Description                                                 | Required | Type    | Default       |
| -------------------- | ----------------------------------------------------------- | -------- | ------- | ------------- |
| `dispatch-workflow`  | Optional workflow file to dispatch after successful release | No       | string  |               |
| `fail-on-no-release` | Fail workflow when no new release is published              | No       | boolean | `false`       |
| `ref`                | Git ref or SHA to release (defaults to current ref)         | No       | string  | (current ref) |

**Outputs:**

| Name                       | Description                         |
| -------------------------- | ----------------------------------- |
| `new-release-published`    | Whether a new release was published |
| `new-release-version`      | New semantic version                |
| `new-release-git-tag`      | Git tag created for the release     |
| `new-release-notes`        | Release notes (markdown)            |
| `new-release-notes-base64` | Base64 encoded release notes        |

**Secrets:**

| Name           | Description                                      |
| -------------- | ------------------------------------------------ |
| `github-token` | Optional token for release creation and dispatch |

### `publish`

**Description:**
Publishes a GitHub release for a provided tag and optional precomputed notes.

**Usage Example:**

```yaml
uses: stairwaytowonderland/actions/.github/workflows/publish.yaml@main
with:
 tag: v1.2.3
 notes-b64: ${{ needs.release.outputs.new-release-notes-base64 }}
secrets:
 github-token: ${{ secrets.GH_PAT_CREATE_RELEASE }}
```

**Inputs:**

| Name        | Description                                    | Required | Type   | Default |
| ----------- | ---------------------------------------------- | -------- | ------ | ------- |
| `tag`       | Tag to publish                                 | Yes      | string |         |
| `notes-b64` | Optional base64-encoded markdown release notes | No       | string |         |

**Outputs:**

| Name                | Description                            |
| ------------------- | -------------------------------------- |
| `release-url`       | Published release URL                  |
| `release-tag`       | Release tag used                       |
| `release-sha`       | Commit SHA associated with release     |
| `release-notes-b64` | Release notes in base64                |
| `release-published` | Whether publish action created release |

**Secrets:**

| Name           | Description                                    |
| -------------- | ---------------------------------------------- |
| `github-token` | Optional token to checkout and publish release |

### `terraform-deploy`

**Description:**
Reusable workflow to plan and apply Terraform deployments with AWS OIDC and manual approval.

**Usage Example:**

```yaml
uses: stairwaytowonderland/actions/.github/workflows/terraform-deploy.yml@main
with:
 working-directory: path/to/dir
 custom-directory: path/to/custom
 terraform-version: 1.6.6
 environment: dev
 aws-region: us-west-2
 oidc-role-name: my-oidc-role
 application-name: my-app
 purpose: main
 plan: true
 apply: false
 destroy: false
 approvers: user1,user2
secrets:
 aws-account-id: ${{ secrets.AWS_ACCOUNT_ID }}
```

**Inputs:**

| Name                | Description                          | Required | Type    | Default |
| ------------------- | ------------------------------------ | -------- | ------- | ------- |
| `working-directory` | Directory to run Terraform in        | No       | string  |         |
| `custom-directory`  | Custom directory to run Terraform in | No       | string  |         |
| `terraform-version` | Terraform version to use             | Yes      | string  |         |
| `environment`       | Environment name                     | Yes      | string  |         |
| `aws-region`        | AWS region                           | Yes      | string  |         |
| `oidc-role-name`    | AWS OIDC role name                   | Yes      | string  |         |
| `application-name`  | Application name                     | Yes      | string  |         |
| `purpose`           | Purpose for the deployment           | Yes      | string  |         |
| `plan`              | Run plan step                        | Yes      | boolean |         |
| `apply`             | Run apply step                       | Yes      | boolean |         |
| `destroy`           | Run destroy step                     | Yes      | boolean |         |
| `approvers`         | Comma-separated list of approvers    | Yes      | string  |         |

**Secrets:**

| Name             | Description                   |
| ---------------- | ----------------------------- |
| `aws-account-id` | AWS Account ID (**required**) |
