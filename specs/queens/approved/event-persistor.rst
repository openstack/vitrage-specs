..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============
Event Persistor
===============

https://blueprints.launchpad.net/vitrage/+spec/event-persistor

The presistor service listens to the RabbitMQ2 (on a different topic) and
asynchronously writes the events to a relational database. All events are
stored after the filter/enrich phase.

Problem description
===================

In order to support some of the main future use cases of Vitrage, including full
HA support, alarm history and RCA history, we will need to store the events
that arrive from the collector in a persistent database.
Explained in `Vitrage HA and History Vision`_.

.. _Vitrage HA and History Vision: https://docs.openstack.org/vitrage/latest/contributor/vitrage-ha-and-history-vision.html

Example of a use case for the stored data:
Reconstructing the graph from the historic data that controlled by the processor,
and will be used in two cases:

    - Upon failure, in order to initiate the standby vitrage-graph process
    - For RCA history

Proposed change
===============

.. _vitrage_persistor: new topic in rabbitMQ2 for the Persistor.

Add `event`_ table in Vitrage database.
Both Datasource driver and Service listener (collector) passes the filtered/enriched
events to the persistor via `vitrage_persistor`_ topic.
Add a Persistor service which listens to `vitrage_persistor`_ topic and writes the
events to `event`_ table.


Alternatives
------------

None

Data model impact
-----------------

.. _event: a relational database table to store the filtered/enriched events.

The table will have the following fields:

+----------------------+------------------------------------------------------+-------------------------------------+
| Field                | Description                                          | Examples                            |
+======================+======================================================+=====================================+
| id                   | INTEGER, Auto-Increment                              | 19588                               |
+----------------------+------------------------------------------------------+-------------------------------------+
| collector_timestamp  | The time the event filtered/enriched in the driver   | 2017-10-09 09:19:50                 |
+----------------------+------------------------------------------------------+-------------------------------------+
| payload              | The enriched event sent from the collector           | JSON representation of the event    |
+----------------------+------------------------------------------------------+-------------------------------------+


REST API impact
---------------

None

Versioning impact
-----------------

None

Other end user impact
---------------------

None

Deployer impact
---------------

None

Developer impact
----------------

None

Horizon impact
--------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  7mode3294

Other contributors:
  None

Work Items
----------
 - Add new topic `vitrage_persistor`_ in rabbitMQ2 for the Persistor.
 - Add `event`_ table in Vitrage database.
 - Both Datasource driver and Service listener (collector) passes the filtered/enriched
   events to the persistor via `vitrage_persistor`_ topic.
 - Add a Persistor service which listens to `vitrage_persistor`_ topic and writes the
   events to `event`_ table.

Dependencies
============

None

Testing
=======

The implementation will be covered by additional unit tests and tempest tests.

Documentation Impact
====================

None

References
==========

- https://docs.openstack.org/vitrage/latest/contributor/vitrage-ha-and-history-vision.html
