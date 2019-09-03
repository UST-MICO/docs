============
Domain Model
============

MicoApplication
===============
Represents an application represented as a set of instances of `MicoService`_

*Required fields*

    * shortName
        A brief name for the application intended for use as a unique identifier.

    * name
        The name of the artifact. Intended for humans.

    * version
        The version of this application.

    * description
        Human readable description of this application.

*Optional fields*

    * services
        The list of included MicoServices

    * serviceDeploymentInfos
        The list of service deployment information this application uses for the deployment of the required services. Null values are skipped.

    * contact
        Human readable contact information for support purposes.

    * owner
        Human readable information for the application owner who is responsible for this application.

MicoApplicationJobStatus
========================
Represents the job status for a `MicoApplication`_. Contains a list of jobs.

*Fields*

    * applicationShortName
        The short name of the application.

    * applicationVersion
        The version of the application.

    * status
        The aggregated status of jobs for the `MicoApplication`_.

    * jobs
        The list of jobs for the `MicoApplication`_.

MicoEnvironmentVariable
=======================
An environment variable represented as a simple key-value pair. Necessary since Neo4j does not allow to persist properties of composite types.

*Fields*

    * name
        Name of the environment variable.

    * value
        Value of the environment variable.

MicoServiceBackgroundJob
========================
Background job for a `MicoService`_. Instances of this class are persisted in the Redis database.

*Fields*

    * future
        The actual job future.

    * serviceShortName
        The short name of the corresponding `MicoService`_.

    * serviceVersion
        The version of the corresponding `MicoService`_.

    * type
        The type of this job.

    * status
        The current status of this Job.

    * errorMessage
        An error message in case the job has failed.

KubernetesDeploymentInfo
========================
Information about the Kubernetes resources that are created through an actual deployment of a `MicoService`_.

*Optional fields*

    * namespace
        The namespace in which the Kubernetes deployment is created.

    * deploymentName
        The name of the Kubernetes deployment created by a `MicoService`_.

    * serviceNames
        The names of the Kubernetes services created by `MicoServiceInterface`_.

MicoLabel
=========
Represents a simple key-value pair label. Necessary since Neo4j does not allow to persist Map implementations.

*Fields*

    * key
        Key of the label.

    * value
        Value of the label.

MicoMessage
===========
A simple message associated with a `Type`_. Note that this class is only used for business logic purposes and instances are not persisted.

*Required fields*

    * content
        The actual message content.

    * type
        The `Type`_ of this message.

*Methods*

    * info(String content)
        Creates a new `MicoMessage`_ instance with the type Type#INFO and the given message content.

    * error(String content)
        Creates a new `MicoMessage`_ instance with the type Type#ERROR and the given message content.

    * warning(String content)
        Creates a new `MicoMessage`_ instance with the type Type#WARNING and the given message content.

Type
----
Enumeration for all types of a `MicoMessage`_.

* INFO
* WARNING
* ERROR


MicoService
===========
Represents a service in the context of MICO.

*Required fields*

    * shortName
        A brief name for the service. In conjunction with the version it must be unique. Pattern is the same as the one for Kubernetes Service names.

    * name
        The name of the artifact. Intended for humans. Required only for the usage in the UI.

    * version
        The version of this service. E.g. the GitHub release tag.

    * description
        Human readable description of this service. Is allowed to be empty (default). Null values are skipped.

    * serviceCrawlingOrigin
        Indicates where this service originates from, e.g., GitHub (downloaded and built by MICO) or DockerHub (ready-to-use image). Null is ignored.

