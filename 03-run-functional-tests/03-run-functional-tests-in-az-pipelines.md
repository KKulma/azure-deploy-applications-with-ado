## Introduction

In this module, you'll add functional tests to the pipeline. These tests verify an application's behavior.

In the Create a multistage pipeline by using Azure Pipelines module, you helped the Tailspin Toys web team design and build a multistage release pipeline. The team uses the pipeline to move changes through a series of stages. Changes move through the Dev stage, the Test stage, and finally the Staging stage, which resembles a production environment.

The stages that you and the team defined provide the overall shape of the pipeline. But you can add more to each stage. For example, in the Test stage, Amita still tests the web application manually as she always has. When she's satisfied, she manually promotes the application to Staging. In Staging, management reviews the new features and decides whether to make the release publicly available.

In the Run quality tests in your build pipeline using Azure Pipelines module, you incorporated unit and code coverage tests into the build process. These tests help avoid regression bugs and ensure that the code meets the company's standards for quality and style. But what kinds of tests can you run after a service is operational and deployed to an environment?

**Learning objectives**
After completing this module, you'll be able to:

- Define the role of functional tests and identify some popular kinds of tests you can run.
- Map manual testing steps to automated test cases.
- Run automated UI tests locally and in the pipeline using Selenium.


## What is functional testing?

**Functional tests** verify that each function of the software does what it should. How the software implements each function isn't important in these tests. What's important is that the software behaves correctly. You provide input and check that the output is what you expect.

**Nonfunctional tests** check characteristics like performance and reliability. An example of a nonfunctional test is checking how many people can simultaneously sign up in to the app. Load testing is another example of a nonfunctional test (performance and reliability).

#### What kinds of functional tests can I run?
There are many kinds of functional tests. They vary by the functionality that you need to test and the time or effort that they typically require to run.

The following sections present some commonly used functional tests.

**Smoke testing**

Smoke testing verifies the most basic functionality of your application or service. These tests are often run before more complete and exhaustive tests. Smoke tests should run quickly.

For example, say you're developing a website. Your smoke test might use curl to verify that the site is reachable and that fetching the home page produces a 200 (OK) HTTP status. If fetching the home page produces another status code, such as 404 (Not Found) or 500 (Internal Server Error), you know that the website isn't working. You also know that there's no reason to run other tests. Instead, you diagnose the error, fix it, and restart your tests.

**Unit testing**

In short, unit testing verifies the most fundamental components of your program or library, such as an individual function or method. You specify one or more inputs along with the expected results. The test runner performs each test and checks to see whether the actual results match the expected results.

As an example, let's say you have a function that performs an arithmetic operation that includes division. You might specify a few values that you expect your users to enter. You also specify edge-case values such as 0 and -1. If you expect a certain input to produce an error or exception, you can verify that the function produces that error.

The UI tests that you'll run later in this module are unit tests.


**Integration testing**

Integration testing verifies that multiple software components work together to form a complete system. For example, an e-commerce system might include a website, a products database, and a payment system. You might write an integration test that adds items to the shopping cart and then purchases the items. The test verifies that the web application can connect to the products database and then fulfill the order.

You can combine unit tests and integration tests to create a layered testing strategy. For example, you might run unit tests on each of your components before running the integration tests. If all unit tests pass, you can move on to the integration tests with greater confidence.


**Regression testing**

A regression occurs when existing behavior either changes or breaks after you add or change a feature. Regression testing helps determine whether code, configuration, or other changes affect the software's overall behavior.

Regression testing is important because a change in one component can affect the behavior of another component. For example, say you optimize a database for write performance. The read performance of that database, which is handled by another component, might unexpectedly drop. The drop in read performance is a regression.

You can use various strategies to test for regression. These strategies typically vary by the number of tests you run to verify that a new feature or bug fix doesn't break existing functionality. However, when you automate the tests, regression testing might involve just running all unit tests and integration tests each time the software changes.

**Sanity testing** 

Sanity testing involves testing each major component of a piece of software to verify that the software appears to be working and can undergo more thorough testing. You can think of sanity tests as being less thorough than regression tests or unit tests, but sanity tests are broader than smoke tests.

Although sanity testing can be automated, it's often done manually in response to a feature change or a bug fix. For example, a software tester who's validating a bug fix might also verify that other features are working by entering some typical values. If the software appears to be working as expected, it can then go through a more thorough test pass.


**User interface testing**

