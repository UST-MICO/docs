Postman
=======

We share our Postman collections and environment configurations via Git.

.. role:: bash(code)
    :language: bash

.. role:: javascript(code)
    :language: javascript

Install
-------

`Install Postman <https://www.getpostman.com/downloads/>`_

Update API
----------

Everyone that changes the API of one of the MICO Microservices should also update the Postman collections and export them to this directory.
An easy way to do that is by using the Swagger UI, copy the API docs, and import it into Postman.

Example with mico-core:

* Port forwarding to mico-core to be able to access the REST API:
  :bash:`kubectl port-forward svc/mico-core -n mico-system 8080:8080`
* Open `<http://localhost:8080/v2/api-docs>`_
  (Swagger-UI would be `<http://localhost:8080/swagger-ui.html>`_)
* Copy the complete JSON
* Postman -> Import
* Optional: Change the property :javascript:`"host":"localhost:8080"` to :javascript:`"host":"{{host}}"` so that Postman will use the host definition from the environment configuration
* Export updated collection
