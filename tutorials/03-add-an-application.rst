How To add an application
=========================

First :doc:`add some services <01-add-a-service>` to your MICO instance.

To add a new application click on the :guilabel:`new Application` button in the dashboard or the application overview.

.. figure:: images/add-new-application.*
   :name: add-new-application

Fill in the values of the dialog. For :samp:`Short Name` and :samp:`Version` the same restrictions as for services apply.

.. figure:: images/new-application-dialog.*
   :name: new-application-dialog

You can choose the services for the new application now with the :guilabel:`pick Services` button.

Services still can be added, removed or changed after the creation.

.. figure:: images/service-picker-for-new-application-dialog.*
   :name: service-picker-for-new-application-dialog

Pick the services with their specific version. Only one version of a service may be directly included in an application.
This does not restrict dependencies of included services.

.. figure:: images/new-application-dialog-with-services.*
   :name: new-application-dialog-with-services

The selected services are shown in the create new application dialog.


After adding the application
----------------------------

You will be redirected to the application detail page, if the application was added successfully.

.. figure:: images/application-detail.*
   :name: application-detail

On the application detail page, you can choose the version of the application in the top right corner.

Right below the application name on the left, you can deploy the application with the :guilabel:`deploy` button,
undeploy a the application with the :guilabel:`undeploy` button,
create a new version of this application with the :guilabel:`create next Version` button or
delete this specific application version with the :guilabel:`delete version` button.

On the right side you can find a :guilabel:`Services` button to add services to the application.
The included services are listed below this button together with their deployment information.

To edit the application data switch to the tab :guilabel:`Application Data`.

.. figure:: images/application-detail-data.*
   :name: application-detail-data


