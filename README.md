# Shared Actions and Workflows

Shared actions and workflows for use by the organization.

## Project structure

> [!NOTE]
> `tree -a -F -L 2 -I '.git|.vscode' --gitignore --dirsfirst .`

```none
./
├── .github/
│   ├── actions/
│   ├── ISSUE_TEMPLATE/
│   ├── workflows/
│   ├── dependabot.yml
│   └── PULL_REQUEST_TEMPLATE.md
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

| Name                | Description                                                     |
| ------------------- | --------------------------------------------------------------- |
| `priority-labels`   | JSON array of priority label objects (name, color, description) |
| `effort-labels`     | JSON array of effort label objects (name, color, description)   |
| `default-labels`    | JSON array of default label objects (name, color, description)  |
| `priority-map`      | JSON object mapping priority values to normalized label names   |
| `effort-map`        | JSON object mapping effort values to normalized label names     |
| `matrix` (required) | JSON matrix of issues to process (from parse-csv output)        |
| `dry-run`           | Dry run (no changes made)                                       |
| `github-token`      | GitHub token for authentication                                 |

**Outputs:**

| Name             | Description                                                         |
| ---------------- | ------------------------------------------------------------------- |
| `created-labels` | JSON array of needed label names that were created or already exist |

### 2. `parse-issue-csv-action`

**Description:**
Parses a CSV file of issues and outputs a grouped matrix by type.

**Inputs:**

| Name                  | Description                                                                                      |
| --------------------- | ------------------------------------------------------------------------------------------------ |
| `csv-path` (required) | Path to the CSV file (relative to repo root)                                                     |
| `types`               | JSON array of types in order of processing (default: `["bug","feature","documentation","task"]`) |
| `status-if-missing`   | Default status to assign if missing (default: `new`)                                             |

**Outputs:**

| Name     | Description                                                       |
| -------- | ----------------------------------------------------------------- |
| `matrix` | Grouped matrix object by type (bug, feature, documentation, task) |
| `types`  | Array of types in the order they were processed                   |

### 3. `issue-create-action`

**Description:**
Creates a GitHub issue from a CSV row and appends to the job summary.

**Inputs:**

| Name                          | Description                                                                      |
| ----------------------------- | -------------------------------------------------------------------------------- |
| `allow-duplicates`            | Allow creating duplicate issues (default: `true`)                                |
| `allow-closed-duplicates`     | Allow creating duplicate issues for closed issues                                |
| `allow-unrecognized-status`   | Allow unrecognized status values (default: `true`, will default to `new` status) |
| `fail-on-unrecognized-status` | Fail the action if an unrecognized status value is encountered                   |
| `batch-json`                  | JSON array of issue rows for batch creation (default: `[]`)                      |
| `title`                       | Issue title (conflicts with `batch-json`)                                        |
| `priority`                    | Priority value (conflicts with `batch-json`)                                     |
| `effort`                      | Effort value (conflicts with `batch-json`)                                       |
| `description`                 | Issue description (conflicts with `batch-json`)                                  |
| `status`                      | Issue status (conflicts with `batch-json`)                                       |
| `completed`                   | Completed flag (conflicts with `batch-json`)                                     |
| `dry-run`                     | Dry run                                                                          |
| `github-token`                | GitHub token for authentication                                                  |

**Outputs:**

| Name         | Description                                                      |
| ------------ | ---------------------------------------------------------------- |
| `issue-url`  | URL of the created issue (empty if dry run)                      |
| `issue-urls` | JSON array of created issue URLs (batch only) (empty if dry run) |

### 4. `terraform-plan`

**Description:**
Reusable composite action for Terraform plan.

**Inputs:**

| Name                | Description                                        |
| ------------------- | -------------------------------------------------- |
| `working-directory` | Directory to run Terraform in                      |
| `custom-directory`  | Custom directory to run Terraform in               |
| `extra-init-args`   | Extra arguments for Terraform init (default: `""`) |
| `extra-plan-args`   | Extra arguments for Terraform plan (default: `""`) |

### 5. `terraform-apply`

**Description:**
Reusable composite action for Terraform apply.

**Inputs:**

| Name                | Description                                         |
| ------------------- | --------------------------------------------------- |
| `working-directory` | Directory to run Terraform in                       |
| `custom-directory`  | Custom directory to run Terraform in                |
| `extra-init-args`   | Extra arguments for Terraform init (default: `""`)  |
| `extra-apply-args`  | Extra arguments for Terraform apply (default: `""`) |

## Reusable Workflows

All workflows in this section are implemented as [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
using `workflow_call`. They are designed to be invoked from other workflows or repositories.

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
 node-version: 24
 max-parallel: 5
secrets:
 github-token: ${{ secrets.GH_PAT }}
```

**Inputs:**

| Name                      | Description                                                          |
| ------------------------- | -------------------------------------------------------------------- |
| `ref`                     | Git ref of the shared actions repo to use (default: `main`)          |
| `node-version`            | Node.js version to use for parsing CSV (default: `24`)               |
| `dry-run`                 | Dry run (default: `true`)                                            |
| `max-parallel`            | Maximum number of parallel issue creations (default: `5`)            |
| `batch`                   | Create issues in batches (default: `true`)                           |
| `allow-duplicates`        | Allow creating duplicate issues (default: `false`)                   |
| `allow-closed-duplicates` | Allow creating duplicate issues for closed issues (default: `false`) |
| `csv-path` (required)     | Path (relative) to the CSV file                                      |

**Secrets:**

| Name           | Description                                                    |
| -------------- | -------------------------------------------------------------- |
| `github-token` | GitHub Personal Access Token with permissions to create issues |

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

| Name     | Description                                                             |
| -------- | ----------------------------------------------------------------------- |
| `config` | Path to the pre-commit config file (default: `.pre-commit-config.yaml`) |

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

| Name                | Description                                      |
| ------------------- | ------------------------------------------------ |
| `working-directory` | Directory to run Terraform in                    |
| `custom-directory`  | Custom directory to run Terraform in             |
| `terraform-version` | Terraform version to use (**required**)          |
| `environment`       | Environment name (**required**)                  |
| `aws-region`        | AWS region (**required**)                        |
| `oidc-role-name`    | AWS OIDC role name (**required**)                |
| `application-name`  | Application name (**required**)                  |
| `purpose`           | Purpose for the deployment (**required**)        |
| `plan`              | Run plan step (boolean, **required**)            |
| `apply`             | Run apply step (boolean, **required**)           |
| `destroy`           | Run destroy step (boolean, **required**)         |
| `approvers`         | Comma-separated list of approvers (**required**) |

**Secrets:**

| Name             | Description                   |
| ---------------- | ----------------------------- |
| `aws-account-id` | AWS Account ID (**required**) |
