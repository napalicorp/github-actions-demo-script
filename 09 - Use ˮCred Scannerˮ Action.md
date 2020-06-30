# 09 - Use "Cred Scanner" Action
1. Create new PR: `.github/workflows/approval-wf.yml` to include the action as below:
```
name: PR Approval Workflow

on: pull_request
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Credentials Scanner
        uses:  napalicorp/find-creds-action@master
        with:
          pathToSearch: '.'
          fileTypes: '.json'
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
2. Add below to `src\Web\appsettings.json` to cause a build-break in the same branch:
```
...
,"somesettings":{
      "api_key": "f400a24e4f1c1f8dbc87c0f761266a45e781e06b"
    }
```
3. Show the action in use in "Actions" tab


[< Back to TOC](README.md)