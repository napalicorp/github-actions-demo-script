# 03 - Create an Artifact on Release

## Produce and Artifact
1. Navigate to `Actions -> New Workflow` and create a workflow from the template "Deploy Node.js to Azure Web App"
2. Rename the file to "deployment-wf"
3. Paste below workflow:
```
name: Deployment Workflow

on:
  release:
    types: [created, edited]

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
      - name: Publish Web App
        run: dotnet publish src/Web -c Release -o out
      - name: Upload a Build Artifact
        uses: actions/upload-artifact@v2
        with:
          name: eshop-web-package
          path: out/
      
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build]
    steps:
      - uses: actions/checkout@v2
      - name: Download a Build Artifact
        uses: actions/download-artifact@v2
        with:
          name: eshop-web-package
          path: out
```
4. Commit directly to `master`
5. Navigate to `Code -> Releases -> Create Release`
6. Create `0.1.0` release as a **pre-release**
7. Show the workflow run in "Actions" and show the generated artifact

[< Back to TOC](README.md)