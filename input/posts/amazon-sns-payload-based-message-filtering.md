Title: Amazon SNS payload-based message filtering
Description: New payload-based message filtering option of SNS help you to offload additional filtering logic to SNS and reduce your application integration costs.
Published: 28/12/2022
Image: /posts/images/amazon-sns-filtering.png
Tags:
  - aws
  - sns
  - sqs
---
By default, an Amazon SNS topic subscriber receives every message that's published to the topic. To receive only a subset of the messages, a subscriber must assign a filter policy to the topic subscription.

A filter policy is a JSON object containing properties that define which messages the subscriber receives. Amazon SNS supports policies that act on the message attributes or on the message body. Earlier filter policy option was only available on message attribute level not on the message body, Now [Amazon introducing payload-based message filtering for Amazon SNS](https://aws.amazon.com/blogs/compute/introducing-payload-based-message-filtering-for-amazon-sns/)

We will use below example to implement payload-based message filtering in Amazon SNS

<img src="/posts/images/amazon-sns-payload-based-filtering.png">

### Step 1: Create Orders topic

1. Open the [AWS Management Console for SNS](https://console.aws.amazon.com/sns/home) in new table or window.
2. Enter a topic name `Orders`
3. Select **Standard topic** for the type of topic you want to create.
4. Select **Create topic**

<img src="/posts/images/amazon-sns-payload-based-filtering-sns.png">

### Step 2: Create Orders queue

1. Open the [AWS Management Console for SQS](https://console.aws.amazon.com/sqs/home) in new table or window.
2. Enter a queue name `Orders`
3. Select **Standard queue** for the type of queue you want to create.
4. Select **Create queue**

### Step 3: Create Orders-EU queue

1. Open the [AWS Management Console for SQS](https://console.aws.amazon.com/sqs/home) in new table or window.
2. Enter a queue name `Orders-EU`
3. Select **Standard queue** for the type of queue you want to create.
4. Select **Create queue**

### Step 4: Subscribe the Orders queue to the Orders SNS topic

1. Select **Orders** queue and open **SNS subscription** tab.
2. Select **Subscribe to Amazon SNS topic**
3. Choose **Orders** topic from dropdown list, then choose **save**.

<img src="/posts/images/amazon-sns-payload-based-filtering-sqs1.png">

### Step 5: Subscribe the Orders-EU queue to the Orders SNS topic

Repeat step 4 to subscribe the **Orders-EQ** queue to the Orders SNS topic.

<img src="/posts/images/amazon-sns-payload-based-filtering-sqs2.png">

### Step 6: apply payload-based message filtering

1. Open **Orders** topic.
2. Select **Orders-EU** subscription and then choose **Edit**.
3. Expand the **Subscription filter policy** section and enable **Subscription filter policy.**
4. Select **Message body** and paste below JSON in JSON editor.

```json
{
  "location": ["EU"]
}
```

5. Select **Save changes**.

We have configured SNS Topic **Orders**, and two SQS queue subscriptions **Orders** queue and **Orders-EU** queue. Now we will send messages to the **Orders** SNS topic which will redirect message to appropriate queue based on filter criteria. 

### Step 7: Publish test messages to Amazon SNS

1. Sample message for US location

```json
{
	"location": "US",
	"OrderId": "132456-564123-789456-897645",
	"Amount": 105,
	"OrderDate": "2022-12-28"
}
```

2. Sample message for EU location

```json
{
	"location": "EU",
	"OrderId": "789456-897645-132456-564123",
	"Amount": 111,
	"OrderDate": "2022-12-28"
}
```

### Step 8: Verify message delivery

We have not applied any filter in Orders queue, hence we should received both the messages in Orders queue and only 1 message in Orders-EQ.

<img src="/posts/images/amazon-sns-payload-based-filtering-response.png">

### Step 9: Cleanup

1. Delete Orders SNS topic.
2. Delete both queue (Orders and Order-EQ)

### Conclusion
The new payload-based message filtering option in SNS empowers subscribers to express their SNS subscription filter policies in terms of the contents of the message.

Happy cloud computing.