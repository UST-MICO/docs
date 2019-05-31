# Error Report Messages

As described by the EAI Patterns [Invalid Message Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/InvalidMessageChannel.html) and [Dead Letter Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DeadLetterChannel.html)
we need a way to handle messages which can not be processed and or routed. We adopt the concept of the Pattern and use topics to realize them. 
Messages which can not be processed are reported to the Invalid Message Topic and messages which can not be routed are reported to the Dead Letter Topic.
The messages in these topics use a CloudEvent envelope and the following error report message format in the data field of the envelope.

## Attributes

**Used attributes in the CloudEvent envelope:**

- **createdfrom** – id of the message which triggered the error


**Error report Message Attributes:**

- **errorMessage** – human-readable error message. 
- **inputTopic** – the original message which triggered the error was read from this topic
- **outputTopic** – the standard output topic. FaaS functions can overwrite this
- **faasGateway** - the gateway of the configured FaaS function
- **faasFunctionName** - the name of the FaaS function
- **reportingComponentName** – the name of the component which reports the error