User interface (UI) testing verifies the behavior of an application's user interface. UI tests help verify that the sequence, or order, of user interactions, leads to the expected result. These tests also help verify that input devices, such as the keyboard or mouse, affect the user interface properly. You can run UI tests to verify the behavior of a native Windows, macOS, or Linux application. Or you can use UI tests to verify that the UI behaves as expected across web browsers.

A unit test or integration test might verify that the UI receives data correctly. But UI testing helps verify that the user interface displays correctly and that the result functions as expected for the user.

For example, a UI test might verify that the correct animation appears in response to a button click. A second test might verify that the same animation appears correctly when the window is resized.

In this module, you work with UI tests that are coded by hand. But you can also use a capture-and-replay system to automatically build your UI tests.


**Usability testing**

Usability testing is a form of manual testing that verifies an application's behavior from the user's perspective. **Usability testing is typically done by the team that builds the software.**

Whereas UI testing focuses on whether a feature behaves as expected, usability testing helps verify that the software is intuitive and meets the user's needs. In other words, usability testing helps verify whether the software is "usable."

For example, say you have a website that includes a link to the user's profile. A UI test can verify that the link is present and that it brings up the user's profile when the link is clicked. However, if humans can't easily locate this link, they might become frustrated when they try to access their profile.


**User acceptance testing**

User acceptance testing (UAT), like usability testing, focuses on an application's behavior from the user's perspective. Unlike acceptance testing, **UAT is typically done by real end users.**

Depending on the software, end users might be asked to complete specific tasks. Or they might be allowed to explore the software without following specific guidelines. For custom software, UAT typically happens directly with the client. For more general-purpose software, teams might run beta tests. In beta tests, users from different geographic regions or those with particular interests receive early access to the software.

Feedback from testers can be direct or indirect. Direct feedback might come in the form of verbal comments. Indirect feedback can come in the form of measuring testers' body language, eye movements, or the time they take to complete certain tasks.



# EXERCISES


## Create the Azure App Service emvironments

Select an Azure region 

```shell
az account list-locations --query "[].{Name: name, DisplayName: displayName}" --output table
```

```shell
az configure --defaults location=ukwest
```

## Create the App Service instances

Create a new resource group 

```shell
az group create --name tailspin-space-game-rg
```

Create a new App Service plan

```shell
az appservice plan create --name tailspin-space-game-asp --resource-group tailspin-space-game-rg --sku B1 --is-linux
```

Create an App Service instances, each for Dev, Test and Staging stages 

```shell
az webapp create --name tailspin-space-game-web-dev-kk --resource-group tailspin-space-game-rg --plan tailspin-space-game-asp --runtime "DOTNET|6.0"
```

```shell
az webapp create --name tailspin-space-game-web-test-kk --resource-group tailspin-space-game-rg --plan tailspin-space-game-asp --runtime "DOTNET|6.0"
```

```shell
az webapp create --name tailspin-space-game-web-staging-kk --resource-group tailspin-space-game-rg --plan tailspin-space-game-asp --runtime "DOTNET|6.0"
```

Check the status of App Service instances

```shell
az webapp list --resource-group tailspin-space-game-rg --query "[].{hostName: defaultHostName, state: state}" --output table
```

# Plan the UI Tests

### What are locators in Selenium?
In a Selenium test, a locator selects an HTML element from the DOM (Document Object Model) to act on. Think of the DOM as a tree or graph representation of an HTML document. Each node in the DOM represents a part of the document.
In a Selenium test, you can locate an HTML element by its:

- id attribute.
- name attribute.
- XPath expression.
- Link text or partial link text.
- Tag name, such as body or h1.
- CSS class name.
- CSS selector.

The locator you use depends on the way your HTML code is written and the kinds of queries you want to perform.

In an HTML document, the id attribute specifies a unique identifier for an HTML element. Here, you'll use the id attribute to query for elements on the page because each identifier must be unique. This makes the id attribute one of the easiest ways to query for elements in a Selenium test.

### Plan the automated tests
Here's what we'll do:

- Create an NUnit project that includes Selenium. The project will be stored in the directory along with the app's source code.
- Write a test case that uses automation to click the specified link. The test case verifies that the expected modal window appears.
- Use the id attribute we saved to specify the parameters to the test case method. This task creates a sequence, or series, of tests.
- Configure the tests to run on Chrome, Firefox, and Microsoft Edge. This task creates a matrix of tests.
- Run the tests and watch each web browser come up automatically.
- Watch Selenium automatically run through the series of tests for each browser.
- In the console window, verify that all the tests pass.


