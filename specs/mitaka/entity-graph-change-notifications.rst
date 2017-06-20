..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Vitrage Resource Processor
==========================

https://blueprints.launchpad.net/vitrage/+spec/entity-graph-change-notifications

When the Entity Graph is changed, different services would like to be informed of the change, and perform operations
accordingly.

Problem description
===================

When the Entity Graph is changed, different services would like to be informed of the change, and perform operations
accordingly. Such as:
(1) When edges \ vertices are added \ removed, the Vitrage Evaluator will run the subgraph matching algorithm on the
graph to find the templates which are matching and perform required actions.
(2) When properties of existing edges \ vertices are modified, this can also impact subgraph matching outcome and thus
the Vitrage Evaluator will need to be notified.

Proposed change
===============

For each CRUD operation on the Entity Graph, we will support adding listeners that can be notified, and receive updates
when each change occurs. The update must include information about each change.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

None

Pipeline impact
---------------

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

None


Other deployer impact
---------------------

None

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
	alexey_weyl <alexey.weyl@alcatel-lucent.com>

Work Items
----------

None

Future lifecycle
================

None

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

None

References
==========

None
