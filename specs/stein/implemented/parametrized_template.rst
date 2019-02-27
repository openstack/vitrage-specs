..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================
Parametrized Template
=====================

StoryBoard link: https://storyboard.openstack.org/#!/story/2004056

This spec suggests a method to quickly create Vitrage templates based on
a template that contains parameters.

Problem description
===================

Vitrage template language is powerful, yet a bit verbose and it's quite easy
to make syntax mistakes when creating a new template from scratch. Moreover,
in real production environments, we see that many templates have a very similar
structure like "an alarm on a host is propagated to its instances". We would
like to make it easier to generate the most common templates.

Another motivation for this feature is for Heat usage. A Heat user should be
able to easily define a Vitrage template for auto-healing or auto-scaling its
stack based on specific alarms. For more information, see Heat_spec_ and
Heat_template_.

.. _Heat_spec: https://review.openstack.org/#/c/578786/
.. _Heat_template: https://review.openstack.org/#/c/583951/


Proposed change
===============

Parts of a template, like alarm names, resource states, etc. can be written as
``parameters``. When creating a new template, the user will have to assign
actual values to each parameter that doesn't have a default value. The concept
is similar to the way that parameters are used in Heat.

A ``Parameters`` section can be added to standard templates. Each ``parameter``
will hold two optional fields:

* ``description``
* ``default``

All values will be strings.

The parameters can be used inside the templates, using a ``get_param(param)``
syntax.

Vitrage ``template add`` API will include a new option to create a template
based on a template with parameters. The template will be added to the database
only if all parameters are set.

Example for a template with parameters
--------------------------------------

.. code-block:: yaml

   metadata:
     version: 3
     type: standard
     name: host_affects_instance_prototype
     description: Template prototype for scenarios where a Zabbix alarm on a host affects its instances
   parameters:
     host_alarm_rawtext:
       description: Zabbix rawtext of the alarm on the host
     instance_alarm_name:
       description: Name of the alarm to be created on the instance
     instance_alarm_severity:
       description: Severity of the alarm to be created on the instance
       default: WARNING
     host_state:
       description: New state to be set for the host
       default: SUBOPTIMAL
     instance_state:
       description: New state to be set for the instance
       default: SUBOPTIMAL
   definitions:
     entities:
      - entity:
         category: ALARM
         type: zabbix
         rawtext: get_param(host_alarm_rawtext)
         template_id: host_alarm
      - entity:
         category: ALARM
         type: vitrage
         name: get_param(instance_alarm_name)
         template_id: instance_alarm
      - entity:
         category: RESOURCE
         type: nova.instance
         template_id: instance
      - entity:
         category: RESOURCE
         type: nova.host
         template_id: host
     relationships:
      - relationship:
         source: host_alarm
         relationship_type: on
         target: host
         template_id : alarm_on_host
      - relationship:
         source: host
         relationship_type: contains
         target: instance
         template_id : host_contains_instance
      - relationship:
         source: instance_alarm
         relationship_type: on
         target: instance
         template_id : alarm_on_instance
   scenarios:
     - scenario:
        condition: alarm_on_host and host_contains_instance
        actions:
         - action:
             action_type: raise_alarm
             action_target:
               target: instance
             properties:
               alarm_name: get_param(instance_alarm_name)
               severity: get_param(instance_alarm_severity)
         - action:
             action_type: set_state
             action_target:
               target: instance
             properties:
               state: get_param(instance_state)
     - scenario:
        condition: alarm_on_host and host_contains_instance and alarm_on_instance
        actions:
         - action:
             action_type: add_causal_relationship
             action_target:
               source: host_alarm
               target: instance_alarm
     - scenario:
        condition: alarm_on_host
        actions:
         - action:
             action_type: set_state
             action_target:
               target: host
             properties:
               state: SUBOPTIMAL


CLI Example
-----------

.. code-block:: console

  vitrage template add --path ./host_affects_instance_prototype.yaml --param 'host_alarm_rawtext'='High CPU load on host' --param 'instance_alarm_name'='High CPU load on instance'

Or:

.. code-block:: console

  vitrage template add --path ./host_affects_instance_prototype.yaml --params 'host_alarm_rawtext'='High CPU load on host', 'instance_alarm_name'='High CPU load on instance'


Alternatives
------------

Hold a template prototype catalog
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Define templates of type ``prototype`` that will be added to Vitrage database
like other templates, but will be ignored by the evaluator.

There will be APIs to:

* Add a template prorotype
* View all template prototypes
* Add a template based on an existing prototype name

Advantages:

* At runtime, Vitrage will hold a template prototype catalog that can be used
  by the end users.

Disadvantages:

* Since users will not add new templates very often, the advantage is not very
  important.
* It will be harder to use the prototype ability in Heat, since the HOT
  template will have to depend on runtime information of Vitrage.
* We will have to decide how to handle prototype updates. The same HOT template
  will behave differently if the prototype in Vitrage has changed.

Implement this feature in Heat
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Instead of adding template prototypes in Vitrage, the prototyping option can be
quite easily implemented in Heat. The main disadvantage is, of course, that
the feature will not be available in Vitrage.

The entire Vitrage template can be written inside a Hot template, in one of two
ways:

1. As part of the Vitrage Template resource in Heat. The disadvantage is that
   the Heat resource will be tightly coupled with Vitrage, and any syntax
   change in Vitrage should be reflected in Heat.

2. As a comment inside the Vitrage Template resource in Heat. The disadvantage
   is that writing a long yaml file as a comment is less clear and may result
   in syntax and alignment errors.

Data model impact
-----------------

None. The template parameters will be given actual values, and standard
templates will be added to the database.

REST API impact
---------------

Parameters should be added to ``vitrage template add`` function call, if the
``path`` points to a template with parameters . All parameters that are
required in the template must be set.

There will be two ways to specify the parameters:

* ``param``: Followed by a key=value pair. There can be few ``param`` pairs.
* ``params``: Followed by a list of key=value parameters, separated by ",".

Versioning impact
-----------------

These changes will be part of Vitrage template version 3.

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

None.
In the future this feature may be used to write a template editor UI.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ifat_afek

Work Items
----------

* Introduce Vitrage template version 3
* Support template loading and parameters injection for template parameters
* Support template validation for templates with parameters
* New parameters for ``template add`` API
* Documentation and tests

Dependencies
============

None

Testing
=======

Unit tests, functional tests and tempest tests

Documentation Impact
====================

The new template format will be documented

References
==========

None
