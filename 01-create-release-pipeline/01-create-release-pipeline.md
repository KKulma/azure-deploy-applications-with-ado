### Introduction

**What is continuous delivery?**

CD by itself is a set of processes, tools, and techniques that enable rapid, reliable, and continuous delivery of software. So CD isn't only about setting up a pipeline, although that part is important. CD is about setting up a working environment where:

- We have a reliable and repeatable process for releasing and deploying software.
- We automate as much as possible.
- We don't put off doing something that's difficult or painful; instead, we do it more often so that we figure out how to make it routine.
- We keep everything in source control.
- We all agree that done means released.
- We build quality into the process. Quality is never an afterthought.
- We're all responsible for the release process. We no longer work in silos.
- We always try to improve.

Continuous delivery provides a consistent way for you and your team to continuously test, deploy, and monitor your application each time you check in your code. When you right-click publish your application, there's no guarantee that the code was properly tested, or will behave as expected under real-world usage.



**What is an environment?**

You've likely used the term environment to refer to where your application or service is running. For example, your production environment might be where your end users access your application.

Following this example, your production environment might be:

- A physical machine or virtual machine (VM).
- A containerized environment, such as Kubernetes.
- A managed service, such as Azure App Service.
- A serverless environment, such as Azure Functions.

An artifact is deployed to an environment. Azure Pipelines makes it easy to deploy to almost any kind of environment, whether it's on-premises or in the cloud.

In Azure Pipelines, the term environment has a second meaning. Here, an environment is an abstract representation of your deployment environment, such as a Kubernetes cluster, an App Service instance, or a virtual machine.

An Azure Pipelines environment records the deployment history to help you identify the source of changes. By using Azure Pipelines environments, you can also define security checks and ways to control how an artifact is promoted from one stage of a pipeline to another. What an environment includes depends on what you want to do with the artifact. An environment where you want to test the artifact will probably be defined differently than one where you want to deploy the artifact for your end users.



## EXERCISES
#### Set up your environment
This part will provide only the project to build and publish as a pipeline artifact, it won't be deployed to any env

