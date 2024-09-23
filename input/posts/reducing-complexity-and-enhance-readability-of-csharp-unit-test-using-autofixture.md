Title: Reducing complexity and enhance readability of C# unit tests using AutoFixture
Description: we are explaining about how to write efficient unit tests with less no. of lines of code.
Published: 20/08/2022
Image: /posts/images/autofixture-unittest.jpg
PrimaryTag: dotnet
Tags:
  - dotnet
  - csharp
  - unittest
  - autofixture
---
Unit tests allows developers to verify their code to make sure that the code works as expected and to identify and fix bugs at early stage. The goal of unit test is to isolate each part of the program and test them individually. When we write Unit Tests for individual code, we mock all external dependencies. The purpose of mocking is to isolate the code being tested without affecting the behavior or state of external dependencies.

If our service has many external dependencies then it adds repeated line of code for each test case which increases complexity and decreases readability. 

Also, It require change in mock objects if any dependency modified otherwise It breaks test execution. For example, if we inject any new dependency in our service through constructor, the existing unit test breaks and we have to add that dependency mock object or pass null value to fix a unit test.

In this post, we are explaining about how to handle above issue and write efficient unit tests with less no. of lines of code.


### Prerequisite
- Basic understanding of Unit Test
- Basic understanding of Moq (Mock) framework

### Method To Test
Here we are taking very basic example, and will optimize this in further steps of unit test versions V1, V2 and V3. So here, we are going to test **Divide** method of **CalculatorService** class:-

```cs
public class CalculatorService : ICalculatorService
{
    private readonly IDataAccessService dataAccessService;
    private readonly ILogger<CalculatorService> logger;
    public CalculatorService(IDataAccessService dataAccessService, ILogger<CalculatorService> logger)
    {
        this.dataAccessService = dataAccessService;
        this.logger = logger;
    }

    public double Divide(double value1, double value2)
    {
        if (value2 == 0)
        {
            this.logger.LogError($"Invalid value : {value2} DivideByZeroException");
                throw new DivideByZeroException("value2 cannot be zero");
        }
        var result = (value1 / value2);
        this.dataAccessService.AddResult("Divide", value1, value2, result);
        this.logger.LogInformation($"Result {result} save sucessfully!");
         return result;
        }
}
```
### Unit Test using XUnit and Moq Framework (V1)
We have taken a simple unit test scenario to verify expected behaviour of Divide method. You can see below most of the code is written in Arrange block. It is about 13 lines of code.

```cs
public class CalculatorServiceTestsV1
{
    [Fact]
    public void Divide_WhenPassedTwoNumber_ShouldReturnExpectedResult()
    {
        // Arrange
        var mockDataAccessService = new Mock<IDataAccessService>();
        var mockLogger = new Mock<ILogger<CalculatorService>>();
        mockLogger.Setup(x => x.Log(
            It.IsAny<LogLevel>(),
            It.IsAny<EventId>(),
            It.IsAny<It.IsAnyType>(),
            It.IsAny<Exception>(),
            (Func<It.IsAnyType, Exception, string>)It.IsAny<object>()));
            var service = new CalculatorService(mockDataAccessService.Object, mockLogger.Object);
            double value1 = 5;
            double value2 = 5;

        // Act
        var result = service.Divide(value1, value2);
        // Assert
        Assert.Equal(1, result);
    }
}
```

### AutoFixture
AutoFixture is an open source library for .NET designed to minimize the Arrange phase of your unit tests in order to maximize maintainability and readability. Its primary goal is to allow developers to focus on what is being tested rather than how to setup the test scenario, by making it easier to create object graphs containing test data.

### Unit Test using XUnit and AutoFixture (v2)
Same unit test is written with the help of  AutoFixture and got rid of most of the arrange block code. The no of lines of code has reduced to 7.

```cs
[Theory, AutoData]
public void Divide_WhenPassedTwoNumber_ShouldReturnExpectedResult(IFixture fixture)
{
    // Arrange
    fixture.Customize(new AutoMoqCustomization());
    var service = fixture.Create<CalculatorService>();
    double value1 = 5;
    double value2 = 5;
    // Act
    var result = service.Divide(value1, value2);
    // Assert
    Assert.Equal(1, result);
}
```

### Unit Test using XUnit and AutoFixture (v3)
Same unit test written with the help of  AutoFixture and AutoData attribute and we completely got rid of all the lines of code from arrange block. The no. of lines of code has reduced to 3.

```cs
[Theory]
[InlineAutoMoqData(5, 5, 25)]
[InlineAutoMoqData(599, 599, 358801)]
public void Multiply_WhenPassedTwoNumber_ShouldReturnExpectedResult(double value1, double value2, double expectedResult, CalculatorService service)
{
    // Arrange
    // Act
    var result = service.Multiply(value1, value2);
    // Assert
    Assert.Equal(expectedResult, result);
}
```

### AutoMockData attribute
Provides auto-generated data specimens generated by AutoFixture with a mocking library as an extension to xUnit.net's Theory attribute.

### InlineAutoMoqData attribute
Provides a data source for a `Theory`, with the data coming from inline values combined with auto-generated data specimens generated by AutoFixture with a mocking library.

### References

[Getting Started with xUnit.net](https://xunit.net/docs/getting-started/netcore/cmdline)

[Moq Repo](https://github.com/moq/moq)

[AutoFixture Repo](https://github.com/AutoFixture/AutoFixture)

[AutoFixture Guide](https://autofixture.github.io/docs/quick-start/#)

### Conclusion
It is a very simple example but think about real project scenario where you might have many dependencies in single class, in that case you can save hundreds of lines of code. Moreover, your unit test will be simple to understand and easy to.

Happy coding.