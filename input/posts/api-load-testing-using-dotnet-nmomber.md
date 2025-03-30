Title: Load Testing APIs with .NET NBomber: A Step-by-Step Guide

Description: Discover how to use NBomber, an open-source load testing framework, to test your .NET applications. This guide walks you through setup, execution, and result analysis for API load testing.
Published: 26/05/2022
Image: /posts/images/nbomber.jpg
PrimaryTag: dotnet
Tags:
  - dotnet
  - nbomber
  - load-testing
  - api-testing
---

[NBomber](https://nbomber.com/) is an open-source load testing framework designed to assist .NET developers in evaluating their applications under various workloads. It’s lightweight, versatile, and supports testing a wide range of systems, including HTTP, WebSockets, GraphQL, gRPC, SQL databases, MongoDB, Redis, and more. With NBomber, you can simulate real-world production scenarios to ensure your application performs reliably under stress.

In this post, I’ll guide you through a simple example of load testing an API using NBomber in C#, covering setup, execution, and result analysis.

### Step 1: Create a Sample WebAPI for Load Testing

To minimize network latency, I created a sample WebAPI project in Visual Studio, accessible at [http://localhost:5104/WeatherForecast](http://localhost:5104/WeatherForecast). You can substitute this with your own application’s endpoint for testing purposes.

### Step 2: Set Up the Load Test Application

NBomber provides two options for running tests:

- **XUnit or NUnit**: Integrate NBomber into your unit or integration test suites.
- **Console Application**: Run NBomber as a standalone console app for quick and simple tests.

For this example, I opted for the console application approach. Here’s how to set it up:

1. Create a new C# console application.
2. Install the **NBomber** NuGet package.

![Adding NBomber NuGet Package](/posts/images/nbomber-1.jpg)

### Step 3: Configure the API Load Test

NBomber relies on three key components:

- **Step**: Represents a single user action, such as sending an API request (e.g., login, logout).
- **Scenario**: A collection of steps that define a workflow virtual users will follow, simulating real-world behavior.
- **NBomberRunner**: Manages scenario execution and offers configuration options for infrastructure, reporting, and plugins.

For this test, I configured NBomber to send 200 requests per second over a 30-second duration. Below is a screenshot of the load test code:

![Load Test Code](/posts/images/nbomber-2.jpg)

### Step 4: Analyze NBomber Load Test Results

During execution, NBomber displays real-time telemetry in the console. Once the test completes, it provides a detailed summary, including the number of successful and failed requests, along with latency metrics for each step.

![Console Test Results](/posts/images/nbomber-3.jpg)

Additionally, NBomber generates an HTML report for a more in-depth analysis. The report’s file path is displayed in the console output.

![HTML Report](/posts/images/nbomber-4.jpg)
![HTML Report Details](/posts/images/nbomber-5.jpg)

### Conclusion

Before NBomber, I relied on custom multi-threaded code for basic load testing, which was time-consuming and less efficient. NBomber streamlines the process, and I look forward to sharing more advanced examples in future posts.

Happy coding!