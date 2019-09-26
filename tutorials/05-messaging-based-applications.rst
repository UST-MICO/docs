How To create a messaging based application
===========================================

MICO supports two different communication types between microservices.
The standard communication type is ``http``.
These connections use the :term:`service interfaces <Service Interface>` as described in :doc:`How To manage an application <04-manage-an-application>`.
The second communication type is :doc:`cloud events <../messaging/cloudevents>` over `Kafka`_.

.. _Kafka: https://kafka.apache.org

Kafka enabled services
----------------------

To use the provided `Kafka`_ broker a service hast to be marked as ``Kafka Enabled``.
Such a service does not need to specify an http :term:`interface <Service Interface>` to be deployable.

See :doc:`How To add a service <01-add-a-service>` for details on how to add and edit a service.

.. note::

    The `UST-MICO Organization <https://github.com/UST-MICO>`_ on GitHub has three example repositories containing `Kafka`_ enabled services that you can import directly.

    *  `UST-MICO/test_message_generator <https://github.com/UST-MICO/test_message_generator>`_
    *  `UST-MICO/msg-validator <https://github.com/UST-MICO/msg-validator>`_
    *  `UST-MICO/http-to-messaging-adapter <https://github.com/UST-MICO/http-to-messaging-adapter>`_

.. hint::

    For all `Kafka`_ enabled services the following environment variables will be set:

    ``KAFKA_BOOTSTRAP_SERVERS``
        The URLs of the bootstrap servers
    ``KAFKA_GROUP_ID``
        The group id to be used when subscribing to a Kafka topic
    ``KAFKA_TOPIC_INPUT``
        The topic to receive messages from
    ``KAFKA_TOPIC_OUTPUT``
        The topic to send messages to
    ``KAFKA_TOPIC_INVALID_MESSAGE``
        The topic, to which invalid messages are forwarded
    ``KAFKA_TOPIC_DEAD_LETTER``
        The topic, to which messages are forwarded that can't/shouldn't be delivered
    ``KAFKA_TOPIC_TEST_MESSAGE_OUTPUT``
        The topic, to which test messages are forwarded at the end of a test

    Input and output topic can be set by the user creating the application.
    The rest are set by the MICO system.


The Kafka-FaaS-Connector
------------------------

The `Kafka-FaaS-Connector <https://github.com/UST-MICO/kafka-faas-connector>`_ is a special service that can be used to implement most of the `Enterprise Integration Patters by Gregor Hohpe and Bobby Woolf <https://www.enterpriseintegrationpatterns.com>`_
The communication with `Kafka`_ and some of the :doc:`cloud event attributes <../messaging/cloudevents>` are managed by the Kafka-FaaS-Connector.
Custom behaviour is implemented as a `OpenFaaS-Function <https://www.openfaas.com/>`_.
The function can be managed with the supplied OpenFaaS dashboard that is linked to from the MICO dashboard.
MICO will import the latest version of the Kafka-FaaS-Connector on startup automatically.

.. TODO:: add screenshot for openfaas link

.. seealso::

    More backround information:

    * :doc:`../adr/0021-kafka-as-messaging-middleware`
    * :doc:`../adr/0023-faas`
    * :doc:`../adr/0022-function-to-component-mapping`
    * :doc:`../adr/0025-generic-component`
    * :doc:`../adr/0027-simple-composition-components`
    * :doc:`../adr/0026-implementation-of-complex-eai-patterns-with-faas`


Creating messaging based networks
---------------------------------

.. note::

   For this tutorial the following assumptions are made:

   The services :samp:`UST-MICO/test_message_generator` and :samp:`UST-MICO/msg-validator`
   have been :ref:`imported from github <importing-a-service-from-github>`.

   Both services have to be edited to set the Kafka Enabled bit!

   The service :samp:`UST-MICO/msg-validator` has an interface
   :samp:`validator` with :samp:`Exposed Port Number` 80, :samp:`Target Port Number` 8090 and :samp:`type` :samp:`TCP`.

   The Application :samp:`New Application` includes both services.


The starting configuration should look like this:

.. figure:: images/new-messaging-app.*
   :name: new-messaging-app

To link a Kafka enabled service with Kafka topic hover over the service and drag one of the appearing link handles.

.. figure:: images/highlighted-link-handles.*
   :name: highlighted-link-handles

The dragged edge can be dropped in the empty space to create a new topic.

.. figure:: images/drag-link-from-service.*
   :name: drag-link-from-service

In the add topic you can specify the topic and the role.
The topic can either be used as a input to the service or as an output from the service.
A service can only have one input and one output topic in the grapheditor.

.. figure:: images/add-topic-dialog.*
   :name: add-topic-dialog

After confirming the dialog the topic appears in the grapheditor.
If a topic is already in the grapheditor services can also be linked directly to the topics by dragging an edge from the topic to the service or from the service to the topic.
The direction of the arrow symbolises the message flow.

.. figure:: images/messaging-app-first-topic.*
   :name: messaging-app-first-topic

Add the topic as input topic to the :samp:`UST-MICO/msg-validator` service.
This is a minimal configuration for a messaging based application.

.. figure:: images/messaging-app-small.*
   :name: messaging-app-small

To remove a link drage the link from the link handle near the link target on the link.

.. figure:: images/delete-link.*
   :name: delete-link

Drop the link on empty space to delete it.

.. figure:: images/link-removed.*
   :name: link-removed

To add a Kafka-FaaS-Connector to the application simply drag a edge from a topic and drop it over empty space.
The Kafka-FaaS-Connector may take a few seconds to appear.

.. figure:: images/add-kafka-faas-connector.*
   :name: add-kafka-faas-connector

Dragg a edge from the Kafka-FaaS-Connector to connect it with a new topic.

.. figure:: images/messaging-app-second-topic.*
   :name: messaging-app-second-topic

.. figure:: images/messaging-app-big.*
   :name: messaging-app-big

To change the FaaS function of the Kafka-FaaS-Connector click on the title of the Kafka-FaaS-Connector node to open the dialog.

.. figure:: images/update-faas-function-dialog.*
   :name: update-faas-function-dialog

The title will change to the set FaaS function.

.. figure:: images/updated-faas-function.*
   :name: updated-faas-function
