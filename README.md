# 📦 GitHub Configuration Self Service

This repository automates the management of GitHub repositories within the G-Research GitHub organization using Terraform Cloud and GitHub Actions. It supports both the **import of existing repositories** and the **provisioning of new repositories**, ensuring GitHub configuration is reproducible, auditable, and version-controlled.

## ✨ Features

- 🛠️ Import existing GitHub repositories into Terraform Cloud state
- 📦 Provision new repositories using declarative YAML configurations
- 🤖 GitHub Actions workflows for automation and CI/CD
- 🔀 Supports both `dev` and `prod` environments
- 👥 Support for multiple Github organizations
- 🧩 Manages metadata including:
    - General repository settings
    - Branch protection rules
    - Default branch
    - Teams and collaborators
    - Repository rulesets

## 📁 Repository Structure

```plaintext
feature/
  ├── github-repo-importer/                 # Logic for importing (reading settings) of existing GitHub repos
  └── github-repo-provisioning/             # Terraform modules for managing GitHub repos
        └── repo_configs/
              ├── dev/
              │   ├── gr-oss-devops/        # gr-oss-devops organization repositories
              │   └── gr-oss-developers/    # gr-oss-developers organization repositories
              └── prod/
                  ├── G-Research/           # G-Research organization repositories
                  └── armadaproject/        # armadaproject organization repositories
```

## ⚙️ Terraform Cloud Integration

This project leverages [Terraform Cloud (TFE)](https://www.terraform.io/cloud) to manage GitHub repositories declaratively. Pull requests into the configuration directories trigger Terraform runs for `plan` and `apply` stages, aligning GitHub state with the YAML definitions.

## 🆕 Creating New Repositories

To provision a **brand-new repository** in the G-Research GitHub organization:

1. Create a pull request targeting the `feature/github-repo-provisioning/prod/{organisation}/` directory
2. Add a YAML file describing the desired repository configuration - name of the YAML file will be the name of the repository (case-sensitive)
3. Submit the PR for review
4. Upon approval and merge, Terraform Cloud will:
    - Plan and apply the configuration
    - Create the repository

> 📝 New repositories must not be forks. Forks follow a different workflow (see below).

## 🍴 Handling Forks

To import a **forked** repository into the organization:

1. Trigger the Create fork workflow
2. Provide input:
    - `upstream_repo`: Repo to fork (in the format of `owner/repo`)
    - `fork_name`: Name of the new forked repository. If left empty, default would be the same as the upstream repo name
    - `default_branch_only`: To create the fork with only the default branch (e.g., `main` or `master`) or all branches. Default is `true`
    - `fork_to_org`: The organization to fork the repository into (e.g., `G-Research` or `armadaproject`)
3. Workflow will fork and import the repository triggering the import workflow. From here, the process is similar to importing an existing repository:
    1. The workflow will generate a YAML config for the forked repository
    2. Place it under `feature/github-repo-provisioning/importer_tmp_dir/{organization}/`
    3. Create a PR against the `prod` branch
    4. _User reviews, approves, and merges the PR_
4. After merge Terraform Cloud will import the forked repository into its state by applying the generated YAML configuration
5. The configuration file is then sanitized (ids removed) and moved to the appropriate directory `feature/github-repo-provisioning/repo_configs/{branch}/{organization}`
6. From here, user can make changes to the configuration file as needed, and create a PR against the `feature/github-repo-provisioning/prod/{organisation}/{repository}.yaml` file to apply further changes

> 📝 We are working on improving this so that the user has the same experience as when creating a new repo

> [!IMPORTANT]
> All important attributes are documented in the [Developer's Guide](DEVELOPERS_GUIDE.md).