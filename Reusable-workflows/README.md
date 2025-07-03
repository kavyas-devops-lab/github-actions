# Reusable-workflows
reusable workflows in github actions will provide a powerful way to avoid duplication and streamline the creation and maintainance of workflows.
instead of copy pasting same job or steps across multiple workflows you can call the reusable workflows multiple times.

## Creating a Reusable Workflow
To create a reusable workflow, you need to add a workflow_call trigger to your workflow file. This makes the workflow callable from other workflows.

```
name: Reusable workflow example
on:
  workflow_call:
    inputs:
      config-path:
        required: true
        type: string
    secrets:
      token:
        required: true
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - name: reusable workflows
      run: echo "hello from reusable workflows"
```
## Calling a Reusable Workflow
To call a reusable workflow, use the 'uses' keyword within a job. You can reference reusable workflows using the following syntax:

1.  calling from within the same repo
```
jobs:
  call-workflow:
    uses: ./.github/workflows/reusable-workflow.yml
```
2.  calling from different repos
```
jobs:
  call-workflow:
    uses: <org-name>/<repo-name>/.github/workflows/reusable-workflow.yml@main
```

IMP:
1. if reusable workflow is from same repo, no need to mention the branch name.
2. if reusable workflow is from same repo, add ./ in uses, so that it goes root directory and then .github/
3. if reusable workflow is present in other repo it is mandatory to mention branch name i.e @main
