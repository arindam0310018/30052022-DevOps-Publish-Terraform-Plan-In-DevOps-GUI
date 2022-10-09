# TERRAFORM PLAN IN DEVOPS GUI

Greetings my fellow Technology Advocates and Specialists.

In this Session, I will demonstrate how to __Publish Terraform Plan in Azure DevOps Graphical User Interface (GUI).__ 

| __LIVE RECORDED SESSION:-__ |
| --------- |
| __LIVE DEMO__ was Recorded as part of my Presentation in __AZURE BACK TO SCHOOL - 2022__ Forum/Platform |
| Duration of My Demo = __37 Mins 04 Secs__ |
| [![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/pGmy-IkytCQ/0.jpg)](https://www.youtube.com/watch?v=pGmy-IkytCQ) |


| __THIS IS HOW IT LOOKS AT THE END!!!__ |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/47i3gg8m903nrqdl89j1.png) |


| __REQUIREMENTS:-__ |
| --------- |

1. Azure Subscription.
2. Azure DevOps Organisation and Project.
3. Service Principal with Delegated Graph API Rights and Required RBAC (Typically __Contributor__ on Subscription or Resource Group)
3. Azure Resource Manager Service Connection in Azure DevOps.
4. __Azure Pipelines Terraform Tasks Extension by Charles Zipp__ Installed in Azure DevOps.


| __EXTENSION DETAILS:-__ |
| --------- |
| __NAME:__ |
| Azure Pipelines Terraform Tasks |
| __WHERE TO FIND:__ |
| https://marketplace.visualstudio.com/items?itemName=charleszipp.azure-pipelines-tasks-terraform&targetId=11c5414f-ba26-4659-87a7-aa40610cf74a&utm_source=vstsproduct&utm_medium=ExtHubManageList |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rsb5mirkb6rwafszoywb.png) | 
| __INSTALLED IN AZURE DEVOPS ORGANISATION:__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i74b8g733o0bhrjku7m9.png) |

| __HOW DOES MY CODE PLACEHOLDER LOOKS LIKE:-__ |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cpwxhzldnfpej2j9ekpa.png) |

| __OBJECTIVE:-__ |
| --------- |
| Deploy a __Resource Group__ and __Log Analytics Workspace.__|
| __Publish the Terraform Plan__ in __Azure DevOps GUI.__ |

| PIPELINE CODE SNIPPET:- | 
| --------- |

| AZURE DEVOPS YAML PIPELINE (azure-pipelines-Publish-TF-Plan-GUI-v1.0.yml):- | 
| --------- |

