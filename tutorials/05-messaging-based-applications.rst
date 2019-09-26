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

    *  `test_message_generator <https://github.com/UST-MICO/test_message_generator>`_
    *  `msg-validator <https://github.com/UST-MICO/msg-validator>`_
    *  `http-to-messaging-adapter <https://github.com/UST-MICO/http-to-messaging-adapter>`_

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

.. TODO:: describe grapheditor features
