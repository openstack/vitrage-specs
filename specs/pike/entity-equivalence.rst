..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Define and handle equivalence among entities
============================================

https://blueprints.launchpad.net/vitrage/+spec/entity-equivalence

Define equivalence among entities to allow mapping two or more entities to same
object and handle it properly in scenario evaluator. It is inspired by Idan
Hefetz's proposal about `ZTE use case of alarm deduction`_, but designed in a
generic way to allow extending to RESOURCE equivalence in future.

Problem description
===================

Introducing entity equivalence will enhance the extensibility of Vitrage,
making it suitable for more use cases, such as

- early `deduction`_ of alarm before it is reported and deal with the real
  alarm followed
- `aggregation`_ of equivalent alarms from multiple monitors
- aggregation of resource information from multiple data sources.

Proposed change
===============

Add a new file (or a set of files) that define equivalence between entities

.. code-block:: yaml

   metadata:
    name: entity equivalence example
   equivalences:
    - equivalence:
       - entity:
          category: ALARM
          type: nagios
          name: host_problem
       - entity:
          category: ALARM
          type: zabbix
          name: host_problem
       - entity:
          category: ALARM
          type: vitrage
          name: host_problem
     - equivalence:
       ...

These definitions will take effect globally, i.e. for every other template

The evaluator will duplicate every scenario for every equivalent alarm
automatically. For example, in case of the condition:: yaml

    condition: nagios_host_problem_on_host and host_contains_vm``

Two conditions will be created internally:: yaml

    condition: nagios_host_problem_on_host and host_contains_vm
    condition: zabbix_host_problem_on_host and host_contains_vm
    condition: vitrage_host_problem_on_host and host_contains_vm

The idea is that the user will write a single condition, and all equivalent
conditions will be created and evaluated automatically.

Equivalences should be defined explicitly. Including one entity in two or more
equivalence definition will result in implicit chaining, thus is considered
invalid. For example, if ``a eq b`` and ``b eq c`` are defined separately, it
will logically result in an implicit ``a eq c``. This will introduce unnecessary
complexity in creating templates and should be restricted in validator.

Alternatives
------------

Separate file vs embedded definition
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Instead of creating a separate file, we may embed the equivalence definitions in
templates by adding a new section ``equivalences``. Entities that are equivalent
to each other are grouped in arrays of their ``template_id``

.. code-block:: yaml

   metadata:
    name: entity equivalence example
   definitions:
    entities:
     - entity:
        category: ALARM
        type: nagios
        name: host_problem
        template_id: nagios_host_problem
     - entity:
        category: ALARM
        type: zabbix
        name: host_problem
        template_id: zabbix_host_problem
     - entity:
        category: ALARM
        type: vitrage
        name: host_problem
        template_id: vitrage_host_problem
     ...
    relationships:
     ...
    equivalences:
     - [nagios_host_problem, zabbix_host_problem, vitrage_host_problem]
   scenarios:
     ...

In this way, there will be fewer duplication of entity definitions.

However, given the fact that once an ``equivalent`` edge is added between two
alarms, then it *logically* means that they are equivalent in *all* other
templates as well. Even if they are not specified this way in the other
templates. Then template will be less clear without the equivalence information
embedded in it.

The duplication of entity definition might be resolved by implementing an
``import`` feature in other blueprint.

Adding equivalent edge vs not
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``equivalent`` edges could be created between every two equivalent alarms.
Since all related scenarios have been duplicated, This does not bring extra
value in the evaluator.

The ``equivalent`` edge could be useful for future evolution such as alarm
aggregation, UI optimization, alarm deduction. It may be implemented in those
blueprints.

Data model impact
-----------------

None

REST API impact
---------------

None

Versioning impact
-----------------

None

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

There are currently three views in ``vitrage-dashboard``

Topology view
^^^^^^^^^^^^^

No impact

RCA view
^^^^^^^^

More alarms and more ``causes`` edges

.. TODO:: (yujunz) include example graph

Entity graph
^^^^^^^^^^^^

- separate vertices for equivalent alarms (nagios, zabbix, vitrage)
- more edges (``equivalent`` and ``on``)

Summary
^^^^^^^

The impacts on RCA view and Entity graph will only be relevant to cases where
both ``equivalence`` and ``vitrage-dashboard`` are used. We will handle it in
future blueprints.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  yujunz

Other contributors:
  None

Work Items
----------

- validate and parse equivalence definition in templates
- duplicate scenarios in the scenario repository
- no changes in sub-graph matching or the evaluator

The following items are not in scope

- aggregation of equivalent alarms
- ``add-equivalent`` action
- support alarm equivalence in UI
- implement causal tree model for alarm deduction enhancement
- resource equivalence

Dependencies
============

None

Testing
=======

The implementation will be covered by additional unit test

Documentation Impact
====================

- documentation on how to define equivalence and when to use it
- declare limitation on resource equivalence
- list known issues when use ``equivalence`` with ``vitrage-dashboard``

References
==========

.. _ZTE use case of alarm deduction: https://goo.gl/FfDLi8
.. _deduction: https://review.openstack.org/#/c/423000/
.. _aggregation: https://blueprints.launchpad.net/vitrage/+spec/alarm-aggregation
