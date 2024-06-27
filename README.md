# Deploy-Docker-containers-to-Azure-App-Service-web-apps
## Overview
This project utilises an Azure DevOps CI/CD pipeline for creating a personalised Docker image, uploading it to Azure Container Registry, and launching it as a container on Azure App Service.
We will create a personalised `Docker image` using a `Linux agent hosted` by Microsoft. Upload the image to `Azure Container Registry`.
Finally, we will deploy a Docker image as a container to Azure App Service by using Azure DevOps.

## Configure the Lab Prerequisites

In this exercise, we will set up the prerequisites for the lab, which consist of a new Azure DevOps project with a repository based on the [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

### Task 1: Create and Configure the Team Project

In this task, you will create an eShopOnWeb Azure DevOps project to be used by several labs.

1. In a browser window log into Azure DevOps organization. Click on `New Project`. Give your project the name `eShopOnWeb` and choose `Scrum` on the `Work Item process` dropdown. Click on `Create`.<p>
![create-project](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/2081b0a0-838b-457c-96be-3486141473b5)<p>
![create-project2](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/d227a157-21f8-4a5f-b3c9-0c926aa046fd)<p>

### Task 2: Import eShopOnWeb Git Repository

This task seeks to import the **eShopOnWeb Git repository** that will be used for this project.

1. In a browser window open your Azure DevOps organization and the previously created eShopOnWeb project. Click on `Repos > Files`, `Import`. On the `Import a Git Repository` window, paste the following URL https://github.com/MicrosoftLearning/eShopOnWeb.git and click on `Import`.<p>
![import-project](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/05449484-38f8-48b8-a28c-8e813ffebbed)<p>
![import-project2](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/4bec7d02-1717-4eda-8487-a8ad955e7f38)<p>

2. The repository is organized the following way:

- `.ado` folder contains Azure DevOps YAML pipelines.
- `.devcontainer` folder container setup to develop using containers (either locally in VS Code or GitHub Codespaces).
- `infra` folder contains Bicep & ARM infrastructure as code templates used in some lab scenarios.
- `.github` folder container YAML GitHub workflow definitions.
- `src` folder contains the .NET 8 website used on the lab scenarios.

### Task 3: Set main branch as default branch

Go to `Repos > Branches`.
Hover on the `main` branch then click the ellipsis on the right of the column.
Click on `Set as default branch`.<p>
![default-branch](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/1e5c306e-590c-436f-ae2a-96843ec2ce20)<p>
![default-branch2](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/7663977d-44e2-41b2-9255-820d63dced46)<p>

Here is the text paraphrased in Markdown format:

## Manage the Service Connection

In this section, we will configure the service connection with the personalised Azure Subscription then import and run the CI pipeline.

### Task 1: Manage the Service Connection

We can create a connection from Azure Pipelines to external and remote services for executing tasks in a job.

In this task, we will create a `service principal` by using the `Azure CLI`, which will allow Azure DevOps to:

- Deploy resources on your azure subscription.
- Push the Docker image to Azure Container Registry.
- Add a role assignment to allow Azure App Service pull the Docker image from Azure Container Registry.

> Note: If you do already have a service principal, you can proceed directly to the next task.

A `service principal` will be needed to deploy Azure resources from Azure Pipelines.

A service principal is automatically created by Azure Pipeline when we connect to an Azure subscription from inside a pipeline definition or when you create a new service connection from the project settings page (automatic option). You can also manually create the service principal from the portal or using Azure CLI and re-use it across projects.

1. Start a web browser, avigate to the Azure Portal, and sign in with the user account that has the Owner role in the Azure subscription you will be using in this lab and has the role of the Global Administrator in the Microsoft Entra tenant associated with this subscription.<p>
![Azure-portal](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/ea551453-8850-4a2d-861a-dda252a8af60)<p>

2. In the Azure portal, click on the Cloud Shell icon, located directly to the right of the search textbox at the top of the page.<p>
![Azure-portal2](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/467c7f0a-fd70-4952-b428-0d248aa465f3)<p>

3. If prompted to select either Bash or PowerShell, select `Bash`.

> Note: If this is the first time you are starting Cloud Shell and you are presented with the "You have no storage mounted" message, select the subscription you are using in this lab, and select `Create storage`.

4. From the Bash prompt, in the Cloud Shell pane, run the following commands to retrieve the values of the Azure subscription ID attribute:

```
subscriptionName=$(az account show --query name --output tsv)
subscriptionId=$(az account show --query id --output tsv)
echo $subscriptionName
echo $subscriptionId
```

> Note: Copy both values to a text file. You will need them later in this lab.

5. From the Bash prompt, in the Cloud Shell pane, run the following command to create a service principal:

```
az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
```

> Note: The command will generate a JSON output. Copy the output to text file. You will need it later in this lab.

6. Next, navigate to the Azure DevOps eShopOnWeb project. Click on `Project Settings > Service Connections (under Pipelines)` and `New Service Connection`.<p>
![connection](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/2518f4e2-9282-4321-8b54-ba164c3c4442)<p>

7. On the `New service connection` blade, select `Azure Resource Manager` and `Next` (may need to scroll down).
The choose `Service principal (manual)` and click on `Next`.<p>
![connection2](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/e2b7df8b-59b3-449f-b31c-ed42b8de312c)<p>
![connection3](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/edd78067-47b3-46cd-81d0-7fb76783eca7)<p>
![connection4](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/6ac3094e-67cc-4bcc-bd57-14251f0cfe8f)<p>

8. Fill in the empty fields using the information gathered during previous steps:
- Subscription Id and Name.
- Service Principal Id (appId), Service principal key (password) and Tenant ID (tenant).
- In `Service connection name` type `azure-connection`. This name will be referenced in YAML pipelines when needing an Azure DevOps Service Connection to communicate with your Azure subscription.<p>
![connection5](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/020b4c94-46c8-4e6f-bf2f-baa11b38dbc4)<p>

Click on `Verify and Save`.<p>
![connection6](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/9017155d-210f-435c-bbac-240913a4a7a5)<p>

## Import and Run the CI Pipeline

In this section, we will import and run the CI pipeline.

### Task 1: Import and Run the CI Pipeline

1. Go to `Pipelines > Pipelines`.
2. Click on `New pipeline` button (or `Create Pipeline` if you don't have other pipelines previously created).
3. Select `Azure Repos Git (YAML)`.
4. Select the `eShopOnWeb` repository.
5. Select `Existing Azure Pipelines YAML file`.
6. Select the `main` branch and the `/.ado/eshoponweb-ci-docker.yml` file, then click on `Continue`.<p>
![pipeline](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/a3eb4ee2-e343-45ff-842b-66db24a4d83c)<p>

7. In the YAML pipeline definition, customize:
   - `YOUR-SUBSCRIPTION-ID` with your Azure subscription ID.
   - `rg-az400-container-NAME` with the resource group name that will be created by the pipeline (it can be an existing resource group too).<p>
![pipeline2](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/e8a8929f-b689-4e2a-82e3-376c9d3b29ef)<p>

8. Click on `Save and Run` and wait for the pipeline to execute successfully.
Pipeline actions:<p>
```
# NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")

resources:
  repositories:
    - repository: self
      trigger: none

variables:
  azureServiceConnection: 'azure-connection'
  subscriptionId: 'YOUR-SUBSCRIPTION-ID'
  resourceGroup: 'rg-az400-container-NAME'
  location: 'uksouth'

stages:
- stage: Build
  jobs:
  - job: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: AzureResourceManagerTemplateDeployment@3
      displayName: Deploy ACR using Bicep
      inputs:
        deploymentScope: 'Resource Group'
        azureResourceManagerConnection: $(azureServiceConnection)
        subscriptionId: $(subscriptionId)
        action: 'Create Or Update Resource Group'
        resourceGroupName: '$(resourceGroup)'
        location: '$(location)'
        templateLocation: 'Linked artifact'
        csmFile: 'infra/acr.bicep'
        deploymentMode: 'Incremental'
        deploymentOutputs: 'outputJson'
    - task: PowerShell@2
      displayName: Parse Bicep Output
      inputs:
        targetType: 'inline'
        script: |
          $var=ConvertFrom-Json '$(outputJson)'
          $value=$var.acrLoginServer.value
          Write-Host "##vso[task.setvariable variable=acrLoginServer;]$value"
    - task: Docker@0
      displayName: 'Build the docker image'
      inputs:
        azureSubscription: $(azureServiceConnection)
        azureContainerRegistry: $(acrLoginServer)
        dockerFile: 'src/Web/Dockerfile'
        defaultContext: false
        context: $(Build.SourcesDirectory)
        includeLatestTag: true
        imageName: eshoponweb/web:$(Build.BuildId)
    - task: Docker@0
      displayName: 'Push the docker images'
      inputs:
        azureSubscription: $(azureServiceConnection)
        azureContainerRegistry: $(acrLoginServer)
        action: 'Push an image'
        imageName: eshoponweb/web:$(Build.BuildId)
        includeLatestTag: true
```
![pipeline3](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/de564b0f-3e79-441a-b1fb-9b77cabdf79e)<p>

> Note: The deployment may take a few minutes to complete.<p>
![pipeline4](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/cbeea778-94b5-4d83-a858-682ddc305d20)<p>
![pipeline5-docker](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/9e2642fe-dbc1-421f-9e86-416822e54c96)<p>
![pipeline6-dockerbuild](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/6d5b4eb6-da91-43a1-88c5-6bd57b637f25)<p>

The CI definition consists of the following tasks:

- `Resources`: It downloads the repository files that will be used in the following tasks.
- `AzureResourceManagerTemplateDeployment`: Deploys the Azure Container Registry using bicep template.
- `PowerShell`: Retrieve the ACR Login Server value from the previous task's output and create a new parameter `acrLoginServer`.
- `Docker - Build`: Build the Docker image and create two tags (Latest and current BuildID).
- `Docker - Push`: Push the images to Azure Container Registry.

9. The pipeline will take a name based on the project name. Let's rename it for identifying the pipeline better. Go to `Pipelines > Pipelines` and click on the recently created pipeline. Click on the ellipsis and `Rename/move` option. Name it `eshoponweb-ci-docker` and click on `Save`.<p>
![pipeline7-renamed](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/1a64c61a-9a19-4998-9a48-9341f961abba)<p>

10. Navigate to the Azure Portal, search for the Azure Container Registry in the recently created Resource Group (it should be named `rg-az400-container-NAME`). On the left-hand side click `Repositories` under Services and make sure that the repository `eshoponweb/web` was created. When you click the repository link, you should see two tags (one of them is `latest`), these are the pushed images. If you don't see this, check the status of your pipeline.<p>
![azure-container](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/b50be430-9cca-45cd-a11c-3f2f2da3d8c6)<p>
![azure-container2](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/01f31d73-3efb-44c9-a439-f59b152a156d)<p>


## Exercise 3: Import and Run the CD Pipeline

In this exercise, we will configure the service connection with your Azure Subscription then import and run the CD pipeline.

### Task 1: Add a New Role Assignment

In this task, we will add a new role assignment to allow Azure App Service pull the Docker image from Azure Container Registry.

1. Navigate to the Azure Portal.
2. In the Azure portal, click on the Cloud Shell icon, located directly to the right of the search textbox at the top of the page.
3. If prompted to select either Bash or PowerShell, select `Bash`.
4. From the Bash prompt, in the Cloud Shell pane, run the following commands to retrieve the values of the Azure subscription ID attribute:

   ```
   subscriptionId=$(az account show --query id --output tsv)
   echo $subscriptionId
   spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
   echo $spId
   roleName=$(az role definition list --name "User Access Administrator" --query "[0].name" --output tsv)
   echo $roleName
   ```

5. After getting the service principal ID and the role name, let's create the role assignment by running this command (replace `<rg-az400-container-NAME>` with your resource group name):

   ```
   az role assignment create --assignee $spId --role $roleName --scope /subscriptions/$subscriptionId/resourceGroups/<rg-az400-container-NAME>
   ```

A JSON output which confirms the success of the command run should be populated.

### Task 2: Import and Run the CD Pipeline

In this task, we will import and run the CD pipeline.
In the project on Azure DevOps: 
1. Go to `Pipelines > Pipelines`.
2. Click on `New pipeline` button.
3. Select `Azure Repos Git (YAML)`.
4. Select the `eShopOnWeb` repository.
5. Select `Existing Azure Pipelines YAML File`.
6. Select the `main` branch and the `/.ado/eshoponweb-cd-webapp-docker.yml` file, then click on `Continue`.<p>
![pipeline-no2](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/e3d2c8f5-b47b-4607-a814-506062c892ca)<p>

7. In the YAML pipeline definition, customize:
   - `YOUR-SUBSCRIPTION-ID` with your Azure subscription ID.
   - `rg-az400-container-NAME` with the resource group name defined before in the lab.<p>
```
#NAME 

resources:
  repositories:
    - repository: self
      trigger: none

variables:
  azureServiceConnection: 'azure-connection'
  resourceGroup: 'rg-az400-container-NAME'
  location: 'westeurope'
  subscriptionId: 'YOUR-SUBSCRIPTION-ID'

stages:
- stage: Deploy
  jobs:
  - job: Deploy
    pool:
      vmImage: ubuntu-latest
    steps:
    - task: AzureResourceManagerTemplateDeployment@3
      displayName: Deploy App Service using Bicep
      inputs:
        deploymentScope: 'Resource Group'
        azureResourceManagerConnection: $(azureServiceConnection)
        subscriptionId: $(subscriptionId)
        action: 'Create Or Update Resource Group'
        resourceGroupName: '$(resourceGroup)'
        location: '$(location)'
        templateLocation: 'Linked artifact'
        csmFile: 'infra/webapp-docker.bicep'
        deploymentMode: 'Incremental'
    - task: AzureResourceManagerTemplateDeployment@3
      displayName: Add Role Assignment using Bicep
      inputs:
        deploymentScope: 'Resource Group'
        azureResourceManagerConnection: $(azureServiceConnection)
        subscriptionId: $(subscriptionId)
        action: 'Create Or Update Resource Group'
        resourceGroupName: '$(resourceGroup)'
        location: '$(location)'
        templateLocation: 'Linked artifact'
        csmFile: 'infra/webapp-to-acr-roleassignment.bicep'
        deploymentMode: 'Incremental'
```
8. Click on `Save and Run` and wait for the pipeline to execute successfully.<p>
![pipeline-no2-3](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/86c566b8-4f29-4368-b39e-d9f41a6be115)<p>

> Note: The deployment may take a few minutes to complete.

The CD definition consists of the following tasks:

- `Resources`: It downloads the repository files that will be used in the following tasks.
- `AzureResourceManagerTemplateDeployment`: Deploys the Azure App Service using bicep template.
- `AzureResourceManagerTemplateDeployment`: Add role assignment using Bicep.<p>
![pipeline-no2-3](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/3a929b43-d9a3-40c1-9ef7-745c3d8a4efe)<p>

> Important: If you receive the error message "TF402455: Pushes to this branch are not permitted; you must use a pull request to update this branch.", you need to uncheck the "Require a minimum number of reviewers" Branch protection rule enabled in the previous labs.

9. The pipeline will take a name based on the project name. Let's rename it for identifying the pipeline better. Go to `Pipelines > Pipelines` and hover on the recently created pipeline. Click on the ellipsis and `Rename/move` option. Name it `eshoponweb-cd-webapp-docker` and click on `Save`.<p>
![pipeline-no2-5 renamed](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/ae3b424d-b2f7-4b48-b819-5965129cb8c0)<p>

> Note 1: The use of the `/infra/webapp-docker.bicep` template creates an app service plan, a web app with system assigned managed identity enabled, and references the Docker image pushed previously: `${acr.properties.loginServer}/eshoponweb/web:latest`.

> Note 2: The use of the `/infra/webapp-to-acr-roleassignment.bicep` template creates a new role assignment for the web app with `AcrPull` role to be able to retrieve the Docker image. This could be done in the first template, but since the role assignment can take some time to propagate, it's a good idea to do both tasks separately.

### Task 3: Test the Solution

1. In the Azure Portal, navigate to the recently created Resource Group, now we can see three resources (App Service, App Service Plan and Container Registry).<p>
![image](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/d3eb011e-f1bb-4c47-a2a2-1f42967ef3b0)<p>

2. Navigate to the App Service, then click `Browse`, this will take us to the website.<p>
The webapp is live!:<p>
![image](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/467528f7-c3a3-4bf9-b46f-9ba29906a268)<p>
![image](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/cd11ff62-3055-45e0-8958-04f8112b0d02)<p>
![image](https://github.com/JonesKwameOsei/Deploy-Docker-containers-to-Azure-App-Service-web-apps/assets/81886509/7e78ebbf-90bd-4ee7-9780-a2b357518210)


Congratulations!, we have deployed a website using a custom Docker image.

## Remove the Azure Lab Resources

In this exercise, you will remove the Azure resources provisioned in this lab to eliminate unexpected charges.

> Note: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

### Task 1: Remove the Azure Lab Resources

Hhaving completed the project, we will use Azure Cloud Shell to remove the Azure resources provisioned in this lab to eliminate unnecessary charges.

1. In the Azure portal, open the Bash shell session within the Cloud Shell pane.
2. List all resource groups created throughout the labs of this module by running the following command:

   ```
   az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
   ```

3. Delete all resource groups you created throughout the labs of this module by running the following command:

   ```
   az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

> Note: The command executes asynchronously (as determined by the `--nowait` parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

## Conclusion
In this project, I demonstrated how to use an `Azure DevOps CI/CD pipeline` to build a custom `Docker image`, push it to `Azure Container Registry`, and deploy it as a container to `Azure App Service`.













