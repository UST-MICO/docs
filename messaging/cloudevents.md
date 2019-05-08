# CloudEvents

[CloudEvents](https://cloudevents.io/) is a specification for describing event data in a common way.
We want to support this vendor-neutral specification.

Currently we depend on the [CloudEvents specification v0.2](https://github.com/cloudevents/spec/blob/v0.2/spec.md).

## Attributes

**Context Attributes (REQUIRED):**

- **type** – cloud event definition
- **specversion** – cloud event definition
- **source** – cloud event definition
- **id** – change id if message was changed significantly (data translation, ignoring addition only changes)
    if id changed add previous message id as **createdFrom** to message (overwriting previous **createdFrom**)

**Context Attributes (OPTIONAL):**

- **time** – cloud event definition
    Change to current time if id changed
- **schemaurl** – cloud event definition
- **contenttype** – cloud event definition, contenttype + schemaurl define the complete message format
- **subject** - wip definition of [CloudEvents spec v0.3](https://github.com/cloudevents/spec/blob/master/spec.md): Identifying the subject of the event in context metadata (opposed to only in the data payload) is particularly helpful in generic subscription filtering scenarios where middleware is unable to interpret the data content. In the above example, the subscriber might only be interested in blobs with names ending with '.jpg' or '.jpeg' and the subject attribute allows for constructing a simple and efficient string-suffix filter for that subset of events.
- **createdfrom** – id of the message this message was directly created from
- **correlationid** – always keep if set, even in answers to this message (can be id of first message in a createdFrom chain)
- **route** – list of topics the message was sent to (updated by sender), keep until destination of message
    example: `["topicA", "topicB", "destinationTopic"]`
- **routingslip** – stack of topics the message should be routed to (last entry is the next destination)
    Maybe allow complex entries in stack to implement recipient lists
- **istestmessage** – boolean flag for test messages
- **filteroutbeforetopic** – only applies to test messages. Message should never be sent to specified topic
- **expirydate** – for messages that should be considered stale after this date
- **sequenceid** – message id for messages split into a sequence of messages
- **sequencenumber** – the position of this message in the sequence as an Integer. The sequence MUST start with a value of `1` increase by `1` for each subsequent value.
- **sequencesize** – the number of messages in the sequence (if known, is not required)
- **returntopic** – the return address for replies to this message (maybe implement this as a stack similar to routing slip)
- **dataref** - useful for claim check pattern. A reference to a location where the event payload is stored.

**Data Attribute:**

- **data** – cloud event definition

## Examples

**Minimal:**

```json
{
    "specversion" : "0.2",
    "type" : "io.github.ust.mico.result",
    "source" : "/router",
    "id" : "A234-1234-1234",
    "time" : "2019-05-08T17:31:00Z",
    "contenttype" : "application/json",
    "data" : {
        "key" : "value"
    }
}
```