*Optional fields*

    * kafkaEnabled
        Indicates whether this service wants to communicate with Kafka. If so this service is handled differently (e.g. it's not mandatory to have interfaces).

    * serviceInterfaces
        The list of interfaces this service provides. Is read only. Use special API for updating.

    * dependencies
        The list of services that this service requires in order to run normally. Is read only. Use special API for updating.

    * contact
        Human readable contact information for support purposes.

    * owner
        Human readable information for the service owner who is responsible for this service.

    * dockerfilePath
        The relative (to vcsRoot) path to the Dockerfile.

    * dockerImageUri
        The fully qualified URI to the image on DockerHub. Either set after the image has been built by MICO (if the service originates from GitHub) or set by the user directly.

MicoServiceDependency
=====================
Represents a dependency of a `MicoService`_.

*Required fields*

    * service
        This is the `MicoService`_ that requires (depends on) the depended service.

    * dependedService
        This is the `MicoService`_ depended by this service.

MicoServiceDeploymentInfo
=========================
Represents the information necessary for deploying a single service.

*Required fields*

    * service
        The `MicoService`_ this deployment refers to.

    * instanceId
        The instanceId of this deployment. It is used to be able to have multiple independent deployments of the same MICO service. Especially for KafkaFaasConnectors this is a must have.


*Optional fields*

    * replicas
        Number of desired instances. Default is 1.

    * labels
        Those labels are key-value pairs that are attached to the deployment of this service. Intended to be used to specify identifying attributes that are meaningful and relevant to users, but do not directly imply semantics to the core system. Labels can be used to organize and to select subsets of objects. Labels can be attached to objects at creation time and subsequently added and modified at any time. Each key must be unique for a given object.

    * enviromentVariables
        Enviroment variables as key-value pairs that are attached to the deployment of this `MicoService`_. These enviroment values can be used by the deployed Micoservice during runtime. This could be useful to pass information to the MicoService that is not known during design time or is likely to change,

    * interfaceConnections
        Interface connections includes all required information to be able to connect a `MicoService`_ with `MicoServiceInterface`_ of other MicoServices. The backend uses the information to set enviroment variables so that e.g. the frontend knows how to connect to the backend

    * topics
        The list of topics that are used in the deployment of this MicoService

    * imagePullPolicy
        Indicates whether and when to pull the image. Default is Always.

    * kubernetesDeploymentInfo
        Information about the actual Kubernetes resource created by a deploy. Contains details about the used Kubernetes and Services.


MicoServiceInterface
====================
 Represents a interface, e.g., REST API, of a `MicoService`_.

*Required fields*

    * serviceInterfaceName
        The name of this `MicoServiceInterface`_. Pattern is the same than for Kubernetes Service names.

    * ports
        The list of ports. Must not be empty.

*Optional fields*

    * description
        Human readable description of this service interface, e.g., the functionality provided.

    * protocol
        The protocol of this interface (e.g. HTTP).

MicoServicePort
===============
Represents a basic port with a port number and port type (protocol).

*Required fields*

    * port
        The port number of the externally exposed port.

    * type
        The type (protocol) of the port. Default port type is MicoPortType.TCP.

    * targetPort
        The port inside the container.

MicoPortType
============
Enumeration for all port types, e.g., TCP, supported by MICO.

* TCP
    Transmission Control Protocol.

* UDP
    User Datagram Protocol.

MicoServiceCrawlingOrigin
=========================
Enumeration for the various places a service may originate from.

* GITHUB
    Indicates that a service originates from some GitHub respository.

* DOCKER
    Indicates that a service originates from Docker.

* NOT_DEFINED
    Undefined.

MicoVersion
===========
Wrapper for a version that adds the functionality for a version prefix, so that versions like, e.g., 'v1.2.3' are possible.

* prefix
    String prefix of this version, e.g., 'v'.

* version
    The actual semantic version.

* valueOf(String version)
    Creates a new instance of MicoVersion as a result of parsing the specified version string.
    Prefixes are possible as everything before the first digit in the given version string is treated as a prefix to the actual semantic version.
    Note that the prefix can only consist of letters.

* forIntegers(int major, int minor, int patch)
    Creates a new instance of MicoVersion for the specified version numbers.

* forIntegersWithPrefix(String prefix, int major, int minor, int patch)
    Creates a new instance of MicoVersion for the specified version numbers with the specified prefix string.

MicoInterfaceConnection
==============================
An interface connection contains the the information needed to connect a `MicoService`_ to an `MicoServiceInterface`_ of another `MicoService`_. Instances of this class are persisted as nodes in the Neo4j database.

**Required fields**

    * environmentVariableName
        Name of the environment variable that is used to set the fully qualified name of an interface.

    * micoServiceInterfaceName
        Name of the `MicoServiceInterface`_ of an `MicoService`_.

    * micoServiceShortName
        Name of the `MicoService`_.


MicoTopic
=========
A Topic represented a kafka-topic. Instances of this class are persisted as nodes in the neo4j database.

**Required Fields**

    * name
        Name of the topic

MicoTopicRole
=============
Represents a role of a `MicoTopic`_. An instance of this class is persisted as a relationship between a MicoServiceDeploymentInfo and a MicoTopic node in the neo4j database.

**Required Fields**

    * serviceDeploymentInfos
        This is the MicoServiceDeploymentInfo that includes the MicoTopicRole

    * topic
        This is the MicoTopic included by the MicoTopicRole#serviceDeploymentInfos

    * role
        This is the role of the MicoTopicRole

Role
----
Enumeration for all topic roles

* INPUT
* OUTPUT
* DEAD_LETTER
* INVALID_MESSAGE
* TEST_MESSAGE_OUTPUT
