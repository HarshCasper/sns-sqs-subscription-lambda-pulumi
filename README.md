# LocalStack & Pulumi Sample to showcase SNS, SQS, and Lambda integration

This example demonstrates the delivery from SNS to SQS queues through subscriptions, triggering Lambda functions via SQS event source mappings, and accessing resources from within a Lambda function. The sample is deployed using **LocalStack** on your local machine and on a CI pipeline without accessing any real cloud resources.

The basic pipeline is:

- Publish to SNS Topic, initiated by user input.
- Transfer from SNS Topic to SQS queue, facilitated by an SNS subscription.
- SQS Queue triggers a Lambda invocation, configured through a Lambda event source mapping.
- Lambda function interacts with SQS using the AWS SDK for Python (`boto3`), executed within the Lambda.
- SQS queue message reception, activated by user input.

![Architecture](./images/architecture.png)

## Prerequisites

- [LocalStack & `localstack` CLI](https://docs.localstack.cloud/getting-started/installation/#localstack-cli)
- [Pulumi (`pulumi`) CLI](https://www.pulumi.com/docs/install/)
- Docker
- AWS CLI & [`awslocal` wrapper script](https://docs.localstack.cloud/user-guide/integrations/aws-cli/#localstack-aws-cli-awslocal)

## Instructions

### Start LocalStack

To start LocalStack, execute:

```bash
localstack start -d
```

### Configure Pulumi

You can use a local backend for Pulumi, which will store the state in a local file. Run the following command to configure the local backend:

```bash
pulumi login --local
```

Configure the Pulumi stack `dev` using the following command:

```bash
pulumi stack init dev
```

### Install dependencies

Install the necessary dependencies with:

```bash
yarn install
```

### Deploy the stack

Zip the Lambda function with the following command:

```bash
zip lambda.zip lambda.py
```

For stack preview and deployment, run:

```bash
pulumi up --yes
```

> You might be prompted for `PULUMI_CONFIG_PASSPHRASE` or `PULUMI_CONFIG_PASSPHRASE_FILE`. If you haven't set a passphrase, you can just press `Enter`.

After a successful deployment, you should see the following output:

```bash
Updating (dev):
     Type                              Name                    Status              
 +   pulumi:pulumi:Stack               sns-sqs-dev             created (0.89s)     
 +   ├─ aws:sns:Topic                  trigger-event-topic     created (1s)        
 +   ├─ aws:sqs:Queue                  lambda-event-queue      created (26s)       
 +   ├─ aws:sqs:Queue                  lambda-result-queue     created (25s)       
 +   ├─ aws:lambda:Function            event-echo-lambda       created (29s)       
 +   ├─ aws:sns:TopicSubscription      lambda-event-queue-sub  created (0.05s)     
 +   └─ aws:lambda:EventSourceMapping  sqs-lambda-trigger      created (0.05s)     

Resources:
    + 7 created

Duration: 57s
```

### Check the stack

Post deployment, verify the available SQS queues & SNS topics with the following commands:

```bash
awslocal sqs list-queues
awslocal sns list-topics
```

## Test the stack

You can publish a message to the SNS topic via:

```bash
awslocal sns publish \
	--topic arn:aws:sns:us-east-1:000000000000:trigger-event-topic \
	--message '{"event_type":"testing","event_payload":"hello world"}'
```

You can receive the message from the SQS queue via:

```bash
awslocal sqs receive-message --wait-time-seconds 10 --visibility-timeout=0 \
	--queue=http://sqs.us-east-1.localhost.localstack.cloud:4566/000000000000/lambda-result-queue
```

You will find the `hello world` message in the output.

## License

This code is available under the Apache 2.0 license.
