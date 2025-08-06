# Azure DevOps CI/CD Pipeline for Node.js Azure Function

This README documents the step-by-step process of setting up a CI/CD pipeline in Azure DevOps for a simple **"Hello, world!"** Node.js Azure Function App.

---

## 🚀 Project Overview

We create a simple HTTP-triggered Azure Function that returns `"Hello, <name>!"` and deploy it using a 3-stage Azure DevOps pipeline:

1. **Build Stage** – Install dependencies and package the function
2. **Test Stage** – Run unit tests
3. **Deploy Stage** – Deploy the function to Azure Function App

---

## 📁 Repository Structure

Azure-function-Demo/
├── src/functions/HttpExample.js
├── package.json
├── azure-pipelines.yml

yaml
---

## 🔧 Azure DevOps Pipeline Configuration

### ✅ 1. Build Stage (CI)
Installs Node.js 20.x, runs `npm install` and `npm run build`, and archives the build output.

```yaml
stages:
- stage: Build
  displayName: "Build Stage"
  jobs:
    - job: Build
      pool:
        vmImage: 'windows-latest'
      steps:
        - task: NodeTool@0
          inputs:
            versionSpec: '20.x'
        - script: |
            npm install
            npm run build
        - task: ArchiveFiles@2
          inputs:
            rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
            archiveType: 'zip'
            archiveFile: '$(Build.ArtifactStagingDirectory)/functionapp.zip'
            replaceExistingArchive: true
        - publish: '$(Build.ArtifactStagingDirectory)/functionapp.zip'
          artifact: drop
✅ 2. Test Stage
Installs dependencies and runs your test suite.

yaml
- stage: Test
  displayName: "Test Stage"
  dependsOn: Build
  jobs:
    - job: Test
      pool:
        vmImage: 'windows-latest'
      steps:
        - task: NodeTool@0
          inputs:
            versionSpec: '20.x'
        - script: |
            npm install
            npm test
✅ 3. Deploy Stage
Downloads the build artifact and deploys to Azure using a service connection.

yaml
- stage: Deploy
  displayName: "Deploy Stage"
  dependsOn: Test
  jobs:
    - job: Deploy
      pool:
        vmImage: 'windows-latest'
      steps:
        - download: current
          artifact: drop
        - task: AzureFunctionApp@1
          inputs:
            azureSubscription: 'new-connection' 
            appType: 'functionApp'
            appName: 'my-azure-functiondemo-hga3hnakf8dkfgb0'   
            package: '$(Pipeline.Workspace)/drop/functionapp.zip'

⚡ Triggering the Pipeline
Commit and push changes to the main branch

Pipeline triggers automatically (if trigger: is defined)

Monitor each stage (Build → Test → Deploy) in the Azure DevOps UI

✅ Verifying the Deployment
Go to Azure Portal > Function App > Functions > HttpExample

Click on Get Function URL (if not anonymous)

Open in browser or Postman:
bash

https://my-azure-functiondemo-hga3hnakf8dkfgb0.canadacentral-01.azurewebsites.net/api/HttpExample

✅ Output:

Hello, World!