- [link](https://learn.microsoft.com/en-us/training/modules/create-release-pipeline/4-set-up-environment) to the instructions

- [link](https://github.com/MicrosoftDocs/mslearn-tailspin-spacegame-web-deploy) to the GitHub repo

#### Deploy the web application to Azure App Service
- Create an App Service instance to host your web application.
- Create a multistage pipeline.
- Deploy to Azure App Service.

**Create the App Service instance**

- Sign in to the Azure portal. 
- Select App Services from the left pane. 
- Select Create > Web App to create a new Web App. 
- On the Basics tab, enter the following values.
  - **Subscription**:  your subscription
  - **Resource Group**: Select Create new, and then enter tailspin-space-game-rg, and select OK.
  - **Name**: Provide a unique name, such as tailspin-space-game-1234. This name must be unique across Azure. It becomes part of the domain name. In practice, choose a name that describes your service. Note the name for later.
  - **Publish**: Code
  - **Runtime stack**: .NET 6 (LTS)
  - **Operating System**: Linux
  - **Region**:	Select a region, preferably one close to you.
    (Pricing plans)	
  - **Linux Plan**:	Accept the default.
  - **Pricing plan**: Select the Basic B1 pricing tier from the drop-down menu.
- Select Review + create, review the form and then select Create. The deployment takes a few moments to complete.
- When deployment is complete, select Go to resource. The App Service Essentials displays details related to your deployment.
- Select the URL to verify the status of your App Service.


**Create a service connection**
- In Azure DevOps, go to your Space Game - web - Release project.
- From the lower-left corner of the page, select Project settings.
- Under Pipelines, select Service connections.
- Select New service connection, and then select Azure Resource Manager then select Next.
- Select Service principal (automatic), and then select Next.
- Fill out the required fields as follows: If prompted, sign in to your Microsoft account. 
  - **Scope level**: Subscription 
  - **Subscription**: Your Azure subscription
  - **Resource Group**:	tailspin-space-game-rg 
  - **Service connection name**: Resource Manager - Tailspin - Space Game
  - Ensure that Grant access permission to all pipelines is selected.
- Select Save.

**Add the Build stage to your pipeline**

A multistage pipeline allows you to define distinct phases that your change passes through as it's promoted through the pipeline. Each stage defines the agent, variables, and steps required to carry out that phase of the pipeline. In this section, you'll define one stage to perform the build. You define a second stage to deploy the web application to App Service.

To convert your existing build configuration to a multistage pipeline, you add a stages section to your configuration, and then you add one or more stage sections for each phase of your pipeline. Stages break down into jobs, which are a series of steps that run sequentially as a unit.

From your project in Visual Studio Code, open azure-pipelines.yml and replace its contents with this code:

```yaml
trigger:
- '*'

variables:
  buildConfiguration: 'Release'

stages:
- stage: 'Build'
  displayName: 'Build the web application'
  jobs: 
  - job: 'Build'
    displayName: 'Build job'
    pool:
      vmImage: 'ubuntu-20.04'
      demands:
      - npm

    variables:
      wwwrootDir: 'Tailspin.SpaceGame.Web/wwwroot'
      dotnetSdkVersion: '6.x'

    steps:
    - task: UseDotNet@2
      displayName: 'Use .NET SDK $(dotnetSdkVersion)'
      inputs:
        version: '$(dotnetSdkVersion)'

    - task: Npm@1
      displayName: 'Run npm install'
      inputs:
        verbose: false

    - script: './node_modules/.bin/node-sass $(wwwrootDir) --output $(wwwrootDir)'
      displayName: 'Compile Sass assets'

    - task: gulp@1
      displayName: 'Run gulp tasks'

    - script: 'echo "$(Build.DefinitionName), $(Build.BuildId), $(Build.BuildNumber)" > buildinfo.txt'
      displayName: 'Write build info'
      workingDirectory: $(wwwrootDir)

    - task: DotNetCoreCLI@2
      displayName: 'Restore project dependencies'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'

    - task: DotNetCoreCLI@2
      displayName: 'Build the project - $(buildConfiguration)'
      inputs:
        command: 'build'
        arguments: '--no-restore --configuration $(buildConfiguration)'
        projects: '**/*.csproj'

    - task: DotNetCoreCLI@2
      displayName: 'Publish the project - $(buildConfiguration)'
      inputs:
        command: 'publish'
        projects: '**/*.csproj'
        publishWebProjects: false
        arguments: '--no-build --configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/$(buildConfiguration)'
        zipAfterPublish: true

    - publish: '$(Build.ArtifactStagingDirectory)'
      artifact: drop
```

**Create the dev environment**

An environment is an abstract representation of your deployment environment. Environments can be used to define specific criteria for your release such as which pipeline is authorized to deploy to the environment. Environments can also be used to set up manual approvals for specific user/group to approve before deployment is resumed.

- From Azure Pipelines, select Environments.
- Select Create environment.
- Under Name, enter dev. 
- Leave the remaining fields at their default values. 
- Select Create. 

**Store your web app name in a pipeline variable**

The Deploy stage that we will be creating will use the name to identify which App Service instance to deploy to e.g: tailspin-space-game-web-1234.

Although you could hard-code this name in your pipeline configuration, defining it as a variable makes your configuration more reusable.

- In Azure DevOps, select Pipelines and then select Library. 
- Screenshot of Azure Pipelines showing the location of the Library menu. 
- Select + Variable group to create a new variable group. 
- Enter Release for the variable group name. 
- Select Add under Variables to add a new variable. 
- Enter WebAppName for the variable name and your App Service instance's name for its value: e.g. tailspin-space-game-web-1234. 
- Select Save.

**Add the deployment stage to your pipeline**

We will extend our pipeline by adding a deployment stage to deploy the Space Game to App Service using the download and AzureWebApp@1 tasks to download the build artifact and then deploy it.

From Visual Studio Code, replace the contents of azure-pipelines.yml with the following yaml:

```yaml
trigger:
- '*'

variables:
  buildConfiguration: 'Release'

stages:
- stage: 'Build'
  displayName: 'Build the web application'
  jobs: 
  - job: 'Build'
    displayName: 'Build job'
    pool:
      vmImage: 'ubuntu-20.04'
      demands:
      - npm

    variables:
      wwwrootDir: 'Tailspin.SpaceGame.Web/wwwroot'
      dotnetSdkVersion: '6.x'

    steps:
    - task: UseDotNet@2
      displayName: 'Use .NET SDK $(dotnetSdkVersion)'
      inputs:
        version: '$(dotnetSdkVersion)'

    - task: Npm@1
      displayName: 'Run npm install'
      inputs:
        verbose: false

    - script: './node_modules/.bin/node-sass $(wwwrootDir) --output $(wwwrootDir)'
      displayName: 'Compile Sass assets'

    - task: gulp@1
      displayName: 'Run gulp tasks'

    - script: 'echo "$(Build.DefinitionName), $(Build.BuildId), $(Build.BuildNumber)" > buildinfo.txt'
      displayName: 'Write build info'
      workingDirectory: $(wwwrootDir)

    - task: DotNetCoreCLI@2
      displayName: 'Restore project dependencies'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'

    - task: DotNetCoreCLI@2
      displayName: 'Build the project - $(buildConfiguration)'
      inputs:
        command: 'build'
        arguments: '--no-restore --configuration $(buildConfiguration)'
        projects: '**/*.csproj'

    - task: DotNetCoreCLI@2
      displayName: 'Publish the project - $(buildConfiguration)'
      inputs:
        command: 'publish'
        projects: '**/*.csproj'
        publishWebProjects: false
        arguments: '--no-build --configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/$(buildConfiguration)'
        zipAfterPublish: true

    - publish: '$(Build.ArtifactStagingDirectory)'
      artifact: drop

- stage: 'Deploy'
  displayName: 'Deploy the web application'
  dependsOn: Build
  jobs:
  - deployment: Deploy
    pool:
      vmImage: 'ubuntu-20.04'
    environment: dev
    variables:
    - group: Release
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop
          - task: AzureWebApp@1
            displayName: 'Azure App Service Deploy: website'
            inputs:
              azureSubscription: 'Resource Manager - Tailspin - Space Game'
              appName: '$(WebAppName)'
              package: '$(Pipeline.Workspace)/drop/$(buildConfiguration)/*.zip'
```
Notice the highlighted section and how we are using the download and AzureWebApp@1 tasks. The pipeline will fetch the $(WebAppName) from the variable group we created earlier.

Also notice how we are using environment to deploy to the dev environment.

**View the deployed website on App Service**

If you still have your App Service tab open, simply refresh the page. Otherwise, navigate to your Azure App Service in the Azure portal and select the instance's URL: for example, https://tailspin-space-game-web-1234.azurewebsites.net


#### Exercise - Clean up your environment
Congratulations! You're all done with creating pipelines and resources. In this unit, you're going to clean up your Azure resources and Azure DevOps environment.

**Clean up Azure resources**

- Navigate to Azure portal.
- Select Resource groups from the left panel.
- Select your resource group (tailspin-space-game-rg).
- Select Delete resource group.
- Type your resource group name in the text box, and then select Delete.
- Select Delete again to confirm deletion.
- Disable pipeline or delete project

- Each module in this learning path provides a template you can run to create a clean environment. When you run multiple templates, you will create multiple projects each pointing to the same GitHub repository. This setup can trigger multiple pipelines to run every time you push a change to your GitHub repository consuming free build minutes on our hosted agents. That's why it's important to disable or delete your pipeline before you move on to the next module in this learning path.

**Option 1: Disable your pipeline**

Choose this option if you want to keep your project and your build pipeline for future reference. You can re-enable your pipeline later if you need to.

In your Azure DevOps project, select Pipelines and then select your pipeline.

- Select the ellipsis button at the far right, and then select Settings.
- Select Disabled, and then select Save. 

Your pipeline will no longer process new run requests.

**Option 2: Delete your project**

Choose this option if you don't need your DevOps project for future reference. This will delete your Azure DevOps project.

- Navigate to your Azure DevOps project. 
- Select Project settings in the lower-left corner. 
- Under Overview, scroll down to the bottom of the page and then select Delete. 
- Type your project name in the text box, and then select Delete.