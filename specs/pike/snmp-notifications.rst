..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================
SNMP Notifications
==================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/snmp-notifications

The Evaluator performs root cause analysis on the Vitrage Graph and may
determine that an alarm should be created, deleted or otherwise updated.
Other components are notified of such changes by the Vitrage Notifier service.
Among others, one Vitrage Notifier is responsible for sending SNMP traps on
Vitrage deduced alarms.

This blueprint describes the implementation of Vitrage Notifier for notifying
SNMP on Vitrage deduced alarms.


Problem description
===================

Vitrage should support registering for SNMP notifications, and sending traps
on raised alarms and deactivated alarms to any registered targets.


Proposed change
===============

Due to definition in Vitrage config file::

 [DEFAULT]
 notifiers = snmp

Vitrage listener will get the alarm events from the message bus and the SNMP
notifier will send SNMP traps on raised deduced alarms and deleted deduced alarms.

The traps will be sent to destinations specified in consumers yaml file.

The traps will be sent only on alarms specified in yaml file which contains
oid mapping for each alarm name.

The format of sent traps will be specified in another yaml file.

All those yaml files' paths should be specified in Vitrage config::

 [snmp]
 consumers = <path to consumers yaml file>
 alarm_oid_mapping = <path to alarm oid mapping yaml file>
 oid_tree = <path to tree format oid configuration yaml file>

The SNMP notifier is pluggable, you can implement your own SNMP sender and use
it (it must inherit from the base class), when there is a default implementation.

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

To use the SNMP notifier there is a need to define it in the Vitrage config
file, and in addition create three yaml files and define them in Vitrage config file.

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
  annarez

Work Items
----------

- Create SNMP notifier

  - Create SNMP sender

    - create base class
- Create unit test for SNMP sender

  - test snmp notifier
  - test snmp sender

Dependencies
============

None

Testing
=======

This blueprint requires unit tests.

Documentation Impact
====================

The usage of the SNMP notifier will be documented


References
==========

`notifier-snmp-plugin.rst <https://github.com/openstack/vitrage/blob/master/doc/source/notifier-snmp-plugin.rst>`_
