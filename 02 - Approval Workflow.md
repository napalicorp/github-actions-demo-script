# 02 - Approval Workflow

## Create a new Workflow for Pull Request Approvals

1. Go to **napalicorp/eshopweb-demo** repo.
2. Go to **Actions->New Worklfow**
3. Select .NET Core Template
  **_Note_**: _Templates can be maintained at org-level `.github` repo_
4. Name workflow file `approval-wf.yml`
5. Remove content and past below:
```
name: PR Approval Workflow

on: pull_request

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: 3.1.101
      - name: Restore Nuget
        run: dotnet restore eShopOnWeb.sln
      - name: Build Projects
        run: dotnet build eShopOnWeb.sln
      - name: Run Unit Tests
        run: dotnet test tests/UnitTests
      - name: Run Integration Tests
        run: dotnet test tests/IntegrationTests
```
6. Understand the similarities and the differences:
  - Familiar editor experience
  - Trigger `on` supports many events. E.g. code push, PR, release label, new issue.
  - Only set triggers are considered unlike Azure Pipelines where `pr: none`.
  - Support `workflow -> jobs -> steps` unlike `pipeline -> stages -> jobs -> steps`.
  - Yaml structure is strict and cannot ignore `jobs -> job-name` if there is only one job like in Azure Pipelines.
  - GA `uses: action-repo/action` instead of `tasks` in Azure Pipelines
  - GA `run` instead of `script` in Azure Pipelines

7. Push to new pull request branch to trigger a workflow.
8. Navigate to `Actions -> Workflow -> build`
9. Show options: logs, search, job-rerun

[< Back to TOC](README.md)