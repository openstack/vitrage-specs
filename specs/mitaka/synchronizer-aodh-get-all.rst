..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================
Aodh Synchronizer Plugin - get_all implementation
=================================================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/synchronizer-aodh-get-all

This blueprint describes the Aodh plugin for Vitrage Synchronizer, and its
implementation for get_all Aodh alarms.

Problem description
===================

Aodh alarms should be added to Vitrage Graph via Vitrage Synchronizer.
This requires writing a Synchronizer plugin for Aodh.

The plugin should support two modes:

* get_all: query all Aodh alarms and send corresponding events to the Synchronizer
* notifications: notify the Synchronizer upon a change in an Aodh alarm definition or state

This blueprint refers to get_all implementation.

Proposed change
===============

Aodh plugin will be configured with:

* Poll interval in seconds (default: 60)

Every poll-interval seconds, Aodh plugin will call Aodh list all alarms API. The alarms will be converted to Vitrage Synchronizer events and passed to Vitrage Graph queue.


Alternatives
------------

None

Data model impact
-----------------

Aodh event will be sent to Vitrage Graph queue with the following properties:

+------------------+----------------------------------------------------------+-----------------------------------------------------+
| Field            | Description                                              | Examples                                            |
+==================+==========================================================+=====================================================+
| alarm_id         | The alarm UUID                                           | e47e1be6-3598-46e6-bb63-0cc9a4e35ad7                |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| name             | The alarm name                                           | CpuUtilAlarm1                                       |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| description      | The alarm description                                    | Alarm when cpu is eq a avg of 300.0 over 60 seconds |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| enabled          | Is the alarm enabled                                     | true/false                                          |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| project_id       | The ID of the project or tenant that owns the alarm      | 5542b27142154f30b32dea6238aa81aa                    |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| repeat_actions   | Should actions be re-triggered on each evaluation cycle  | true/false                                          |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| severity         | The alarm severity                                       | low/moderate/critical                               |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| state            | The alarm state                                          | ok/alarm/insufficient data                          |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| time_constraints | Optional. Evaluate the alarm within this time constraint | ?                                                   |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| type             | The alarm type                                           | threshold/event                                     |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| user_id          | The ID of the user who created the alarm                 | 8ab65ef808b245e3ba234b7b3554cb94                    |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| resource_id      | Optional. The resource id in the query of the alarm rule | b0bf3635-d9e8-4624-9793-7aac82948c0a                |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| sync_type        | The source of information                                | aodh (constant value)                               |
+------------------+----------------------------------------------------------+-----------------------------------------------------+


Aodh threshold alarms will include in addition:

+------------------+----------------------------------------------------------+-----------------------------------------------------+
| Field            | Description                                              | Examples                                            |
+==================+==========================================================+=====================================================+
| state_timestamp  | The date of the last alarm state changed                 | 2016-03-01T17:13:08.847812                          |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| timestamp        | The date of the last alarm definition update             | 2016-03-01T17:13:08.847812                          |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| threshold_rule   | The threshold rule, for a threshold alarm                | dictionary                                          |
+------------------+----------------------------------------------------------+-----------------------------------------------------+


Aodh event alarms will include in addition:

+------------------+----------------------------------------------------------+-----------------------------------------------------+
| Field            | Description                                              | Examples                                            |
+==================+==========================================================+=====================================================+
| event_type       | Event type for event alarm                               | compute.instance.update                             |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| timestamp        | Does not exist in Aodh? current time will be used        | 2016-03-01T17:13:08.847812                          |
+------------------+----------------------------------------------------------+-----------------------------------------------------+


Gnocchi alarms, combination alarms and composite alarms are still TBD.

TBD: how this information will be transformed to a Vitrage Graph vertex


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

Aodh plugin should be configured

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

None

Dependencies
============

None

Testing
=======

This blueprint requires unit tests and Tempest tests.

Documentation Impact
====================

Aodh Configuration should be documented

References
==========

Synchronizer main blueprint:
https://github.com/openstack/vitrage-specs/blob/master/specs/mitaka/vitrage-synchronizer.rst
