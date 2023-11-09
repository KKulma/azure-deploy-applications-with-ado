# Introduction
Non-functional tests check characteristics like performance and reliability.

# What is nonfunctional testing?

In short, functional tests verify that each function of the software does what it should. In other words, **functional tests verify an application's functionality**.

**Nonfunctional tests check nonfunctional aspects of an application, such as performance and reliability.** You can also run nonfunctional tests on systems that aren't apps, such as infrastructure components. One example of a nonfunctional test is to determine how many people can simultaneously sign in to an application without causing a problem, such as slower response times.

Here are some of the questions that nonfunctional tests can answer:

- How does the application perform under normal circumstances?
- How does the application perform when many users sign in concurrently?
- How secure is the application?

## Performance testing
The goal of performance testing is to improve the speed, scalability, and stability of an application. Testing for speed determines how quickly an application responds. Testing for scalability determines the maximum user load an application can handle. Testing for stability determines whether the application remains stable under different loads. Two common types of performance tests are load tests and stress tests.

### Load testing
Load tests determine the performance of an application under realistic loads. For example, load tests can determine how well an application performs at the upper limit of its service-level agreement (SLA). Basically, load testing determines the behavior of the application when multiple users need it at the same time.

Users aren't necessarily people. For example, a load test for printer software might send the application large amounts of data. A load test for a mail server might simulate thousands of concurrent users.

Load testing is also a good way to uncover problems that exist only when the application is operating at its limits. That's when issues such as buffer overflow and memory leaks can surface.

In this module, you use Apache JMeter to perform load tests. You use a set of simulated users that access the website simultaneously.

### Stress testing
Stress tests determine the stability and robustness of an application under heavy loads. The loads go beyond what's specified for the application. The stress tests determine whether the application will crash under these loads. If the application fails, the stress test checks to ensure that it fails gracefully. A graceful failure might, for example, issue an appropriate, informative error message.

Scenarios in which applications must operate under abnormally heavy loads are common. For example, in case your video goes viral, you'll want to know how well the servers can handle the extra load. Another typical scenario is high traffic on shopping websites during holiday seasons.

## Security testing
Security testing ensures that applications are free from vulnerabilities, threats, and risks. Thorough security testing finds all the possible loopholes and weaknesses of the system that might cause an information breach or a loss of revenue.

There are many types of security testing. Two of them are penetration testing and compliance testing.

### Penetration testing
Penetration testing, or pen testing, is a type of security testing that tests the insecure areas of the application. In particular, it tests for vulnerabilities that an attacker could exploit. An authorized, simulated cyber attack is usually a part of penetration testing.

### Compliance testing
Compliance testing determines whether an application is compliant with some set of requirements, inside or outside the company. For example, healthcare organizations usually need to comply with HIPAA (Health Insurance Portability and Accountability Act of 1996), which provides data privacy and, security provisions for safeguarding medical information.

An organization might also have its own security requirements. Software must be tested to make sure that it follows these requirements. For example, on Linux systems, the default user mask must be 027 or more restrictive. A security test needs to prove that this requirement is met.



# Plan load tests by using Apache JMeter

## Create the test plan

Tim brings up **Apache JMeter** on a laptop that runs Linux.

Tim then creates a new test plan file named _LoadTest.jmx_. To the file, he adds a **Thread Group**. Each simulated user runs on its own thread. A thread group controls the number of users and the number of each user's requests.

The following example shows **10 simulated users (threads)**. Each user makes 10 requests. So the system gets a total of 100 requests.

A _sampler_ is a single request made by JMeter. JMeter can query servers over HTTP, FTP, TCP, and several other protocols. Samplers generate the results that are added to the report

Next, Tim adds **Http Request Defaults** and an **Http Request** sampler to the thread group. He provides the hostname of the Space Game website that runs in the staging environment on Azure App Service.


## Run the test plan

YOu can run your test plan via terminal:

Tim opens a terminal window and runs the test plan:

Bash

