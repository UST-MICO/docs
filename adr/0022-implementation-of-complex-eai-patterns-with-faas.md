# Implementation of complex EAI-Patterns with FaaS


## Context and Problem Statement

Some [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com) have a complex structure where parts of the behaviour can be implemented generically while some parts need to be modifiable by the end user (in our case the system admin using mico).
We have already decided to use a FaaS plattform to provide this modifiability in form of code as configuration.
While this works well for most patterns, for some of the more complex patterns it is not easy to allow modifiability via FaaS.
This is especially so because the user would want to write as little code as possible meaning the generic part of the component has to be implemented by the mico team.

<!-- TODO reference other adrs -->


## Decision Drivers

* Modifiability of the patterns must be provided via a FaaS function
* The function should only have to contain as little code as possible
* Generic code for the pattern should be provided by the mico plattform either as a library to import into the FaaS function or in the component that calls said function


## Challenges

### Where/how to implement generic logic

* Implement all logic in the FaaS function
* Implement custom logic + some of the generic logic in the FaaS function
* Implement only custom logic in the FaaS function

### State of configuration channel

How can the FaaS function get the current state of the configuration (e.g. dynamic router).

* Let the FaaS function subscribe to kafka
* Have a seperate db
* Send the current configuration together with the message

### Sending to multiple destinations/unknown destinations

A router does not have a specific endpoint to send all  messages to and may even send messages to many endpoints.

* Implement generic routing with recipientList/routingSlip only and let FaaS function write the route into the message header
* Let the function send the messages directly to kafka

### Stateful components

Some patterns are inherently stateful.

* Store state in a db
* Store state in generic component and send to FaaS function with message

## Affected patterns

| Pattern             | dynamic config | destinations | stateful |
|:--------------------|:--------------:|:------------:|:--------:|
| Router              | no             | yes          | maybe    |
| Dynamic Router      | yes            | yes          | maybe    |
| Conent based Router | maybe          | yes          | naybe    |

<!-- TODO add more affected patterns -->

## Decision Outcome

TODO
