..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Vitrage Resource Processor
==========================

https://blueprints.launchpad.net/vitrage/+spec/vitrage-resource-processor

When Vitrage initializes we need to create the Entity Graph on which the Vitrage will run it’s algorithms (sub graph matching, BFS, DFS etc.) and perform the actions (RCA, deduced alarms etc.). After the initialization of the graph, the resources changes are being processed and pushed to the Entity Graph.

Problem description
===================

Vitrage doesn’t have a full Entity Graph of the resources and their state when it initializes.
Vitrage needs to be consistent with the updated and new resources.

Proposed change
===============

When Vitrage initializes we need to build the up to date entity graph, so we can run the different algorithms on it and perform the required actions. To perform we require the full resources list from the synchronizer.
In order for the Entity Graph to be constructed, it will need updates on changes in the cloud (e.g., machine creation/termination), and guidance on how resources should be linked to each other (e.g., a virual machine resides on a physical host, which in turn belongs to an availability zone, etc.). For this purpose, each synchronizer will also update a configuration file - the entity graph template - which specifies for each resource what and how to locate the specific resources to link to.

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
	dan-ofek <dan.ofek@alcatel-lucent.com>

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