```shell
apache-jmeter-5.4.1/bin/./jmeter -n -t LoadTest.jmx -o Results.xml
```

The `-n` argument specifies to run JMeter in non-GUI mode. 

The `-t` argument specifies the test plan file, _LoadTest.jmx_. 

The `-o` argument specifies the report file, _Results.xml_.


JMeter runs and produces the report file, Results.xml (the first few results):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testResults version="1.2">
<httpSample t="180" it="0" lt="95" ct="35" ts="1569306009772" s="true" lb="HTTP Request" rc="200" rm="OK" tn="Thread Group 1-1" dt="text" by="40871" sby="144" ng="1" na="1">
  <java.net.URL>http://tailspin-space-game-web-staging-1234.azurewebsites.net/</java.net.URL>
</httpSample>
<httpSample t="174" it="0" lt="96" ct="38" ts="1569306009955" s="true" lb="HTTP Request" rc="200" rm="OK" tn="Thread Group 1-1" dt="text" by="40869" sby="144" ng="1" na="1">
  <java.net.URL>http://tailspin-space-game-web-staging-1234.azurewebsites.net/</java.net.URL>
</httpSample>
<httpSample t="160" it="0" lt="121" ct="35" ts="1569306010131" s="true" lb="HTTP Request" rc="200" rm="OK" tn="Thread Group 1-1" dt="text" by="40879" sby="144" ng="2" na="2">
  <java.net.URL>http://tailspin-space-game-web-staging-1234.azurewebsites.net/</java.net.URL>
</httpSample>
```

Each sample produces a node in the report. The `t` attribute specifies the response time in milliseconds (ms). Here you see three requests that took 180 ms, 174 ms, and 160 ms.

We can visualize the results if we provide them in a format that Azure Pipelines understands. Azure Pipelines can parse an XML file that contains test results, but the file needs to be in a supported format. Supported formats include CTest, JUnit (including PHPUnit), NUnit 2, NUnit 3, Visual Studio Test (TRX), and xUnit 2. We can write an XSLT file that converts the JMeter results to **JUnit**.

## Transform the report to JUnit

XSLT stands for XSL Transformations, or eXtensible Stylesheet Language Transformations. An XSLT file resembles an XML file, but it enables you to transform an XML document to another XML format.

Rrequirements for load tests:

- The average request time should be less than one second.
- No more than 10 percent of requests should take more than one second.

The new XSLT file looks as like this:

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:math="http://exslt.org/math">
  <xsl:output method="xml" indent="yes" encoding="UTF-8"/>
  <xsl:template match="/testResults">
    <xsl:variable name="times" select="./httpSample/@t" />
    <xsl:variable name="failures" select="./httpSample/assertionResult/failureMessage" />
    <xsl:variable name="threshold" select="1000" />
    <testsuite>
      <xsl:attribute name="tests"><xsl:value-of select="count($times)" /></xsl:attribute>
      <xsl:attribute name="failures"><xsl:value-of select="count($failures)" /></xsl:attribute> 
      <testcase>
          <xsl:variable name="avg-time" select="sum($times) div count($times)" />
          <xsl:attribute name="name">Average Response Time</xsl:attribute>
          <xsl:attribute name="time"><xsl:value-of select="format-number($avg-time div 1000,'#.##')"/></xsl:attribute>
          <xsl:if test="$avg-time > $threshold">
            <failure>Average response time of <xsl:value-of select="format-number($avg-time,'#.##')"/> exceeds <xsl:value-of select="$threshold"/> ms threshold.</failure>
          </xsl:if> 
      </testcase>
      <testcase>
          <xsl:variable name="exceeds-threshold" select="count($times[. > $threshold])" />
          <xsl:attribute name="name">Max Response Time</xsl:attribute>
          <xsl:attribute name="time"><xsl:value-of select="math:max($times) div 1000"/></xsl:attribute>
          <xsl:if test="$exceeds-threshold > count($times) * 0.1">
            <failure><xsl:value-of select="format-number($exceeds-threshold div count($times) * 100,'#.##')"/>% of requests exceed <xsl:value-of select="$threshold"/> ms threshold.</failure>
          </xsl:if>
      </testcase>
    </testsuite>
  </xsl:template>
</xsl:stylesheet>
```

