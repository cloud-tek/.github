# workflows.md

All workflows and workflow templates provided by this repository are located under:
- `.github/workflows`
- `.github/workflow-templates`

## Critical Rule: Naming conventions for workflows

When working with workflows the following elements need to strictly adhere to the following rules:

### Workflow inputs

- Use kebab-case (e.g., `image-name`, `deploy-environment`) — this is what GitHub uses in their docs and examples
- Keep names descriptive but concise

### Workflow outputs

- Use kebab-case (e.g., `artifact-path`, `build-version`) — this is the prevalent convention
- Keep names descriptive but concise

### Workflow secrets

- SCREAMING_SNAKE_CASE is the strong convention (e.g., `NPM_TOKEN`, `DEPLOY_KEY`)
- This aligns with environment variable conventions and makes secrets visually distinct from regular inputs
- GitHub's built-in secrets like `GITHUB_TOKEN` follow this pattern

### Complete Example

```yml
on:
  workflow_call:
    inputs:
      environment-name:
        required: true
        type: string
      enable-cache:
        required: false
        type: boolean
        default: true
    secrets:
      REGISTRY_PASSWORD:
        required: true
    outputs:
      deployed-url:
        description: "The deployment URL"
        value: ${{ jobs.deploy.outputs.url }}
```