```
###############################
#PIPELINE TRIGGER CONDITION:-
###############################
trigger: none

######################
#DECLARE PARAMETERS:-
######################

parameters:
- name: envName
  displayName: Select Environment
  default: NonProd
  values:
  - NonProd

- name: actionToPerform
  displayName: Deploy or Destroy
  default: Deploy
  values:
  - Deploy

######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  resourceGroup: tfpipeline-rg
  storageAccount: tfpipelinesa
  storageAccountSku: Standard_LRS
  container: terraform
  tfstateFile: PUBLISH-TF-PLAN/LogaPublishTFPlan.tfstate
  BuildAgent: ubuntu-latest
  terraform_ver: latest
  workingDir: $(System.DefaultWorkingDirectory)/Publish-TF-Plan-In-GUI
  target: $(build.artifactstagingdirectory)/AMTF
  artifact: AM

#########################
# Declare Build Agents:-
#########################
pool:
  vmImage: $(BuildAgent)

###################
# Declare Stages:-
###################
stages:

- stage: PUBLISH_PLAN
  jobs:
  - job: PUBLISH
    displayName: PUBLISH
    steps:
# Install Terraform Installer in the Build Agent:-
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@0
      displayName: INSTALL TERRAFORM VERSION
      inputs:
        terraformVersion: '$(terraform_ver)'
# Terraform Init:-
    - task: TerraformCLI@0
      displayName: TERRAFORM INIT
      inputs:
        command: 'init'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        backendServiceArm: '$(ServiceConnection)' 
        backendAzureRmResourceGroupName: '$(resourceGroup)' 
        backendAzureRmStorageAccountName: '$(storageAccount)'
        backendAzureRmStorageAccountSku: '$(storageAccountSku)'
        backendAzureRmContainerName: '$(container)'
        backendAzureRmKey: '$(tfstateFile)'
# Terraform Validate:-
    - task: TerraformCLI@0
      displayName: TERRAFORM VALIDATE
      inputs:
        command: 'validate'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        environmentServiceName: '$(ServiceConnection)'
# Terraform Plan:-
    - task: TerraformCLI@0
      displayName: TERRAFORM PLAN
      inputs:
        command: 'plan'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        commandOptions: "--var-file=loga.tfvars --out=tfplan"
        environmentServiceName: '$(ServiceConnection)'
        publishPlanResults: 'tfplan'

- stage: BUILD
  jobs:
  - job: BUILD
    displayName: BUILD
    steps:
# Install Terraform Installer in the Build Agent:-
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@0
      displayName: INSTALL TERRAFORM VERSION
      inputs:
        terraformVersion: '$(terraform_ver)'
# Terraform Init:-
    - task: TerraformCLI@0
      displayName: TERRAFORM INIT
      inputs:
        command: 'init'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        backendServiceArm: '$(ServiceConnection)' 
        backendAzureRmResourceGroupName: '$(resourceGroup)' 
        backendAzureRmStorageAccountName: '$(storageAccount)'
        backendAzureRmStorageAccountSku: '$(storageAccountSku)'
        backendAzureRmContainerName: '$(container)'
        backendAzureRmKey: '$(tfstateFile)'
# Terraform Validate:-
    - task: TerraformCLI@0
      displayName: TERRAFORM VALIDATE
      inputs:
        command: 'validate'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        environmentServiceName: '$(ServiceConnection)'
# Terraform Plan:-
    - task: TerraformCLI@0
      displayName: TERRAFORM PLAN
      inputs:
        command: 'plan'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        commandOptions: "--var-file=loga.tfvars --out=tfplan"
        environmentServiceName: '$(ServiceConnection)'
# Copy Files to Artifacts Staging Directory:-
    - task: CopyFiles@2
      displayName: COPY FILES ARTIFACTS STAGING DIRECTORY
      inputs:
        SourceFolder: '$(workingDir)'
        Contents: |
          **/*.tf
          **/*.tfvars
          **/*tfplan*
        TargetFolder: '$(target)'
# Publish Artifacts:-
    - task: PublishBuildArtifacts@1
      displayName: PUBLISH ARTIFACTS
      inputs:
        targetPath: '$(target)'
        artifactName: '$(artifact)' 

- stage: DEPLOY
  condition: |
     and(succeeded(),
       eq('${{ parameters.actionToPerform }}', 'Deploy'), 
       eq(variables['build.sourceBranch'], 'refs/heads/main')
     )
  jobs:
  - deployment: 
    displayName: DEPLOY
    environment: '${{ parameters.envName }}'
    pool:
      vmImage: $(BuildAgent)
    strategy:
      runOnce:
        deploy:
          steps:
# Download Artifacts:-
          - task: DownloadBuildArtifacts@0
            displayName: DOWNLOAD ARTIFACTS
            inputs:
              buildType: 'current'
              downloadType: 'single'
              artifactName: '$(artifact)'
              downloadPath: '$(System.ArtifactsDirectory)' 
# Install Terraform Installer in the Build Agent:-
          - task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@0
            displayName: INSTALL TERRAFORM VERSION
            inputs:
              terraformVersion: '$(terraform_ver)'
# Terraform Init:-
          - task: TerraformCLI@0
            displayName: TERRAFORM INIT
            inputs:
              command: 'init'
              backendType: 'azurerm'
              workingDirectory: '$(System.ArtifactsDirectory)/$(artifact)/AMTF/' 
              backendServiceArm: '$(ServiceConnection)' 
              backendAzureRmResourceGroupName: '$(resourceGroup)' 
              backendAzureRmStorageAccountName: '$(storageAccount)'
              backendAzureRmStorageAccountSku: '$(storageAccountSku)'
              backendAzureRmContainerName: '$(container)'
              backendAzureRmKey: '$(tfstateFile)'
# Terraform Apply:-
          - task: TerraformCLI@0
            displayName: TERRAFORM APPLY 
            inputs:
              command: 'apply'
              backendType: 'azurerm'
              workingDirectory: '$(System.ArtifactsDirectory)/$(artifact)/AMTF'
              commandOptions: '--var-file=loga.tfvars'
              environmentServiceName: '$(ServiceConnection)'

```

