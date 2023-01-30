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

Rules use **event patterns** to select events and route them to targets. **A pattern either matches an event or it doesn't.** Event patterns are represented as JSON objects with a structure that is similar to that of events. For example, the following event pattern allows you to subscribe to only events from Amazon EC2.

```YAML
1
2
3
{
  "source": [ "aws.ec2" ]
}
```
The pattern simply quotes the fields you want to match and provides the values you are looking for.

The sample event above, like most events, has a nested structure. Suppose you want to process all instance-termination events. Create an event pattern like the following.

```YAML
{
  "source": [ "aws.ec2" ],
  "detail-type": [ "EC2 Instance State-change Notification" ],
  "detail": {
    "state": [ "terminated" ]
  }
}
```
**Important**

- For a pattern to match an event, **the event must contain all the field names listed in the pattern.** The field names must appear in the event with the same nesting structure.
- Other **fields of the event not mentioned in the pattern are ignored**; effectively, there is a "*" : "*" wildcard for fields not mentioned.
- The **matching is exact** (character-by-character), without case-folding or any other string normalization.
- The **values being matched follow JSON rules**: Strings enclosed in quotes, numbers, and the unquoted keywords true, false, and null.
- **Number matching is at the string representation level.** For example, 300, 300.0, and 3.0e2 are not considered equal.


# EventBridge using API Gateway

**GOAL** 

Process all events for source **com.aws.orders** via an **Amazon EventBridge API Destination**. In this use case we are demonstrating how you might integrate EventBridge with third party public API endpoints using an outbound webhook with Basic Auth configured.

