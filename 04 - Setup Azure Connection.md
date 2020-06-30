
# 04 - Setup Azure Connection

## Setup Service Principal
1. Open [http://portal.azure.com](http://portal.azure.com)
2. Use "Cloud Shell" to execute below command:
```
az ad sp create-for-rbac --name "eshopwebprivate-dep-creds-aa" --role contributor --scopes /subscriptions/f516f70b-8870-404d-aa5a-153e929ce9c4/resourceGroups/eshopwebprivate-aa --sdk-auth

```
3. Explain the resulting credential, that looks like below:
```
{
  "appId": "clientid-goes-in-here",
  "displayName": "githubactions-dep-creds-aa",
  "name": "http://githubactions-dep-creds-aa",
  "password": "clientsecretgoes.inhere",
  "tenant": "6767677-8622-47d2-aa7442-ddbba4584471"
}
```
4. Go to "Settings -> Secrets" and create a new secret named `AZURE_CREDENTIALS`
5. Paste the output JSON from above step
6. Update the deployment workflow to below:
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
      - name: Azure Login
        uses: Azure/login@v1.1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
```
7. Show execution in "Actions"

[< Back to TOC](README.md)