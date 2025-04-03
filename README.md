# Azure Pipelines Templates for Docker Build, Scan, and Helm Deployment

This repository contains reusable Azure Pipelines YAML templates for:
- Building and pushing Docker images
- Scanning Docker images for vulnerabilities (using Trivy)
- Deploying applications using Helm

## How to Use the Templates

In your main pipeline, you can reference these templates from this repository (or an external repository) by declaring the repository as a resource and then using the `template` keyword with the repository alias.

Below is an example pipeline that uses our templates. In this example, the external repository is referenced with the alias `myTemplatesRepo`, and we deploy to three environments (dev, staging, prod):

```yaml
trigger:
- main

resources:
  repositories:
  - repository: myTemplatesRepo        
    type: github
    name: github-user/github-repo
    endpoint: <azure service connection name>

stages:
- template: build-push-template.yml@myTemplatesRepo  
  parameters:
    dockerRegistryServiceConnection: '5x4m5pl3-1ff-2fa3-dasd2-7acf4ec678f2'
    imageRepository: 'myImage'
    containerRegistry: 'myregistry.azurecr.io'
    dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
    tag: '$(Build.BuildID)'
    agentPool: 'MyLocalPool'

- template: scan-template.yml@myTemplatesRepo  
  parameters:
    containerRegistry: 'myregistry.azurecr.io'
    containerImage: 'myregistry.azurecr.io/myImage'
    severity: 'HIGH,CRITICAL'
    trivyVersion: '0.61.0'
    agentPool: 'MyLocalPool'
    tag: 'latest'

- template: deploy-template.yml@myTemplatesRepo  
  parameters:
    helmChartPath: '$(Build.SourcesDirectory)/mychart'    
    releaseName: 'myrelease'
    baseValuesFile: '$(Build.SourcesDirectory)/env-files/values.yaml'
    envValuesFile: '$(Build.SourcesDirectory)/env-files/values-dev.yaml'
    namespace: 'dev'
    agentPool: 'MyLocalPool'
    timeout: '5m'      

- template: deploy-template.yml@myTemplatesRepo  
  parameters:
    helmChartPath: '$(Build.SourcesDirectory)/mychart'    
    releaseName: 'myrelease'
    baseValuesFile: '$(Build.SourcesDirectory)/env-files/values.yaml'
    envValuesFile: '$(Build.SourcesDirectory)/env-files/values-staging.yaml'
    namespace: 'staging'
    agentPool: 'MyLocalPool'
    timeout: '5m'  

- template: deploy-template.yml@myTemplatesRepo  
  parameters:
    helmChartPath: '$(Build.SourcesDirectory)/mychart'    
    releaseName: 'myrelease'
    baseValuesFile: '$(Build.SourcesDirectory)/env-files/values.yaml'
    envValuesFile: '$(Build.SourcesDirectory)/env-files/values-prod.yaml'
    namespace: 'prod'
    agentPool: 'MyLocalPool'
    timeout: '5m'
