# Simple Composition Components

Technical Story: 
- Depends on: https://github.com/UST-MICO/mico/issues/724
- Related issues: 
  - https://github.com/UST-MICO/mico/issues/717
  - https://github.com/UST-MICO/mico/issues/718

## Context and Problem Statement
A Simple Composition Component is a [Generic Component](https://github.com/UST-MICO/mico/issues/724), that fulfills the requirements of an [Enterprise Integration Pattern](https://www.enterpriseintegrationpatterns.com) by invoking a user-defined (or predefined) openFaaS function and acting upon its results. In contrast to the [Complex Integration Components](https://github.com/UST-MICO/docs/blob/master/adr/0025-implementation-of-complex-eai-patterns-with-faas.md) there are no further efforts (such as maintaining a state) needed for implementing the corresponding Simple Composition Component. 


## Decision Drivers
We want to have a list of all Enterprise Integration Patterns, that can be implemented as Simple Composition Components with a reasonable amount of effort.

## Considered Options
Theoretically, all Enterprise Integration Pattern could be implemented as Simple Composition Components

## Decision Outcome
In the following outcome it is assumed that we have two types of generic components:
- Generic Transformation Component: Provide each message to a user-defined function and forward the result to a predefined target topic
- Generic Routing Component: Provide each message to a user-defined function and forward the result to the topic, that is returned by the user-defined function.
We also assume that the user can not provide external resources (like a database)

### Routing Patterns

| Name                       | Implementation Strategy                                                                                                              | Return Value                                                                |         Possible as Simple Composition Component         |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|:--------------------------------------------------------:|
| Content-Based router       | All rules are implemented in the user-defined function                                                                               | The topic, to which it is routed                                            | Yes                                                      |
| Message Filter             | All rules are implemented in the user-defined function                                                                               | Either nothing, or the topic to which it is routed                          | Yes                                                      |
| Dynamic Router             | All rules are implemented in the user-defined function                                                                               | The topic, to which it is routed                                            | No                                                       |
| Recipient List             | The recipients are either hard coded in the user-defined function, or in a external database, which can be accessed by the function. | All topics, to which the message shall be forwarded                         | No                                                       |
| Splitter                   | The splitting rules are implemented in the user-defined function                                                                     | The sub messages, that contain the splitted content of the original message | Yes                                                      |
| Aggregator                 | The user provides a message storage, in which the messages can be stored until they are aggregated                                   | The aggregated message                                                      | No                                                       |
| Resequencer                | The user provides a message storage, in which the messages can be stored until they are forwarded in the right order                 | The (correct) sequence of messages                                          | No                                                       |
| Composed Message processor | Can be implemented by combining a Splitter, Router and Aggregator                                                                    | The composed message                                                        | No (since the aggregator is not considered to be simple) |
| Scatter-Gather             | Can be implemented by combining an aggregator, multiple transformers and a topic, to which all transformers subscribe                | the transformed and aggregated message                                      | No (since the aggregator is not considered to be simple) |
| Routing Slip               | Not appropriate as Simple Composition Component                                                                                      | -                                                                           | No                                                       |
| Process Manager            | Not appropriate as Simple Composition Component                                                                                      | -                                                                           | No                                                       |
| Message Broker             | Not appropriate as Simple Composition Component                                                                                      | -                                                                           | No                                                       |


### Transformation Patterns

| Name                 | Implementation Strategy                                                                                                                                                                        | Return Value                     | Possible as Simple Composition Component |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------|------------------------------------------|
| Content Enricher     | In most cases: User provides database which contains the information that shall be added to a message. The user defined-function implements the database access and the transformation process | The transformed message          | No                                       |
| Content Filter       | The user-defined function implements the whole process of filtering out content of a message                                                                                                   | The filtered message             | Yes                                      |
| Envelope Wrapper     | A Content Enricher for wrapping a message in an envelope. A Content Filter for unwrapping                                                                                                      | -                                | Yes                                      |
| Claim Check          | Combination of a special Content Filter for replacing message data with a 'claim', and a Content Enricher for recovering the data. Also a database for storing the data until it is recovered  | -                                | No                                       |
| Normalizer           | A combination of a Router, and a set of Translators                                                                                                                                            | The transformed message          | Yes                                      |
| Canonical Data Model | The user-defined function is implemented in such a way, that receives/returns messages with a canoncial data format                                                                            | Message in canonical data format | Yes                                      |

