..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Vitrage Get Topology API
========================

https://blueprints.launchpad.net/vitrage/+spec/vitrage-api-get-topology

Vitrage needs to supply an api for listing a full or partial resources and alarms topology.

Problem description
===================

We would like to be able to retrieve the Vitrage topology, which will include both the resources and their appropriate alarms


Proposed change
===============

Create API to retrieve the resources and alarms topology :

#.	Retrieve the full topology.
#.	List partial topology with filter :

    #.   Full connected components of a given element
    #.   Connected component of a given element, filtered according to edges of specific type(s) (i.e., labels and properties)



Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

A new api will be added called topology-show
the url will be "GET /v1/topology"
returned value will be a json describing the topology according to this spec http://jsongraphformat.info/

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
	dan-ofek <dan.ofek@alcatel-lucent.com>

Work Items
----------

None

Future lifecycle
================

None

Dependencies
============

Depends on the blueprints of the graph and the synchronizer

Testing
=======

Tempest tests also need to be added to test "get full topology" and "get partial topology" API.



Documentation Impact
====================
The new api should be documented

References
==========
