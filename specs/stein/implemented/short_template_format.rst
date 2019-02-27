..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================
Short Template Format
=====================

StoryBoard link: https://storyboard.openstack.org/#!/story/2004871

This spec suggests a shorter format for Vitrage template files.
The new format should be shorter, read faster and written more easily.

Problem description
===================

Vitrage template language is powerful, yet a bit verbose and it's quite easy
to make syntax mistakes when creating a new template from scratch.
We would like to make it easier to generate any template.

For more information on the current format, see Template_format_.

.. _Template_format: https://docs.openstack.org/vitrage/latest/contributor/vitrage-template-format.html

Proposed change
===============

Several simplifications that will result in a shorter template file
while preserving it's capabilities.

 1. Remove ``definitions`` level nesting, ``entities`` will directly replace it
 2. Remove ``relationships``, these will be represented inline in the condition,
    example usage ``condition: host_ssh_alarm [ on ] host``
 3. ``entities`` will be changed from a list to a dictionary, ``template_id``
    is removed and the entity key will be used instead.
 4. In ``actions`` the field ``action_type`` is removed and the action key will be used instead
 5. In each action ``action_target`` nesting level is removed
 6. In ``scenarios`` removed the ``scenario`` nesting level
 7. Action ``set_state`` will be replaced with ``set_suboptimal``, ``set_error`` and ``set_ok``.
 8. ``raise_alarm`` action will have a new optional field ``causing_alarm`` this will add a causal relationship
    between the new alarm and the causing alarm, de-necessitating the use of ``add_causal_relationship``


Example for a short format template
-----------------------------------

.. code-block:: yaml

   metadata:
    version: 3
    name: zabbix alarm for network interface and ssh affects host instances
    description: zabbix alarm for network interface and ssh affects host instances
   entities:
    host_network_alarm:
     type: zabbix
     rawtext: host network interface is down
    host_ssh_alarm:
     type: zabbix
     rawtext: host ssh is down
    instance:
     type: nova.instance
    host:
     type: nova.host
   scenarios:
    - condition: host_ssh_alarm [ on ] host
      actions:
        - set_suboptimal:
           target: host
    - condition: host_network_alarm [ on ] host AND host_ssh_alarm [ on ] host
      actions:
        - add_causal_relationship:
           source: host_network_alarm
           target: host_ssh_alarm
    - condition: host_ssh_alarm [ on ] host AND host [ contains ] instance
      actions:
        - raise_alarm:
           target: instance
           alarm_name: instance is down
           severity: WARNING
           causing_alarm: host_ssh_alarm
        - set_error:
           target: instance

Example for an equivalent template in the previous format
---------------------------------------------------------

.. code-block:: yaml

   metadata:
    version: 2
    type: standard
    name: zabbix alarm for network interface and ssh affects host instances
    description: zabbix alarm for network interface and ssh affects host instances
   definitions:
    entities:
     - entity:
      category: ALARM
      type: zabbix
      rawtext: host network interface is down
      template_id: host_network_alarm
     - entity:
      category: ALARM
      type: zabbix
      rawtext: host ssh is down
      template_id: host_ssh_alarm
     - entity:
      category: ALARM
      type: vitrage
      name: instance is down
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
      source: host_network_alarm
      relationship_type: on
      target: host
      template_id : network_alarm_on_host
     - relationship:
      source: host_ssh_alarm
      relationship_type: on
      target: host
      template_id : ssh_alarm_on_host
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
        condition: ssh_alarm_on_host
        actions:
         - action:
             action_type: set_state
             action_target:
               target: host
             properties:
               state: SUBOPTIMAL
    - scenario:
        condition: network_alarm_on_host AND ssh_alarm_on_host
        actions:
         - action:
             action_type: add_causal_relationship
             action_target:
               source: host_network_alarm
               target: host_ssh_alarm
    - scenario:
        condition: ssh_alarm_on_host AND host_contains_instance
        actions:
         - action:
             action_type: raise_alarm
             action_target:
               target: instance
             properties:
               alarm_name: instance is down
               severity: WARNING
         - action:
             action_type: set_state
             action_target:
               target: instance
             properties:
               state: ERROR
    - scenario:
        condition: ssh_alarm_on_host AND host_contains_instance AND alarm_on_instance
        actions:
         - action:
             action_type: add_causal_relationship
             action_target:
               source: host_ssh_alarm
               target: instance_alarm


Alternatives
------------

None.


Data model impact
-----------------

None.
Templates will be stored in the same data model.

REST API impact
---------------

None

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

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  idan_hefetz

Work Items
----------

* Introduce Vitrage template version 3
* Support template validation and loading
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
