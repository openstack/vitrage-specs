..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============
Synchronizer
============


https://blueprints.launchpad.net/vitrage/+spec/synchronizer

The synchronizer is a service which supplies both a snapshot(all the entities)
of openstack services (such as HEAT/NOVA/Cinder and so on) via poll requests,
or, change notifications for entities which comprise the same services,
via push notifications.
Each synchronizer instance has the capability of sampling any of these
openstack services at the same time, and vitrage might work against several
of these synchronizers for scaling purposes.

::

 +------+                      +------------------+
 |      |             +-----------+               |
 |      +---------+---> Heat      |               |
 |      |         |   | Plugin    |               |
 |      |         |   +-----------+               |
 |      |         |            |                  |                                 get all
 | HEAT |         |   +--------+--+               | <---------------------------------------------------------------------------+-------------------+
 |      |         |   | Cinder    |               |                                                                             |                   |
 |      |  +----------> Plugin    |   Synchronizer|                                                                             |                   |
 |      |  |      |   +--------+--+               |                                                                             |                   |
 |      |  |      |            |                  |                         +----------------------------+                      |       Vitrage     |
 |      |  |      |   +--------+--+               |                         |                            |                      |                   |
 +------+  |      |   | Nova      |               +----------------------------->                     <-------------------------+                   |
           |   +------> Plugin    |               |       HEAT, Cinder,     |                            |      register for    +-------------------+
 +------+  |   |  |   +--------+--+               |       Nova, etc.        |                            |      change
 |      |  |   |  |            |                  |       CRUD              |                            |      notifications
 |      |  |   |  |            +------------------+       notifications     |                            |      over entities
 |      |  |   |  |                                                         |            Queue           |      of a specific type
 |      |  |   |  |            +-----------------------+                    |                            |      (topic per type)
 |Cinder|  |   |  |            |                       |                    |                            |
 |      |  |   |  |   +--------+--+                    |                    |                            |
 |      |  |   |  +---> Heat      |                    +------------------------>                        |
 |      |  |   |      | Plugin    |                    | HEAT, Cinder,      |                            |
 |      +--+   |      +--------+--+                    | Nova, etc.         |                            |
 +------+  |   |               |                       | CRUD               +----------------------------+
           |   |      +--------+--+                    | notifications
 +------+  |   |      | Cinder    |   Synchronizer     |
 |      |  +----------> Plugin    |                    |
 |      |      |      +--------+--+                    |
 |      |      |               |                       |
 | Nova |      |      +--------+--+                    |
 |      +------+------> Nova      |                    |
 |      |             | Plugin    |                    |
 |      |             +--------+--+                    |
 +------+                      |                       |
                               +-----------------------+


 Alternative diagram

 ::


                                                                                                                                  +-------------+
                                            +----------+             +--------------+       get all response                      |             |
                                            | entity   |             |              +------------------------------------------------>          |
                                     +------+ type x <---------------+            <-----------------------------------------------+ Vitrage     |
                                     |      |          | polling via |              |   get all request          +----------+     | graph       |
    +-----------------------------+  |      +----------+ api or via  |              |                            | queue    |     |             |
    | Openstack services:         |  |                   message     | Synchronizer |                            |          |     +-------------+
    | Keystone, Nova,           <----+      +----------+ queue       |            <------------------------------+     ^    |
    | Neutron, Glance,            |         | entity   |             |              |  register for change       +----------+
    | Cinder, Heat              <-----------+ type y <---------------+              |  notifications over              |
    |                             |         |          |             |              |  entities of a specific          |
    | other polling mechanisms:   |         +----------+             |              |  type                            |
    | Nagios                    <----+                               |              |                                  |
    | Ceilometer                  |  |      +----------+             |              |        entity changes            |
    |                             |  |      | entity   |             |            +------------------------------------+
    |                             |  +------+ type z <---------------+              |
    |                             |         |          |             +--------------+
    +-----------------------------+         +----------+






Problem description
===================

Enable Vitrage and to maintain the most up-to-date view of OS services it samples.

Proposed change
===============

periodically samples OS services and produces up to date entities once changed (in contrast to deltas)



design
------

northbound interface
^^^^^^^^^^^^^^^^^^^^

get all
"""""""

- return the whole latest snapshot
- through polling
- using RPC
- interface signature - **TBD**

change notifications
""""""""""""""""""""

- interface signature - **TBD**
- push notifications
- sent to:
    - openstack message bus (if installed on openstack)
    - or to a queue (which would be installed as part of the synchronizer deployment)

southbound interface
^^^^^^^^^^^^^^^^^^^^

