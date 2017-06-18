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
