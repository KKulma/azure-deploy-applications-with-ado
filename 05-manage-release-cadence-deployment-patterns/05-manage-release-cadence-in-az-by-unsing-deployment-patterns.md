# Introduction

In this module, you help the team solve another problem, which is to implement a deployment pattern to release to production in a way that's best for the company and their users. You'll help them evaluate the possibilities and then implement the one they choose.

## Learning objectives
After completing this module, you'll be able to:

- Explain why deployment patterns matter.
- Compare different deployment patterns so that you can choose the one that best suits your needs.
- Apply the blue-green deployment pattern in your pipeline.

# What are deployment patterns?

**A deployment pattern is an automated way to smoothly roll out new application features to your users**. An appropriate deployment pattern **helps you minimize downtime**. **Some patterns also enable you to roll out new features progressively**. That way, you can validate new features with select users before you make those features available to everyone.
 A deployment pattern is an automated way to do the cutover. **It's how we move the software from the final pre-production stage to live production.**

In this section, you'll learn about some common deployment patterns. You'll also learn how Azure App Service will help implement the pattern that the Tailspin team chooses.


Options we can consider:

- Blue-green deployment
- Canary releases
- Feature toggles
- Dark launches
- A/B testing
- Progressive-exposure deployment


## Blue-green deployments
A blue-green deployment reduces risk and downtime by running two identical environments. These environments are called blue and green. At any time, only one of the environments is live. A blue-green deployment typically involves a router or load balancer that helps control the flow of traffic.

Diagram of a load balancer distributing traffic in a blue-green deployment.

Let's say blue is live. As we prepare a new release, we do our final tests in the green environment. After the software is working in the green environment, we just switch the router so that all incoming requests go to the green environment.

Blue-green deployment also gives us a fast way to do a rollback. If anything goes wrong in the green environment, then we just switch the router back to the blue environment.

## Canary releases
A canary release is a way to identify potential problems early without exposing all users to the issue. The idea is that we expose a new feature to only a small subset of users before we make it available to everyone.

Diagram of a load balancer sending traffic to a canary version.

In a canary release, we monitor what happens when we release the feature. If the release has problems, then we apply a fix. After the canary release is known to be stable, we move it to the actual production environment.

## Feature toggles
Use feature toggles to "flip a switch" at runtime. We can deploy new software without exposing any other new or changed functionality to our users.

In this deployment pattern, Mara and I build new features behind a toggle. When a release occurs, the feature is "off" so that it doesn't affect the production software. Depending on how we configure the toggle, we can flip the switch to "on" and expose the feature how we want.

Diagram of a coded if statement for an on-off feature.

For example, we could expose the feature first to a few users to see how they react. That random sample of users sees the feature. Or we could just let the feature go live to everyone.

But this deployment pattern might benefit Mara and me more than anyone else. A big advantage to the feature toggles pattern is that it helps us avoid too much branching. Merging branches can be painful.


## Dark launches
A dark launch is similar to a canary release or switching a feature toggle. Rather than expose a new feature to everyone, in a dark launch we release the feature to a small set of users.

Diagram of a load balancer sending traffic to the new feature.

Those users don't know they're testing the feature for us. We don't even highlight the new feature to them. That's why it's called a dark launch. The software is gradually or unobtrusively released to users so we can get feedback and can test performance.


## A/B testing
A/B testing compares two versions of a webpage or app to determine which one performs better. A/B testing is like an experiment.

Diagram of two apps and their analytics.

In A/B testing, we randomly show users two or more variations of a page. Then we use statistical analysis to decide which variation performs better for our goals.


## Progressive-exposure deployment
Progressive-exposure deployment is sometimes called ring-based deployment. It's another way to limit how changes affect users while making sure those changes are valid in a production environment.

Rings are basically an extension of the canary stage. The canary release, releases to a stage to measure effect. Adding another ring is essentially the same idea.

Diagram of a progression of larger groups.

In a ring-based deployment, we deploy changes to risk-tolerant customers first. Then we progressively roll out to a larger set of customers.



# Exercises

## Implement the blue-green deployment pattern
In Create a multistage pipeline by using Azure Pipelines, you built a basic deployment pipeline that deploys a web application to Azure App Service in these stages: Dev, Test, and Staging.

Here you add to that workflow by applying the blue-green deployment pattern during Staging.

To do so, you:
- Add a deployment slot to the App Service instance that corresponds to Staging.
- Add a task to the pipeline to swap the deployment slots.

## Add a deployment slot
Here you add a deployment slot to the App Service instance that corresponds to Staging.

By default, every App Service instance provides a default slot, named production. You deployed to the production slot when you set up the pipeline in the previous section.

An App Service instance can have multiple slots. Here you add a second deployment slot to the App Service instance that corresponds to Staging. The deployment slot is named swap.

