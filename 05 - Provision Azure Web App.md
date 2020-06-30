# 05 - Provision Azure Web App
## Create and Apply ARM template
1. Open `VS Code` and show `Azure Resource Manager Tools` extension
2. Crate a basic ARM template using `arm-*` snippets
3. Show the final ARM template available in `arm\azuredeploy.json` in the repo.
4. Add a step to provision web app using the ARM template as shown below:
```
name: Deployment Workflow

on:
  release:
    types: [created, edited]

env:
  AZURE_RESOURCE_GROUP_NAME: 'eshopwebprivate-aa'

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
      - name: Provision Azure Resources
        run: |
          az group deployment create -g ${{ env.AZURE_RESOURCE_GROUP_NAME }} --template-file arm/azuredeploy.json --parameters envPrefix=prep
```
5. Create a new release and show the ARM template applied in azure portal

[< Back to TOC](README.md)