.. file containing all term definitions relevant for mico documentation

Glossary
========

.. glossary::

    MICO Application
        An application is a set of services.

    MICO Service
        MICO Service is the resource that is managed by the MICO system, is part of the MICO domain model and relates to the term "micro-service". Such a MICO Service originates either from a GitHub repository or from DockerHub.

    Kubernetes Service
        Kubernetes Service "is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service". Pods themselves "serve as unit of deployment, horizontal scaling, and replication". (quoted by Kubernetes documentation)
        The relationship between a MICO Service and a Kubernetes Service is quite confusing because of following association:

            - MICO Service → Kubernetes Deployment
            - MICO Service Interface → Kubernetes Service (MICO Service Interface is a different resource within the mico system, but is part of a MICO Service, actually it is a 1:n releationship)

    Service Layer
        The Service Layer includes *Infrastructure Services* that are used by the domain entities. For example for the use-case of importing MICO services based on GitHub repositories the service `GitHubCrawler` exists. More examples are the `ImageBuilder` for creating images for MICO services, the `MicoStatusService` for retrieving data from Prometheus and `MicoKubernetesClient` as the service to operate with Kubernetes.

    Broker
        Broker is part of the domain layer that extends the domain model with more business logic. A Broker adds operations to the model in ubiquitous language (not CRUD, that would be a Repository), but is completely stateless.

    Service
        An application service retrieve the information required by an entity, effectively setting up the execution environment, and provide it to the entity. For example for the use-case of importing MicoServices based on GitHub repositories the Application Service GitHubCrawler exists.

    External Service
        An external service is a service not managed by the mico system but by a third party, e.g. a weather api from weatherbit which could be used by a MICO Service. External services are currently not supported.

    Service Interface
        A service interface is an API (REST over http, gRPC, etc.) the service provides for other services to use over the network.
        One MICO service can have multiple service interfaces, but at least one.
        A MICO service interface is representet through a Kubernetes Service.

    Kafka
        Kafka is a MOM(Message oriented Middleware) that uses topics instead of traditional message queues.

    Topic
        Messages that are send over Kafka are organized in kafka topics. A service has multiple topics (input, output, dead letter, etc) from where it can consume messages or push new messages.

    OpenFaaS-function
        A OpenFaaS-function is used for manipulating messages based on different messaging patterns using the FaaS(Function as a Service) approach.

    KafkaFaaSConnector
        The Kafka FaaS Connector is used by MICO as a generic message processor component to route messages from Kafka in the CloudEvents format to an OpenFaaS function.