![image](https://user-images.githubusercontent.com/91480603/215527518-93d7f9c9-a9da-405e-849f-4cdc69fdad7a.png)

**1. Identify the API URL**
1. Open the CloudFormation console: **https://console.aws.amazon.com/cloudformation**
2. View the **Outputs** tab and **Apiurl**

![image](https://user-images.githubusercontent.com/91480603/214955099-0c833723-8dfa-485c-b64c-38648aa35826.png)

**2. Configure the EventBridge API Destination with basic auth security**

1. Open the EventBridge console **https://console.aws.amazon.com/events**
2. In the navigation pane, choose **Integration** and select **API destinations**
3. Choose **Create API destination**

![image](https://user-images.githubusercontent.com/91480603/214955757-275d4087-4619-498a-8efb-6fa822abb527.png)

4. **Create API destination** detail

- Enter *api-destination* as the **Name**
- Enter the **API URL** identified in Step 1 as the **API destination endpoint**
- Select POST as the **HTTP method**
- Select **Create a new connection** for the **Connection**
- Enter *basic-auth-connection* as the **Connection name**
- Select **Basic (Username/Password)** as the **Authorization type**
- Enter *myUsername* as the **Username**
- Enter *myPassword* as the **Password**

![image](https://user-images.githubusercontent.com/91480603/214957590-56c6066f-256c-42de-b399-1fd803994ba7.png)
![image](https://user-images.githubusercontent.com/91480603/214957791-68c8eb70-3ce5-4dfc-9dfd-e94bdb409f1d.png)
![image](https://user-images.githubusercontent.com/91480603/214957847-d9bedd72-64d3-46d9-9673-31aa329e4740.png)

5. Click **Create**

![image](https://user-images.githubusercontent.com/91480603/214958145-755e1399-0b4e-49c6-bab9-ae09d0e0bfba.png)

**3. Configure an EventBridge rule to target the EventBridge API Destination**

1. In the navigation pane, choose **Buses** and select **Rules**
2. From the Event bus dropdown, select the **Orders** Event bus
3. Click **Create rule**
4. On the **Define rule detail** page

- Enter *OrdersEventsRule* as the **Name** of the rule
- Enter *Send com.aws.orders source events to API Destination* for **Description**
5. Select **Next**
6. Under **Build event pattern**

- Choose **Other** for your **Event source**
- Choose **Enter my own** copy and paste the following:

```YAML
{
    "source": [
        "com.aws.orders"
    ]
}
```
![image](https://user-images.githubusercontent.com/91480603/214961102-3048b477-b5bf-412e-82fc-91b1f31dd7f3.png)

7. Select **Next** to specify your target
8. **Select target(s)** 
- Select **EventBridge API destination** as the target type.
- Select *api-destination* from the list of **existing API destinations**
- All other defaults

![image](https://user-images.githubusercontent.com/91480603/214961634-1b2bc7b9-046f-4779-834a-c9095f88c1b5.png)

9. Click **Skip to Review and create**
10. Click **Create rule**

**4. Send test Orders event**

Use this sample data to test your rule

```YAML
{ "category": "lab-supplies", "value": 415, "location": "us-east" }
```
Using the Event Generator, send the following *Order Notification* events from the source *com.aws.orders:*

![image](https://user-images.githubusercontent.com/91480603/214965375-38f7bcfe-76b6-4638-8d25-c36e4780c037.png)

![image](https://user-images.githubusercontent.com/91480603/214965215-c7f1e45a-2e63-4bd2-95b6-77d2653bf3f6.png)

**5. Verify API Destination**

1. Open the CloudWatch console **https://console.aws.amazon.com/cloudwatch/home?**
2. In the navigation pane, choose **Logs** and **Log groups**
3. Select the Log group with an **API-Gateway-Execution-Logs** prefix

![image](https://user-images.githubusercontent.com/91480603/214965953-423b3926-2f9c-4cea-80a1-ac405ab2e2ab.png)

4. Select the **Log streams**

![image](https://user-images.githubusercontent.com/91480603/214966215-2ee2c474-40cc-4d87-8d87-20bc0ab5b884.png)

5. Toggle the log event to verify the basic authorization was successful and the API responds with a 200 status.

![image](https://user-images.githubusercontent.com/91480603/214966414-4f55b659-0efd-45af-81a5-5ad6df0f3bda.png)
![image](https://user-images.githubusercontent.com/91480603/214966511-3da41737-2630-46bd-aa74-54547148113a.png)


# EventBridge using Step Functions

**GOAL**

Process **only orders from locations in the EU** (eu-west or eu-east) using a **AWS Step Functions** target (**OrderProcessing**). In this use case, we are demonstrating how a Step Function execution can be triggered to process orders as they are published by the Orders bus.

![image](https://user-images.githubusercontent.com/91480603/215527996-883e62d6-5dca-477c-9978-ee000942798a.png)

**1. Implement an EventBridge rule to target Step Functions**

1. Open the EventBridge console **https://console.aws.amazon.com/events**
2. Navigate to **Rules** and select Event Bus **Orders**
3. Click **Create rule** with the Name **EUOrdersRule**
4. **Rule with an event pattern** **Next** 
5. Event source **Other**
6. Define an **Event pattern** to match events with a detail location in **eu-west** or **eu-east**

```YAML
{
  "source": ["com.aws.orders"],
  "detail-type": ["Order Notification"],
  "detail": {
    "category": ["office-supplies"],
    "value": [300],
    "location": ["eu-west"]
  }
}
```
7. Target the **OrderProcessing** Step Functions state machine

8. Click **Skip to Review and create** **Create rule**

**2. Send test EU Orders events**

1. Open Event Generator **https://event-generator.awsworkshops.io/#/**
2. Send the following *Order Notification* events from the *source com.aws.orders*:

```YAML
{ "category": "office-supplies", "value": 300, "location": "eu-west" }
```

**3. Verify Step Functions workflow execution**

If the event sent to the **Orders** event bus matches the pattern in your rule, then the event will be sent to the **OrderProcessing** Step Functions state machine for execution.

1. Open the **Step Functions** console **https://console.aws.amazon.com/states/home**
2. Navigate and select **State machines**
3. Enter *OrderProcessing* in the **Search for state machines** box and verify the state machine execution has succeeded

![image](https://user-images.githubusercontent.com/91480603/215188212-525b91e0-f4c1-46c7-ab5f-d753efe473a5.png)


# EventBridge using SNS

![image](https://user-images.githubusercontent.com/91480603/215528518-26cc749e-2a8b-405b-b5c9-e05ab81c5db2.png)

**1. Implement an EventBridge rule to target SNS**

1. Open the EventBridge console **https://console.aws.amazon.com/events**
2. Click **Create rule** with the Name **USLabSupplyRule**
3. **Rule with an event pattern** **Next** 
5. Event source **Other**
6. Define an **Event pattern** to match events with a detail location in **eu-west** or **eu-east**

```YAML
{
  "source": ["com.aws.orders"],
  "detail-type": ["Order Notification"],
  "detail": {
    "category": ["lab-supplies"],
    "value": [300],
    "location": ["us-east"]
  }
}
```
7. Target the **Orders** SNS topic

![image](https://user-images.githubusercontent.com/91480603/215202787-2ad5c350-64f2-42ab-8990-3570321feb95.png)

8. Click **Skip to Review and create** **Create rule**

**2. Send test US Orders events**

1. Open Event Generator **https://event-generator.awsworkshops.io/#/**
2. Send the following *Order Notification* events from the *source com.aws.orders*:

```YAML
{ "category": "lab-supplies", "value": 415, "location": "us-east" }
```

```YAML
{ "category": "office-supplies", "value": 1050, "location": "us-west", "signature": [ "John Doe" ] }
```

**3. Verify SNS topic**

If the event sent to the Orders event bus matches the pattern in your rule, then the event will be sent to the Orders SQS Queue (via Orders SNS Topic.

1. Open the **SQS** console **https://console.aws.amazon.com/sqs/v2/home**
2. Navigate and select **Orders** queue
3. Select the **Send and receive messages** button

![image](https://user-images.githubusercontent.com/91480603/215204475-56e9ee24-b964-44e0-9752-6543f07251ab.png)

5. Select **Poll for Messages** and verify the first message was delivered and the second was not

![image](https://user-images.githubusercontent.com/91480603/215204607-0bb104f2-cd01-41eb-902a-3e9f7aebbe21.png)

**First message confirmed**

![image](https://user-images.githubusercontent.com/91480603/215205063-1e0b2a91-c60e-4c23-830b-1ea7d5ce127e.png)