::

                                                          +
                          ^                               |
                          |                               |
 +------------------------------------------------------------------------------------------------+
 |                        |          synchronizer         |                                       |
 |                        |                               |get all                                |
 |                        |                               |                                       |
 |                        |                               |                                       |
 |                        |                         +-----v-----+                                 |
 |                        |                         | subprocess|                                 |
 |                        |                         |           |                                 |
 |                        |                         |           |                                 |
 |                        | notification            |           |                                 |
 |                        |                         +---+-------+                                 |
 |                        |                             |                                         |
 | +----------------------|-----------------------------|-------+  +-----------+  +------------+  |
 | |                      |   plugin                    |       |  | plugin    |  |  plugin    |  |
 | |                      |                             |       |  |           |  |            |  |
 | |                      |                             |       |  |           |  |            |  |
 | | +--------------------+-------------------------+   |get    |  |           |  |            |  |
 | | |                                              |   |all    |  |           |  |            |  |
 | | |            subprocess                        |   |       |  |           |  |            |  |
 | | |                                              |   |       |  |           |  |            |  |
 | | |                                              |   |       |  |           |  |            |  |
 | | |           +----------------------------+     |   |       |  |           |  |            |  |
 | | |  baseline:|hash0|hash1|hash2|hash3|... |     |   |       |  |           |  |            |  |
 | | |           +----------^-----------------+     |   |       |  |           |  |            |  |
 | | |                      |if                     |   |       |  |           |  |            |  |
 | | |                      |hash(item)!=hash1 ==>  |   |       |  |           |  |            |  |
 | | |                      |propagate item as a    |   |       |  |           |  |            |  |
 | | |                      |change notification    |   |       |  |           |  |            |  |
 | | |                      |                       |   |       |  |           |  |            |  |
 | | |          +-----------+---------------+       |   |       |  |           |  |            |  |
 | | |          |sampling worker (one/more?)|       |   |       |  |           |  |            |  |
 | | |          +----------^----------------+       |   |       |  |           |  |            |  |
 | | |                     |                        |   |       |  |           |  |            |  |
 | | |                     |                        |   |       |  |           |  |            |  |
 | | +---------------------|------------------------+   |       |  |           |  |            |  |
 | |                       |                            |       |  |           |  |            |  |
 | +-----------------------|----------------------------|-------+  +-----------+  +------------+  |
 |                         |                            |                                         |
 +-------------------------|----------------------------|-----------------------------------------+
                           |                            |
               +-----------+-------------------+        |
               |      OS service               |        |
               |                               <--------+
               +-------------------------------+

- sampling of OS services
- via each OS service REST API
- method of collection:
    - retrieve OS service elements list. For the purposes of:
        - 'get all' - as a response for a 'get all' northbound interface call
            - run on its own subprocess, separated from the collecting sub processes (cont.)
        - change notifications - as part of the periodic collection of the latest snapshot
            - we'd use this method of collection against OS services which doesn't propagates change notification.
            - run on its own collection sub processes (**one/more? - TBD**)
            - how to discover a change
                - once a snapshot is collected, we'd like to know which entity was changed from the latest time a snapshot was taken
                - in order to enable this, we'd keep a baseline - a data structure which contains for each OS service entity (such as a vm instance), its ID + it latest collection timestamp or a hash which represents its latests state.
                - by comparing the latest snapshot of elements against the baseline, we'd know for which entity we'd like to propagate a notification for.
    - collect deltas for services which reveals this functionality
        - for the purposes of change notifications
        - this is the easy case, where change notifications are simply passed on to whoever registered for them
- type of collection:
    - against each OS service we'd like to sample, a plugin library would enable BOTH the collection sub processes and the 'get all' method to retrieve BOTH a complete snapshot of the OS service (all the VM instances / all of the ports / etc.) AND to sample for change notifications, as described above.
- deployment - as a library

consumer flow
^^^^^^^^^^^^^
- 'get all' - for a complete snapshot of OS services, the consumer would call the 'get all' interface
- 'change notification' - for a streaming change notifications, the consumer would normally:
    - register for change notifications against the queue
    - then immediately call the 'get all' interface to have the latest snapshot
    - over time, exercise each notification which was sampled after this snapshot, over it, in order to have the latest view of the OS services


Alternatives
------------

None

Data model impact
-----------------

**TBD**

REST API impact
---------------

**TBD**

Versioning impact
-----------------

**TBD**

Other end user impact
---------------------

- 'get all' - **TBD**
- 'change notification' - **TBD**


Deployer impact
---------------

**TBD**

Developer impact
----------------

**TBD**

Horizon impact
--------------

None


Implementation
==============

Assignee(s)
-----------

**TBD**

Work Items
----------

**TBD**


Dependencies
============

**TBD**


Testing
=======
Unit Tests - Tox
Integration Tests - Tempest


Documentation Impact
====================

**TBD**


References
==========

**TBD**