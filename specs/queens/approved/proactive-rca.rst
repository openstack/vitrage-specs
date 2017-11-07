..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================
Proactive RCA fault model
=========================

https://blueprints.launchpad.net/vitrage/+spec/proactive-rca

This blueprint proposes a solution for proactive RCA. It aims to be an
umbrella for all related blueprints.


Problem description
===================

Currently, Vitrage relies on the monitored events for root cause analysis. It
may deduce alarms on virtual resources, but it is always in a confirmed state.
But in the real world, things could be more complicated. Suppose there are two
possible causes (A or B) for fault C. When fault C is monitored, it is
suspicious that A or B could happen and be the root cause. We need a way to
take action in such case to find the root cause more proactively.


Proposed change
===============

A fault model with deductive reasoning is proposed here to resolve the problem above.

Fault model
-----------

Given the template defined for the fault model above.

.. graphviz::

    digraph G {
        // styles
        style="filled";
        color="lightgrey";

        A -> C [label="causes"];
        B -> C [label="causes"];
    }

The underlying entity graph could be breakdown as

.. graphviz::

    digraph G {
        // styles

        node [style="filled", color="lightgrey"]

        // equivalences

        subgraph cluster_2 {
            label="Fault A";
            a_m -> a_v [label="eq"][dir="both"];
        }

        subgraph cluster_3 {
            label="Fault B";
            b_m -> b_v [label="eq"][dir="both"];
        }

        subgraph cluster_4 {
            c_m -> c_v [label="eq"][dir="both"];
            label="Fault C";
        }

        // expanded RCA rule

        a_m -> c_m [label="causes"];
        a_v -> c_m [label="causes"];
        a_m -> c_v [label="causes"];
        a_v -> c_v [label="causes"];
        b_m -> c_m [label="causes"];
        b_v -> c_m [label="causes"];
        b_m -> c_v [label="causes"];
        b_v -> c_v [label="causes"];
    }

Deductive reasoning
-------------------

Suppose we received a series of events in the following order:

#. t0: initial status, no faults
#. t1: fault C active monitored
#. t2: fault A active and fault B inactive is reported from diagnose action
#. t3: fault C inactive monitored
#. t4: fault A inactive returned from diagnostic action

Let's illustrate the deductive reasoning in graphs with following legend.

- cluster: aggregate fault
- nodes: raw fault
- edges:

  - dashed: the target state is deduced from source state by Vitrage
  - dotted: the target state is updated by diagnose action triggered by source state

- border:

  - dashed: suspect

- colors:

  - lightgrey: fault in undefined state
  - red: fault in confirmed state

- labels:

  - ``Fault A``: aggregated
  - ``a_v``: deduced by Vitrage
  - ``a_m``: monitored

t0: initial status
^^^^^^^^^^^^^^^^^^

All faults undefined, no reasoning

.. graphviz::

    digraph G {
        node [style="filled", color="lightgrey"]

        // fixed layout with cluster and invisible edges
        subgraph cluster_1 {label="Fault A" a_m a_v}
        subgraph cluster_2 {label="Fault B" b_m b_v}
        subgraph cluster_4 {label="Fault C" c_m c_v}
        a_m -> c_v [label="deduces" style="invis"]
        b_m -> c_v [label="deduces" style="invis"]
        c_m -> a_v [label="deduces" style="invis"]
        c_m -> b_v [label="deduces" style="invis"]
        a_v -> a_m [label="diagnose" style="invis"]
        b_v -> b_m [label="diagnose" style="invis"]
        c_v -> c_m [label="diagnose" style="invis"]
    }


t1: downstream fault monitored, upstream fault deduced
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Vitrage deduces that fault A and fault B are active in suspect state
#. execute diagnose action to confirm suspect state

.. graphviz::

    digraph G {
        node [style="filled" color="lightgrey"]

        // fixed layout with cluster and invisible edges
        subgraph cluster_1 {label="Fault A" color="red" graph[style="dashed"] a_m a_v}
        subgraph cluster_2 {label="Fault B" color="red" graph[style="dashed"] b_m b_v}
        subgraph cluster_4 {label="Fault C" color="red" c_m c_v}
        a_m -> c_v [label="deduces" style="invis"]
        b_m -> c_v [label="deduces" style="invis"]
        //c_m -> a_v [label="deduces" style="invis"]
        //c_m -> b_v [label="deduces" style="invis"]
        a_v -> a_m [label="diagnose" style="invis"]
        b_v -> b_m [label="diagnose" style="invis"]
        c_v -> c_m [label="diagnose" style="invis"]

        // downstream fault monitored
        c_m [color="red"];

        // upstream fault deduced, in suspect state
        c_m -> a_v [label="deduces" style="dashed"];
        a_v [color="red" style="dashed"]

        // upstream fault deduced, in suspect state
        c_m -> b_v [label="deduces" style="dashed"];
        b_v [color="red" style="dashed"]
    }


t2: diagnose action executed, upstream fault confirmed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. monitored states updated by diagnose action
#. as a side-effect, Vitrage will deduce that fault C is active in parallel of monitored fault C

