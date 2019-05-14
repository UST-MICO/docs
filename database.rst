Database Schema
===============


Overview
--------

Initial graph based on :doc:`adr/0007-database-technology`.

.. graphviz::

    digraph G {
        concentrate=true;

        {
            service [label=<<U>Service</U>>]

            serviceInterface [label=<<U>Service Interface</U>>]

            application [label=<<U>Application</U>, <U>Service</U>>]

            deployInfo [label="Deploy Information", shape=plaintext]
            vizData [label="Visualization Data", shape=plaintext]
        }

        service -> service [label="depends on; predecessor", constraint=false]

        service -> serviceInterface [label=has, headlabel="1..n", taillabel=1]

        application -> {deployInfo, vizData} [dir=none, constraint=false]

        application -> service [label=includes, headlabel="1..n", taillabel=1]

    }


Service
-------

Attributes describing a :term:`MICO Service` are mainly taken from `Pivio`_.

.. _Pivio: http://pivio.io/docs/#_general

.. list-table::
   :header-rows: 1
   :widths: 20 40 40

   * - Attribute
     - Description
     - Example
   * - ID*
     - UUID (Unique for every version)
     - :samp:`7e7e1828-e845-11e8-9f32-f2801f1b9fd1`
   * - shortName*
     - Unique short name (java Package Style)
     - :samp:`de.uni-stuttgart.mico.hello-world`
   * - name*
     - Display name
     - Hello World
   * - description*
     - Human readable description of the service
     - A simple Hello World Service.
   * - version*
     - `Semantic version`_ version number, commit hash or revision
     - :samp:`1.0.2`, :samp:`1.2.0a`, :samp:`d99ada433bb1d69571...`
   * - vcsroot
     - Url to vcs root (e.g. GitHub link with branch)
     - :samp:`https://github.com/UST-MICO/mico`
   * - dockerfile
     - Path to dockerfile relative to vcsroot
     - :samp:`mico-core/Dockerfile`
   * - contact
     - Human readable contact information.
     - :samp:`Max Musterman, musterman@muster.de`
   * - tags
     - To be defined
     -
   * - links
     - To be defined
     -
   * - owner
     - Human readable team information.
     - :samp:`Max Musterman, Company Y`
   * - externalService
     - A boolean which indicates if a service is external
     - :samp:`true`

.. _`Semantic version`: https://semver.org

.. todo:: Define Tags, Check if lifecycle and links is needed

DependsOn
---------

DependsOn is an edge between two services. A service depends on another service. Additionally, it provides the attribute version. This attribute allows to implement
an auto-update feature. It contains a version string in the `semantic version <https://semver.org>`_ format.
This allows to determine the most recent compatible service.

Service Interface
-----------------

.. list-table::
   :header-rows: 1
   :widths: 20 40 40

   * - Attribute
     - Description
     - Example
   * - serviceInterfaceName*
     - Unique short name suffix for :samp:`shortName` of service.
     - :samp:`rest`
   * - description*
     - Human readable description
     - :samp:`This is a service interface`
   * - port*
     - This is a port number or a variable name
     - :samp:`1024`
   * - protocol*
     - The name of the protocol which is supported by this interface
     - :samp:`gRPC`, :samp:`SQL`, :samp:`CYPHER`
   * - transport_protocol*
     - This protocol is used to transport the previous protocol
     - :samp:`http`, :samp:`mqtt`
   * - public_dns*
     - To be defined
     -


.. todo:: Define Tag public_dns



Application
-----------

Because an application is a service itself it also includes all attributes of a service even when not explicitly stated in the table.

.. list-table::
   :header-rows: 1
   :widths: 20 40 40

   * - Attribute
     - Description
     - Example
   * - ID*
     - UUID (Unique for every version)
     - :samp:`7e7e1828-e845-11e8-9f32-f2801f1b9fd1`

.. todo:: Add necessary attributes for deployments, list all attributes
