How To add a service
====================

To add a new service click on the :guilabel:`new Service` button in the dashboard or the service overview.

.. figure:: images/add-new-service.*
   :name: add-new-service


Manual service creation
-----------------------

To create a service manually fill in all the fields in the dialog:

.. figure:: images/new-service-dialog-manual.*
   :name: new-service-dialog-github-manual

.. note::

    :samp:`Short Name` must be all lowercase contain only (latin) letters numbers and `-` and must start with a letter.

    :samp:`Version` must consist of three numbers formatted like in https://semver.org/spec/v2.0.0.html



Importing a service from GitHub
-------------------------------

Open the :guilabel:`GitHub` tab in the dialog and enter the url to the github repository.

.. hint::

    The `UST-MICO Organization <https://github.com/UST-MICO>`_ on GitHub has two example repositories that you can import directly.

    *  `spring-boot-realworld-example-app <https://github.com/UST-MICO/spring-boot-realworld-example-app>`_
    *  `react-redux-realworld-example-app <https://github.com/UST-MICO/react-redux-realworld-example-app>`_


.. figure:: images/new-service-dialog-github-step1.*
   :name: new-service-dialog-github-step1


In the next step you can choose to import the `latest` version or a specific version.

.. warning:: The import from GitHub feature only works if the repository has realeases that match the `semantic versioning <https://semver.org/spec/v2.0.0.html>`_ schema!


.. figure:: images/new-service-dialog-github-step2.*
   :name: new-service-dialog-github-step2

You can also choose a specific release to import.

.. figure:: images/new-service-dialog-github-step2-versionselect.*
   :name: new-service-dialog-github-step2-versionselect



After adding the service
------------------------

You will be redirected to the service detail page if the service was added successfully.

.. figure:: images/service-detail.*
   :name: service-detail

In the top right corner you can select the version of the service you want to view.

Right under the service name on the left you can delete this specific service version with the :guilabel:`delete version` button.

On the right side you can find a button to :guilabel:`Edit` the service and Buttons to add :term:`service interfaces <service interface>` :guilabel:`Provides` and to add dependencies :guilabel:`Dependees` to this service.

.. figure:: images/service-detail-edit.*
   :name: service-detail-edit

