..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================
Vitrage-Collector on demand
===========================

https://blueprints.launchpad.net/vitrage/+spec/rpc-collector

A simpler implementation of Vitrage high availability, collector service
should meet these new requirements:

 - collector should be active-active.
 - Dependency between the collector and graph services should be removed.
 - vitrage-graph should be able to request updates from the collector when needed.

Problem description
===================

The current implementation, where vitrage collector and vitrage graph are
always restarted simultaneously, is not desired, but is due to the lack of a
better triggering event for get_all. In addition message bus may overload
occasionaly due to this behaviour, when vitrage-graph is down and does not
consume the messages.


Proposed change
===============

Collector receives synchronous RPC and retrieves a list of all the events.

vitrage-collector implementation:
 - Remove timers for get_all and get_changes
 - get_all and get_changes can be called by RPC
 - get_all and get_changes receive a list parameter `datasource_names`.
 - Will return all the entities from all the specified data-sources
 - Will open a thread to write all the events to the Persistor message queue
   in the background.
 - Remove dependency between the collector and graph services (.service files)

vitrage-graph implementation:
 - Should manage the RPC calls to get_all and get changes.
 - Add appropriate timers calling get_all and get_changes by RPC
 - List of events should be processed using an event coordinator
   (before `TwoPrirityQueue`)
 - A snapshot should be taken after each get_all once the events have been processed.
   So the majority of events do not need to be replayed



Alternatives
------------

None


Data model impact
-----------------

None

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

vitrage-collector could be restarted individually from vitrage-graph

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
  idan-hefetz

Other contributors:
  None

Work Items
----------

None

Dependencies
============

None

Testing
=======

New behaviour is already covered by existing tempest

Documentation Impact
====================

None

References
==========

None

