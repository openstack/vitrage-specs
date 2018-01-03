..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========
Webhooks
========

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/configurable-notifications

The Evaluator performs root cause analysis on the Vitrage Graph and may
determine that an alarm should be created, deleted or otherwise updated.
Other components are notified of such changes by the Vitrage Notifier service.
Among others, Vitrage Notifier is responsible for sending http post
notifications on Vitrage deduced alarms.

This blueprint describes the implementation of Vitrage Notifier for
webhooks on Vitrage alarms and state changes.


Problem description
===================

Vitrage should support webhooks for notfications, which are sent on raised
alarms, deactivated alarms, state changes, RCA or other to any
registered targets.
Furthermore any registered recipient should supply a regex to filter the alarms
sent to that recipient.


Proposed change
===============

Needed definitions in Vitrage config file::

 [DEFAULT]
 notifiers = webhook

Vitrage listener will get the alarm events from the message bus and the webhook
notifier will send http post notifications on raised deduced alarms and deleted deduced alarms.

The filtered notifications will be sent to the destinations that are written in
the database, as configured via API requests.

The notifications will be sent only on alarms which meet the regex filter specified in the
webhook specification.

The format of sent notifications will be hard coded.

As Vitrage notifiers are pluggable, you can write your own notifier and use it.
Specifically in this case, you can inherit the webhook base class and implement your own webhook notifier.



Alternatives
------------

None

Data model impact
-----------------

New DB table, to represent registration details
Preliminary columns :

- ID

- Date

- Address

- Headers

- Filter

REST API impact
---------------

An API which supports adding, removing and listing webhooks

Versioning impact
-----------------

None

Other end user impact
---------------------

None

Deployer impact
---------------

To use webhooks one needs to define it in the Vitrage config file.

Developer impact
----------------

None

Horizon impact
--------------

Future support for webhooks in Horizon

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  nivo

Work Items
----------
- Add DB table
- Add API
- Implement notifier
- Update docs
- Tests

Dependencies
============

None

Testing
=======

This blueprint requires tempest tests and unit tests.

Documentation Impact
====================

The usage of webhooks will be documented


References
==========

Example on http post notifications in AODH
`http post request <https://github.com/openstack/aodh/blob/master/aodh/notifier/rest.py#L60-L109>`_
