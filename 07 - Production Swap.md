---
tags: [GithubActions Demo Script]
title: 07 - Production Swap
created: '2020-06-27T09:42:04.567Z'
modified: '2020-06-27T09:47:15.168Z'
---

# 07 - Production Swap
## Swap Staging/Production slot on Release != "pre-release"
1. Product slot swap happens when Release is edited to uncheck **pre-release**
2. `build` and `deploy-stagin` jobs should only run on `pre-release` mode
3. New `deploy-prod` job to swap web app slots:
```
name: Deployment Workflow

on:
  release:
    types: [created, edited]

env:
  AZURE_RESOURCE_GROUP_NAME: 'eshopwebprivate-aa'

jobs:
  build:
    if: "github.event.release.prerelease"
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
    if: "github.event.release.prerelease"
    env:
      AZURE_WEBAPP_NAME: 'prep-napalieshop-web'
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
      - name: Azure WebApp
        uses: Azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          slot-name: staging
          package: ./out
        
  deploy-prod:
    if: "!github.event.release.prerelease"
    env:
      AZURE_WEBAPP_NAME: 'prep-napalieshop-web'
    runs-on: ubuntu-latest
    steps:
      - name: Azure Login
        uses: Azure/login@v1.1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      - name: Swap staging and prod slots
        run: |
          az webapp deployment slot swap  -g ${{ env.AZURE_RESOURCE_GROUP_NAME }} -n ${{ env.AZURE_WEBAPP_NAME }} --slot staging --target-slot production
```
4. Edit a release to uncheck "pre-release" and modify the text as well.
  **_Note_**: _Change release text, otherwise action won't be triggerred_
5. Show the final deployed production application

[< Back to TOC](README.md)