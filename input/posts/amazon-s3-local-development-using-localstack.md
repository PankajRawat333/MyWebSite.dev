Title: Amazon S3 local development using LocalStack
Published: 27/09/2022
Image: /posts/images/localstack.png
Tags:
  - aws
  - s3
  - dotnet
  - localstack
  - docker
---

It's always a good practice, before moving your code into cloud, do integration test from local dev environment. Let's say if you don't have AWS account access from your development machine Or you don't want to create resources in AWS account just for integration testing purpose in that situation LocalStack is best choice.

### What is LocalStack?

[LocalStack](https://localstack.cloud/)  is a cloud service emulator that runs in a single container on your laptop or in your CI 
environment. With LocalStack, you can run your AWS applications or Lambdas entirely 
on your local machine without connecting to a remote cloud provider!

LocalStack supports a growing number of AWS services, like AWS S3, Lambda, DynamoDB, Kinesis, SQS, SNS, and many more! The Pro version of LocalStack supports additional APIs and advanced features.

### How to setup LocalStack?

There are many different way to setup LocalStack such as CLI, Docker, cockpit and helm. In this post, I’ll use docker the easiest way to start and manage LocalStack.

#### Prerequisites

Make sure that you have a working [docker](https://docs.docker.com/get-docker/) environment and Python3 on your machine before moving on.

**Starting LocalStack with CLI**

`pip install awscli-local`

**Starting LocalStack with Docker**

`docker run --rm -d -p 4566:4566 -p 4510-4559:4510-4559 localstack/localstack`

**Create LocalStack S3 bucket using CLI**

`awslocal s3api create-bucket --bucket sample-bucket`

`awslocal s3api list-buckets`

<img src="/posts/images/localstack-command.png" width="100%">


**Create LocalStack S3 bucket using .NET 6**

To interact with Amazon S3 using .NET, you need to create AmazonS3Client object from `AWSSDK.S3` Nuget package.

```cs
var s3Client = new AmazonS3Client();
```

To interact with LocalStack you need specify `ServiceURL` while creating `AmazonS3Client` object.

```cs
var s3Client = new AmazonS3Client(
new AmazonS3Config
            {
                ServiceURL = "http://localhost:4566",
                ForcePathStyle = true,
            });
```

Here is the complete sample code.

```cs
public async Task S3IntegrationTest(string env)
{
    IAmazonS3 s3Client;
    if (env == "local")
    {
        s3Client = new AmazonS3Client(new AmazonS3Config
        {
            ServiceURL = "http://localhost:4566",
            ForcePathStyle = true,
        });
    }
    else
    {
        s3Client = new AmazonS3Client();
    }
    string s3bucketName = $"test-bucket-{Guid.NewGuid()}";
    //1. Create S3 bucket
    await s3Client.PutBucketAsync(s3bucketName);
    //2. Put Object into S3 bucket
    string keyName = Guid.NewGuid().ToString();
    await s3Client.PutObjectAsync(new Amazon.S3.Model.PutObjectRequest
    {
        BucketName = s3bucketName,
        Key = keyName,
        ContentType = "text/plain",
        ContentBody = "Hello, Test message from visual studio"
    });
    //3. Delete Object from S3bucket
    await s3Client.DeleteObjectAsync(s3bucketName, keyName);
    //4. Delete S3 bucket
    await s3Client.DeleteBucketAsync(s3bucketName);
}
```

### Conclusion

The code remain same either you are interacting with Amazon S3 or LocalStack S3. For AWS local development, also to run integration test LocalStack is really useful.

Happy cloud computing.