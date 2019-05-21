# Function to component mapping

* Status: [accepted | superseeded by [ADR-0005](0005-example.md) | deprecated | …] <!-- optional -->
* Deciders: [list everyone involved in the decision] <!-- optional -->
* Date: [YYYY-MM-DD when the decision was last updated] <!-- optional -->

Technical Story: [Issue #713](https://github.com/UST-MICO/mico/issues/713) <!-- optional -->

## Context and Problem Statement

To implement the EAI patterns we use a combination of a generic component
which handles the communication with Kafka and a FaaS solution. The business logic of the EAI patterns (message splitting/aggregation or transformation) is provided via functions which are hosted on the FaaS solution. The generic component communicates with Kafka and de/serializes the messages. We need a means to wire the instances of the generic component with the functions. E.g. A user wants to insert a message splitter between two message-based components. To realize this an instance of the generic component in combination with a splitting FaaS function will be used. The generic component needs the address of the FaaS gateway and the function name (e.g. http://address:8080/function/msg-payload-splitter) to call the function. To provide the necessary information to instances of the generic component we considered the following techniques. 

## Decision Drivers <!-- optional -->

* [driver 1, e.g., a force, facing concern, …]
* [driver 2, e.g., a force, facing concern, …]
* … <!-- numbers of drivers can vary -->

## Considered Options

* [option 1]
* [option 2]
* [option 3]
* … <!-- numbers of options can vary -->

## Decision Outcome

Chosen option: "[option 1]", because [justification. e.g., only option, which meets k.o. criterion decision driver | which resolves force force | … | comes out best (see below)].

### Positive Consequences <!-- optional -->

* [e.g., improvement of quality attribute satisfaction, follow-up decisions required, …]
* …

### Negative consequences <!-- optional -->

* [e.g., compromising quality attribute, follow-up decisions required, …]
* …

## Pros and Cons of the Options <!-- optional -->

### [option 1]

[example | description | pointer to more information | …] <!-- optional -->

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]
* … <!-- numbers of pros and cons can vary -->

### [option 2]

[example | description | pointer to more information | …] <!-- optional -->

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]
* … <!-- numbers of pros and cons can vary -->

### [option 3]

[example | description | pointer to more information | …] <!-- optional -->

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]
* … <!-- numbers of pros and cons can vary -->

## Links <!-- optional -->

* [Link type] [Link to ADR] <!-- example: Refined by [ADR-0005](0005-example.md) -->
* … <!-- numbers of links can vary -->
