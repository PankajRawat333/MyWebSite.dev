Title: Building a Serverless Conversational AI App with Amazon Q Business

Description: Learn how to build a conversational generative AI application using Amazon Q Business without writing any code, integrating seamlessly with your private data.
Published: 24/06/2024
Image: /posts/images/amazon-q-business-gen-ai-app/amazon-q-app.jpeg
Tags:
  - aws
  - genai
  - amazon-q
  - amazon-q-business
  - textract
  - bedrock
  - lambda
  - s3
---

[Amazon Q Business](https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/what-is.html) is a conversational assistant powered by generative artificial intelligence (AI) that enhances workforce productivity by answering questions based on your data. It streamlines tasks, accelerates problem-solving, and allows you to create and share task automation applications or perform routine actions like submitting time-off requests and sending meeting invites.

In this blog post, you’ll discover how to develop a conversational assistant using Amazon Q Business to access your private data—no coding required. This method can be adapted for enterprises with diverse data sources, such as SharePoint documents, application logs in S3 buckets, or Google Drive files.

[Connectors](https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/connectors-list.html) simplify synchronizing data from multiple repositories with your Amazon Q index. They can be scheduled to automatically sync, ensuring your searches always reflect the latest content securely.

For this experiment, I used personal documents (e.g., Aadhaar, PAN, Driving License, Voter ID, Bank Passbook) uploaded as scanned images to an Amazon S3 bucket. Extracting text from images can be challenging without the right tools. Initially, I used [Amazon Textract](https://aws.amazon.com/textract/), an Optical Character Recognition (OCR) service, for its robust data extraction capabilities. You can explore the source code for this approach on [ServerlessLand](https://serverlessland.com/patterns/textract-lambda-cdk-dotnet).

However, challenges arose with diverse document types (labeled, unlabeled, and multilingual). While training Amazon Textract improved results, it required significant time and sample documents, which wasn’t practical for an experimental app. Here’s why I moved away from Textract:

- My documents were too varied to standardize.
- Custom model training required hard-to-obtain sample documents.
- Training time was too long for an experimental setup.

Instead, I turned to [Amazon Bedrock](https://aws.amazon.com/bedrock/), a managed service offering foundational AI models. I used **Anthropic’s Claude 3 Sonnet** model, which includes vision capabilities, to extract text from scanned documents. The source code for this solution is available on [ServerlessLand](https://serverlessland.com/patterns/bedrock-lambda-cdk-dotnet).

With the text extracted, I built a conversational assistant atop my personal data using Amazon Q Business.

## Solution Overview

The diagram below outlines the high-level architecture of this conversational assistant built with Amazon Q Business.

![High-level architecture diagram of Amazon Q Business conversational AI application](/posts/images/amazon-q-business-gen-ai-app/solution-overview.png)

Users interact with the Amazon Q Business application through a web browser via the Amazon Q Web Experience, secured by AWS IAM Identity Center. Scanned documents are uploaded to an S3 bucket, processed by an AWS Lambda function using Amazon Bedrock for text extraction, and stored in another S3 bucket as the data source for Amazon Q Business.

**Key Components:**

1. **Amazon S3 Bucket**: Stores extracted text as the data source for Amazon Q Business.
2. **AWS Lambda Function**: Extracts text from scanned documents using Amazon Bedrock.
3. **Anthropic’s Claude 3 on Amazon Bedrock**: Processes images within the Lambda function.
4. **Amazon Q Business Application**: Powers the conversational assistant and web experience.
5. **AWS IAM Identity Center**: Secures the Amazon Q Web Experience.

## Configure an Amazon Q Business Application

Follow these steps to set up your application:

1. **Create the Application**:
   - Log in to your AWS account and open Amazon Q Business.
   - Click **Create application**.
   - Enter an **Application name**.
   - Keep the default **Service Access** option: *“Amazon Q Business requires permissions to use other services on your behalf”* (adjust if needed).
   - Click **Create**.

   ![Screenshot of Amazon Q Business application creation first step](/posts/images/amazon-q-business-gen-ai-app/create-app-step1.png)

2. **Select Retriever**:
   - Choose **Use native retriever** (default) unless you need a pre-existing [Amazon Kendra](https://aws.amazon.com/kendra/) index or storage for over 20,000 documents.
   - Retain default **Index provisioning** settings.
   - Click **Next**.

   ![Screenshot of retriever selection in Amazon Q Business setup](/posts/images/amazon-q-business-gen-ai-app/create-app-step2.png)

3. **Connect Data Sources**:
   - Select **Amazon S3** as the data source.

   ![Screenshot of data source selection showing Amazon S3 option](/posts/images/amazon-q-business-gen-ai-app/create-app-step4.png)

   - Enter a **Data source name**.
   - For **IAM role**, select **Create a new service role (Recommended)**.

   ![Screenshot of data source name and IAM role configuration](/posts/images/amazon-q-business-gen-ai-app/create-app-step3.png)

   - Browse and select your S3 bucket. If your data is in a folder (e.g., `output/`), set it as the include pattern.
   - Set **Sync mode** to **Full sync** and **Sync run schedule** to **Run on demand** (adjust frequency as needed).

   ![Screenshot of S3 bucket selection interface for data source](/posts/images/amazon-q-business-gen-ai-app/choose-s3-bucket.png)

   - Click **Add data source**, then **Next**.

4. **Add Groups and Users**:
   - Go to the **Users** tab and click **Add or assign users and groups**.
   - Select **Add new users**, enter details, and click **Next**.

   ![Screenshot of user addition interface for Amazon Q Business](/posts/images/amazon-q-business-gen-ai-app/add-user.png)

   - Assign the user and click **Create application**.

5. **Accept Invitation**:
   - Check your email for an invitation, accept it, and set up your account.

   ![Screenshot of email invitation acceptance for Amazon Q Business](/posts/images/amazon-q-business-gen-ai-app/accept-invitation.png)

6. **Finalize Application**:
   - For **Web experience service access**, keep the default **Create and use a new service role**.
   - Click **Create application**.

7. **Sync Data Source**:
   - Select your application, go to the **Data source** section, choose **s3-datasource**, and click **Sync now**.

   ![Screenshot of data source selection interface in Amazon Q Business](/posts/images/amazon-q-business-gen-ai-app/select-data-source.png)

   - Wait for the sync to complete.

   ![Screenshot of data source synchronization process in Amazon Q Business](/posts/images/amazon-q-business-gen-ai-app/sync-data-source.png)

8. **Start Conversation**:
   - Return to the application, select **Web experience**, and log in with your credentials.
   - Begin asking questions about your documents.

   ![Screenshot of conversation interface with Amazon Q Business chatbot](/posts/images/amazon-q-business-gen-ai-app/conversation-chat.png)

   - Amazon Q Business includes **source** information with each response for verification.

   ![Screenshot showing source attribution in Amazon Q Business conversation responses](/posts/images/amazon-q-business-gen-ai-app/conversation-chat-source.png)

## Conclusion

Amazon Q Business lets you build a private conversational AI assistant without coding. Its connectors integrate effortlessly with your data sources, allowing natural language queries. I chose Amazon Bedrock over Textract for text extraction due to my experimental needs, but Amazon Textract excels at extracting text, handwriting, and layout elements from scanned documents in most cases.

Happy cloud computing!