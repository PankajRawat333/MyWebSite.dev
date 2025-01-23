Title: How to Deserialize DynamoDb stream JSON to Object in .NET Lambda?
Description: We can write an extension method to convert Amazon DynamoDB stream JSON to .NET object.
Published: 13/10/2022
Image: /posts/images/programming.jpg
Tags:
  - aws
  - dynamodb
  - lambda
  - dotnet
---
Whenever an application creates, updates, or deletes items in the table, DynamoDB Streams writes a stream record with the primary key attributes of the items that were modified. You can configure the stream so that the stream records capture additional information, such as the "before" and "after" images of modified items.

You can enable a stream and select type of data you need in stream
- Key attributes only - Only the key attributes of the modified item.
- New image - The entire item, as it appears after it was modified.
- Old image - The entire item, as it appeared before it was modified.
- New and old images - Both the new and the old images of the item.

### Extension Method
We can write an extension method to convert Amazon DynamoDB stream JSON to .NET object.

```cs
public static class DynamoDBExtensions
{
   private static readonly AmazonDynamoDBConfig clientConfig;
   private static readonly DynamoDBContext dynamoDBContext;
   
   static DynamoDBExtensions()
   {
      clientConfig = new AmazonDynamoDBConfig();
      dynamoDBContext = new DynamoDBContext(new AmazonDynamoDBClient(clientConfig));
   }

   public static T Convert<T>(this Dictionary<string, AttributeValue> dynamoDBImage)
   {
      var dynamoDocument = Document.FromAttributeMap(dynamoDBImage);
      return dynamoDBContext.FromDocument<T>(dynamoDocument);
   }
}
```
First it converts `Dictionary<string, AttributeValue>` to an Amazon DynamoDB Document and from that to .NET object.

### Lambda Function
Amazon DynamoDB is integrated with AWS Lambda so that you can create triggers that automatically respond to events in DynamoDB Streams.

In the below example, we have configured New image in DynamoDB stream and Call `Convert<T>` extension method to get .NET object.

```cs
public class Function
{
    public async Task FunctionHandler(DynamoDBEvent dynamoEvent, ILambdaContext context)
    {
        context.Logger.LogInformation($"Beginning to process {dynamoEvent.Records.Count} records...");

        foreach (var record in dynamoEvent.Records)
        {
            var newimage = record.Dynamodb.NewImage;
            Employee employee = newimage.Convert<Employee>();

            context.Logger.LogInformation($"Employee data: {JsonSerializer.Serialize(employee)}");
        }

        context.Logger.LogInformation("Stream processing complete.");
        await Task.CompletedTask;
    }
}
```

### Conclusion
In this blog, we have created extension method to convert Amazon DynamoDB stream JSON to .NET object. Make sure you have same data type in .NET model and Amazon DynamoDB table.

Happy coding