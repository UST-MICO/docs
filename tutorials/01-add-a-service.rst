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

    :samp:`Short Name` must be all lowercase, contain only (latin) letters, numbers and the character :samp:`-`. Also the fist character must be a letter.

    :samp:`Version` must consist of three numbers formatted like in https://semver.org/spec/v2.0.0.html (semantic versioning).


.. _importing-a-service-from-github:

Importing a service from GitHub
-------------------------------

Open the :guilabel:`GitHub` tab in the dialog and enter the url to the github repository.

.. hint::

    The `UST-MICO Organization <https://github.com/UST-MICO>`_ on GitHub has two example repositories that you can import directly.

    *  `spring-boot-realworld-example-app <https://github.com/UST-MICO/spring-boot-realworld-example-app>`_
    *  `react-redux-realworld-example-app <https://github.com/UST-MICO/react-redux-realworld-example-app>`_


.. figure:: images/new-service-dialog-github-step1.*
   :name: new-service-dialog-github-step1


In the next step you can choose to import the `latest` or a specific version.

.. warning:: 

    The import from GitHub feature only works if the repository has releases that match 
    the `semantic versioning <https://semver.org/spec/v2.0.0.html>`_ schema! It is also necessary to 
    include a dockerfile in the root folder of the repository.


.. figure:: images/new-service-dialog-github-step2.*
   :name: new-service-dialog-github-step2

You can also choose a specific release to import.

.. figure:: images/new-service-dialog-github-step2-versionselect.*
   :name: new-service-dialog-github-step2-versionselect



After adding the service
------------------------

You will be redirected to the service detail page, if the service was added successfully.

.. TODO update picture

.. figure:: images/service-detail.*
   :name: service-detail

In the top right corner you can select the services version which is displayed.

Right below the service name on the left, you can delete this specific service version with the :guilabel:`delete version` button.
Further, you can create the next service version, if the currently selected version is the latest.
Thereby, the next version number (next major, next minor, next patch) can be selected.
The new version will be a copy of the previous latest version with an updated version number.

.. TODO insert picture of promote service dialog

On the right side you can find a button to :guilabel:`Edit` the service and Buttons to add :term:`service interfaces <service interface>` :guilabel:`Provides` and to add dependencies :guilabel:`Dependees` to this service.

.. figure:: images/service-detail-edit.*
   :name: service-detail-edit

