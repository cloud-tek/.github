# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the `.github` repository for the cloud-tek organization. It contains:
- **Reusable workflow templates** (`workflow-templates/`) that appear in the GitHub UI when creating new workflows in other repositories
- **Reusable workflows** (`.github/workflows/`) that can be called from other repositories

## Architecture

### Workflow Templates
Located in `workflow-templates/`, these are starter workflows that appear in GitHub's workflow creation UI. Each template consists of:
- A `.yml` workflow file
- A `.properties.json` metadata file that defines:
  - `name`: Display name in GitHub UI
  - `description`: Template description
  - `iconName`: Octicon icon to display
  - `categories`: Tags for filtering
  - `filePatterns`: Which files must exist for this template to be suggested

**Available Templates:**
1. **claude-code-review.yml**: Automated PR code review using Claude AI
   - Triggers on PR open/sync/reopen
   - Uses `anthropics/claude-code-action@v1`
   - Requires secrets: `CLOUDTEK_AUTOMATION_TOKEN`, `CLAUDE_CODE_OAUTH_TOKEN`
   - Runs `/dotnet-code-review` command with 25 max turns
   - Uses `claude-sonnet-4-5-20250929` model
   - Limited to specific tools for PR comments and viewing

2. **claude.yml**: Claude comment-driven workflow
   - Triggers on issue/PR comments mentioning @claude
   - Uses `anthropics/claude-code-action@v1`
   - Requires secrets: `CLOUDTEK_AUTOMATION_TOKEN`, `CLAUDE_CODE_OAUTH_TOKEN`
   - Uses `claude-sonnet-4-5-20250929` model

3. **dotnet-validate-pr.yml**: .NET PR validation using Dagger
   - Triggers on PR events for .NET-related files
   - Calls reusable workflow `dagger/cloudtek-build.yml@v0.2.0`
   - Uses Dagger module: `git@github.com/cloud-tek/dagger/dotnet/cloudtek-build@v0.2.0`
   - Requires secrets: `CLOUDTEK_AUTOMATION_TOKEN`, `NUGET_API_KEY`, `DAGGER_CLOUD_TOKEN`, `DAGGER_SSH_PRIVATE_KEY`

### Reusable Workflows
Located in `.github/workflows/`, these can be called from other repositories using `uses:`.

**dagger-cloudtek-build.yml**: Reusable Dagger-based build workflow
- Called via `workflow_call` with inputs and secrets
- Configurable inputs:
  - `Directory`: Project directory
  - `NetVersion` / `NetSdkVersion`: .NET version configuration
  - `DaggerModule`: Which Dagger module to use
  - `CloudTekBuildVersion`: Build tool version
  - `Target`: NUKE build target (default: "All")
  - `Skip`: NUKE targets to skip
- Outputs: `ArtifactName` for downstream jobs
- Steps:
  1. Checkout with full history
  2. Configure git for SSH access to private repos
  3. Setup SSH agent with private key
  4. Create/verify `global.json` for .NET SDK version
  5. Run Dagger build module
  6. Upload build artifacts to `artifacts/` (unless Pack/Publish skipped)
  7. Display results and artifacts trees
  8. Generate XUnit test report from `.trx` files

## Common Workflows

### Adding a New Workflow Template
1. Create the workflow YAML file in `workflow-templates/`
2. Create the corresponding `.properties.json` file with metadata
3. Ensure `filePatterns` accurately reflects when the template should be suggested
4. Test by checking if it appears in another repo's workflow creation UI

### Modifying the Reusable Build Workflow
The `dagger-cloudtek-build.yml` is critical infrastructure. Changes should:
- Maintain backward compatibility with existing callers
- Update version tags when making breaking changes
- Test with actual .NET projects before merging

### Testing Workflow Changes
- Workflow templates: Must be in the `main` branch to appear in GitHub UI
- Reusable workflows: Typically versioned with git tags (e.g., `@v0.2.0`)
- Test reusable workflows by calling them from a test repository first

## Repository-Specific Notes

- All Claude workflows require organization-level secrets for authentication
- The Dagger workflow uses SSH for private repository access (configured via `git config --global url."git@github.com:".insteadOf`)
- Artifacts are uploaded to a standard name (`artifacts`) for consistency across projects
- Test results are always reported via `dorny/test-reporter` even on failure
