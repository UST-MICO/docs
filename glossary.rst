.. file containing all term definitions relevant for mico documentation

Glossary
========

.. glossary::

    service
        A service is described by a Pivio description. A service can consist of other services.

    external service
        An external service is a :term:`service` not managed by the mico system but by a third party.

    service interface
        A service interface is an API (REST over http, gRPC, etc.) the service provides for other services to use over the network.
        One service can have multiple service interfaces.

    application
        An application is a set of services. An application can be used as a service in another application.

    admin user
        At the moment there is no difference between a user and an admin user.
