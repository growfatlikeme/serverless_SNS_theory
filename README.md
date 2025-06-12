# Serverless SNS theory

## Does AWS SNS guarantee exactly once delivery to subscribers?

No guarantee, instead it follows an "at-least-once" delivery model, meaning messages can occasionally be delivered more than once (duplicates) which could happen due to retries (network issue, subscriber failures etc)

## What is the purpose of the Dead-letter Queue (DLQ), which is a feature available to SQS/SNS/EventBridge?

DLQ is a secondary queue where messages are sent after failing to be processed successfully multiple times in the main queue.

It primary helps in isolating problematic message, prevent queue clogging from repeated retries, and also enable debugging by storing these failed messages for later analysis.

## How to enable notification to your email when messages are added to the DLQ?

A few methods would allow us to achieve that. One of the recommended way is to create SNS topic, subcribe your email to the topic, configure SQS DLQ (see below) to publish to SNS when message arrive.

From console:
Go to SQS → Select your DLQ → Configure Queue → Dead-Letter Queue tab).
Enable "Redrive Allow Policy" (if not set).
Under "Notifications", link the SNS topic that was configured.
Make sure that queue policies has been set up to allow SQS to publish messages to SNS topic.

```json
{
  "Version": "2012-10-17",
  "Id": "AllowSQSToPublishToSNS",
  "Statement": [
    {
      "Sid": "AllowSQSToSendToSNSTopic",
      "Effect": "Allow",
      "Principal": {
        "Service": "sqs.amazonaws.com"
      },
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:REGION:ACCOUNT_ID:YOUR_SNS_TOPIC_NAME",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "arn:aws:sqs:REGION:ACCOUNT_ID:YOUR_DLQ_NAME"
        }
      }
    }
  ]
}
```
