..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================
Support External Actions in Vitrage Templates
=============================================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/support-external-actions

Currently, the Vitrage templates support the following actions: raise alarm,
set state, mark causal relationship and mark-down.
The implementation of mark-down is patchy (update a property on the vertex and
call the notifier).

We need a way to define in the template an external action that should be
taken. One example is the mark-down. Another example is to execute a Mistral
workflow or a Congress policy.

Problem description
===================

The following use cases should be supported:

1. **mark-down**. This is an existing use case. If a mark-down action exists,
   Vitrage will call Nova mark-down API. The behavior should remain the same
   after the change.
2. **Execute a Mistral workflow**. The user should be able to execute a Mistral
   workflow based on Vitrage insights.
3. **Execute other external actions**, like a Congress policy.

Proposed change
===============

Add a new action for each external engine we would like to support. The action
will have metadata with parameters to be sent to the engine.

Examples
--------

.. code-block:: yaml

   action:
     action_type: execute_nova
     metadata:
       api_call: mark-down
       host: host1


.. code-block:: yaml

   action:
     action_type: execute_mistral
     metadata:
       workflow: evacuate_host
       failed_host: host1


The idea behind adding a new external_* action for each engine is to emphasize
that Vitrage supports a predefined set of external actions, and specifically
it does not support runinng scripts.

The internal implementation will be identical for Nova, Mistral, Congress, etc.
During the template loading phase, the execute_* action will be converted to
a generic Execute action, with a notifier parameter. The template validator
will verify that the wanted notifier is enabled in vitrage.conf, otherwise
the template loading will fail.

When executing the Execute action, the evaluator will send an event to the
message bus of the wanted notifier. This way, only the Mistral notifier will
handle the execute_mistral actions.

Note that currently only the Vitrage graph sends notification to the notifier.
This behavior will be changed, so notifications will be sent both from the
graph (for deduced alarms and set state) and from the evaluator.


Alternatives
------------

There are two alternatives:

Send Message bus notifications
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Vitrage can have a message bus notifier, that will send notifications about
raise/delete alarm, set state and mark causal-relationship. Mistral will be
configured to execute certain workflows based on the notifications received
from Vitrage.

Advantages:

* No extra configuration needed in Vitrage
* Can work with the current notifiers mechanism

Disadvantages:

* Messgae bus notifications might get lost
* Less powerful. In Vitrage template the user can decide to execute a workflow
  based on a combination of alarms or a specific topology. Once Mistral gets
  the notifications, the topology context is no longer there.
* mark-down use case is not supported by this solution

Configuration Done on the specific notifier
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
A Mistral notifier will be written and receive notifications from the graph.
It will be configured by a yaml file that will determine which workflow to
execute per specific alarm.

Advantages:

* No extra configuration needed in Vitrage
* Can work with the current notifiers mechanism
* No lost messages

Disadvantages:

* Less powerful. In Vitrage template the user can decide to execute a workflow
  based on a combination of alarms or a specific topology. In Mistral notifier
  we no longer have this context.
* mark-down use case is not supported by this solution


Calculate the action_target
---------------------------
Every "regular" action (raise_alarm, set_state, add_causal_relationship) has
an action_target block. The action_target has no meaning for the external
actions, since they are managed by an external engine that does not have a
"target" terminology.

The problem is that the action_target is essential for
the subgraph-matching calculations, in order to calculate the connected
components.

The proposed solution
~~~~~~~~~~~~~~~~~~~~~
For external actions, we will automatically calculate an action
target. The target will be selected arbitrarily from the set of possible
targets. In complex conditions that involve {and, or, not} finding the target
might not be trivial. Some more-complex conditions will not be supported for
external actions, due to this reason.


The calculation method:

* And condition - any vertex that is part of the condition can be a target
* Not condition - no vertex that is part of the condition can be a target
* Or condition - the target should be a vertex that appears in any "positive"
  part (i.e. one that does not have a 'not' in front of it) of the Or condition

Examples:
~~~~~~~~~

**Note:** In some cases it is impossible to find a valid target. They are
marked with '/'.

The template validation should fail in these cases, for **all kinds** of
actions.


+----------------------------------------------------------+-----------------+
| Condition                                                | Possible targets|
+----------------------------------------------------------+-----------------+
| a_in_error_status                                        | a               |
+----------------------------------------------------------+-----------------+
| a_contains_b                                             | a, b            |
+----------------------------------------------------------+-----------------+
| a_contains_b or a_contains_c                             | a               |
+----------------------------------------------------------+-----------------+
| a_contains_b or a_contains_c or a_contains_d             | a               |
+----------------------------------------------------------+-----------------+
| a_in_error_status or a_contains_c or a_contains_d        | a               |
+----------------------------------------------------------+-----------------+
| a_contains_b or a_contains_c or b_contains_d             | Not Supported   |
+----------------------------------------------------------+-----------------+
| a_contains_b and a_contains_c                            | a, b, c         |
+----------------------------------------------------------+-----------------+
| a_contains_b and a_contains_c and a_contains_d           | a, b, c, d      |
+----------------------------------------------------------+-----------------+
| a_contains_b or (a_contains_c and a_contains_d)          | a               |
+----------------------------------------------------------+-----------------+
| a_contains_b or (a_contains_c and b_contains_d)          | a,b             |
+----------------------------------------------------------+-----------------+
| not a_contains_b                                         | Not Supported   |
+----------------------------------------------------------+-----------------+
| not a_in_error_status                                    | Not Supported   |
+----------------------------------------------------------+-----------------+
| a_contains_b or not a_contains_c                         | Not Supported   |
+----------------------------------------------------------+-----------------+
| a_contains_b or (a_contains_c and not a_contains_d)      | a               |
+----------------------------------------------------------+-----------------+
| a_contains_b or (not a_contains_c and not a_contains_d)  | Not Supported   |
+----------------------------------------------------------+-----------------+


Alternatives to calculating the action_target
---------------------------------------------
We can decide that an external action requires an action_target like all other
actions. This will simplify the implementation, but is logically wrong.
The action_target will serve as a "helper entity" for the subgraph-matching,
and it should not be the end user's role to help fixing implementation issues.
As far as the external engine is concerned, the action_target has no meaning.


Data model impact
-----------------

A new Execute action will be added to the evaluator.

REST API impact
---------------

None

Versioning impact
-----------------

We should introduce a versioning mechanism to the templates. This will be done
when modifying the implementation of mark-down.

Other end user impact
---------------------

None

Deployer impact
---------------

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
  ifat-afek

Work Items
----------

* Enhance the template language (template loading and validation)
* Update the documentation
* Execute the external actions from the evaluator

Dependencies
============

None

Testing
=======

The implementation will be covered by unit tests and tempest tests.

Documentation Impact
====================

The new action should be documented

References
==========

None
