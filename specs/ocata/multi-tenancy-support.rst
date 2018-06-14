..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================
Vitrage Multi Tenancy Support
=============================

When a tenant uses Vitrage APIs we would like to show him only the data that is relevant to him.

Problem description
===================

Vitrage needs to show in its API an CLI only the data relevant to that tenant (it can't show all the data due to irrelevancy and privacy of each tenant).
Thus for each datasource and entity we need to know what relevant data to show for that tenant.
We would also like to show all of the data if someone adds the all_tenants property.

Proposed change
===============

Here is a description, for each of the Vitrage APIs, how it should behave for each tenant:

Get alarms:
 1. Find all the alarms with the requested project_id (if the project is admin then show also alarms that has no project_id property)
 2. Find all the resources with the requested project_id and return all the alarms that are attached to them.
 3. Merge the results from the previous steps and return it.

Get RCA:
 1.	Find all the alarms that this alarm has caused, recursively. When reaching an alarm that is not of same project_id (or on resource of same project_id), stop the recursion, including this last alarm in the response.
 2. Find all the alarms that caused this alarm, recursively. When reaching an alarm that is not of same project_id (or on resource of same project_id), stop the recursion, including this last alarm in the response.
 3. Merge the results from the previous steps and return it.

Get Topology:
 1.	Find all the connected components for project_id.
 2. For each component, select 1 entity and find all paths (without cycles) to "openstack.cluster" entity. Add all of the vertices in the path to the component.
 3.	Merge all the components and return it.


Remark: API and CLI needs to behave the same.

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
