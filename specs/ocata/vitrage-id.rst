..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========.
Vitrage ID
===========.

Vitrage ID will be standard generated UUID.

Problem description
===================

Currently Vitrage ID is actually a set of properties. This can create duplicates and is very misleading.
Furthermore it will impair us once we have history, since same alarms can happen multiple times. Thus,
it should be the same as any other service in Openstack and provide an ID based on Openstack UUID.
generating algorithm.

Proposed change
===============

Changing Vitrage ID from it's current algorithm to an OpenStack compliant "UUIDUtils" generated UUID.

All the documentations and examples in Vitrage project should be updated to use an OpenStack compliant UUID.

A few Mock Json files exist for “mock api” requests purposes, for example, alarms.sample.json.
They contain the “old” Vitrage ID. The “mock” api should be deleted, and the same should go with
these file. We can also just fix the examples in the mock.


Alarm IDs: When creating Vitrage ID for an alarm, we also need to put it in the metadata when
updating AODH / other. Afterwards, when getting the updated alarm back, we will to update the
alarm ID in Vitrage, In case we will have simultaneous multiple alarm engines, we might need to
have an “ServiceName / ID” map for the alarm, and the Alarm ID in Vitrage will be “Vitrage ID”.


Template IDs should be changed back from a calculated String to generated uuid, in scenario_repository.


Datasources:
 - key / value tests : fix field names.
 - Transformers: Instead of the current neighbor Vitrage ID, generate UUID. It will be dropped at a later stage in the processor, in case the entity already exists in the Graph.
 - Processor: Checking if an entity exists in the graph: The entity is currently queried in the Graph according to its Vitrage ID. Instead, it will be queried according to the parameters set. If the entity exists, it’s original Vitrage ID will be used. Otherwise, the newly generated vitrage ID is used.


Update all necessary tests.


No changes are needed in Evaluator Action / Recipes or in Consistency enforcer.


Performance/Scalability Impacts
-------------------------------

Performance needs to be tested, since after the development of this blueprint, all graph queries will use parameters
instead of a single index.

As long as Vitrage uses an in memory graph database, starting from this change, standard HA will be buggy,
to say the least. An entity's Vitrage ID will have different values in each HA "instance". Using Pacemaker
equivalent HA (stonith) will solve this.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  doffek <dany.offek@gmail.com>


Dependencies
============

Aodh : Need to change the notification from Vitrage to Aodh.

Testing
=======

Unit tests and tempest tests.

Documentation Impact
====================

All documentation regarding the creation of Vitrage ID will be updated.

References
==========

None
