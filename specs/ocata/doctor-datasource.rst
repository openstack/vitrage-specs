..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================
Doctor Data Source
==================

https://blueprints.launchpad.net/vitrage/+spec/doctor-datasource

This blueprint describes the datasource that will receive notifications from
OPNFV Doctor monitor

Problem description
===================
In order for Vitrage to be accepted as a reference implementation for the
OPNFV Doctor Inspector component, it should be able to receive alarm
notifications from the Doctor monitor.

Proposed change
===============
The Doctor datasource will receive notifications in the format defined by
Doctor SB API (see the reference below):

::

    {
        'event': {
            'time': '2016-04-12T08:00:00',
            'type': 'compute.host.down',
            'details': {
                'hostname': 'compute-1',
                'source': 'sample_monitor',
                'cause': 'link-down',
                'severity': 'critical',
                'status': 'down',
                'monitor_id': 'monitor-1',
                'monitor_event_id': '123',
            }
        }
    }


Upon receiving such a notification, the Doctor datasource will create or
delete a corresponding alarm in Vitrage, based on the 'status' field.

In addition, a new evaluator template will be added in order to:
 - Create deduced alarms on the VMs running on the host
 - Modify the states of the host and the VMs to ERROR
 - Call Nova force-down API to mark that the host is down


REST API impact
---------------

The Doctor monitor sends its alarms in REST format. Another blueprint discusses
the SB API that should be added to Vitrage in order to support it.
See https://blueprints.launchpad.net/vitrage/+spec/support-inspector-sb-api


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ifat-afek

Work Items
----------

- Implement the Doctor datasource
- Write a template for creating deduced alarms on the VMs and calling Nova
  mark host down

Testing
=======

The changes will be tested by unit tests, and later on also by Doctor test
scripts.

References
==========

- https://wiki.opnfv.org/display/doctor/Doctor+Home
- http://artifacts.opnfv.org/doctor/docs/requirements/05-implementation.html
  section 4.5.6

