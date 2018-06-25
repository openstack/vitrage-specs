..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================
Vitrage-graph fast failover
===========================

https://blueprints.launchpad.net/vitrage/+spec/vitrage-fast-failover

vitrage-graph high availability should meet these requirements:
 - Single active instance of vitrage-graph (managed by pacemaker).
 - Initialize quickly upon failover without requesting updates.
 - In case of a long downtime, vitrage-graph startup will request
   collector updates


Problem description
===================

Vitrage-graph is active standby. Currently on a failover, vitrage-graph
needs to pull all the data again from the collector data-sources.
This takes a considerable amount of time, in which data is inconsistent.
As we wish to continue working with an in-memory graph (due to performance),
vitrage-graph service will remain active-standby. Therefore, downtime must
be minimized in failover events.

Proposed change
===============

 - after every get_all, vitrage-graph stores a full entity graph snapshot in
   the db, so the majority of events do not need to be replayed.
 - Vitrage-graph sends each processed event to vitrage-persistor so these
   are stored in the order of handling.
 - Upon init vitrage-graph queries the db table graph_snapshots, fetching the
   latest entry, it will be used if it is not older than snapshot_interval.

Init with a snapshot - on failover
 - Unpickle stored snapshot to get the graph.
 - Run the processor on all the events (from events table) that occurred after
   the snapshot.
 - Enable the evaluators.
 - Process all the events that are waiting in the message bus.

Init without a snapshot - a fresh start (This is the current behaviour).
 - Start with a new empty graph.
 - RPC to Collector to run get_all for all drivers, then process the events.
 - Process all the events that are waiting in the message bus.
 - Enable evaluator and iterate all graph.

Alternatives
------------

Using a persistent graph database can improve vitrage-graph high availability
as fail-over will be quick due to running active-active. This may be a
preferred solution in terms of high availability, but overall, when comparing
performance compared to in-memory networkx, the degradation is not reasonable


Data model impact
-----------------

May require minor changes, TBD.


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

This will be enabled by default.
Deployer may disable in by adding the following to vitrage.conf
[persistancy]
enable_persistancy=false

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

Additional tempest will be added for fail-over, as persistence is already
covered by existing tempest.
Unit tests will not be affective here as changes are mostly in the init process
and scheduler. This feature mostly reuses existing (tested) functionality.

Documentation Impact
====================

None

References
==========

None

