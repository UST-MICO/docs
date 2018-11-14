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

Attributes based on :doc:`service_description`.

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


.. _`Semantic version`: https://semver.org

.. todo:: list all attributes


Service Interface
-----------------

.. list-table::
   :header-rows: 1
   :widths: 20 40 40

   * - Attribute
     - Description
     - Example
   * - serviceName*
     - Unique short name suffix for :samp:`shortName` of service.
     - :samp:`rest`

.. todo:: list all attributes



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

.. todo:: list all attributes
