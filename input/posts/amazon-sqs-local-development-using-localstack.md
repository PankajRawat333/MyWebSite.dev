Title: Amazon SQS local development using .NET
Published: 27/09/2022
Image: /posts/images/localstack.png
Tags:
  - aws
  - sqs
  - dotnet
  - localstack
  - docker
---

This is a continuation of my previous blog [Amazon S3 local development using .NET](). If you are new to LocalStack, I highly recommend you to check previous post where I have covered basic of LocalStack.

#### Prerequisites
- Make sure that you have a working [docker](https://docs.docker.com/get-docker/) environment on your machine before moving on.
- Make sure LocalStack is running on port 4566.

**Create LocalStack SQS using CLI**

`awslocal sqs create-queue --queue-name sample-queue`

`awslocal sqs list-queues`

`awslocal sqs send-message --queue-url http://localhost:4566/00000000000/sample-queue --message-body test-message`

<img src="/posts/images/sqs-localstack.JPG" width="100%">


**Create LocalStack SQS using .NET 6**

To interact with Amazon SQS using .NET, you need to create AmazonS3Client object from `AWSSDK.SQS` Nuget package.

```cs
var sqsClient = new AmazonSQSClient();
```

To interact with LocalStack you need specify `ServiceURL` while creating `AmazonSQSClient` object.

```cs
var sqsClient = new AmazonSQSClient(
new AmazonSQSConfig
            {
                ServiceURL = "http://localhost:4566"
            });
```

Here is the complete sample code.

```cs
public async Task SqsIntegrationTest(string env)
{
    IAmazonSQS sqsClient;
    if (env == "local")
    {
        sqsClient = new AmazonSQSClient(new AmazonSQSConfig
        {
            ServiceURL = "http://localhost:4566"
        });
    }
    else
    {
        sqsClient = new AmazonSQSClient();
    }
    string queueName = $"test-queue1";
    string queueUrl = $"http://localhost:4566/000000000000/{queueName}";
    //1. Create SQS queue
    await sqsClient.CreateQueueAsync(queueName);
    //2. Put message into SQS queue
    await sqsClient.SendMessageAsync(queueUrl, "SQS test message");
    //3. Get message from SQS queue
    var messages = await sqsClient.GetAttributesAsync(queueUrl);
    //4. Delete SQS queue
    await sqsClient.DeleteQueueAsync(queueUrl);
}
```

### Conclusion

The code remain same either you are interacting with Amazon SQS or LocalStack SQS. For AWS local development, also to run integration test LocalStack is really useful.

Happy cloud computing.