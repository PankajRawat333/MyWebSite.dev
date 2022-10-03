Title: Run .NET Lambda Function Locally Using LocalStack
Published: 05/10/2022
Image: /posts/images/localstack.png
Tags:
  - aws
  - lambda
  - dotnet
  - localstack
  - docker
---

This is a continuation of my previous blog [Amazon S3 local development using .NET](https://rawatpankaj.com/posts/amazon-s3-local-development-using-localstack). If you are new to LocalStack, I highly recommend you to check previous post where I have covered basic of LocalStack.

### Prerequisites
- Make sure that you have a working [docker](https://docs.docker.com/get-docker/) environment on your machine before moving on.
- Make sure [LocalStack](https://localstack.cloud/) is running on port 4566.
- [Visual Studio](https://visualstudio.microsoft.com/downloads/) or [Visual Studio Code](https://code.visualstudio.com/download) downloaded and installed
- AWS Toolkit for Visual Studio (see [setup instructions](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/setup.html))
- [.NET 6](https://dotnet.microsoft.com/en-us/download/dotnet/6.0)
- An active AWS account (Optional).
- [AWS Command Line Interface (AWS CLI) version 2.x](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html), installed and [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) on macOS, Linux, or Windows (Optional).


## Create Lambda Function
Before running lambda function in LocalStack, we need to create lambda function in our development machine.
- Open Visual Studio and create new project
- Select `AWS Lambda Project (.NET Core - C#)`.
- Enter project name `SampleLambdaFunction` and click on create button.
- Select `Empty Function` Template and click Finish.

We can quickly test the newly created Lambda function from Visual Studio before moving ahead.

To test a Lambda function in AWS or LocalStack, we need to publish the output of lambda function. To get that output in zip format, Add below lines in `SampleLambdaFunction.csproj`

```xml
<Target Name="ZipOutput" AfterTargets="Build">
		<ZipDirectory SourceDirectory="$(OutputPath)" DestinationFile="$(MSBuildProjectDirectory)\$(MSBuildProjectName).zip" Overwrite="true"></ZipDirectory>
	</Target>
```
After adding above lines `SampleLambdaFunction.csproj` file look like below
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <GenerateRuntimeConfigurationFiles>true</GenerateRuntimeConfigurationFiles>
    <AWSProjectType>Lambda</AWSProjectType>
    <!-- This property makes the build directory similar to a publish directory and helps the AWS .NET Lambda Mock Test Tool find project dependencies. -->
    <CopyLocalLockFileAssemblies>true</CopyLocalLockFileAssemblies>
    <!-- Generate ready to run images during publishing to improve cold start time. -->
    <PublishReadyToRun>true</PublishReadyToRun>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Amazon.Lambda.Core" Version="2.1.0" />
    <PackageReference Include="Amazon.Lambda.Serialization.SystemTextJson" Version="2.3.0" />
  </ItemGroup>
	<Target Name="ZipOutput" AfterTargets="Build">
		<ZipDirectory SourceDirectory="$(OutputPath)" DestinationFile="$(MSBuildProjectDirectory)\$(MSBuildProjectName).zip" Overwrite="true"></ZipDirectory>
	</Target>
</Project>
```

## Setup Lambda Function Integration Test
In this step, we will create a integration test project to verify lambda function running in LocalStack Or AWS Account.

- Add new project in same solution from Visual Studio
- Select `xUnit Test Project` and click Next.
- Enter test project name `SampleLambdaFunction.IntegrationTests` and click Create.
- Select framework `.NET 6` and click Create.

 Add below code in integration test project.

```cs
using Amazon.IdentityManagement;
using Amazon.Lambda;
using Amazon.Lambda.Model;
using System.Text.Json;
using Xunit;

namespace SampleLambdaFunction.IntegrationTests
{
    public class FunctionUnitTest
    {
        private IAmazonLambda _lambdaClient;
        private readonly bool _isRunningOnLocal;
        private readonly string _functionName;
        private readonly string _serviceUrl;

        public FunctionUnitTest()
        {
            _isRunningOnLocal = true;
            _functionName = "SampleLambdaFunction";
            _serviceUrl = "http://localhost:4566";
        }
        [Fact]
        public async Task ProcessSuccessful()
        {
            //Arrange
            if (_isRunningOnLocal)
            {
                await SetupLocalStackLambda();
            }
            else
            {
                _lambdaClient = new AmazonLambdaClient();
            }
            var request = new InvokeRequest
            {
                FunctionName = _functionName,
                // force sync lambda invocation
                InvocationType = InvocationType.RequestResponse,
                LogType = LogType.Tail,
                Payload = JsonSerializer.Serialize("hello")
            };

            // Act
            var result = await _lambdaClient.InvokeAsync(request);

            // Assert
            Assert.Equal(200, result.StatusCode);
            Assert.NotEqual("Unhandled", result.FunctionError);
        }

        private async Task SetupLocalStackLambda()
        {
            // Create the IAM client object.
            using (var client = new AmazonIdentityManagementServiceClient(new AmazonIdentityManagementServiceConfig
            {
                ServiceURL = _serviceUrl
            }))
            {
                _lambdaClient = new AmazonLambdaClient(new AmazonLambdaConfig
                {
                    ServiceURL = _serviceUrl
                });
                string lambdaRoleName = "lambda-ima-role-for-update-status";
                Amazon.IdentityManagement.Model.CreateRoleResponse createRoleResponse = await client.CreateRoleAsync(new Amazon.IdentityManagement.Model.CreateRoleRequest
                {
                    AssumeRolePolicyDocument = "{\"Version\": \"2012-10-17\", \"Statement\": [{ \"Effect\": \"Allow\", \"Principal\": {\"Service\": \"lambda.amazonaws.com\"}, \"Action\": \"sts:AssumeRole\"}]}",
                    RoleName = lambdaRoleName
                });

                _ = await client.AttachRolePolicyAsync(new Amazon.IdentityManagement.Model.AttachRolePolicyRequest
                {
                    RoleName = lambdaRoleName,
                    PolicyArn = "arn:aws:iam::aws:policy/AWSLambda_FullAccess"
                });
                
                string lambdaArtifact = "../SampleLambdaFunction/SampleLambdaFunction.zip";
                byte[] bytes = await File.ReadAllBytesAsync(lambdaArtifact);
                MemoryStream stream = new(bytes);

                _ = await _lambdaClient.CreateFunctionAsync(new CreateFunctionRequest
                {
                    FunctionName = _functionName,
                    Timeout = 900,
                    Code = new FunctionCode
                    {
                        ZipFile = stream
                    },
                    Handler = "SampleLambdaFunction::SampleLambdaFunction.Function::FunctionHandler",
                    Runtime = "dotnet6",
                    Role = createRoleResponse.Role.Arn,
                    Environment = new Amazon.Lambda.Model.Environment
                    {
                        Variables = new Dictionary<string, string> { { "ENVIRONMENT", "test" } }
                    }
                });
            }
        }
    }
}
```

We have assigned local variables value in `FunctionUnitTest` constructor for simpilycity.

## Run integration test using LocalStack
To run integration test using LocalStack, we need to create a Lambda function with appropriate permission in LocalStack. We can create Lambda function from LocalStack CLI but we want to keep it automated using .NET for every test run, hence We called method `SetupLocalStackLambda` in Arrange section to setup and deploy lambda function in Localstack on every test run.

- Set `_isRunningOnLocal` variable value as `true`.
- Run Integration test from visual studio.

## Run integration test using AWS (Optional)
- [Deploy](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/lambda-creating-project-in-visual-studio.html) LambdaFunction into your AWS account. Make sure you have given function name `SampleLambdaFunction`.
- Make sure you have given appropriate permission to lambda function.
- Make sure deployed lambda function running and tested.
- Set `_isRunningOnLocal` variable value as `false`.
- Run Integration test from visual studio.

## Troubleshooting
1. To verify lambda function response, we need to make sure `result.StatusCode` is `200` and `result.FunctionError` is not `Unhandled`. In most of the cases, lambda function response code is 200 even if it is throwing error.
2. If lambda function response `result.FunctionError` is `Unhandled`, it means lambda function throwing error which you can check through docker desktop or you can use `result.LogResult`.
    
    <img src="/posts/images/localstack-docker-error.jpg" width="100%">

3. `result.LogResult` return message in base64 format, you can convert into text to see the message.

    <img src="/posts/images/lambda-localstack-logresult.jpg" width="100%">
    <img src="/posts/images/lambda-localstack-logresult-convert.jpg" width="100%">

4. If you're running LocalStack using CLI or Docker Compose, you may not get this error. This error only comes with docker when you have not passed all the parameters required to run Lambda in LocalStack. To fix the above error, you can use the docker command to run LocalStack.
    
    `docker run -e "LAMBDA_EXECUTOR=docker" -e "LOCALSTACK_HOSTNAME=127.0.0.1" -e "DOCKER_HOST=unix:///var/run/docker.sock" -e "DEFAULT_REGION=us-east-1" -e "TEST_AWS_ACCOUNT_ID=000000000000" -e "DATA_DIR=/tmp/localstack/data" -v /var/run/docker.sock:/var/run/docker.sock -v ./create-resources.sh:/docker-entrypoint-initaws.d/create-resources.sh --rm -d -p 4566:4566 -p 4510-4559:4510-4559 localstack/localstack`
5. If your integration test runs successfully using LocalStack Or AWS, you can convert `result.LogResult` from base64 to text to see the response from Lambda function.

    <img src="/posts/images/localstack-log-result.jpg" width="100%">

### Conclusion
To run Lambda function in a LocalStack required some additional effort as compare to S3 and SQS which we have seen in our previous blog post. You can check [LocalStack Lambda](https://docs.localstack.cloud/aws/lambda/) for more detail and don't forget to share your thoughts in comments section.

Happy cloud computing.