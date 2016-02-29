..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Vitrage Resource Processor
==========================

https://blueprints.launchpad.net/vitrage/+spec/entity-graph-consistency-validator

The Entity Graph may have mistakes, such as an incorrect resource state, and we would like to detect and repair such errors.

Problem description
===================

The Entity Graph is a living graph which is updated all the time with the data received from the synchronizer(s). The Entity Graph may have errors and become inconsistent compared to the state of the Cloud, which may occur due to:
(1)	The Vitrage Resource Processor may perform incorrect operations during graph updates, for example due to problems resulting from multi-threading.
(2)	Missing updates from the synchronizer(s).

Proposed change
===============

We would like to check the Entity Graph each set interval (configurable) and validate the consistency of its resources and state. If the Entity Graph is incorrect, it will repair it. When needed, it might consult the synchronizer(s) to retrieve missing data.

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
