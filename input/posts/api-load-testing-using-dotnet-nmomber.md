Title: API Load Testing using .NET NBomber
Published: 26/05/2022
Image: /posts/images/nbomber.jpg
Tags:
  - dotnet
  - nbomber
  - load-testing
  - api
---
NBomber is an open source load testing framework which help .NET developers to load test their applications. NBomber is a lightweight framework for writing load tests which you can use to test literally any system and simulate any production workload. With NBomber you can test any PULL or PUSH system (HTTP, WebSockets, GraphQl, gRPC, SQL Database, MongoDb, Redis etc).

In this post, I'll show you a very simple example of API load testing using NBomber C#. 

#### Step 1: Create WebAPI for Load Test

To avoid network latency, I have created sample WebAPI project from visual studio for load test which is accessible using [http://localhost:5104/WeatherForecast](http://localhost:5104/WeatherForecast) endpoint. you can use your existing application endpoint to do load test.

#### Create Load Test Application
There are two ways to run a test with NBomber:

- **XUnit or NUnit**: You can write your NBomber load test same as your Unit/Integration test.
- **Console application**: You can run NBomber load test using console application which will shows result in console window output. You can write only one test with this approach.

To keep it simple, I have created simple C# console application for NBomber load test and add **NBomber** nuget package on it.

<img src="/posts/images/nbomber-1.jpg" width="100%">

#### Setup API Load Test
There are few important building blocks of NBomber:

- **Step**: Step and Scenario play the most important role in building real-world simulations with NBomber. To represent users behaviors, testers should define scenarios with steps. The scenario is basically a workflow that virtual users will follow. The step represents a single user action like login, logout, etc. 
- **Scenario**: Scenario is basically a workflow that virtual users will follow. It helps you organize steps into user actions.
- **NBomberRunner**: NBomberRunner is responsible for registering and running scenarios under Test Suite. Also it provides configuration points related to infrastructure, reporting, loading plugins. 

By using all three build blocks, I have written my load test code to send 200 request per-second for 30 seconds duration.
<img src="/posts/images/nbomber-2.jpg" width="100%">

#### NBomber Load Test Result
While the application is running it will write some basic telemetry to the console about the progress of the tests. Once load test complete, console will show you the result of load test with succeeded, failed and latency for each step.
<img src="/posts/images/nbomber-3.jpg" width="100%">

NBomber also generates HTML report which you can find from NBomber console result window.

<img src="/posts/images/nbomber-4.jpg" width="100%">
<img src="/posts/images/nbomber-5.jpg" width="100%">

### Conclusion
I used to write multi-thread custom code to do a basic load test of my application. I really liked NBomber and I'll share more sophisticated example in my future post.

Happy coding.