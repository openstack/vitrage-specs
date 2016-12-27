This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================
Collectd Data Source
====================

https://blueprints.launchpad.net/vitrage/+spec/collectd-datasource

This blueprint describes the datasource that will receive notifications from
collectd.

Problem description
===================
Vitrage should be able to accept a collectd notification.


Proposed change
===============
The Collectd datasource will receive notifications in the following format:

::

    {
        "host": "compute-1",
        "plugin": "ovs_events",
        "plugin_instance": "br-ex",
        "type": "gauge",
        "type_instance": "link_status",
        "message": "link state of "br-ex" interface has been changed to "WARNING,"",
        "severity": "WARNING",
        "time": 1482409029.062524,
        "id": "46c7eba7753efb0e6f6a8de24c949c52"
    }


Upon receiving such a notification, the Collectd datasource will create a
corresponding alarm in Vitrage. When receiving an ok
notification, the alarm will be deleted.

In addition, a new evaluator template will be added in order to:
 - Create deduced alarms on the VMs running on the host
 - Modify the states of the host and the VMs to ERROR

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  eyal bar ilan

Work Items
----------

- Implement the Collectd datasource
- Write a template for creating deduced alarms on the VMs and calling Nova
  mark host down

Testing
=======

The changes will be tested by unit tests

References
==========

- https://collectd.org/
