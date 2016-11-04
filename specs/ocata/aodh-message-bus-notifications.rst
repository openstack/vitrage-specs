..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================
Get Aodh Alarms From Message Bus Notifications
==============================================

Get Aodh alarms immediately via notification from the message bus.

Problem description
===================

Currently Aodh datasource queries Aodh alarms periodically. It is not in time
and it is also not efficient, since most query will get no change when the
interval of alarm evaluation in Aodh is larger than the interval of pulling
Aodh alarm in Vitrage.

Proposed change
===============

Aodh `update_method` in vitrage config file will be configured with:
/etc/vitrage/vitrage.conf
```
    [aodh]
    update_method = push

Add a new notification topic in Aodh config file:
/etc/aodh/aodh.conf
```
    [oslo_messaging_notifications]
    topics = vitrage_notifications

Vitrage listener will get the alarm events from the message bus. Aodh driver
will filter the needed event types, enrich the events and then send them to
the queue for the Aodh transformer to create, update or delete the entity 
vertex.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

None

Pipeline impact
---------------

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

performance improvement of getting Aodh alarms


Other deployer impact
---------------------

** Config `update_method` as 'push' in `Aodh` section in vitrage config file.
** Add the `vitrage_notifications` topic in `oslo_messaging_notifications`
   section in Aodh config file.

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  dongwenjuan <dong.wenjuan@zte.com.cn>

Work Items
----------

None

Future lifecycle
================

None

Dependencies
============

None

Testing
=======

Unit tests and tempest tests.

Documentation Impact
====================

Documentation will be modified to describe how to configure the notification
topics in Aodh when using devstack to deploy OpenStack env.

References
==========

None
