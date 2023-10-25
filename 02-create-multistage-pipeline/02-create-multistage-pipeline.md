## Introduction

In this module, you'll design a realistic release pipeline that contains multiple stages. You'll also learn different ways to control how an artifact is promoted from one stage to the next.

A good release-management workflow enables you to release more frequently and more consistently. In practice, you'll want to define a process that maps to your team's needs. Here you'll create a basic workflow. That means first designing the environments. The environments define the runtimes of each stage in the pipeline. You'll then deploy the Space Game web app to these stages: Dev, Test, and Staging. Each stage deploys the app to its own App Service instance.

After completing this module, you'll be able to:

- Identify the stages, or major divisions of the pipeline, that you need to implement in a multistage pipeline.
- Explain when to use conditions, triggers, and approvals to promote changes from one stage to the next.
- Promote a build through these stages: Dev, Test, and Staging.


## Design the pipeline

When you plan a release pipeline, you usually begin by identifying the stages, or major divisions, of that pipeline. Each stage typically maps to an environment.

After you define which stages you need, consider how changes are promoted from one stage to the next. Each stage can define the success criteria that must be met before the build can move to the next stage. Azure Pipelines provides several ways to help you control how and when changes move through the pipeline. As a whole, these approaches are used for **release management**.

In this section, you'll:

- Learn the differences between common pipeline stages, such as Build, Dev, Test, and Staging.
- Understand how to use manual, scheduled, and continuous deployment triggers to control when an artifact moves to the next stage in the pipeline.
- See how a release approval pauses the pipeline until an approver accepts or rejects the release.

### What pipeline stages do you need?

Consider dev, test, staging and prod.

### What are conditions?

In Azure Pipelines, use a condition to run task or job based on the state of the pipeline. You worked with conditions in previous modules.

Remember, some of the conditions that you can specify are:

- Only when all previous dependent tasks have succeeded.
- Even if a previous dependency has failed, unless the run was canceled.
- Even if a previous dependency has failed, even if the run was canceled.
- Only when a previous dependency has failed.
- Some custom condition.

For example: 

```yaml
steps:
  - script: echo Hello!
    condition: always()
```

The `always()` condition causes this task to print "Hello!" unconditionally, even if previous tasks failed.


The `succeeded` condition is used if you don't specify a condition:
```yaml
condition: succeeded()
```
It checks whether the previous task succeeded. If the previous task fails, this task and later tasks that use the same condition are skipped.


In our example, you want to build a condition that specifies:
- The previous task succeeded.
- The name of the current Git branch is "release".

To build this condition, you use the built-in `and()` function. This function checks whether each of its conditions is true. If any condition isn't true, the overall condition fails.

To get the name of the current branch, you use the built-in `Build.SourceBranchName` variable. You can access variables within a condition in a few ways. Here you use the `variables[]` syntax.

To test a variable's value, you can use the built-in `eq()` function. This function checks whether its arguments are equal.

With that in mind, you apply this condition to run the Dev stage only when the current branch name is "release":

```yaml
condition: |
  and
  (
    succeeded(),
    eq(variables['Build.SourceBranchName'], 'release')
  )
```

In YAML, you use the pipe `(|)` syntax to define a string that spans multiple lines. You could define the condition on a single line, but we write it this way to make it more readable.

