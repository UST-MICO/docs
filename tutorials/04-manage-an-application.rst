How To manage an application
============================

First :doc:`add some services <01-add-a-service>` and an :doc:`application <03-add-an-application>` to your mico instance.

.. note::

   For this tutorial the following assumptions are made:

   The services :samp:`UST-MICO/react-redux-realworld-example-app` and :samp:`UST-MICO/spring-boot-realworld-example-app`
   have been :ref:`imported from github <importing-a-service-from-github>`.

   The service :samp:`UST-MICO/react-redux-realworld-example-app` has an interface
   :samp:`frontend` with :samp:`Exposed Port Number` 80, :samp:`Target Port Number` 80 and :samp:`type` :samp:`TCP`.

   The service :samp:`UST-MICO/spring-boot-realworld-example-app` has an interface
   :samp:`backend` with :samp:`Exposed Port Number` 80, :samp:`Target Port Number` 8080 and :samp:`type` :samp:`TCP`.

   The Application :samp:`New Application` includes both services.


To add the services to the application, use the :guilabel:`Services` button on the right.


.. figure:: images/application-detail.*
   :name: application-detail


Editing deployment information of services
------------------------------------------

Some services need environment variables at runtime to work correctly.
This is also true for the :samp:`UST-MICO/react-redux-realworld-example-app` service.

The service needs the environment variable :envvar:`BACKEND_REST_API` to `find the backend at runtime <https://github.com/UST-MICO/react-redux-realworld-example-app#docker-build>`_.

The deployment info for a service contains these environment variables and other deployment specific information, like the number of replicas, to start.
It is displayed right below the service in the box on the right.

To edit the deployment information use the gear icon.

In the dialog you can edit the following settings:

Replicas
    The number of replicas to scale the service.
MicoLabelRequestDTO
    Key value pairs that get mapped to kubernetes labels.
MicoEnvironmentVariableRequestDTO
    Environment variables to set for the container.
MicoInterfaceConnectionRequestDTO
    Connections to service interfaces of other services. The address of the service interface gets stored in the environment variable.
Image Pull Policy
    When to reload image from dockerhub.

To connect the frontend with the backend service we need to add an interface connection with the following values to the frontend service.

*  **Environment Variable Name:** :envvar:`BACKEND_REST_API`
*  **Interface Name:** :samp:`backend`
*  **Service Short Name:** :samp:`spring-boot-realworld-example-app`

.. warning::

    The actual values depend on the included services of the application!

    The assumptions for this tutorial are listed in the note at the top of this tutorial.

.. figure:: images/edit-frontend-deployment-info-dialog.*
   :name: edit-frontend-deployment-info-dialog


Deploying an Application
------------------------

To deploy an application, use the :guilabel:`deploy` button below the application name.
The actual deployment can take a few minutes depending on the services included in the application.

Upon completion of the deployment you can see a list of ips under :guilabel:`Public IPs`.
Each service has its own ip.



Undeploying an Application
--------------------------

To undeploy a deployed application, use the :guilabel:`undeploy` button below the application name.




Create a new Application Version
--------------------------------

To create a new application version, use the :guilabel:`create next Version` Button and choose the next version to create.

.. figure:: images/create-new-application-version-dialog.*
   :name: create-new-application-version-dialog

The new version of the application will have all services and deployment information of the old version.

