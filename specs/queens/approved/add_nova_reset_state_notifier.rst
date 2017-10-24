..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================
Add Nova reset-state notifier
=============================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/vitrage/+spec/add-nova-reset-state-notifier

When a host is marked as down, all the servers which are launched at the host
should be with error status. A new notifier will be added to call Nova API to
reset server state.

Problem description
===================

As Vitrage works as the OPNFV Doctor Inspector component, when Vitrage
receives alarm notifications from the Doctor monitor, it should map the
physical resources to virtual resources and set their states appropriately.
For the host down scenario, currently Vitrage only support to mark host down,
the states of the servers which are launched at the host are still 'Ok' in
Nova. So a new notifier should be added to call Nova API to reset server state.


Proposed change
===============

Add a 'execute_nova' action that under host down conditions in Vitrage
template which will call Nova api: 'reset-state' to set instance state.

Examples
--------

.. code-block:: yaml

 - scenario:
    condition: host_down_alarm_on_host and host_contains_instance and alarm_on_instance
    actions:
     - action:
        action_type: execute_nova
　　　　　properties:
          api_call: reset-state
          parameters:
            instance: instance1
            state: error

Alternatives
------------

None

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

To use the Nova notifier there is a need to define it in the Vitrage config
file, and in addition add the host_down_scenario template in Vitrage.

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
  dong wenjuan <dong.wenjuan@zte.com.cn>

Work Items
----------

- Implement the nova reset-state notifier and tests
- Modify the host_down_scenario template for calling Nova reset-state

Dependencies
============

None

Testing
=======

Unit tests and tempest tests need to be added.

Documentation Impact
====================

The usage of the nova reset-state notifier will be documented.


References
==========

`Doctor inspector design guideline <https://github.com/opnfv/doctor/blob/master/docs/development/design/inspector-design-guideline.rst>`_
`Support external actions in Vitrage templates <https://specs.openstack.org/openstack/vitrage-specs/specs/pike/external-actions.html>`_
