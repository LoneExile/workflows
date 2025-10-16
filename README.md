# GitLeak Workflow Distribution

Automated workflow distribution system that copies security scanning workflows to multiple repositories using GitHub App authentication.

## Overview

This repository contains GitHub Actions workflows that automatically distribute a security scanning workflow (gitleaks) to multiple target repositories. It uses GitHub App authentication for secure, scalable access across your organization.

## Features

- **GitHub App Authentication**: Uses GitHub App tokens instead of Personal Access Tokens for better security and higher rate limits
- **Batch Repository Processing**: Deploy workflows to multiple repositories in a single run
- **Flexible Repository Naming**: Supports both `repo-name` and `owner/repo-name` formats
- **Dry Run Mode**: Preview changes before creating pull requests
- **Automated PR Creation**: Creates pull requests in target repositories with customizable titles and descriptions
- **Comprehensive Reporting**: Provides detailed summaries of deployment status for each repository

## Prerequisites

### 1. Create a GitHub App

You need to create a GitHub App with the following permissions:

**Repository Permissions:**
- Contents: Read and Write
- Pull Requests: Read and Write
- Metadata: Read-only (automatically granted)

**Steps to create a GitHub App:**

1. Navigate to your organization settings or personal account settings
2. Go to **Developer settings** → **GitHub Apps** → **New GitHub App**
3. Fill in the required fields:
   - **GitHub App name**: Choose a descriptive name (e.g., "Workflow Distribution Bot")
   - **Homepage URL**: Your repository or organization URL
   - **Webhook**: Uncheck "Active" (not needed for this use case)
4. Set the permissions as listed above
5. Click **Create GitHub App**
6. After creation, note down the **App ID**
7. Generate a private key:
   - Scroll to **Private keys** section
   - Click **Generate a private key**
   - Save the downloaded `.pem` file securely

### 2. Install the GitHub App

1. Go to your GitHub App settings
2. Click **Install App** in the left sidebar
3. Select the organization or account where you want to install it
4. Choose either:
   - **All repositories** (recommended for organization-wide deployment)
   - **Only select repositories** (if you want to limit access)
5. Complete the installation

### 3. Configure Repository Secrets

Add the following secrets to this repository:

1. Navigate to **Settings** → **Secrets and variables** → **Actions**
2. Add the following secrets:

   - **GITLEAKSCAN_ID**: The App ID from your GitHub App (found in app settings)
   - **GITLEAKSCAN_PRIVATE_KEY**: The entire contents of the `.pem` file you downloaded
     - Open the `.pem` file in a text editor
     - Copy everything including `-----BEGIN RSA PRIVATE KEY-----` and `-----END RSA PRIVATE KEY-----`
     - Paste the entire content as the secret value

## Usage

### Running the Workflow

1. Navigate to **Actions** tab in this repository
2. Select **Copy Workflow to Repos** from the workflows list
3. Click **Run workflow**
4. Fill in the parameters:

   - **repos**: Comma or space-separated list of repository names
     - Examples: 
       - `repo1, repo2, repo3` (uses current organization)
       - `myorg/repo1, myorg/repo2` (explicit owner)
       - `repo1 repo2` (space-separated)
   
   - **target_path**: Where to place the workflow file in target repos
     - Default: `.github/workflows/self-scan.yml`
   
   - **pr_title**: Title for the pull request
     - Default: `chore: add security scan workflow`
   
   - **pr_body**: Description for the pull request
     - Default: `Adds automated security scanning workflow with gitleaks.`
   
   - **dry_run**: Preview mode
     - `true`: Lists what would be done without creating PRs
     - `false`: Actually creates PRs (default)

5. Click **Run workflow**

### Example Scenarios

#### Deploy to multiple repos in your organization:
```
repos: frontend-app, backend-api, data-processor
target_path: .github/workflows/self-scan.yml
dry_run: false
```

#### Test deployment with dry run:
```
repos: test-repo-1, test-repo-2
dry_run: true
```

#### Deploy to repos across different organizations:
```
repos: myorg/repo1, otherorg/repo2, personal/repo3
```

## How It Works

1. **Token Generation**: The workflow generates a short-lived GitHub App token with the necessary permissions
2. **Source Checkout**: Checks out this repository to access the source workflow file
3. **Repository Processing**: For each target repository:
   - Validates the repository exists
   - Clones the target repository
   - Creates a new branch
   - Copies the workflow file to the specified path
   - Commits and pushes the changes
   - Creates a pull request
4. **Reporting**: Generates a summary showing success/failure for each repository

## Workflow Files

### Primary Workflows

- **`copy-workflow-to-repos.yml`**: Main distribution workflow that copies files to multiple repositories
- **`self-scan.yml`**: The security scanning workflow that gets distributed to target repositories

### Source Workflow

The workflow being distributed (`.github/workflows/self-scan.yml`) runs gitleaks to scan for secrets and credentials in your code. Once deployed, it will:

- Run on push to main/master branches
- Run on pull requests
- Scan for exposed secrets using gitleaks
- Report findings as workflow annotations

## Troubleshooting

### Common Issues

**"Resource not accessible by integration" error:**
- Ensure your GitHub App has the correct permissions (Contents: Write, Pull Requests: Write)
- Verify the app is installed on the target repositories
- Check that the GITLEAKSCAN_ID and GITLEAKSCAN_PRIVATE_KEY secrets are correctly configured

**"Repository not found" error:**
- Verify the repository name is correct
- Ensure the GitHub App is installed with access to that repository
- Check that the repository exists and is accessible

**Clone failures:**
- Verify network connectivity
- Check that the GitHub App token has sufficient permissions
- Ensure the default branch name is correct

**No PRs created:**
- Check if dry_run is set to `true`
- Review the workflow run logs for specific errors
- Verify the source workflow file exists at `.github/workflows/self-scan.yml`

### Debugging

Enable debug logging by setting repository secrets:
- `ACTIONS_STEP_DEBUG`: `true`
- `ACTIONS_RUNNER_DEBUG`: `true`

View detailed logs in the Actions tab for each workflow run.

## Security Considerations

- **GitHub App tokens are short-lived**: They automatically expire after 1 hour, limiting exposure
- **Principle of least privilege**: Only grant the minimum required permissions to your GitHub App
- **Private key security**: Store the GITLEAKSCAN_PRIVATE_KEY secret securely and never commit it to version control
- **Audit trail**: All PRs created are tracked and can be reviewed before merging
- **Organization-level control**: GitHub App installations can be managed at the organization level

## Benefits Over PAT (Personal Access Token)

1. **Better security**: Tokens are short-lived and scoped to specific repositories
2. **Higher rate limits**: GitHub Apps get higher API rate limits
3. **Organization ownership**: Not tied to a specific user account
4. **Fine-grained permissions**: More granular control over access
5. **Audit logging**: Better tracking of app activity
6. **No user impersonation**: Actions are performed as the app, not a user

## Contributing

To modify the distributed workflow:

1. Edit `.github/workflows/self-scan.yml` in this repository
2. Run the distribution workflow to deploy changes
3. Target repositories will receive PRs with the updated workflow

## License

MIT License - See LICENSE file for details

## Support

For issues or questions:
- Open an issue in this repository
- Check the GitHub Actions logs for detailed error messages
- Review the troubleshooting section above