To add the slot:

1. Go to the Azure portal and sign in. 
2. On the menu, select Cloud Shell. When you're prompted, select the Bash experience. 
3. Run the following command to get the name of the App Service instance that corresponds to Staging and to store the result in a Bash variable that's named staging.

```shell
staging=$(az webapp list \
  --resource-group tailspin-space-game-rg \
  --query "[?contains(@.name, 'tailspin-space-game-web-staging')].{name: name}" \
  --output tsv)
```

The `--query` argument uses JMESPath, which is a query language for JSON. The argument selects the App Service instance whose name field contains "tailspin-space-game-web-staging".

4. Print the staging variable to verify that you get the correct name.

```yaml
echo $staging
```

Here's an example of the output:
```shell
tailspin-space-game-web-staging-1234
```

5. Run the following command to add a slot named swap to your staging environment.

```shell
az webapp deployment slot create \
  --name $staging \
  --resource-group tailspin-space-game-rg \
  --slot swap
```

6. Run the following command to list your deployment slot's host name.

```shell
az webapp deployment slot list \
    --name $staging \
    --resource-group tailspin-space-game-rg \
    --query [].hostNames \
    --output tsv
```

The result resembles this output:

```shell
tailspin-space-game-web-staging-25391-swap.azurewebsites.net
```

Make note of this host name for later.

7. As an optional step, go to your site in a browser. You see the default home page because you haven't yet deployed your code to this slot.

By default, a deployment slot is accessible from the internet. In practice, you could configure an Azure virtual network that places your swap slot in a network that's not routable from the internet but that only your team can access. Your production slot would remain reachable from the internet.

### Swap deployment slots in Staging
Here you use the `AzureAppServiceManage@0` task to swap deployment slots in your Staging environment.

You can also use this task to start, stop, or delete a slot. Or you can use it to install site extensions or to enable continuous monitoring on App Service.

In Visual Studio Code, modify azure-pipelines.yml by using this code:

**Tip**: You can replace the entire file or just update the part that's highlighted.

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

- stage: 'Dev'
  displayName: 'Deploy to the dev environment'
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
              appName: '$(WebAppNameDev)'
              package: '$(Pipeline.Workspace)/drop/$(buildConfiguration)/*.zip'

- stage: 'Test'
  displayName: 'Deploy to the test environment'
  dependsOn: Dev
  jobs:
  - deployment: Deploy
    pool:
      vmImage: 'ubuntu-20.04'
    environment: test
    variables:
    - group: 'Release'
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
              appName: '$(WebAppNameTest)'
              package: '$(Pipeline.Workspace)/drop/$(buildConfiguration)/*.zip'

- stage: 'Staging'
  displayName: 'Deploy to the staging environment'
  dependsOn: Test
  jobs:
  - deployment: Deploy
    pool:
      vmImage: 'ubuntu-20.04'
    environment: staging
    variables:
    - group: 'Release'
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
              deployToSlotOrASE: 'true'
              resourceGroupName: 'tailspin-space-game-rg'
              slotName: 'swap'
              appName: '$(WebAppNameStaging)'
              package: '$(Pipeline.Workspace)/drop/$(buildConfiguration)/*.zip'
          - task: AzureAppServiceManage@0
            displayName: 'Swap deployment slots'
            inputs:
              azureSubscription: 'Resource Manager - Tailspin - Space Game'
              resourceGroupName: 'tailspin-space-game-rg'
              webAppName: '$(WebAppNameStaging)'
              sourceSlot: 'swap'
              targetSlot: 'production'
              action: 'Swap Slots'
```

Note these changes:

The AzureWebApp@1 task now specifies these values:
- `deployToSlotOrASE`, when set to true, deploys to an existing deployment slot.
- `resourceGroupName` specifies the name of the resource group. This value is required when deployToSlotOrASE is true.
- `slotName` specifies the name of the deployment slot. Here you deploy to the slot that's named swap. 
- The new task, `AzureAppServiceManage@0`, swaps the deployment slots. 
- `sourceSlot` and `targetSlot` specify the slots to swap.
- `action` specifies the action to take. Recall that you can use this task to start, stop, or delete a slot. Here, "Swap Slots" specifies to swap the source and target slots.

This configuration always deploys to the swap slot. It then swaps the production and swap slots. The swap process ensures that production points to the more recent deployment.

In the integrated terminal, add azure-pipelines.yml to the index. Commit the changes, and then push the branch up to GitHub.

**Tip**: Save `azure-pipelines.yml` before you run these Git commands.

```shell
git add azure-pipelines.yml
git commit -m "Swap deployment slots"
git push origin blue-green
```

In Azure Pipelines, trace the build through each of the steps.