Now, let me explain each part of YAML Pipeline for better understanding.

| PART #1:- | 
| --------- |

| BELOW FOLLOWS PIPELINE RUNTIME VARIABLES CODE SNIPPET:- | 
| --------- |

```
######################
#DECLARE PARAMETERS:-
######################

parameters:
- name: envName
  displayName: Select Environment
  default: NonProd
  values:
  - NonProd

- name: actionToPerform
  displayName: Deploy or Destroy
  default: Deploy
  values:
  - Deploy

```

| THIS IS HOW IT LOOKS WHEN YOU EXECUTE THE PIPELINE FROM AZURE DEVOPS:- | 
| --------- |

| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vdbxd39806hir2knng4i.png) | 
| --------- |


| NOTE:- | 
| --------- |
| No User Input Required. | 


| PART #2:- | 
| --------- |

| BELOW FOLLOWS PIPELINE VARIABLES CODE SNIPPET:- | 
| --------- |

```
######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  resourceGroup: tfpipeline-rg
  storageAccount: tfpipelinesa
  storageAccountSku: Standard_LRS
  container: terraform
  tfstateFile: PUBLISH-TF-PLAN/LogaPublishTFPlan.tfstate
  BuildAgent: ubuntu-latest
  terraform_ver: latest
  workingDir: $(System.DefaultWorkingDirectory)/Publish-TF-Plan-In-GUI
  target: $(build.artifactstagingdirectory)/AMTF
  artifact: AM

```

| __NOTE:-__ |
| --------- |
| Please feel free to change the values of the variables. | 
| The entire YAML pipeline is build using Parameters and variables. No Values are Hardcoded. |
| "**Working Directory**" Path should be based on your Code Placeholder. |


| PART #3:- | 
| --------- |

| __PIPELINE STAGE DETAILS FOLLOW BELOW:-__ |
| --------- |

1. This is a __3 Stage__ Pipeline with 2 Runtime Variables - 1) DevOps Environment, and 2) Action To Perform 
2. The Names of the Stages are - 1) PUBLISH_PLAN 2) BUILD, and 3) DEPLOY


| __PIPELINE STAGE - PUBLISH_PLAN:-__ |
| --------- |

```
- stage: PUBLISH_PLAN
  jobs:
  - job: PUBLISH
    displayName: PUBLISH
    steps:
# Install Terraform Installer in the Build Agent:-
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@0
      displayName: INSTALL TERRAFORM VERSION
      inputs:
        terraformVersion: '$(terraform_ver)'
# Terraform Init:-
    - task: TerraformCLI@0
      displayName: TERRAFORM INIT
      inputs:
        command: 'init'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        backendServiceArm: '$(ServiceConnection)' 
        backendAzureRmResourceGroupName: '$(resourceGroup)' 
        backendAzureRmStorageAccountName: '$(storageAccount)'
        backendAzureRmStorageAccountSku: '$(storageAccountSku)'
        backendAzureRmContainerName: '$(container)'
        backendAzureRmKey: '$(tfstateFile)'
# Terraform Validate:-
    - task: TerraformCLI@0
      displayName: TERRAFORM VALIDATE
      inputs:
        command: 'validate'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        environmentServiceName: '$(ServiceConnection)'
# Terraform Plan:-
    - task: TerraformCLI@0
      displayName: TERRAFORM PLAN
      inputs:
        command: 'plan'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        commandOptions: "--var-file=loga.tfvars --out=tfplan"
        environmentServiceName: '$(ServiceConnection)'
        publishPlanResults: 'tfplan'

```

