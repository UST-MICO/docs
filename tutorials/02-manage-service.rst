How To manage a service
=======================


.. figure:: images/service-detail.*
   :name: service-detail-manage


Manage service interfaces
-------------------------

The :term:`service interfaces <service interface>` are used to map the open ports of the service docker container to ports of the kubernetes service.

.. todo:: link kubernetes service to glossar or somewhere

To add a new service interface click on the :guilabel:`Provides` button in the :ref:`service detail <service-detail-manage>` page.

.. figure:: images/new-service-interface-dialog.*
   :name: new-service-interface-dialog

.. warning:: At least one port must be provided per service interface!

.. note::

    :samp:`Exposed Port Number` is the port that will be exposed in kubernetes.

    :samp:`Target Port Number` is the port exposed by the docker container.

After adding a interface to a service, the detail page should look something like this:

.. figure:: images/service-detail-with-service-interface.*
   :name: service-detail-with-service-interface

To edit a service intervace, click on the text of the service interface.

To delete a service interface, use the trashbin icon right beside the service interface.



Manage service dependencies
---------------------------

To add an existing service as a dependency to this service, use the :guilabel:`Dependees` button.

You can then select the service (and the specific version of that service) that you want to have as a dependency.

.. figure:: images/choose-service-dependency-dialog.*
   :name: choose-service-dependency-dialog

If you have added a dependency, the name of the dependency will show up below the :guilabel:`Dependees` button.

.. figure:: images/service-detail-with-dependency.*
   :name: service-detail-with-dependency

You can directly go to the detail page of a dependency by clicking on the name of the dependency.

To remove a dependency, use the trashbin icon right beside its name.

You can also view the whole dependency graph for the service in the :guilabel:`Dependency Graph` tab.

.. figure:: images/service-detail-dependency-graph.*
   :name: service-detail-dependency-graph

In the dependency graph you can directly change the version of a dependency if the dependency is directly connected to this service (the blue service node).
To change the version of a dependency click on its version number and choose a different version in the dialog.

.. warning:: Changing  the dependency graph may break existing applications that use this service.


