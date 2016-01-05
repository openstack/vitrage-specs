..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================================================
Nagios Synchronizer Plugin - get_all implementation
===================================================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/synchronizer-nagios-get-all

This blueprint describes the Nagios plugin for Vitrage Synchronizer, and its
implementation for get_all nagios services (tests).

Problem description
===================

Nagios test results should be added to Vitrage Graph via Vitrage Synchronizer.
This requires writing a Synchronizer plugin for Nagios.

The plugin should support two modes:

* get_all: query the results of all Nagios tests and send corresponding events to the Synchronizer
* notifications: notify the Synchronizer upon a change in one of Nagios test results

This blueprint refers to get_all implementation.

Proposed change
===============

The current Nagios plugin will support Nagios 3.x.

Nagios plugin will be configured with:

* Nagios URI, for example:
  http://10.45.1.10/monitoring/nagios/cgi-bin/status.cgi?host=all&limit=0
* Nagios credentials
* Poll interval in seconds (default: 60)

Every poll-interval seconds, Nagios plugin will call Nagios URI to query the
results of Nagios services (tests). It will parse the returned html (there is
no REST API for Nagios 3.x), create Nagios events and send them to Vitrage
Graph queue.


Alternatives
------------

None

Data model impact
-----------------

Nagios event will look like that:

+-------------+--------------------------------------+------------------------+
| Field       | Description                          | Examples               |
+=============+======================================+========================+
| host        | name of Nagios host                  | - compute-0-0.local    |
|             |                                      | - os-glance-00.local   |
|             |                                      | - ilo.node14           |
+-------------+--------------------------------------+------------------------+
| service     | name of Nagios service (test)        | - CPU load             |
|             |                                      | - check_ceph_health    |
+-------------+--------------------------------------+------------------------+
| status      | the status of the service            | - OK                   |
|             |                                      | - WARNING              |
|             |                                      | - CRITICAL             |
|             |                                      | - UNKNOWN              |
+-------------+--------------------------------------+------------------------+
| last_check  | last time the service was checked    | 2016-01-04 19:17:10    |
+-------------+--------------------------------------+------------------------+
| duration    | duration since the last status change| 1d 2h 55m 48s          |
+-------------+--------------------------------------+------------------------+
| attempt     | how many attempts were made          | 1/3                    |
+-------------+--------------------------------------+------------------------+
| status_info | additional information               | OK - 15min load 1.66   |
|             |                                      |      at 32 CPUs        |
+-------------+--------------------------------------+------------------------+



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

Nagios plugin should be configured with Nagios URI, credentials and poll
interval.

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

None

References
==========

Synchronizer main blueprint:
https://github.com/openstack/vitrage-specs/blob/master/specs/mitaka/vitrage-synchronizer.rst
