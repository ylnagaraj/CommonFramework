# IRIS STAFFOLOGY - TEST AUTOMATION

## Overview

In Staffology, there are 3 different layers in scope for test automation, as mentioned below:

	1. API (RestAPIs)
	2. Web (.net application)
	3. Sharpe Engine (nuget package)

## Tool Stack

Following test libraries and technologies are used for automation of Staffology testing

### Test Libraries
	1. API Layer: RestSharp
	2. Web Layer: Selenium Webdriver
	3. Sharpe Engine: C# Coding
	
### Other Libraries and Technologies
	TestCase Writing			: SpecFlow (Behavior Driven Testing)
	Unit Testing Framework		: MSTest
	Assertion Library			: FluentAssertions
	Continuous Integration		: Azure DevOps
	Config files				: .JSON files
	Reporting					: Extent Reports
	Logging						: NLog
	JSON Framework				: NewtonSoft.JSON
	Dependent Test Framework	: Iris.Payroll.Core.TestFramework
	Dependent Nuget Package		: Staffology.Payroll (Sharpe Engine)
	
## Test Automation Framework Architecture

### Architecture
![alt text][snapshot]
[snapshot]: https://github.com/ylnagaraj/CommonFramework/IRIS_Staffology-TestAutomationFramework_Architecture.png "Staffology Test Framework"

### Project Structure in the Solution
![alt text][snapshot]
[snapshot]: https://github.com/ylnagaraj/CommonFramework/ProjectStructure.jpg "Project Structure"

## Setup

### Installation

Installing the framework is very straight forward.

Clone this repo [staffology-testing](https://dev.azure.com/iris-payroll/Staffology/_git/staffology-testing)
Create a [Personal Access Token](https://dev.azure.com/iris-payroll/_usersSettings/tokens) to access the private NuGet feed in Azure DevOps Auth with the private NuGet feed

Restore all NuGet packages and build the solution

### Configuration

The Test Framework can be configured via a json file in Staffology.Automation.Base/Config/appsettings.json
This json file determines the user, environment, and any additional behaviours of the tests that will run.
User details are loaded from Staffology.Automation.Base/Config/Users/{user}.usersettings.json
Environment details are loaded from Staffology.Automation.Base/Config/Environments/{Environment}.envsettings.json

To add additional users/environments, simply follow the same format.

### Execution

Execution is as simple as building the solution, then using test explorer to run a test inside Visual Studio.
You can also use the dotnet CLI dotnet test --filter "Category=X", where X is the category of Test(s) that you want to run (See feature files, categories include, Web, Sharpe, and API at the time of writing)
Alternatively, use Docker to run tests in an isolated execution environment.
Test Automation is also included in the deployment pipeline. The latest tests/functionality merged into the master branch of the test framework, will be executed.

## Integration & Orchestration

At the time of writing, the test automation framework has been integrated into Azure DevOps to facilate full automation. This is facilitated through Docker and Azure DevOps pipelines.

### Docker
The Test Framework has been built to run inside a Docker container. We currently use the official Microsoft dotnet5 SDK base image, installing the latest Chrome web driver, and copying in our automation framework. Tests are executed using the native dotnet test, with results stored in /app_result in trx format, that can be mounted to the executing host. This allows test results to be parsed, either on a local machine, or on a deployment agent.
User, Environment and Filters, are all passed through to the docker container on execution, to shape the functionality and target of the test framework. These environment variables override any configuration within the appsettings file stored within the repo, to ensure reliable execution.
Tests can be executed with the run_test.ps1 script locally, or using docker compose (The syntax VAR=val docker compose up, is the best syntax to use, using a bash terminal)

### Azure DevOps Pipelines
The Test Framework is automatically re-built via an Azure DevOps pipeline. We have two triggers for a build - a commit/merge to the master branch of the test framework repo, or on completion of the Sharpe pipeline, which publishes a NuGet package.
This pipeline builds the Test Framework solution, the drops it into a container, which is stored inside a private Azure Container Registry 

Details of the pipeline can be seen [here](https://dev.azure.com/iris-payroll/Staffology/_build?definitionId=53)

On deployment, testing tasks are triggered - these run docker compose, with the appropriate environment variables (such as environment, test selection), which runs automated tests on the hosted Azure agents.
Results are then published to the pipeline. See [Task Groups](https://dev.azure.com/iris-payroll/Staffology/_taskgroups) 