.. graphviz::

    digraph G {
        node [style="filled", color="lightgrey"]

        // fixed layout with cluster and invisible edges
        subgraph cluster_1 {label="Fault A" color="red" a_m a_v}
        subgraph cluster_2 {label="Fault B" color="green" b_m b_v}
        subgraph cluster_4 {label="Fault C" color="red" c_m c_v}
        //a_m -> c_v [label="deduces" style="invis"]
        b_m -> c_v [label="deduces" style="invis"]
        c_m -> a_v [label="deduces" style="invis"]
        c_m -> b_v [label="deduces" style="invis"]
        //a_v -> a_m [label="diagnose" style="invis"]
        //b_v -> b_m [label="diagnose" style="invis"]
        c_v -> c_m [label="diagnose" style="invis"]

        // old status
        c_m [color="red"];
        a_v [color="red" style="dashed"]
        b_v [color="red" style="dashed"]

        // diagnose action executed
        a_v -> a_m [label="diagnose", style="dotted"]
        b_v -> b_m [label="diagnose", style="dotted"]

        // upstream fault confirmed
        a_m [color="red"]
        b_m [color="green"]

        // downstream fault deduced as a side effect
        a_m -> c_v [label="deduce", style="dashed"]
        c_v [color="red"]
    }


t3: downstream recovery monitored, upstream recovery deduced
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. from the fault model, we can deduce fault A and fault B are inactive
#. execute diagnose action to resolve status inconsistency between fault A deduced (inactive) and monitored (active)

.. graphviz::

    digraph G {
        node [style="filled", color="lightgrey"]

        // fixed layout with cluster and invisible edges
        subgraph cluster_1 {label="Fault A" color="green" graph[style="dashed"] a_m a_v}
        subgraph cluster_2 {label="Fault B" color="green" b_m b_v}
        subgraph cluster_4 {label="Fault C" color="green" c_m c_v}
        a_m -> c_v [label="deduces" style="invis"]
        b_m -> c_v [label="deduces" style="invis"]
        //c_m -> a_v [label="deduces" style="invis"]
        //c_m -> b_v [label="deduces" style="invis"]
        a_v -> a_m [label="diagnose" style="invis"]
        b_v -> b_m [label="diagnose" style="invis"]
        c_v -> c_m [label="diagnose" style="invis"]

        // old status
        a_m [color="red"]
        b_m [color="green"]
        c_v [color="red"]

        // downstream recovery monitored
        c_m [color="green"]

        // upstream recovery deduced
        c_m -> a_v [label="deduces" style="dashed"]
        a_v [color="green"]

        // upstream recovery deduced
        c_m -> b_v [label="deduces" style="dashed"]
        b_v [color="green"]
    }

t4: upstream recovery confirmed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. graphviz::

    digraph G {
        node [style="filled", color="lightgrey"]

        // fixed layout with cluster and invisible edges
        subgraph cluster_1 {label="Fault A" color="green" a_m a_v}
        subgraph cluster_2 {label="Fault B" color="green" b_m b_v}
        subgraph cluster_4 {label="Fault C" color="green" c_m c_v}
        //a_m -> c_v [label="deduces" style="invis"]
        b_m -> c_v [label="deduces" style="invis"]
        c_m -> a_v [label="deduces" style="invis"]
        c_m -> b_v [label="deduces" style="invis"]
        //a_v -> a_m [label="diagnose" style="invis"]
        b_v -> b_m [label="diagnose" style="invis"]
        c_v -> c_m [label="diagnose" style="invis"]

        // old status
        a_v [color="green"]
        b_v [color="green"]
        b_m [color="green"]
        c_m [color="green"]

        // upstream recovery confirmed
        a_v -> a_m [label="diagnose" style="dotted"]
        a_m [color="green"]

        // upstream recovery confirmed
        a_m -> c_v [label="deduces" style="dashed"]
        c_v [color="green"]
    }

Changes required
----------------

The main change would be allowing raise alarms in suspect status which can be used to trigger a diagnostic action.

- Add support for diagnose action, like:

  - an immediate pull from data source
  - force a push from data source
  - launch external tools to get state and post events to Vitrage API

- Suspect states for alarm

  - for deduced alarm, it is suspect if the source is from downstream
  - for aggregated alarm, it is suspect if inconsistency is detected among underlying alarms (deduced and monitored)

Specially, it is not suspect if **newer** monitored state is different from **old** suspect deduced state. Because a
suspect state means it could be either active or inactive. So it is reasonable to trust the latest update from monitor.

However, when we bring in proactive RCA, the entity numbers in the graph may grow a lot. We shall need to create
deduced alarm for each monitored alarm and set suspect state in some condition. The relationships (edges) will also
grow. So there are some additional work to be done to improve user experience, such as:

- Aggregate underlying entity graph to simplify user view
- Add new API for querying aggregated entity graph
- Simplify the template definition by defining fault model

Backward compatibility
----------------------

A **fully functional** proactive RCA fault model requires every fault can be monitored with a diagnose action. But the
deductive reasoning procedure is also applicable for fault model lacking diagnose action or missing monitoring on some
fault.

For example, if there is no diagnose action for confirming Fault A, the deductive reasoning will still continue once
the status of Fault A get updated passively from monitor.

If there is no monitor for Fault A, then it will stay in suspect status. It helps the user to narrow down the scope of
root cause to make manual investigation easier.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  yujunz

Other contributors:
  no

Work Items
----------

See dependent blueprints.


Dependencies
============

The implementation of proactive RCA depends on several blueprints. They will be listed below once got approved.


Testing
=======

See dependent blueprints.


Documentation Impact
====================

See dependent blueprints.


References
==========

- https://etherpad.openstack.org/p/vitrage-fault-model
- https://etherpad.openstack.org/p/vitrage-ptg-queens