| __PUBLISH_PLAN STAGE PERFORMS BELOW:-__ |
| --------- |

| __##__ | __TASKS__ |
| --------- | --------- |
| 1. | Terraform Installer installed in Azure DevOps Build Agent.|
| 2. | Terraform Init |
| 3. | Terraform Validate |
| 4. | Terraform Plan |
| 5. | Publish Terraform Plan in Azure DevOps GUI. |


| NOTE:- |
| --------- |

```
- task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@0

```
| EXPLANATION:- |
| --------- |
| Instead of using __TerraformInstaller@0__ YAML Task, I have specified the Full Name. This is because I have __two Terraform Extensions__ in my DevOps Organisation and with each of the Terraform Extension, exists the Terraform Install Task |
| The Names of the Extensions are listed below:- |
| 1. Terraform by Microsoft DevLabs   |
| 2. Azure Pipelines Terraform Tasks by Charles Zipp |
| If __Full Name is not provided__, then __below Error is Encountered__:- | 
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tvul3qk468st3qbshyrj.png) |
 
__Alternatively__, below can also be used as Full Name:-

```
- task: charleszipp.azure-pipelines-tasks-terraform.azure-pipelines-tasks-terraform-installer.TerraformInstaller@0

```

| DIFFERENCES BETWEEN TERRAFORM EXTENSIONS (Terraform by Microsoft DevLabs VS Azure Pipelines Terraform Tasks by Charles Zipp) :- |
| --------- |

| CATEGORY | TERRAFORM BY MICROSOFT DEVLABS | AZURE PIPELINES TERRAFORM TASKS BY CHARLES ZIPP |
| --------- | --------- | --------- |
| __Terraform Installer Task__ | - task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@0 | - task: charleszipp.azure-pipelines-tasks-terraform.azure-pipelines-tasks-terraform-installer.TerraformInstaller@0 |
| __Terraform Task__ | TerraformTaskV2@2 | TerraformCLI@0 |
| __Terraform Init Input Parameter__ | This is not Available | backendAzureRmStorageAccountSku |
| __Terraform Init, Validate, Plan and Apply Input Parameter__ | provider: 'azurerm' | backendType: 'azurerm' |
| __Terraform Validate and Plan Input Parameter__ | environmentServiceNameAzureRM | environmentServiceName |
| __Terraform Plan Input Parameter__ | This is not Available | publishPlanResults |


| HOW TERRAFORM PLAN IS GETTING PUBLISHED IN AZURE DEVOPS GUI :- |
| --------- |
| This is achieved by using the __publishPlanResults__ Input Parameters in the __Terraform Plan Task__ in __Publish_Plan Stage__ |

```
# Terraform Plan:-
    - task: TerraformCLI@0
      displayName: TERRAFORM PLAN
      inputs:
        command: 'plan'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        commandOptions: "--var-file=loga.tfvars --out=tfplan"
        environmentServiceName: '$(ServiceConnection)'
        publishPlanResults: 'tfplan'

```
| __PIPELINE STAGE - BUILD:-__ |
| --------- |

```
- stage: BUILD
  jobs:
  - job: BUILD
    displayName: BUILD
    steps:
# Install Terraform Installer in the Build Agent:-
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@0
      displayName: INSTALL TERRAFORM VERSION
      inputs:
        terraformVersion: '$(terraform_ver)'
# Terraform Init:-
    - task: TerraformCLI@0
      displayName: TERRAFORM INIT
      inputs:
        command: 'init'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        backendServiceArm: '$(ServiceConnection)' 
        backendAzureRmResourceGroupName: '$(resourceGroup)' 
        backendAzureRmStorageAccountName: '$(storageAccount)'
        backendAzureRmStorageAccountSku: '$(storageAccountSku)'
        backendAzureRmContainerName: '$(container)'
        backendAzureRmKey: '$(tfstateFile)'
# Terraform Validate:-
    - task: TerraformCLI@0
      displayName: TERRAFORM VALIDATE
      inputs:
        command: 'validate'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        environmentServiceName: '$(ServiceConnection)'
# Terraform Plan:-
    - task: TerraformCLI@0
      displayName: TERRAFORM PLAN
      inputs:
        command: 'plan'
        backendType: 'azurerm'
        workingDirectory: '$(workingDir)'
        commandOptions: "--var-file=loga.tfvars --out=tfplan"
        environmentServiceName: '$(ServiceConnection)'
# Copy Files to Artifacts Staging Directory:-
    - task: CopyFiles@2
      displayName: COPY FILES ARTIFACTS STAGING DIRECTORY
      inputs:
        SourceFolder: '$(workingDir)'
        Contents: |
          **/*.tf
          **/*.tfvars
          **/*tfplan*
        TargetFolder: '$(target)'
# Publish Artifacts:-
    - task: PublishBuildArtifacts@1
      displayName: PUBLISH ARTIFACTS
      inputs:
        targetPath: '$(target)'
        artifactName: '$(artifact)'

```

