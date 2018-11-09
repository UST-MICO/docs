.. file containing all attributes needed to describe a service

Service Description
===================

Attributes describing a :term:`service`. The attributes are mainly taken from `Pivio`_.

.. _Pivio: http://pivio.io/docs/#_general

Mandatory Fields
----------------

id
   Unique ID. Needs to be generated automatically.

name
    The name of the artefact. This is intended for humans.

short_name
    A very brief name for the service.

description
    What does this service do? Should be a human readable description.

Additional Fields
-----------------

vcsroot
    Where can I find the source code?

dockerfile
    Location of the Dockerfile for this service.

contact
    Who should be contacted if one has a question.

tags
    One possible tag is the distinction between service and application.

lifecycle
    In which lifecycle is this component? Only in development, in production or out of service.

links
    e.g.: Homepage, Buildchain, API docs, ...

type
    The type of this artefact. Values could be service, library or mobile_app.

owner
    Which team is responsible for this artefact.

service
    provides
        What and where does this artefact provide services?
        This part describes a :term:`service interface`.

        description
            See above.

        service_name
            Unique identification of the particular interface.

        port
            TCP/UDP Port

        protocol
            SOAP, gRPC, GRaphQL, SQL, CYPHER, etc.

        transport_protocol
            http, mqtt, etc.

        public_dns
            .. todo:: Define function of this field: service vs. application

    depends_on
        To which other service_name (from provides) services does this service talk?

        service_name
            Unique identification of the particular interface.

        port
            TCP/UDP Port

        protocol
            SOAP, gRPC, GRaphQL, SQL, CYPHER, etc.

        transport_protocol
            http, mqtt, etc.

        public_dns
            .. todo:: Define function of this field: service vs. application
