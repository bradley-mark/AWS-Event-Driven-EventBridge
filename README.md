# AWS-Event-Driven-EventBridge

**High Level View**



Amazon EventBridge is a serverless event bus service that makes it easy to connect your applications with data from a variety of sources. EventBridge delivers a stream of real-time data from your own applications, Software-as-a-Service (SaaS) applications, and AWS services and routes that data to targets such as AWS Lambda. You can set up routing rules to determine where to send your data to build application architectures that react in real time to all of your data sources.

EventBridge simplifies the process of building event-driven architectures. With EventBridge, your event targets don’t need to be aware of event sources, and vice versa, leading to loosly coupled architectures. EventBridge lets you filter the events before they are sent ot the targets, making sure that each target receives only the events they are interested in. Event-driven architectures are loosely coupled and distributed, which improves developer agility as well as application resiliency.

**Concepts**

**Events** - An event indicates a change in an environment. This can be an AWS environment, an SaaS partner service or application, or one of your own custom applications or services.

**Rules** - A rule matches incoming events and routes them to targets for processing. A single rule can route to multiple targets, all of which are processed in parallel. Rules aren't processed in a particular order. This enables different parts of an organization to look for and process the events that are of interest to them. A rule can customize the JSON sent to the target, by passing only certain parts or by overwriting it with a constant.

**Targets** - A target processes events. Targets can include Amazon EC2 instances, Lambda functions, Kinesis streams, Amazon ECS tasks, Step Functions state machines, Amazon SNS topics, Amazon SQS queues, and built-in targets. A target receives events in JSON format.

**Event buses** - An event bus receives events. When you create a rule, you associate it with a specific event bus, and the rule is matched only to events received by that event bus. Your account has one default event bus, which receives events from AWS services. You can create custom event buses to receive events from your custom applications. You can also create partner event buses to receive events from SaaS partner applications.
