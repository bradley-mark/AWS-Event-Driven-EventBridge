# AWS-Event-Driven-EventBridge

**High Level Design**

![image](https://user-images.githubusercontent.com/91480603/214858136-0a52e3d8-a00c-4e4b-bc9e-64208371f309.png)

Amazon EventBridge is a serverless event bus service that makes it easy to connect your applications with data from a variety of sources. EventBridge delivers a stream of real-time data from your own applications, Software-as-a-Service (SaaS) applications, and AWS services and routes that data to targets such as AWS Lambda. You can set up routing rules to determine where to send your data to build application architectures that react in real time to all of your data sources.

EventBridge simplifies the process of building event-driven architectures. With EventBridge, your event targets don’t need to be aware of event sources, and vice versa, leading to loosly coupled architectures. EventBridge lets you filter the events before they are sent ot the targets, making sure that each target receives only the events they are interested in. Event-driven architectures are loosely coupled and distributed, which improves developer agility as well as application resiliency.

**Concepts**

- **Events** - An event indicates a change in an environment. This can be an AWS environment, an SaaS partner service or application, or one of your own custom applications or services.

- **Rules** - A rule matches incoming events and routes them to targets for processing. A single rule can route to multiple targets, all of which are processed in parallel. Rules aren't processed in a particular order. This enables different parts of an organization to look for and process the events that are of interest to them. A rule can customize the JSON sent to the target, by passing only certain parts or by overwriting it with a constant.

- **Targets** - A target processes events. Targets can include Amazon EC2 instances, Lambda functions, Kinesis streams, Amazon ECS tasks, Step Functions state machines, Amazon SNS topics, Amazon SQS queues, and built-in targets. A target receives events in JSON format.

- **Event buses** - An event bus receives events. When you create a rule, you associate it with a specific event bus, and the rule is matched only to events received by that event bus. Your account has one default event bus, which receives events from AWS services. You can create custom event buses to receive events from your custom applications. You can also create partner event buses to receive events from SaaS partner applications.

# Setup

![image](https://user-images.githubusercontent.com/91480603/214859142-d3a4c8b5-fd63-4e6b-92e4-7797e5534e16.png)

We will create a custom EventBridge event bus, **Orders**, and an EventBridge rule, **OrderDevRule**, which matches all events sent to the Orders event bus and sends the events to a CloudWatch Logs log group, **/aws/events/orders**.

**Event bus and targets**

**Create a custom event bus**

1. Open the EventBridge console **https://console.aws.amazon.com/events**
2. In the navigation pane, choose **Events** and select **Event buses**
3. Choose **Create event bus**
4. 
![image](https://user-images.githubusercontent.com/91480603/214860597-bda27217-af94-4af8-be91-f09febde28e2.png)

5. Name the event bus *Orders*
6. Leave **Event archive** and **Schema discovery** disabled, **Resource-based policy** blank
7. Click **Create**

![image](https://user-images.githubusercontent.com/91480603/214861568-19efda9f-e83a-406c-91e2-cc705b3fe50e.png)

![image](https://user-images.githubusercontent.com/91480603/214861901-8387ae67-ddb7-4f17-8bcd-bf2555518d84.png)

**Setup CloudWatch target (for dev work)**

A simple way to test the rules you create for your event bus is to use Amazon CloudWatch as a target. We will create a rule for the Orders bus that will act as a "catch-all" for every event passed to the bus, irrespective of source.
  
1. In the navigation pane, choose **Rules**
2. From the **Select event bus** dropdown, select **Orders**

![image](https://user-images.githubusercontent.com/91480603/214862921-4090d088-ef7d-4599-887a-8da034b2abf0.png)

3. Click **Create rule**
4. **Define rule detail**

Add *OrdersDevRule* as the **Name** of the rule

Add *Catchall rule for development purposes* for **Description**

Select *Rule with an event pattern* for the **Rule type**

![image](https://user-images.githubusercontent.com/91480603/214864217-1c8c48c5-49c6-4574-aace-657f8f33e7ac.png)

5. Select **Next**
6. Under **Build event pattern** **Event source** choose **Other**

![image](https://user-images.githubusercontent.com/91480603/214864589-a1dd2e67-c911-4df8-b7fc-9591bdf24ebe.png)

7. Under **Event pattern** - **Insert** to catch all events from com.aws.orders:

```YAML
{
   "source": ["com.aws.orders"]
}
```

8. Select **NEXT**
9. **Select target(s)** Target type **AWS service** Select a target dropdown, select **CloudWatch log group**
10. Log Group: name **/aws/events/*orders***

![image](https://user-images.githubusercontent.com/91480603/214866717-59c56e02-b7d9-494c-acb8-948ef3a77bc6.png)

11. Click **Skip to Review and create**
12. Click **Create rule**

**Configure the Event Generator**

Use an AWS Cognito account with access to send test events to your Amazon SNS topics or Amazon EventBridge event buses

**Note:** This requires an AWS Account ID for Cognito

**Building event-driven architectures on AWS > Getting started > Self Hosted** Launch the AWS CloudFormation template
Template URL **https://aws-event-driven-architecture-workshop-assets.s3.amazonaws.com/master-v2.yaml**
Stack Description: **Master stack: AWS Event-driven Architectures Workshop**

![image](https://user-images.githubusercontent.com/91480603/214880680-03d837c9-79eb-47ac-b0f8-59a146d970c0.png)

Once the stack creation has been completed navigate to the stack Outputs tab. Here you will find all the values necessary to configure the Event Generator to work with your account.

1. **EventGeneratorConfigurationUrl** - this url will pre-populate the Event Generator with all the settings you need.
2. **CognitoPassword** - your password for logging into the Event Generator
3. **CognitoUsername** - your user name (defaults to "user")

![image](https://user-images.githubusercontent.com/91480603/214882436-f7658d7d-4963-4d59-b35f-4ba3d6dbf9fd.png)

![image](https://user-images.githubusercontent.com/91480603/214882526-7ad29886-6abb-4f9c-834b-f27485b9b97e.png)

4. Right-click the url in the **EventGeneratorConfigurationUrl** output, opening a new browser window
5. This will open the Event Generator website and pre-populate the modal dialog box with values for a Cognito User Pool and Cognito Identity Pool provisioned in your AWS account. Click **Configure Cognito User Pool** to view the sign-in page
6. On the Sign In page add your the **CognitoUsername** and **CognitoPassword** from the CloudFormation Stack Output page
7. Click **Configure**

![image](https://user-images.githubusercontent.com/91480603/214889273-5a9fac09-5ef0-49a6-9fd6-e175394f75b0.png)

AWS Account ID is prepopulated

![image](https://user-images.githubusercontent.com/91480603/214889535-bfb8d8a9-cfa5-4dd1-96f4-883462034cdb.png)

**Test the rule**

1. Open Event Generator **https://event-generator.awsworkshops.io/#/**
2. Check and enter following:
- **AWS Region** should be to the region in which you are running the workshop
- **Event Bus** - *Orders*
- **Source** - *com.aws.orders*
- **Detail** - *Order Notification*
- **Detail Template**:

```YAML
{
   "category": "lab-supplies",
   "value": 415,
   "location": "eu-west"
}
```
3. Click **PUBLISH EVENT**

![image](https://user-images.githubusercontent.com/91480603/214892130-8f85b5ee-99b5-43c3-8e23-cee9e2f46c34.png)

4. Open the console for CloudWatch **https://console.aws.amazon.com/cloudwatch/home**

5. Choose **Log groups** in the left navigation and select the **/aws/events/orders** log group

![image](https://user-images.githubusercontent.com/91480603/214894969-6e66b8ed-390c-4ea9-abaa-82faf6e79378.png)

6. Select the **Log stream**

![image](https://user-images.githubusercontent.com/91480603/214895482-2f3d5249-41a0-4b38-82d2-d279ec7e9f69.png)

7. Toggle the log event to verify that you received the event

![image](https://user-images.githubusercontent.com/91480603/214895792-7cf1bec9-c834-40a0-a489-6cc6458792bf.png)

**Review event structure**

Sample event for reference

```YAML
{
    "version": "0",
    "id": "c04cc8c1-283c-425e-8cf6-878bbc67a628",
    "detail-type": "Order Notification",
    "source": "com.aws.orders",
    "account": "111111111111",
    "time": "2020-02-20T23:10:29Z",
    "region": "us-west-2",
    "resources": [],
    "detail": {
        "category": "lab-supplies",
        "value": 415,
        "location": "eu-west"
    }
}
```

# Working with EventBridge rules

![image](https://user-images.githubusercontent.com/91480603/214896547-582bc171-3d9d-4ad6-8122-3de6f4ce2534.png)

Rules match incoming events and routes them to targets for processing. A single rule can route to multiple targets, all of which are processed in parallel. Rules aren't processed in a particular order. A rule can customize the JSON sent to the target, by passing only certain parts or by overwriting it with a constant.

Next steps to create an Orders event bus rule to match an event with a com.aws.orders source and to send the event to an Amazon API Gateway endpoint, invoke a AWS Step Function, and send events to an Amazon Simple Notification Service (Amazon SNS) topic.

**Rule matching basics**

Events in Amazon EventBridge are represented as JSON objects and have the following envelope signature:

```YAML

2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
{
  "version": "0",
  "id": "6a7e8feb-b491-4cf7-a9f1-bf3703467718",
  "detail-type": "EC2 Instance State-change Notification",
  "source": "aws.ec2",
  "account": "111111111111",
  "time": "2017-12-22T18:43:48Z",
  "region": "us-west-1",
  "resources": [
    "arn:aws:ec2:us-west-1:123456789012:instance/ i-1234567890abcdef0"
  ],
  "detail": {
    "instance-id": " i-1234567890abcdef0",
    "state": "terminated"
  }
}
```