| __BUILD STAGE PERFORMS BELOW:-__ |
| --------- |

| __##__ | __TASKS__ |
| --------- | --------- |
| 1. | Terraform Installer installed in Azure DevOps Build Agent.|
| 2. | Terraform Init |
| 3. | Terraform Validate |
| 4. | Terraform Plan |
| 5. | Copy the Terraform files (Most Importantly __Terraform Plan Output__) to Artifacts Staging Directory. |
| 6. | Publish Artifacts |


| __PIPELINE STAGE - DEPLOY:-__ |
| --------- |

```
- stage: DEPLOY
  condition: |
     and(succeeded(),
       eq('${{ parameters.actionToPerform }}', 'Deploy'), 
       eq(variables['build.sourceBranch'], 'refs/heads/main')
     )
  jobs:
  - deployment: 
    displayName: DEPLOY
    environment: '${{ parameters.envName }}'
    pool:
      vmImage: $(BuildAgent)
    strategy:
      runOnce:
        deploy:
          steps:
# Download Artifacts:-
          - task: DownloadBuildArtifacts@0
            displayName: DOWNLOAD ARTIFACTS
            inputs:
              buildType: 'current'
              downloadType: 'single'
              artifactName: '$(artifact)'
              downloadPath: '$(System.ArtifactsDirectory)' 
# Install Terraform Installer in the Build Agent:-
          - task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@0
            displayName: INSTALL TERRAFORM VERSION
            inputs:
              terraformVersion: '$(terraform_ver)'
# Terraform Init:-
          - task: TerraformCLI@0
            displayName: TERRAFORM INIT
            inputs:
              command: 'init'
              backendType: 'azurerm'
              workingDirectory: '$(System.ArtifactsDirectory)/$(artifact)/AMTF/' 
              backendServiceArm: '$(ServiceConnection)' 
              backendAzureRmResourceGroupName: '$(resourceGroup)' 
              backendAzureRmStorageAccountName: '$(storageAccount)'
              backendAzureRmStorageAccountSku: '$(storageAccountSku)'
              backendAzureRmContainerName: '$(container)'
              backendAzureRmKey: '$(tfstateFile)'
# Terraform Apply:-
          - task: TerraformCLI@0
            displayName: TERRAFORM APPLY 
            inputs:
              command: 'apply'
              backendType: 'azurerm'
              workingDirectory: '$(System.ArtifactsDirectory)/$(artifact)/AMTF'
              commandOptions: '--var-file=loga.tfvars'
              environmentServiceName: '$(ServiceConnection)'

```

| __DEPLOY STAGE PERFORMS BELOW:-__ |
| --------- |

| __##__ | __TASKS__ |
| --------- | --------- |
| 1. | __DEPLOY__ Stage will Execute only if the following conditions are met -  1) __BUILD__ Stage gets completed successfully.  2) Option __Deploy__ is selected from DevOps Runtime Parameters 3) Source Branch = Main. If not, __DEPLOY__ Stage will get Skipped Automatically. |
| 2. | __DEPLOY__ Stage will Execute only after Approval. The Approval is integrated with Environment defined in the Pipeline Parameters Section and applied in Deploy Stage.|
| 3. | Download the Published Artifacts. |
| 4. | Terraform Installer installed in Azure DevOps Build Agent.|
| 5. | Terraform Init |
| 6. | Terraform Apply |


