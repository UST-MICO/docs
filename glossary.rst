.. file containing all term definitions relevant for mico documentation

Glossary
========

.. glossary::

    service
        A service is in the context of MICO.

    external service
        An external service is a :term:`service` not managed by the mico system but by a third party. Such a service originates either from a GitHub repository or from DockerHub.

    service interface
        A service interface is an API (REST over http, gRPC, etc.) the service provides for other services to use over the network.
        One service can have multiple service interfaces.

    application
        An application is a set of services.

    admin user
        At the moment there is no difference between a user and an admin user.
