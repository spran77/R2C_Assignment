1. **Build Pipeline CI**:

- Service: Azure Pipelines
- Purpose: Build and test the application
- Configuration:Trigger on code commit or pull request
  Example YAML configuration for pipeline as code
  Including tasks for restoring dependencies, building the application, running tests, and publishing artifacts

```yml
trigger:
  - main
resources:
  repositories:
    - repository: self
      type: git
      name: <your-project-name>/<your-repo-name>

pool:
  vmImage: "ubuntu-latest"

steps:
  - task: UseDotNet@2
    inputs:
      packageType: "sdk"
      version: "6.x"
      installationPath: $(Agent.ToolsDirectory)/dotnet

  - task: DotNetCoreCLI@2
    inputs:
      command: "restore"
      projects: "**/*.csproj"

  - task: DotNetCoreCLI@2
    inputs:
      command: "build"
      projects: "**/*.csproj"

  - task: DotNetCoreCLI@2
    inputs:
      command: "test"
      projects: "**/*.csproj"
      arguments: "--configuration $(buildConfiguration)"

  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: "$(Build.ArtifactStagingDirectory)"
      artifactName: "drop"
```

2. **Release Pipeline**:

- Service: Azure Pipelines
- Purpose: Deploy the application to different environments (e.g., Dev, Test, Prod)
- Configuration: Stages for each environment
  deployment slots for zero-downtime deployment
  Configured approvals and gates for promoting releases
  Example YAML configuration for pipeline as code

```yml
trigger: none

resources:
  pipelines:
    - pipeline: buildPipeline # Name of the resource to be used in this pipeline
      source: <your-build-pipeline-name> # Name of the build pipeline that produces the artifacts
      trigger:
        branches:
          include:
            - main

stages:
  - stage: Staging
    jobs:
      - job: DeployStaging
        pool:
          vmImage: "ubuntu-latest"
        steps:
          - download: current
            artifact: drop

          - task: AzureWebApp@1
            inputs:
              azureSubscription: "YOUR_AZURE_SUBSCRIPTION"
              appName: "your-staging-app-name"
              package: $(System.DefaultWorkingDirectory)/drop/*.zip

          - task: AzureKeyVault@1
            inputs:
              azureSubscription: "YOUR_AZURE_SUBSCRIPTION"
              KeyVaultName: "your-keyvault-name"
              SecretsFilter: "*"

          - task: AzureResourceManagerTemplateDeployment@3
            inputs:
              deploymentScope: "Resource Group"
              azureResourceManagerConnection: "YOUR_AZURE_SUBSCRIPTION"
              action: "Create Or Update Resource Group"
              resourceGroupName: "your-resource-group-name"
              location: "your-resource-location"
              templateLocation: "Linked artifact"
              csmFile: "$(System.DefaultWorkingDirectory)/drop/your-template-file.json"
              csmParametersFile: "$(System.DefaultWorkingDirectory)/drop/your-parameters-file.json"

  - stage: Production
    jobs:
      - job: DeployProduction
        pool:
          vmImage: "ubuntu-latest"
        steps:
          - download: current
            artifact: drop

          - task: AzureWebApp@1
            inputs:
              azureSubscription: "YOUR_AZURE_SUBSCRIPTION"
              appName: "your-production-app-name"
              package: $(System.DefaultWorkingDirectory)/drop/*.zip

          - task: AzureKeyVault@1
            inputs:
              azureSubscription: "YOUR_AZURE_SUBSCRIPTION"
              KeyVaultName: "your-keyvault-name"
              SecretsFilter: "*"

          - task: AzureResourceManagerTemplateDeployment@3
            inputs:
              deploymentScope: "Resource Group"
              azureResourceManagerConnection: "YOUR_AZURE_SUBSCRIPTION"
              action: "Create Or Update Resource Group"
              resourceGroupName: "your-resource-group-name"
              location: "your-resource-location"
              templateLocation: "Linked artifact"
              csmFile: "$(System.DefaultWorkingDirectory)/drop/your-template-file.json"
              csmParametersFile: "$(System.DefaultWorkingDirectory)/drop/your-parameters-file.json"
```
