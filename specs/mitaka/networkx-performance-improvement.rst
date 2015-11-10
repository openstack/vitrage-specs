..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================
NetworkX Performance Improvement
================================

https://blueprints.launchpad.net/vitrage/+spec/networkx-performance-improvement

Many CRUD operations can be performed on the Entity Graph simultaneously. This can cause inconsistency regarding the resources and states on the graph.

Problem description
===================

Many CRUD operations can be performed on the Entity Graph simultaneously. This can cause inconsistency regarding the resources and states on the graph.

Proposed change
===============

We would like to have graph versioning, which will held two copies of the Entity Graph: (1) Stable copy of the last consistent graph, and (2) Dynamic copy of the graph which is updated by the Vitrage Resource Processor. Every set interval the changes inserted to the dynamic graph will be updated to the stable graph, resulting in the stable graph being up to date.

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

This change needs to be tested by unit tests.

Documentation Impact
====================


References
==========