# Write the UI Tests 

## Write the unit test code

### Define the HomePageTest class

```
# HomePageTest.cs
public class HomePageTest
{
} 
```
(We need to mark this class as public so that it's available to the NUnit framework.)

### Add the IWebDriver member variable

`IWebDriver` is the programming interface that you use to launch a web browser and interact with webpage content.

Think of an interface as a specification or blueprint for how a component should behave. An interface provides the methods, or behaviors, of that component. But the interface doesn't provide any of the underlying details. You or someone else would create one or more concrete classes that implement that interface. Selenium provides the concrete classes that we need.

The diagram shows three of the methods that `IWebDriver` provides: `Navigate`, `FindElement`, and `Close`.

The three classes shown here, `ChromeDriver`, `FirefoxDriver`, and `EdgeDriver`, each implement IWebDriver and its methods. There are other classes, such as SafariDriver, that also implement `IWebDriver`. Each driver class can control the web browser that it represents.

```
public class HomePageTest
{
    private IWebDriver driver;
}
```


### Define the test fixtures

We can use **test fixtures** to run the entire set of tests multiple times, one time for each browser that we want to test on

In NUnit, you use the TestFixture attribute to define your test fixtures

```
[TestFixture("Chrome")]
[TestFixture("Firefox")]
[TestFixture("Edge")]
public class HomePageTest
{
    private IWebDriver driver;
}
```


Next, we need to define a constructor for our test class. The constructor is called when NUnit creates an instance of this class. As its argument, the constructor takes the string that we attached to our test fixtures. Here's what the code looks like:

```
[TestFixture("Chrome")]
[TestFixture("Firefox")]
[TestFixture("Edge")]
public class HomePageTest
{
    private string browser;
    private IWebDriver driver;

    public HomePageTest(string browser)
    {
        this.browser = browser;
    }
}
```

We added the browser member variable so that we can use the current browser name in our setup code. Let's write the setup code next.

### Define the Setup method

Next, we need to assign our `IWebDriver` member variable to a class instance that implements this interface for the browser we're testing on. The `ChromeDriver`, `FirefoxDriver`, and `EdgeDriver` classes implement this interface for Chrome, Firefox, and Edge, respectively.

Let's create a method named Setup that sets the driver variable. We use the `OneTimeSetUp` attribute to tell NUnit to run this method one time per test fixture.

```
[OneTimeSetUp]
public void Setup()
{
}
```

In the `Setup` method, we can use a `switch` statement to assign the `driver` member variable to the appropriate concrete implementation based on the browser name. Let's add that code now.

```
// Create the driver for the current browser.
switch(browser)
{
    case "Chrome":
    driver = new ChromeDriver(
        Environment.GetEnvironmentVariable("ChromeWebDriver")
    );
    break;
    case "Firefox":
    driver = new FirefoxDriver(
        Environment.GetEnvironmentVariable("GeckoWebDriver")
    );
    break;
    case "Edge":
    driver = new EdgeDriver(
        Environment.GetEnvironmentVariable("EdgeWebDriver"),
        new EdgeOptions
        {
            UseChromium = true
        }
    );
    break;
    default:
    throw new ArgumentException($"'{browser}': Unknown browser");
}
```

The constructor for each driver class takes an optional path to the driver software Selenium needs to control the web browser. Later, we'll discuss the role of the environment variables shown here.

In this example, the EdgeDriver constructor also requires additional options to specify that we want to use the Chromium version of Edge.

### Define the helper methods

We'll need to repeat two actions throughout the tests:

- Finding elements on the page, such as the links that we click and the modal windows that we expect to appear
- Clicking elements on the page, such as the links that reveal the modal windows and the button that closes each modal

Let's write two helper methods, one for each action. We'll start with the method that finds an element on the page.

#### Write the FindElement helper method
         
When you locate an element on the page, it's typically in response to some other event, such as the page loading or the user entering information. Selenium provides the WebDriverWait class, which allows you to wait for a condition to be true. If the condition isn't true within the given time period, WebDriverWait throws an exception or error. We can use the WebDriverWait class to wait for a given element to be displayed and to be ready to receive user input.

To locate an element on the page, use the By class. The By class provides methods that let you find an element by its name, by its CSS class name, by its HTML tag, or in our case, by its id attribute:

```
private IWebElement FindElement(By locator, IWebElement parent = null, int timeoutSeconds = 10)
{
    // WebDriverWait enables us to wait for the specified condition to be true
    // within a given time period.
    return new WebDriverWait(driver, TimeSpan.FromSeconds(timeoutSeconds))
        .Until(c => {
            IWebElement element = null;
            // If a parent was provided, find its child element.
            if (parent != null)
            {
                element = parent.FindElement(locator);
            }
            // Otherwise, locate the element from the root of the DOM.
            else
            {
                element = driver.FindElement(locator);
            }
            // Return true after the element is displayed and is able to receive user input.
            return (element != null && element.Displayed && element.Enabled) ? element : null;
        });
}
```

#### Write the ClickElement helper method

 Next, let's write a helper method that clicks links. Selenium provides a few ways to write that method. One of them is the IJavaScriptExecutor interface. With it, we can programmatically click links by using JavaScript. This approach works well because it can click links without first scrolling them into view.

ChromeDriver, FirefoxDriver, and EdgeDriver each implement IJavaScriptExecutor. We need to cast the driver to this interface and then call ExecuteScript to run the JavaScript click() method on the underlying HTML object: 

```
private void ClickElement(IWebElement element)
{
    // We expect the driver to implement IJavaScriptExecutor.
    // IJavaScriptExecutor enables us to execute JavaScript code during the tests.
    IJavaScriptExecutor js = driver as IJavaScriptExecutor;

    // Through JavaScript, run the click() method on the underlying HTML object.
    js.ExecuteScript("arguments[0].click();", element);
}
```

### Define the test method

Now, we're ready to define the test method. Based on the manual tests that we ran earlier, let's call this method `ClickLinkById_ShouldDisplayModalById`. It's a good practice to give test methods descriptive names that define precisely what the test accomplishes. Here, we want to click a link defined by its id attribute. Then we want to verify that the proper modal window appears, also by using its id attribute:

```
public void ClickLinkById_ShouldDisplayModalById(string linkId, string modalId)
{
}
```

Let's define what this test should do:

- Locate the link by its id attribute and then click the link.
- Locate the resulting modal.
- Close the modal.
- Verify that the modal was displayed successfully.
- Ignore the test if the driver couldn't be loaded
- Close the modal only if the modal was successfully displayed.

```
public void ClickLinkById_ShouldDisplayModalById(string linkId, string modalId)
{
    // Skip the test if the driver could not be loaded.
    // This happens when the underlying browser is not installed.
    if (driver == null)
    {
        Assert.Ignore();
        return;
    }

    // Locate the link by its ID and then click the link.
    ClickElement(FindElement(By.Id(linkId)));

    // Locate the resulting modal.
    IWebElement modal = FindElement(By.Id(modalId));

    // Record whether the modal was successfully displayed.
    bool modalWasDisplayed = (modal != null && modal.Displayed);

    // Close the modal if it was displayed.
    if (modalWasDisplayed)
    {
        // Click the close button that's part of the modal.
        ClickElement(FindElement(By.ClassName("close"), modal));

        // Wait for the modal to close and for the main page to again be clickable.
        FindElement(By.TagName("body"));
    }

    // Assert that the modal was displayed successfully.
    // If it wasn't, this test will be recorded as failed.
    Assert.That(modalWasDisplayed, Is.True);
}
```

### Define test case data

In NUnit, you can provide data to your tests in a few ways. Here, we use the `TestCase` attribute. This attribute takes arguments that it later passes back to the test method when it runs. We can have multiple `TestCase` attributes that each test a different feature of our app. Each `TestCase` attribute produces a test case that's included in the report that appears at the end of a pipeline run.

Andy adds these `TestCase` attributes to the test method. These attributes describe the `Download` game button, one of the game screens, and the top player on the leaderboard. Each attribute specifies two id attributes: one for the link to click and one for the corresponding modal window.

```
// Download game
[TestCase("download-btn", "pretend-modal")]
// Screen image
[TestCase("screen-01", "screen-modal")]
// // Top player on the leaderboard
[TestCase("profile-1", "profile-modal-1")]
public void ClickLinkById_ShouldDisplayModalById(string linkId, string modalId)
{
...
```

For each `TestCase` attribute, the first parameter is the id attribute for the link to click on. The second parameter is the id attribute for the modal window that we expect to appear. You can see how these parameters correspond to the two-string arguments in our test method.


# Exercise - Run the UI tests locally and in the pipeline

## Export environment variables

Later in this module, you'll run Selenium tests on Windows Server 2019.
For each driver, you have the environment variable that maps to the location of that driver. For example, `ChromeWebDriver` maps to the location of the Chrome driver.

The unit tests code is already set up to read these environment variables. These variables tell Selenium where to find the driver executable files. To run the unit tests locally, you need to export these same environment variables.


```shell
driverDir="/Users/user/mslearn-tailspin-spacegame-web-deploy/Tailspin.SpaceGame.Web.UITests/bin/Release/net6.0"
```

```shell
export ChromeWebDriver=$driverDir
export EdgeWebDriver=$driverDir
export GeckoWebDriver=$driverDir
```


## Run the UI tests locally

The `Setup` method in HomePageTest.cs navigates to the Space Game home page after it sets the `driver` member variable.

Although you could hard-code the site URL, here we read the URL from an environment variable named `SITE_URL`. This way, you can run the tests multiple times against different URLs.


``` 
// Navigate to the site.
// The site name is stored in the SITE_URL environment variable to make 
// the tests more flexible.
string url = Environment.GetEnvironmentVariable("SITE_URL");
driver.Navigate().GoToUrl(url + "/");
```

To run the tests locally:

1. Run the following commands in the new terminal window:

```shell
dotnet build --configuration Release
dotnet run --configuration Release --no-build --project Tailspin.SpaceGame.Web
```

2. Make a note of the local website link, in this example it is `http://localhost:5000`.
3. Switch back to the terminal window where you set the environment variables in the previous step, and ensure that you're in your project's root directory. Here's an example:

```shell
cd ~/mslearn-tailspin-spacegame-web-deploy
```

4. Export the `SITE_URL` environment variable. Use the locally running link that you got from the previous step.
```shell
export SITE_URL="http://localhost:5000"
```

This variable points to the Space Game website that Microsoft hosts

5. Run the UI tests.

```shell
dotnet test --configuration Release Tailspin.SpaceGame.Web.UITests
```

This code runs the tests that are located in the Tailspin.SpaceGame.Web.UITests project.

As the tests run, one or more browsers appear. Selenium controls each browser and follows the test steps that you defined.

6. From the terminal, trace the output of each test. Also note the test-run summary at the end.

This example shows that out of nine tests, all nine tests succeeded and zero tests were skipped:

```shell
Passed!  - Failed:     0, Passed:     9, Skipped:     0, Total:     9, Duration: 5 s
```


### Add the SITE_URL variable to Azure Pipelines
In ADO, add SITE_URL env var. As its value, enter the URL of the App Service instance that corresponds to your **test** environment, such as `http://tailspin-space-game-web-test-10529.azurewebsites.net`.

### Modify the pipeline configuration

The file includes these three changes:

- The `dotnetSdkVersion` variable is moved to the top of the file so that multiple stages can access it. Here the Build stage and Test stage require this version of .NET Core.

- The _Build_ stage publishes only the Space Game website package as the build artifact. Previously, you published the artifacts like this:

```yaml
- task: DotNetCoreCLI@2
  displayName: 'Publish the project - $(buildConfiguration)'
  inputs:
    command: 'publish'
    projects: '**/*.csproj'
    publishWebProjects: false
    arguments: '--no-build --configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/$(buildConfiguration)'
    zipAfterPublish: true
```

This task generates two build artifacts: the Space Game website package and the compiled UI tests. We build the UI tests during the Build stage to ensure that they'll compile during the Test stage, but we don't need to publish the compiled test code. We build it again during the Test stage when the tests run.

- The _Test_ stage includes a second job that builds and runs the tests. This job resembles the one that you used in the Run quality tests in your build pipeline by using Azure Pipelines module. In that module, you ran NUnit tests that verified the leaderboard's filtering functionality.

    Recall that a deployment job is a special type of job that plays an important role in your deployment stages. The second job is a normal job that runs the Selenium tests on a Windows Server 2019 agent. Although we use a Linux agent to build the application, here we use a Windows agent to run the UI tests. We use a Windows agent because Amita runs manual tests on Windows, and that's what most customers use.

    The `RunUITests` job depends on the `Deploy` job to ensure that the jobs run in the correct order. You'll deploy the website to App Service before you run the UI tests. If you don't specify this dependency, jobs within the stage can run in any order or run in parallel.




To stop the App Service instance, just run the code below

```shell
az webapp stop --resource-group tailspin-space-game-rg --name tailspin-space-game-web-staging-kk
```

