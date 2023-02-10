Title: Handling Partial Batch Failure When Processing SQS Messages with a Lambda Function
Description: how you can handle partial batch failure when processing SQS messages with a Lambda function.
Published: 10/02/2023
Image: /posts/images/localstack.png
PrimaryTag: aws
Tags:
  - aws
  - sqs
  - dotnet
  - lambda
---
Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications. One common use case for SQS is to process messages from a queue as a batch using a Lambda function. However, in some cases, you may encounter partial batch failure, where some messages in the batch are processed successfully, while others fail. In this case, you need to handle the partial batch failure to ensure that the messages that failed to process are retried, while the messages that succeeded are not processed again.

In this blog post, we will explore how you can handle partial batch failure when processing SQS messages with a Lambda function.

There are several strategies you can use to handle partial batch failure when processing SQS messages with a Lambda function:

 
#### Visibility Timeout
One solution to handle partial batch failure when processing SQS messages with a Lambda function is to use the visibility timeout feature of SQS. When you receive a batch of messages from an SQS queue, the messages are hidden from other consumers for a specified period of time, known as the visibility timeout. If a message fails to process, you can extend the visibility timeout of that message so that it remains hidden and can be retried later.

To implement this solution, you can add logic to your Lambda function to detect when a message fails to process, and then extend the visibility timeout of that message. This can be done using the **Amazon SDK for .NET** or any other SDK that supports SQS.

Here is an example of how you can extend the visibility timeout of a failed message in .NET:

```cs
var extendMessageVisibilityRequest = new ExtendMessageVisibilityRequest
{
    QueueUrl = queueUrl,
    ReceiptHandle = receiptHandle,
    VisibilityTimeout = visibilityTimeout
};

await sqsClient.ExtendMessageVisibilityAsync(extendMessageVisibilityRequest);
```

In this example, the `queueUrl` and `receiptHandle` variables contain the URL of the SQS queue and the receipt handle of the message, respectively. The `visibilityTimeout` variable contains the length of time in seconds that the message should be hidden from other consumers.

By using the visibility timeout feature of SQS, you can handle partial batch failure by automatically retrying the processing of failed messages.

### Batch Failure Report

AWS Lambda now supports partial batch response for SQS as an event source. With this feature, when messages on an SQS queue fail to process, Lambda marks a batch of records in a message queue as partially successful and allows reprocessing of only the failed records. By processing information at a record-level instead of batch-level, AWS Lambda has removed the need of repetitive data transfer, increasing throughput and making Amazon SQS message queue processing more efficient. 

Until now, a batch being processed through SQS polling would either be completely successful, in which case the records would be deleted from the SQS queue, or would completely fail, and the records would be kept on the queue to be reprocessed after a ‘visibility timeout’ period. The Partial Batch Response feature an SQS queue will only retain those records which could not be successfully processed, improving processing performance.

In Lambda function SQS trigger configuration, you need to enable **Report batch item failures** option which is optional by default.

<img src="/posts/images/sqs-report-batchitem-failure.JPG">

Here is an example of how you can implement the Batch Failure Report solution in C# for Lambda function:

```cs
public class Function
{
    public async Task<SQSBatchResponse> FunctionHandler(SQSEvent evnt, ILambdaContext context)
    {
        List<SQSBatchResponse.BatchItemFailure> batchItemFailures = new List<SQSBatchResponse.BatchItemFailure>();
        context.Logger.LogInformation($"Message received count {evnt.Records.Count}");

        foreach (var message in evnt.Records)
        {
            try
            {
                await ProcessMessageAsync(message, context);
            }
            catch (Exception ex)
            {
                context.Logger.LogError(ex.Message);
                //await ChangeVisibility(message.ReceiptHandle);
                batchItemFailures.Add(new SQSBatchResponse.BatchItemFailure
                {
                    ItemIdentifier = message.MessageId
                });
            }
        }
        context.Logger.LogInformation($"BatchItemFailure count {batchItemFailures.Count}");
        return new SQSBatchResponse(batchItemFailures);
    }
}
```

The FunctionHandler method processes each message in the batch and adds an entry to the `batchItemFailures` list for each failed message. This lets your function return a partial success, which can help reduce the number of unnecessary retries on records.

With this implementation, you have a flexible for handling partial batch failure when processing SQS messages with a .NET Core-based Lambda function.

### Conclusion
The Partial Batch Response feature an SQS queue will only retain those records which could not be successfully processed, improving processing performance.


Happy cloud computing.