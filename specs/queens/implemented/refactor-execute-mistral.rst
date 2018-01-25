..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Refactor execute-mistral action
===============================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/refactor-execute-mistral-definition

The definition of the execute-mistral action should be changed, to better
support the integration of Vitrage and Mistral.

Problem description
===================

With the current execute-mistral action definition, it is possible to pass
string values to the Mistral workflow, but not dynamic attributes like the id
or ip address of the instance that was matched in the template condition.
We should enhance the template language to support passing dynamic attributes
to Mistral.

Another issue is that the structure of the action definition should be changed.
In the current structure, optional input parameters to the workflow appear on
the same level as the 'workflow' property, which is a mandatory part of the
action definition:

.. code-block:: yaml

   - action:
       action_type: execute_mistral
       properties:
         workflow: evacuate_host
         timeout: 10
         force: false

Instead, we should gather all optional parameters (timeout and force) under
an 'input' section.


Proposed change
===============

The first part of the change is to create a new 'input' section, under which
all input parameters of the workflow should be placed. A new versioning
mechanism should be introduced to Vitrage templates, to allow validating and
loading of both the old format and the new format. At the first stage both
formats will be supported, but in a version or two we should deprecate the old
format.

The second part is to define a way to describe which attribute of a specific
entity should be passed to the workflow. The suggested solution is to use
a syntax that is similar to the HOT template.

We will introduce a get_attr() function with the following parameters:

* ``resource template_id``: the id of the resource inside the template. Note
  that the resource must be part of the condition.

* ``attribute``: the name of the attribute to use. The attribute will be taken
  from the resource vertex in the graph, if the condition is met.


.. code-block:: yaml

 - scenario:
     condition: host_down_alarm_on_host
     actions:
       - action:
           action_type: execute_mistral
           properties:
             workflow: evacuate_host
             input:
               host_id: get_attr(host, "id")
               host_ip_addr: get_attr(host, "ip_address")
               timeout: 10
               force: false


Alternatives
------------

One alternative is to replace the get_attr with a shorter syntax. In this case,
we will refer to an entity attribute by the entity template-id and the
attribute name. For example, host.ip_address will mark the ip_address
attribute of the instance.

The example above will look like:

.. code-block:: yaml

   - action:
       action_type: execute_mistral
       properties:
         workflow: evacuate_host
         input:
           host_id: host.id
           host_ip_addr: host.ip_address
           timeout: 10
           force: false

This looks much nicer, but has two main disadvantages:

* It is not clear, just by looking at the template, that this is a reference to
  an attribute. What if the user wants to pass a string which is "host.id"?

* It is less generic. On the other hand, if we add get_attr as a function
  syntax, we will be able to add other functions in the future with a similar
  syntax.


Data model impact
-----------------

None

REST API impact
---------------

None

Versioning impact
-----------------

The suggested change is not backward-compatible with Pike. Vitrage templates
should be enhanced to support versioning, and both the old version and the new
one should be supported for now.

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

* Support versioning in Vitrage templates. Allow per-version validators and
  loaders for specific actions.
* Move optional input parameters under 'input' section
* Support get_attr

Dependencies
============

None

Testing
=======

The implementation will be covered by unit tests and tempest tests.

Documentation Impact
====================

The changes in the action definition should be documented

References
==========

`Vitrage template format: <https://docs.openstack.org/vitrage/latest/contributor/vitrage-template-format.html>`_