| __DETAILS AND ALL TERRAFORM CODE SNIPPETS FOLLOWS BELOW:-__ |
| --------- |

| TERRAFORM (main.tf):- | 
| --------- |

```
terraform {
  required_version = ">= 1.2.0"

   backend "azurerm" {
    resource_group_name  = "tfpipeline-rg"
    storage_account_name = "tfpipelinesa"
    container_name       = "terraform"
    key                  = "PUBLISH-TF-PLAN/LogaPublishTFPlan.tfstate"
  }
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.2"
    }   
  }
}
provider "azurerm" {
  features {}
  skip_provider_registration = true
}

```

| TERRAFORM (loga.tf):- | 
| --------- |

```
## Azure Resource Group:-
resource "azurerm_resource_group" "rg" {
  name     = var.rg-name
  location = var.rg-location
}

## Azure log Analytics Workspace:-

resource "azurerm_log_analytics_workspace" "loga" {
  name                = var.loga-name
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  sku                 = var.loga-sku 
  retention_in_days   = var.loga-retention

  depends_on          = [azurerm_resource_group.rg]
}

```

| TERRAFORM (variables.tf):- | 
| --------- |

```
variable "rg-name" {
  type        = string
  description = "Name of the Resource Group"
}

variable "rg-location" {
  type        = string
  description = "Resource Group Location"
}

variable "loga-name" {
  type        = string
  description = "Name of the Log Analytics Workspace"
}

variable "loga-sku" {
  type        = string
  description = "SKU the Log Analytics Workspace"
}

variable "loga-retention" {
  type        = string
  description = "Retention Period of the Log Analytics Workspace"
}

```

| TERRAFORM (loga.tfvars):- | 
| --------- |

```
rg-name         = "AMTESTRG100"
rg-location     = "West Europe"
loga-name       = "AMLOGA100"
loga-sku        = "PerGB2018"
loga-retention  = "30"

```

| __ITS TIME TO TEST:-__ |
| --------- |
| __DESIRED RESULT__: Stages - __PUBLISH_PLAN__, __BUILD__ and __DEPLOY__ should Complete Successfully. __Terraform Plan__ Gets __Published__ Successfully in Azure DevOps GUI. __Resource Group and Log Analytics Workspace__ Resources gets deployed. |
| __PIPELINE RUNTIME PARAMETERS WITH POPULATED VALUES:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vdbxd39806hir2knng4i.png) |
| __PIPELINE STAGE PUBLISH_PLAN EXECUTED SUCCESSFULLY:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wxdbse1eb4gu4paw215o.png) |
| __TERRAFORM PLAN GETS PUBLISHED IN AZURE DEVOPS GUI:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/47i3gg8m903nrqdl89j1.png) |
| __PIPELINE STAGE BUILD EXECUTED SUCCESSFULLY:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dr7n3fwghlcyqlv5ffs6.png) |
| __PIPELINE STAGE DEPLOY WAITING APPROVAL:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wzjlklpsu5ibwa5edewz.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xlp3vq6pm0lxf6b298k3.png) |
| __PIPELINE STAGE DEPLOY EXECUTED SUCCESSFULLY:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/od3o06kevpu0gffo9cqb.png) |
| __PIPELINE OVERALL EXECUTION STATUS:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lprvjqopm4svpwvugo55.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wvvandofuz89sa1x9az3.png) |
| __VALIDATE RESOURCES DEPLOYED IN PORTAL:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ej3p2khzbg0eve8v0kok.png) |

| __IMPORTANT TO NOTE:-__ |
| --------- |
| Terraform Plan are __NOT PUBLISHED__ in Azure DevOps GUI unless there is a change in Infrastructure - __ADD__, __DESTROY__ or __CHANGE__ |
| In order to demonstrate, the same pipeline was re-run. |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l5zo1swcpsuye6575dtu.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k4qsr6hrutl5a6e5jypk.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g0i35z11u12w366b6mmm.png) |

Hope You Enjoyed the Session!!!

__Stay Safe | Keep Learning | Spread Knowledge__
