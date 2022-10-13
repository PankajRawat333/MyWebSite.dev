Title: Access Amazon DynamoDB using Token Vending Machine
Description: We will use Amazon DynamoDB using Token Vending Machine to achieve tenant isolation.
Published: 20/03/2022
Image: /posts/images/tenant-isolation.jpg
Tags:
  - aws
  - dynamodb
  - token-vending-machine
  - saas
  - iam
  - dotnet
  - lambda
---

When we are working in multi-tenant SaaS application, we need to ensure tenant isolation is maintained. SaaS application can be built with a variety of different architecture. Regulatory, competitive, strategic, cost efficiency, and market considerations all have some influence on the shape of your SaaS architecture. Primarily there are 3 different patterns to design multi-tenant SaaS applications.

1. Silo model refers to an architecture where each tenant has dedicated resources.
2. Pool model refers to an architecture where all tenants share resources.
3. Bridge model refer to combination of both (Silo & Pool) model where some of the system is implemented in silo model and some is in pool model.

Tenant isolation is a key challenge in pool model where all tenants share the same resources. In this blog, we will use Amazon DynamoDB to store multi-tenant application data and AWS Lambda function to retrieve tenant specific data. Token Vending Machine ensure tenants are not allowed to cross tenant boundaries when accessing resources.

<img src="/posts/images/tenant-isolation-1.png">

You can check AWS blog post Isolating SaaS Tenants with Dynamically Generated IAM Policies to understand Token Vending Machine concept in detail. In this post, we'll focus on the steps required to setup Token Vending Machine for DynamoDB using .NET Lambda function.

### Step 1: Create multi-tenant Amazon DynamoDB table
Create DynamoDB **Employee** table from AWS Console with **Tenant **as partition key and **Id **as sort key.

<img src="/posts/images/tenant-isolation-2.png">

For demo purpose we can create sample records in DynamoDB **Employee **table for multiple tenants.

<img src="/posts/images/tenant-isolation-3.png">

### Step 2. Create AWS Lambda with IAM Role
We'll fetch data from DynamoDB using Lambda function, Create a Lambda Function (.NET 6/ .NET Core 3.1) and IAM role with basic Lambda permissions.

<img src="/posts/images/tenant-isolation-4.png">

### Step 3: Create User for Development (optional)
If you have already configured AWS CLI in your local development machine you can skip this step.
We need AWS IAM user for local development. Create new user from IAM and enable an access key Id and secret access key for the AWS SDK. 

<img src="/posts/images/tenant-isolation-5.png">

### Step 4: Create Dynamic Policy Template
We can create IAM policy for each tenant to restrict cross tenant access but this often creates an explosion of tenant policies, which can push the account limits of IAM. Session policies are advanced policies that you pass as a parameter when you programmatically create a temporary session for a role or federated user. The permissions for a session are the intersection of the identity-based policies for the IAM entity (user or role) used to create the session and the session policies. 
I have created below policy for Amazon DynamoDB Employee table from visual editor. In below policy (line 16) limits tenant to only get rows with a key that begins with a specific tenant identifier value.


<img src="/posts/images/tenant-isolation-6.png">

### Step 5. Create Assume Role

Assuming a role means asking Security Token Service (STS) to provide you with a set of temporary credentials **role credentials** that are specific to the role you want to assume. (Specifically, a new **session** with that role.) 
For our example, the role must contain an inline policy allowing access to an Amazon DynamoDB resource. It allows anyone access to DynamoDB resources without any tenant-specific limitations.
For simplicity, created below role with full permission on DynamoDB. 

<img src="/posts/images/tenant-isolation-7.png">

Who can assume this role? We need to setup trust so specific user (development user) and role (lambda role) can assume this role.

<img src="/posts/images/tenant-isolation-8.png">

### Step 6: Implement Token Vending Machine in .NET Lambda
For demo purpose, I have created a simple lambda function which takes tenant as input parameter. In the real world, you may get tenant information from auth token. For simplicity I have skipped auth part and directly passed tenant in lambda function. 

Then, I have created TokenVendingMachine object which returns temporary credential (session token) for a specific tenant. We can use TokenVendingMachine temporary credential while creating DynamoDB object.
Function.cs file will look like this

<img src="/posts/images/tenant-isolation-9.png">

TokenVendingMachine takes assume role ARN and dynamic policy template to create tenant specific session policy.
TokenVendingMachine.cs file will look like this.

<img src="/posts/images/tenant-isolation-10.png">

### Step 7: Verify Token Vending Machine
Once code deploy on AWS Lambda, we can directly verify lambda function from AWS console Test tab. Token Vending Machine ensures a tenant (Company1) cannot access another tenant's (Company2) data.


<img src="/posts/images/tenant-isolation-11.png">

<img src="/posts/images/tenant-isolation-12.png">

### Conclusion
In this blog, we have setup Token Vending Machine for Amazon DynamoDB but you can use Token Vending Machine in other AWS services also.

Happy cloud computing.