..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
Vitrage RCA Support
===================

https://blueprints.launchpad.net/vitrage/+spec/support-rca

Vitrage should support RCA calculation.
This feature will allow us to track the cause-and-effect path of an alarm raised in OpenStack. In order to track the
local causal relationships between alarm-pairs we shall use one or more RCA templates which will specify which alarms
cause which alarms.

Problem description
===================

In case of a major failure in the system, we might get a lot of alarms, which will be hard to track. We would like to
identify the root cause of the alarms, so the user can focus on understanding and fixing this alarm.

Proposed change
===============

The Vitrage Evaluator serves as workflow manager controlling the analysis and activation of templates and execution
of template actions. One of its responsibilities is to listen to changes in Vitrage Graph, and upon a change execute
the matching templates. This is a general mechanism that should work for all kinds of templates and perform several
kinds of actions.

The aim of this blueprint is to make sure RCA functionality works properly end to end.

Whenever the Vitrage Graph is updated, we will calculate RCA and optionally connect alarm vertices with "causes" edges.
When RCA relations are queried for a certain alarm (i.e. which alarm(s) caused it and which alarm(s) were caused by it),
we will traverse the already-existing "causes" edges and return the RCA tree.

Example for a graph with causes edges:

::

  +---------------+              +-------------+
  |               |     on       |             |
  | switch alarm  | +----------> |   switch    |
  |               |              |             |
  +------+--------+              +-------+-----+
         |                               |
 causes  |                               | attached
         |                               |
         v                               v

  +---------------+              +-------------+
  |               |     on       |             |
  | host alarm    | +----------> |   host      |
  |               |              |             |
  +------+--------+              +-------+-----+
         |                               |
 causes  |                               | contains
         |                               |
         |                               |
         v                               v

  +---------------+              +-------------+
  |               |     on       |             |
  | instance alarm| +----------> |  instance   |
  |               |              |             |
  +---------------+              +-------------+


Alternatives
------------

We could re-calculate the RCA relationship whenever someone queries it, but this would be inefficient. Calculating
in advance and keeping the results in Vitrage Graph makes more sense.

Data model impact
-----------------

None

REST API impact
---------------

The API is defined in a separate blueprint: https://blueprints.launchpad.net/vitrage/+spec/rca-api

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

Performance should be tested.
Most of the performance risk is in the common blueprints like https://blueprints.launchpad.net/vitrage/+spec/networkx-graph-driver
(see also https://blueprints.launchpad.net/vitrage/+spec/networkx-performance-improvement). However, we will also need
to have specific tests for RCA.

Other deployer impact
---------------------

None

Developer impact
----------------

None

Horizon impact
--------------

We should develop horizon UI plugin for viewing the RCA relationship. This should be described in a separate blueprint.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
    ifat_afek <ifat.afek@alcatel-lucent.com>

Work Items
----------

The blueprint includes:

- Define the exact syntax for RCA templates
- Mark the causal relationship between two alarms. We would implement it using an action that adds a "causes" edge between the alarm vertices in Vitrage Graph.
- Define and implement the method to query the RCA relations for a given alarm


Future lifecycle
================

None

Dependencies
============

- Vitrage Graph
- Vitrage Engine

Testing
=======

This change needs to be tested by unit tests.

Documentation Impact
====================

None

References
==========

https://wiki.openstack.org/wiki/Vitrage

