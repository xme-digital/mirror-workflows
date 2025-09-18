### README.md

### GitHub Repository Mirroring Action

This GitHub Action automatically mirrors specified repositories from the `xm-online` organization to the `xme-digital` organization. It's designed to keep the mirrored repositories up-to-date by running on a schedule and can also be triggered manually.

#### 🚀 Features

  * **Scheduled Runs:** The action runs every 30 minutes to ensure synchronization.
  * **Manual Trigger:** You can manually trigger the workflow from the GitHub Actions tab.
  * **Selective Mirroring:** Mirrors all branches and tags, but explicitly avoids mirroring problematic "hidden refs" (like Pull Requests), which can cause errors.
  * **Secure Authentication:** Uses a personal access token (PAT) for authenticated pushing to private repositories.

-----

#### 🛠️ Setup

1.  **Generate a GitHub Personal Access Token (PAT):**

      * Go to your GitHub **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**.
      * Click **Generate new token**.
      * Give it a descriptive name (e.g., `mirroring-token`).
      * Select the **`repo`** scope to grant access to repositories.
      * Copy the generated token.

2.  **Add the Token to Your Repository Secrets:**

      * In the `xme-digital/your-mirroring-repo` repository, go to **Settings** → **Secrets and variables** → **Actions**.
      * Click **New repository secret**.
      * Name the secret **`GH_MIRROR_TOKEN`**.
      * Paste the token you copied into the **Secret** field.

3.  **Create the Workflow File:**

      * In your `xme-digital` repository, create a new file at `.github/workflows/mirror.yml`.
      * Copy and paste the entire YAML code provided below into this file.

-----

#### 💻 Workflow Code

```yaml
name: Bulk Repository Mirroring

on:
  schedule:
    - cron: '*/30 * * * *'
  workflow_dispatch:

jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - name: Mirror Repositories
        run: |
          REPOS=(
            "generator-jhipster-xm"
            "tmf-ms-account"
            "tmf-ms-activation"
            "tmf-ms-communication"
            "tmf-ms-customer"
            "tmf-ms-document"
            "tmf-ms-interaction"
            "tmf-ms-loyalty"
            "tmf-ms-offering"
            "tmf-ms-order"
            "tmf-ms-prepaybalance"
            "tmf-ms-product"
            "tmf-ms-promotion"
            "tmf-ms-qualification"
            "tmf-ms-resource"
            "tmf-ms-resource-pool"
            "xm-commons"
            "xm-gate"
            "xm-lep"
            "xm-ms-balance"
            "xm-ms-config"
            "xm-ms-config-repository"
            "xm-ms-dashboard"
            "xm-ms-entity"
            "xm-ms-otp"
            "xm-ms-scheduler"
            "xm-ms-template"
            "xm-ms-timeline"
            "xm-online"
            "xm-online-idea-plugin"
            "xm-uaa"
            "xm-webapp"
          )
          for REPO in "${REPOS[@]}"; do
            echo "--- Mirroring ${REPO} ---"
            
            git clone --mirror "https://github.com/xm-online/${REPO}.git"
            echo "Cloned ${REPO}"

            cd "${REPO}.git"
            echo "Changed directory to ${REPO}.git"
            
            echo "Setting up push mirror to xme-digital/${REPO}"
            git remote add --mirror=fetch push-mirror "https://x-access-token:${{ secrets.GH_MIRROR_TOKEN }}@github.com/xme-digital/${REPO}.git"
            echo "Configured push-mirror remote"
            
            git config remote.push-mirror.push '+refs/heads/*:refs/heads/*'
            echo "Configured push-mirror for branches"
            
            git config --add remote.push-mirror.push '+refs/tags/*:refs/tags/*'
            echo "Configured push-mirror for tags"
            
            # The next line is commented out to prevent errors with hidden GitHub refs like Pull Requests.
            # git config --add remote.push-mirror.push '+refs/pull/*/head:refs/pull/*/head'
            # echo "Configured push-mirror for pull requests"

            git push --mirror push-mirror
            echo "Pushed all refs to push-mirror"
            cd ..
            rm -rf "${REPO}.git"
            echo "--- Done with ${REPO} ---"
          done
```