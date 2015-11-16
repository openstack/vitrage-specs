..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================
Vitrage Support for Deduced Alarms
==================================

https://blueprints.launchpad.net/vitrage/+spec/support-deduced-alarms

Vitrage should support raising deduced alarms.
When an alarm is raised on a certain resource, we might conclude that it results in problems in other related
resources. In this case, we would like to raise deduced alarms on these other resources, even if we got no indication
of a problem from other sources of information.

Problem description
===================

There are cases where we know that a failure in one resource should result in problems in other resources. For example,
high CPU load on a host may cause suboptimal performance of all instances running on this host, thus performance
problems in the applications running on these instances. As far as Nova is concerned, all instances are up and
running, and there is no indication of a problem.


Proposed change
===============

The Vitrage Evaluator serves as workflow manager controlling the analysis and activation of templates and execution
of template actions. One of its responsibilities is to listen to changes in Vitrage Graph, and upon a change execute
the matching templates. This is a general mechanism that should work for all kinds of templates and perform several
kinds of actions.

The aim of this blueprint is to make sure deduced alarms functionality works properly end to end.

Whenever a new alarm is raised, Vitrage Graph is updated with a new vertex for this alarm, connected to the relevant
resource. Then, the vitrage evaluator engine looks for templates that could match this new alarm. If deduced alarms
templates are found, the engine will try to find a full match for the entire template in Vitrage Graph. If found,
the engine will ask the notifier to raise new alarms according to the template.


Example for a graph with deduced alarms:

::

 Original alarm:
 +-----------+               +------------+
 |Host High  |     on        |            |
 |CPU load   | +-----------> |  Host      |
 |           |               |            |
 +-----------+               +-----+------+
                                   |
                                   | contains
 Deduced alarm                     |
 to raise:                         |
 +-----------+               +-----v------+
 |Instance   |     on        |            |
 |Suboptimal | +-----------> |  Instance  |
 |Performance|               |            |
 +-----------+               +------------+




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

Horizon impact
--------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
    ifat_afek <ifat.afek@alcatel-lucent.com>

Work Items
----------

The blueprint includes:

- Define the exact syntax for deduced alarms templates
- Call the notifier to raise alarms


Future lifecycle
================

None

Dependencies
============

- Vitrage Graph
- Vitrage Engine
- Vitrage Notifier

Testing
=======

This change needs to be tested by unit tests.

Documentation Impact
====================

None

References
==========

https://wiki.openstack.org/wiki/Vitrage

