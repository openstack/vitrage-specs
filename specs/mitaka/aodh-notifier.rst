..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============
Aodh Notifier
=============

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/aodh-notifier

The Evaluator performs root cause analysis on the Vitrage Graph and may determine that an alarm should be created, deleted or otherwise updated.
Other components are notified of such changes by the Vitrage Notifier service. Among others, Vitrage Notifier is responsible for handling Aodh Alarms.

This blueprint describes the implementation of Vitrage Notifier for notifying Aodh on Vitrage alarms.

::

        +------------------+           +------------------+          +------------------+
        |   Aodh           <--+        |                  |          |                  |
        +------------------+  | Update |      Vitrage     |  Raise   |      Vitrage     |
                              +--------|                  <----------|                  |
        +------------------+  | Alarm  |      Notifier    |  Alarm   |      Evaluator   |
        | Other components <--+        |                  |          |                  |
        +------------------+           +------------------+          +------------------+


Problem description
===================

Vitrage should be capable of creating, deleting and otherwise updating alarms as requested by the Evaluator Engine.
The notifier is responsible for ensuring these updates are executed. Specifically we will start here with Aodh alarms.

Main challenges:

* There is no way to define a 'custom alarm' in Aodh
* Vitrage alarms are based on resources. There is a need to pass the resource information to Aodh
* Several alarms of the same type can be triggered at the same time, each for a different resource. For example, in case there is an alarm on a host, Vitrage will raise a deduced alarm on every instance in this host.
* How can someone ask for notifications on updates of Vitrage alarms?


Proposed change
===============

The Vitrage Notifier will be separate from the Evaluator, as the two will have different demands of scale and other performance considerations.
The Vitrage Notifier will supply an API used by the Vitrage Evaluator, containing create/delete/update alarm.

In Aodh, Vitrage alarms will be defined as event alarms, this seems like the most appropriate option. The resource id will be defined in the alarm query.

Vitrage deduced alarms will look like this:

+---------------------------+---------------------------------------------------------+
| Property                  | Value                                                   |
+---------------------------+---------------------------------------------------------+
| alarm_actions             | []                                                      |
+---------------------------+---------------------------------------------------------+
| alarm_id                  | 4a3cb988-a620-4bf3-87f7-077c751c408f                    |
+---------------------------+---------------------------------------------------------+
| description               | Instance is unreachable                                 |
+---------------------------+---------------------------------------------------------+
| enabled                   | True                                                    |
+---------------------------+---------------------------------------------------------+
| event_type                | vitrage.alarm.instance_unreachable                      |
+---------------------------+---------------------------------------------------------+
| insufficient_data_actions | []                                                      |
+---------------------------+---------------------------------------------------------+
| name                      | vitrage_instance_unreachable_1                          |
+---------------------------+---------------------------------------------------------+
| ok_actions                | []                                                      |
+---------------------------+---------------------------------------------------------+
| project_id                | 5542b27142154f30b32dea6238aa81aa                        |
+---------------------------+---------------------------------------------------------+
| query                     | [{field': 'resource_id', 'type': '', 'value':           |
|                           | 'b0bf3635-d9e8-4624-9793-7aac82948c0a', 'op': 'eq'}]    |
+---------------------------+---------------------------------------------------------+
| repeat_actions            | False                                                   |
+---------------------------+---------------------------------------------------------+
| severity                  | moderate                                                |
+---------------------------+---------------------------------------------------------+
| state                     | alarm                                                   |
+---------------------------+---------------------------------------------------------+
| type                      | event                                                   |
+---------------------------+---------------------------------------------------------+
| user_id                   | 8ab65ef808b245e3ba234b7b3554cb94                        |
+---------------------------+---------------------------------------------------------+

In this example, Vitrage triggers a deduced alarm that an instance is unreachable due to a failure in the public switch (which was detected by Nagios).
There will be several alarms with the same event_type and different instance ids in their query.


There are two options how to trigger Vitrage alarms in Aodh, none is perfect.


Alternative 1
-------------

Vitrage will create an event alarm in Aodh.
Then, it will send a notification to the message bus. The notification will be converted to a Ceilometer event, which will trigger the Aodh alarm.

The exact notification and event format are still TBD.

The main problem with this solution is that the Aodh alarm will be created on-the-fly and triggered immediately, so it will be impossible for another project to register a web-hook on the alarm before it is triggered.
It will be possbile to see Vitrage alarms in list-alarms, but not to be notified when they are first triggered.


Alternative 2
-------------

Vitrage will create an event alarm in Aodh, with 'alarm' state. The event itself will never be sent, so the alarm state will remain 'alarm'.

The problem with this solution is that Aodh will not send a notification about the alarm being triggered. But since in Alternative 1 it is also impossible to register on the alarm, there is no real difference between the two options.


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

For Alternative 1 - there is a need to define the notification->event configuration

For Alternative 2 - None

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
  idan-hefetz

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

For Alternative 1 - there is a need to document the notification->event configuration

For Alternative 2 - None

References
==========

Vitrage wiki page: https://wiki.openstack.org/wiki/Vitrage

Vitrage use cases: https://github.com/openstack/vitrage/blob/master/doc/source/vitrage-use-cases.rst
