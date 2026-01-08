# cloud-tek/.github

Central repository for reusable GitHub Actions workflows and templates for the cloud-tek organization.

## Overview

This repository provides:
- **Reusable Workflows** - Callable workflows for common CI/CD tasks
- **Workflow Templates** - Starter workflows that appear in GitHub's "Actions" tab

## Reusable Workflows

### dagger-cloudtek-build

Dagger-based build workflow for .NET projects using CloudTek.Build tooling.

**Location:** `.github/workflows/dagger-cloudtek-build.yml`

**Usage:**

```yaml
jobs:
  build:
    uses: cloud-tek/.github/.github/workflows/dagger-cloudtek-build.yml@v0.3.0
    with:
      directory: .
      dagger-module: github.com/cloud-tek/dagger/dotnet/cloudtek-build@v0.2.0
      cloudtek-build-version: '10.0.0'
      net-version: '10.0'
      net-sdk-version: '10.0.100'
      target: 'All'
      skip: ''
    secrets:
      GH_TOKEN: ${{ secrets.CLOUDTEK_AUTOMATION_TOKEN }}
      NUGET_API_KEY: ${{ secrets.NUGET_API_KEY }}
      DAGGER_CLOUD_TOKEN: ${{ secrets.DAGGER_CLOUD_TOKEN }}
      DAGGER_SSH_PRIVATE_KEY: ${{ secrets.DAGGER_SSH_PRIVATE_KEY }}
```

**Inputs:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `directory` | string | yes | - | Project directory |
| `net-version` | string | no | `10.0` | .NET version |
| `net-sdk-version` | string | no | `10.0.100` | .NET SDK version |
| `dagger-module` | string | no | `github.com/cloud-tek/dagger/dotnet/cloudtek-build@v0.2.0` | Dagger module to use |
| `cloudtek-build-version` | string | no | `10.0.0` | CloudTek.Build.Tool version |
| `target` | string | no | `All` | NUKE build target |
| `skip` | string | no | `""` | NUKE targets to skip (comma-separated) |

**Secrets:**

| Name | Required | Description |
|------|----------|-------------|
| `GH_TOKEN` | no | GitHub token for private repo access |
| `NUGET_API_KEY` | no | NuGet API key for package operations |
| `DAGGER_CLOUD_TOKEN` | no | Dagger Cloud authentication token |
| `DAGGER_SSH_PRIVATE_KEY` | yes | SSH private key (dagger@cloud-tek.io) |

**Outputs:**

| Name | Description |
|------|-------------|
| `artifact-name` | Name of the uploaded artifact (typically `artifacts`) |

**Features:**
- Full git history checkout
- SSH configuration for private repository access
- Automatic `global.json` creation if missing
- Artifact upload (NuGet packages, build outputs)
- XUnit test reporting from `.trx` files
- Dagger Cloud integration

## Workflow Templates

Templates appear in the "Actions" tab when creating new workflows in organization repositories.

### claude-code-review

Automated PR code review using Claude AI.

**File:** `workflow-templates/claude-code-review.yml`

**Triggers:** PR opened, synchronized, reopened

**Features:**
- Runs `/dotnet-code-review` command
- 25 max conversation turns
- Creates inline PR comments
- Uses Claude Sonnet 4.5

**Required Secrets:**
- `CLOUDTEK_AUTOMATION_TOKEN`
- `CLAUDE_CODE_OAUTH_TOKEN`

### claude

Comment-driven Claude assistance on issues and PRs.

**File:** `workflow-templates/claude.yml`

**Triggers:** Issue/PR comments mentioning @claude

**Features:**
- Interactive Claude assistance
- Comment-based workflow execution
- Uses Claude Sonnet 4.5

**Required Secrets:**
- `CLOUDTEK_AUTOMATION_TOKEN`
- `CLAUDE_CODE_OAUTH_TOKEN`

### dotnet-validate-pr

.NET PR validation using the Dagger build workflow.

**File:** `workflow-templates/dotnet-validate-pr.yml`

**Triggers:**
- PR events on default branch
- Changes to: workflows, `.sln`, `.*sproj`, `.cs` files
- Manual workflow dispatch

**Features:**
- Full .NET build validation
- Test execution and reporting
- NuGet package creation
- Artifact generation

**Required Secrets:**
- `CLOUDTEK_AUTOMATION_TOKEN`
- `NUGET_API_KEY`
- `DAGGER_CLOUD_TOKEN`
- `DAGGER_SSH_PRIVATE_KEY`

## Version History

### v0.3.0 (Breaking Changes)

**Breaking:** Workflow inputs and outputs now use `kebab-case` instead of `PascalCase`:
- `Directory` → `directory`
- `NetVersion` → `net-version`
- `NetSdkVersion` → `net-sdk-version`
- `DaggerModule` → `dagger-module`
- `CloudTekBuildVersion` → `cloudtek-build-version`
- `Target` → `target`
- `Skip` → `skip`
- `ArtifactName` → `artifact-name`

Repositories calling `dagger-cloudtek-build.yml` must update their workflow files.

### v0.2.0

- Artifact emission support
- Upload artifact version fixes

## Development Guidelines

### Naming Conventions

Following GitHub Actions best practices:

**Workflow Inputs/Outputs:** Use `kebab-case`
```yaml
inputs:
  environment-name:
    type: string
outputs:
  artifact-path:
    value: ${{ jobs.build.outputs.path }}
```

**Secrets:** Use `SCREAMING_SNAKE_CASE`
```yaml
secrets:
  REGISTRY_PASSWORD:
    required: true
```

See `.claude/rules/workflows-conventions.md` for complete guidelines.

### Testing Changes

1. **Workflow Templates** - Must be merged to `main` to appear in GitHub UI
2. **Reusable Workflows** - Tag with version (e.g., `v0.3.0`) after testing
3. **Breaking Changes** - Use semver major version bump or clear version increment

### Required Secrets

Organization-level secrets required:
- `CLOUDTEK_AUTOMATION_TOKEN` - GitHub PAT for automation
- `CLAUDE_CODE_OAUTH_TOKEN` - Claude Code API token
- `NUGET_API_KEY` - NuGet.org API key
- `DAGGER_CLOUD_TOKEN` - Dagger Cloud authentication
- `DAGGER_SSH_PRIVATE_KEY` - SSH key for private repos (dagger@cloud-tek.io)

## Contributing

1. Create a feature branch
2. Make changes following naming conventions
3. Test in a separate repository
4. Submit PR with clear description
5. Tag release after merge (for reusable workflows)

## Resources

- [GitHub Actions: Reusing Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [GitHub Actions: Workflow Templates](https://docs.github.com/en/actions/using-workflows/creating-starter-workflows-for-your-organization)
- [Dagger Documentation](https://docs.dagger.io)
- [Claude Code Documentation](https://claude.ai/code)

## License

Internal use within the cloud-tek organization.