For a more complete description of conditions in Azure Pipelines, see [expressions documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/expressions?view=azure-devops#job-status-functions).

### Add the Test stage

- dev release is triggered when all the previous tasks finished and changes merged to a `release` branch
- test release happens once a day during non-office hours (e.g. 3am)


In Azure Pipelines, we can use triggers. A trigger defines when a stage runs. Azure Pipelines provides a few types of triggers. Here are our choices:

- Continuous integration (CI) trigger
- Pull request (PR) trigger
- Scheduled trigger
- Build completion trigger

CI and PR triggers let you control which branches participate in the overall process. For example, you want to build the project when a change is made in any branch. A scheduled trigger starts a deployment at a specific time. A build completion trigger runs a build when another build, such as one for a dependent component, completes successfully. It seems like we want a scheduled trigger. 

**What are scheduled triggers?**
A scheduled trigger uses cron syntax to cause a build to run on a defined schedule.

On Unix and Linux systems, cron is a popular way to schedule jobs to run on a set time interval or at a specific time. In Azure Pipelines, scheduled triggers use the cron syntax to define when a stage runs.

```yaml
mm HH DD MM DW
 \  \  \  \  \__ Days of week
  \  \  \  \____ Months
   \  \  \______ Days
    \  \________ Hours
     \__________ Minutes
```

For example, this cron expression describes "3 A.M. every day": `0 3 * * *`

To set up a scheduled trigger in Azure Pipelines, you need a schedules section in your YAML file. Here's an example:

```yaml
schedules:
- cron: '0 3 * * *'
  displayName: 'Deploy every day at 3 A.M.'
  branches:
    include:
    - release
  always: false
```

In this schedules section:

- `cron` specifies the cron expression.
- `branches` specifies to deploy only from the release branch.
- `always` specifies whether to run the deployment unconditionally (true), or only when the release branch has changed since the last run (false). Here, you specify false because you need to deploy only when the release branch has changed since the last run.

```yaml
- stage: 'Test'
  displayName: 'Deploy to the Test environment'
  condition: and(succeeded(), eq(variables['Build.Reason'], 'Schedule'))
```

This stage, Test, runs only when the previous stage succeeds and the built-in Build.Reason pipeline variable equals Schedule.

### Add the Staging stage

Having a staging, or preproduction environment is important. This staging environment is often the last stop before a feature or bug fix reaches our users.


#### What are release approvals?
A release approval is a way to pause the pipeline until an approver accepts or rejects the release. To define your release workflow, you can combine approvals, conditions, and triggers.

Your environment can include specific criteria for your release. The criteria can specify which pipelines can deploy to that environment and what human approvals are needed to promote the release from one stage to the next.


###  The plan

1. Produce a build artifact each time we push a change to GitHub. This step happens in the Build stage.
2. Promote the build artifact to the Dev stage. This step happens automatically when the build stage succeeds and the change is on the release branch.
3. Promote the build artifact to the Test stage each morning at 3 A.M. We use a scheduled trigger to promote the build artifact automatically.
4. Promote the build artifact to Staging after Amita tests and approves the build. We use a release approval to promote the build artifact.

Every deployment stage in our pipeline also has its own environment. For example, when we deploy the app to Dev or Test, the environment is an App Service instance.

Finally, we test only one release at a time. We never change releases in the middle of the pipeline. We use the same release in the Dev stage as in the Staging stage, and every release has its own version number. If the release breaks in one of the stages, we fix it and build it again with a new version number. That new release then goes through the pipeline from the very beginning.


### A few words about quality
You've just seen the team design a pipeline that takes their app all the way from build to staging. The whole point of this pipeline isn't just to make their lives easier. It's to ensure the quality of the software they're delivering to their customers.

How do you measure the quality of your release process? The quality of your release process can't be measured directly. What you can measure is how well your process works. If you're constantly changing the process, this might be an indication that there's something wrong. Releases that fail consistently at a particular point in the pipeline might also indicate that there's a problem with the release process.

Do the releases always fail on a particular day or time? Do they always fail after you deploy to a particular environment? Look for these and other patterns to see if some aspects of the release process are dependent or related.

A good way to keep track of your release process quality is to create visualizations of the quality of the releases. For example, add a dashboard widget that shows you the status of every release.

When you want to measure the quality of a release itself, you can perform all kinds of checks within the pipeline. For example, you can execute different types of tests, such as load tests and UI tests while running your pipeline.

Using a quality gate is also a great way to check the quality of your release. There are many different quality gates. For example, work item gates can verify the quality of your requirements process. You can also add additional security and compliance checks. For example, do you comply with the 4-eyes principle or do you have the proper traceability?

As you progress through this learning path, you'll see a number of these techniques put into practice.

Lastly, when you design a quality release process, think about what kind of documentation or release notes that you'll need to provide to the user. Keeping your documentation current can be difficult. You might want to consider using a tool, such as the [Azure DevOps Release Notes Generator](https://learn.microsoft.com/en-us/samples/azure-samples/azure-devops-release-notes/azure-devops-release-notes-generator/). The generator is a function app that contains an HTTP-triggered function. By using Azure Blob Storage, it creates a Markdown file whenever a new release is created in Azure DevOps.


## Exercise - Set up your Azure DevOps environment

#### Create the App Service instances

1. To create an App Service Plan, run:

```shell
az appservice plan create \
  --name tailspin-space-game-asp \
  --resource-group tailspin-space-game-rg \
  --sku B1 \
  --is-linux
```

The `--sku` argument specifies the B1 plan. This plan runs on the Basic tier. The `--is-linux` argument specifies to use Linux workers.

To create the three App Service instances, one for each environment (Dev, Test, and Staging), run the following az webapp create commands.

```shell
webappsuffix=$RANDOM
az webapp create --name tailspin-space-game-web-dev-$webappsuffix --resource-group tailspin-space-game-rg --plan tailspin-space-game-asp --runtime "DOTNET|6.0"

az webapp create --name tailspin-space-game-web-test-$webappsuffix --resource-group tailspin-space-game-rg --plan tailspin-space-game-asp --runtime "DOTNET|6.0"

az webapp create --name tailspin-space-game-web-staging-$webappsuffix --resource-group tailspin-space-game-rg --plan tailspin-space-game-asp --runtime "DOTNET|6.0"
```

For learning purposes, here, you apply the same App Service plan, B1 Basic, to each App Service instance. In practice, you would assign a plan that matches your expected workload.

For example, for the environments that map to the Dev and Test stages, B1 Basic might be appropriate because you want only your team to access the environments.

For the Staging environment, you would select a plan that matches your production environment. That plan would likely provide greater CPU, memory, and storage resources. Under the plan, you can run performance tests, like load tests, in an environment that resembles your production environment. You can run the tests without affecting live traffic to your site.


2. To list the host name and state of each App Service instance, run the following az webapp list command.

```shell
az webapp list \
  --resource-group tailspin-space-game-rg \
  --query "[].{hostName: defaultHostName, state: state}" \
  --output table
```

The commands I ran: 

```shell
az login

# to create an App Service Plan
az appservice plan create  --name azure-deploy-applications-with-ado-tutorial-asp  --resource-group my-resource-group    --sku B1  --is-linux

# to create different App Service instances 
az webapp create  --name azure-deploy-applications-with-ado-tutorial-test --resource-group my-resource-group  --plan azure-deploy-applications-with-ado-tutorial-asp  --runtime "DOTNET|6.0"

az webapp create  --name azure-deploy-applications-with-ado-tutorial-staging --resource-group my-resource-group  --plan azure-deploy-applications-with-ado-tutorial-asp  --runtime "DOTNET|6.0"

# to start the existing App Service
az webapp start --name azure-deploy-applications-with-ado-tutorial --resource-group my-resource-group 

# to list the existing App Service instances + their status
az webapp list  --resource-group my-resource-group  --query "[].{hostName: defaultHostName, state: state}"  --output table
```

3. Create pipeline variables in Azure Pipelines
In Create a release pipeline with Azure Pipelines, you added a variable to your pipeline that stores the name of your web app in App Service. Here you do the same. But this time, you add one variable for each App Service instance that corresponds to a Dev, Test, or Staging stage in your pipeline.

You could hard-code these names in your pipeline configuration, but if you define them as variables, your configuration will be more reusable. Additionally, if the names of your App Service instances change, you can update the variables and trigger your pipeline without modifying your configuration.


