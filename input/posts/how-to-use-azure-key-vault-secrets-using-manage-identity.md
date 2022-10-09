Title: How to use Azure Key Vault Secrets using Manage Identity
Published: 15/01/2019
Lead: In this post, I shared how to use Azure Key Vault Secrets in Azure Function using Manage Idenity
Image: /posts/images/manage-identity.jpg
Tags:
  - azure
  - key-vault
  - secrets
  - identity
---

One of the common concern of every application, either On-premises or Cloud, how to manage application keys and secrets? Keeping credential, secrets and other keys is an important task. Keeping sensitive information in code is not a good practice. Most people keep application keys and secrets in application config file (JSON or XML) and change those value according to environments. I followed the same practice with Azure Function, WebJob and Web App.

### Problem with Azure Function App Setting

Azure function app setting is a plain-text file, User can retrieve keys and secrets through API or Portal. Let's create Azure function with EventHub trigger.

Search **Function App** from Azure marketplace and create Azure function with EventHub trigger.
<img src="/posts/images/manage-identity1.jpg" class="img-fluid centered-img">

Once function app is created, you can see below Azure Function application settings. **EventHubConnectionString** is sensitive information in my function app, Any user has function app access can see and use that connection string anywhere.
<img src="/posts/images/manage-identity2.jpg" class="img-fluid centered-img">
So, Where should I keep my application secrets?

### Key Vault Solution

Key Vault is Azure PaaS service and all-in-one solution to keep your keys, secrets and certificate secure. Key Vault help us to protect sensitive information so only authorized user or authorized application can retrieve those values. In this article, I will only walk you through Key Vault Secrets. Let’s create Azure key Vault to store Azure Function EventHub connection string.
No alt text provided for this image
- Search key vault in Azure marketplace and create with a unique name.
- Optional: Setup Access Policies for application or User (I will setup later in this article).
<img src="/posts/images/manage-identity3.jpg" class="img-fluid centered-img">

Click on Generate/Import button and add EventHub Connection string as secret.
<img src="/posts/images/manage-identity4.jpg" class="img-fluid centered-img">
Once secret is created, we need secret identifier to retrieve EventHub Connection string. Only authorized user or application can retrieve secret from URL. we can control permission (get, set, delete etc.) from Key Vault Access Policies tab. Copy secret identifier URL.
<img src="/posts/images/manage-identity6.jpg" class="img-fluid centered-img">

### Enable a managed identity in Azure Function

Azure Key Vault provides a way to securely store credentials and other secrets, but your code needs to authenticate to Key Vault to retrieve them. Mange identities helps to solve this problem by giving Azure services an automatically managed identity in Azure AD. You can use this identity to authenticate to any service that supports Azure AD authentication, including Key Vault, without having any credentials in your code.
<img src="/posts/images/manage-identity7.jpg" class="img-fluid centered-img">
<img src="/posts/images/manage-identity8.jpg" class="img-fluid centered-img">

### Key Vault Secret Read Access
After enabling mange service identities in function app, assign permission to azure function to read secrets from key vault.
<img src="/posts/images/manage-identity9.jpg" class="img-fluid centered-img">
<img src="/posts/images/manage-identity10.jpg" class="img-fluid centered-img">

Now time to update Azure function applications settings. "EventHubConnectionString" new value something like this @Microsoft.KeyVault(SecretUri={theSecretUri})
<img src="/posts/images/manage-identity11.jpg" class="img-fluid centered-img">

And That’s it 😊. Now, you can see my function app working same as earlier.
<img src="/posts/images/manage-identity12.jpg" class="img-fluid centered-img">
No code changes required in function app to get secrets from Key Vault with manage identity.

### References
[What are managed identities for Azure resources](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)

Happy cloud computing.