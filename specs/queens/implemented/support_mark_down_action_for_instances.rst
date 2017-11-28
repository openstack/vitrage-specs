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
should be with error status. Nova notifier will be extended to reset server
state.

Problem description
===================

As Vitrage works as the OPNFV Doctor Inspector component, when Vitrage
receives alarm notifications from the Doctor monitor, it should map the
physical resources to virtual resources and set their states appropriately.
For the host down scenario, currently Vitrage only support to mark host down,
the states of the servers which are launched at the host are still 'Ok' in
Nova. And also in a real scenario, notifying Nova would help the user to get
a clear picture of the state of its instances. So Nova notifier should be
extended to call 'reset-state' API to reset server state.

Proposed change
===============

Reuse 'mark_down' action type and set 'instance' as 'action_target' in Vitrage
template which will call Nova api: 'reset-state' to set instance state.

Doctor Example
---------------

.. code-block:: yaml

 - scenario:
    condition: host_down_alarm_on_host and host_contains_instance and alarm_on_instance
    actions:
     - action:
        action_type: mark_down
        action_target:
          target: instance

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

To use the Nova notifier, there is a need to define it in the Vitrage config
file, and in addition use the 'mark_down' action for instances in Vitrage
template.

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

- Implement the 'mark_down' action for instances and tests
- Modify the host_down_scenario template for calling Nova reset-state

Dependencies
============

None

Testing
=======

Unit tests and tempest tests need to be added.

Documentation Impact
====================

The usage of the 'mark_down' action for instances will be documented.


References
==========

`Doctor inspector design guideline <https://github.com/opnfv/doctor/blob/master/docs/development/design/inspector-design-guideline.rst>`_
`Support external actions in Vitrage templates <https://specs.openstack.org/openstack/vitrage-specs/specs/pike/external-actions.html>`_