To summarize, this file first collects the following data from the JMeter output:

- Each HTTP request time: It collects this data by selecting the t attribute from each httpSample element. (./httpSample/@t)
- Each failure message: It collects this data by selecting all failureMessage nodes from the document. (./httpSample/assertionResult/failureMessage)

The XSLT file also sets the threshold value to 1,000 ms. This response time is the maximum that Tim defined earlier.

Given these variables, the XSLT file provides the total number of tests and the total number of failures. It then provides these two test cases:

- The average response time, and a failure if the average exceeds the threshold of 1,000 ms.
- The maximum response time, and a failure if more than 10 percent of requests exceed the threshold of 1,000 ms.

The results of the XSLT match the JUnit format, which Azure Pipelines understands. Mara and Tim name their XSLT file JMeter2JUnit.xsl.

Next, we need an XSLT processor. Let's use **xsltproc**.

Tim installs xsltproc:

```shell
sudo apt-get install xsltproc
```

Now we need to transform the JMeter report to JUnit:

```shell
xsltproc JMeter2JUnit.xsl Results.xml > JUnit.xml
```

Here's the resulting JUnit file, _JUnit.xml_:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testsuite xmlns:math="http://exslt.org/math" tests="100" failures="0">
  <testcase name="Average Response Time" time="0.17"/>
  <testcase name="Max Response Time" time="0.373"/>
</testsuite>
```

In this example, the average response time is 170 ms. The maximum response time is 373 ms. Neither test case generates a failure because both times fall below the threshold of 1,000 ms.

Shortly, you'll run these tests in the pipeline. You can think about Results.xml, the file that JMeter writes, as an intermediate file that's not published to the pipeline's test results. JUnit.xml is the final report file. This file gets published to the pipeline so that the team can visualize the results.


# Exercise - Run load tests in Azure Pipelines

## Fetch the branch from GitHub:

```shell
git fetch upstream jmeter
git checkout -B jmeter upstream/jmeter
```
## Add variables to Azure Pipelines

When you run JMeter from the command line, you use the `-J` argument to set the hostname property. Here's an example:

```shell
apache-jmeter-5.4.3/bin/./jmeter -n -t LoadTest.jmx -o Results.xml -Jhostname=tailspin-space-game-web-staging-1234.azurewebsites.net
```


Here you set the `STAGING_HOSTNAME` variable in Azure Pipelines. This variable points to your site's hostname that runs on App Service in your staging environment. You also set the jmeterVersion to specify the version of JMeter to install.

When the agent runs, these variables are automatically exported to the agent as environment variables. So your pipeline configuration can run JMeter this way:

```shell
apache-jmeter-5.4.3/bin/./jmeter -n -t LoadTest.jmx -o Results.xml -Jhostname=$(STAGING_HOSTNAME)
```


Add a second variable named jmeterVersion. For its value, specify 5.4.3.

Here's a summary of the changes to the `azure-pipelines.yml` file:

- The `RunLoadTests` job does load testing from a Linux agent.
- The `RunLoadTests` job depends on the `Deploy` job to ensure that the jobs are run in the correct order. You need to deploy the website to App Service before you can run the load tests. If you don't specify this dependency, jobs within the stage can run in any order or run in parallel.
- The first script task downloads and installs `JMeter`. The `jmeterVersion` pipeline variable specifies the version of JMeter to install.
- The second script task runs `JMeter`. The `-J` argument sets the hostname property in JMeter by reading the STAGING_HOSTNAME variable from the pipeline.
- The third script task installs `xsltproc`, an XSLT processor, and transforms the `JMeter` output to `JUnit`.
- The `PublishTestResults@2` task publishes the resulting JUnit report, `JUnit.xml`, to the pipeline. Azure Pipelines can help you visualize the test results.



