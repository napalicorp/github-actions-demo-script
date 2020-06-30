# 06 - Deploy Web App

## Deploy artifact to created web app
1. Add `azure/webapps-deploy@v2` action to deploy:
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
    env:
      AZURE_WEBAPP_NAME: 'demo-napalieshop-web'
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
          az group deployment create -g ${{ env.AZURE_RESOURCE_GROUP_NAME }} --template-file arm/azuredeploy.json --parameters envPrefix=demo
      - name: Azure WebApp
        uses: Azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          slot-name: staging
          package: ./out
```
2. Show deployed web application

[< Back to TOC](README.md)