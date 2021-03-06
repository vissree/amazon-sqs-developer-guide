# Tutorial: Creating an Amazon SQS queue with Server\-Side Encryption \(SSE\)<a name="sqs-create-queue-sse"></a>

You can enable SSE for a queue to protect its data\. For more information about using SSE, see [Encryption at Rest](sqs-server-side-encryption.md)\.

**Important**  
All requests to queues with SSE enabled must use HTTPS and [Signature Version 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)\.

In this tutorial you learn how to create an Amazon SQS queue with SSE enabled\. Although the example uses a FIFO queue, SSE works with both standard and FIFO queues\.

**Topics**
+ [AWS Management Console](#sqs-create-queue-sse-console)
+ [AWS SDK for Java](#sqs-create-queue-sse-java)

## AWS Management Console<a name="sqs-create-queue-sse-console"></a>

1. Sign in to the [Amazon SQS console](https://console.aws.amazon.com/sqs/)\.

1. Choose **Create New Queue\.**

1. On the **Create New Queue** page, ensure that you're in the correct region and then type the **Queue Name**\.
**Note**  
The name of a FIFO queue must end with the `.fifo` suffix\.

1. **Standard** is selected by default\. Choose **FIFO**\.

1. Choose **Configure Queue**, and then choose **Use SSE**\.

1. Specify the customer master key \(CMK\) ID\. For more information, see [Key terms](sqs-server-side-encryption.md#sqs-sse-key-terms)\. 

   For each CMK type, the **Description**, **Account**, and **Key ARN** of the CMK are displayed\.
**Important**  
If you aren't the owner of the CMK, or if you log in with an account that doesn't have the `kms:ListAliases` and `kms:DescribeKey` permissions, you won't be able to view information about the CMK on the Amazon SQS console\.  
Ask the owner of the CMK to grant you these permissions\. For more information, see the [AWS KMS API Permissions: Actions and Resources Reference](https://docs.aws.amazon.com/kms/latest/developerguide/kms-api-permissions-reference.html) in the *AWS Key Management Service Developer Guide*\.
   + The AWS managed CMK for Amazon SQS is selected by default\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-tutorials-server-side-encryption-default-service-cmk.png)
**Note**  
Keep the following in mind:  
If you don't specify a custom CMK, Amazon SQS uses the AWS managed CMK for Amazon SQS\. For instructions on creating custom CMKs, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\.
The first time you use the AWS Management Console to specify the AWS managed CMK for Amazon SQS for a queue, AWS KMS creates the AWS managed CMK for Amazon SQS\.
Alternatively, the first time you use the `SendMessage` or `SendMessageBatch` action on a queue with SSE enabled, AWS KMS creates the AWS managed CMK for Amazon SQS\.
   + To use a custom CMK from your AWS account, select it from the list\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-tutorials-server-side-encryption-custom-cmk.png)
**Note**  
For instructions on creating custom CMKs, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\.
   + To use a custom CMK ARN from your AWS account or from another AWS account, select **Enter an existing CMK ARN** from the list and type or copy the CMK\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-tutorials-server-side-encryption-custom-cmk-arn.png)

1. \(Optional\) For **Data key reuse period**, specify a value between 1 minute and 24 hours\. The default is 5 minutes\. For more information, see [Understanding the data key reuse period](sqs-key-management.md#sqs-how-does-the-data-key-reuse-period-work)\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-tutorials-server-side-encryption-data-key-reuse-period.png)

1. Choose **Create Queue**\.

   Your new queue is created with SSE\. The encryption status, alias of the CMK, **Description**, **Account**, **Key ARN**, and the **Data Key Reuse Period** are displayed on the **Encryption** tab\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-tutorials-server-side-encryption-details.png)

## AWS SDK for Java<a name="sqs-create-queue-sse-java"></a>

 The following example uses the AWS Java SDK\. To install and set up the SDK, see [Set up the AWS SDK for Java](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-install.html) in the *AWS SDK for Java Developer Guide*\.

Before you run the example code, configure your AWS credentials\. For more information, see [Set up AWS Credentials and Region for Development](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html) in the *AWS SDK for Java Developer Guide*\. 

You must ensure that the AWS KMS key policies allow encryption of queues and encryption and decryption of messages\. For more information, see [Configuring AWS KMS permissions](sqs-key-management.md#sqs-what-permissions-for-sse)

**To create a queue with SSE**

1. Obtain the customer master key \(CMK\) ID\. For more information, see [Key terms](sqs-server-side-encryption.md#sqs-sse-key-terms)\. 
**Note**  
Keep the following in mind:  
If you don't specify a custom CMK, Amazon SQS uses the AWS managed CMK for Amazon SQS\. For instructions on creating custom CMKs, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\.
The first time you use the AWS Management Console to specify the AWS managed CMK for Amazon SQS for a queue, AWS KMS creates the AWS managed CMK for Amazon SQS\.
Alternatively, the first time you use the `SendMessage` or `SendMessageBatch` action on a queue with SSE enabled, AWS KMS creates the AWS managed CMK for Amazon SQS\.

1. To enable server\-side encryption, specify the CMK ID by setting the `KmsMasterKeyId` attribute of the `[CreateQueue](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_CreateQueue.html)` or `[SetQueueAttributes](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SetQueueAttributes.html)` action\.

   The following code example creates a new queue with SSE using the AWS managed CMK for Amazon SQS:

   ```
   final AmazonSQS sqs = AmazonSQSClientBuilder.defaultClient();
   final Map<String, String> attributes = new HashMap<String, String>();
   final CreateQueueRequest createRequest = new CreateQueueRequest("MyQueue").withAttributes(attributes);
    
   // Enable server-side encryption by specifying the alias for the
   // AWS managed CMK for Amazon SQS.
   final String kmsMasterKeyAlias = "alias/aws/sqs";
   attributes.put("KmsMasterKeyId", kmsMasterKeyAlias);
    
   // (Optional) Specify the length of time, in seconds, for which Amazon SQS can reuse 
   attributes.put("KmsDataKeyReusePeriodSeconds", "60");
   
   final CreateQueueResult createResult = client.createQueue(createRequest);
   ```

   The following code example creates a new queue with SSE using a custom CMK:

   ```
   final AmazonSQS sqs = AmazonSQSClientBuilder.defaultClient();
   final Map<String, String> attributes = new HashMap<String, String>();
   final CreateQueueRequest createRequest = new CreateQueueRequest("MyQueue").withAttributes(attributes);
    
   // Enable server-side encryption by specifying the alias ARN of the custom CMK.
   final String kmsMasterKeyAlias = "alias/MyAlias";
   attributes.put("KmsMasterKeyId", kmsMasterKeyAlias);
    
   // (Optional) Specify the length of time, in seconds, for which Amazon SQS can reuse 
   // a data key to encrypt or decrypt messages before calling AWS KMS again.
   attributes.put("KmsDataKeyReusePeriodSeconds", "86400");
    
   final CreateQueueResult createResult = client.createQueue(createRequest);
   ```

1. \(Optional\) Specify the length of time, in seconds, for which Amazon SQS can [reuse a data key](sqs-server-side-encryption.md#sqs-sse-key-terms) to encrypt or decrypt messages before calling AWS KMS again\. Set the `KmsDataKeyReusePeriodSeconds` attribute of the `[CreateQueue](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_CreateQueue.html)` or `[SetQueueAttributes](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SetQueueAttributes.html)` action\. Possible values may be between 60 seconds \(1 minute\) and 86,400 seconds \(24 hours\)\. If you don't specify a value, the default value of 300 seconds \(5 minutes\) is used\.

   The first code example above sets the data key reuse time period to 60 seconds \(1 minute\)\. The second code example sets it to 86,400 seconds \(24 hours\)\. The following code example sets the data key reuse period to 60 seconds \(1 minute\):

   ```
   // (Optional) Specify the length of time, in seconds, for which Amazon SQS can reuse 
   // a data key to encrypt or decrypt messages before calling AWS KMS again.
   attributes.put("KmsDataKeyReusePeriodSeconds", "60");
   ```

For information about retrieving the attributes of a queue, see [Examples](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_GetQueueAttributes.html#API_GetQueueAttributes_Examples) in the *Amazon Simple Queue Service API Reference*\.

To retrieve the CMK ID or the data key reuse period for a particular queue, use the `KmsMasterKeyId` and `KmsDataKeyReusePeriodSeconds` attributes of the `[GetQueueAttributes](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_GetQueueAttributes.html)` action\.

For information about switching a queue to a different CMK with the same alias, see [Updating an Alias](https://docs.aws.amazon.com/kms/latest/developerguide/programming-aliases.html#update-alias) in the *AWS Key Management Service Developer Guide*\.