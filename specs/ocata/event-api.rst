..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========
Event API
=========

https://blueprints.launchpad.net/vitrage/+spec/support-inspector-sb-api

This blueprint defines an API for sending events to Vitrage datasources.


Problem description
===================
As a reference implementation to OPNFV Doctor Inspector project, Vitrage has
to implement the Inspector SB API. This is a REST API that each monitoring
service should call in order to push events to the Inspector.


Proposed change
===============

New API will be added to allow sending events to the Vitrage datasources.
The events will be sent to Oslo message bus, and will be consumed by the one of
the datasource drivers according to the ``type`` property.

For example, the Doctor datasource will handle events of type ``compute.host.down``,
while the Nova instance datasource will handle events of type ``compute.instance.delete.start``.
(Using this API for Nova is not a real use case, but can be used for debugging).

For the Doctor datasource, the events will contain the details defined in the
Doctor specification. Future datasources may also use this API in order to send
their own events to Vitrage.


REST API impact
---------------

POST /v1.0/event/
^^^^^^^^^^^^^^^^^

Post an event to Vitrage message queue, to be consumed by a datasource driver.

Headers
"""""""

-  X-Auth-Token (string, required) - Keystone auth token
-  Accept (string) - application/json
-  User-Agent (String)
-  Content-Type (String): application/json

Path Parameters
"""""""""""""""

None.

Query Parameters
""""""""""""""""

None.

Request Body
""""""""""""

An event to be posted. Will contain the following fields:

- time: a timestamp of the event. In case of a monitor event, should specify when the fault has occurred.
- type: the type of the event.
- details: a key-value map of metadata.

A list of some potential details, copied from the Doctor SB API reference:

- hostname: the hostname on which the event occurred.
- source: the display name of reporter of this event. This is not limited to monitor, other entity can be specified such as ‘KVM’.
- cause: description of the cause of this event which could be different from the type of this event.
- severity: the severity of this event set by the monitor.
- status: the status of target object in which error occurred.
- monitorID: the ID of the monitor sending this event.
- monitorEventID: the ID of the event in the monitor. This can be used by operator while tracking the monitor log.
- relatedTo: the array of IDs which related to this event.

Request Examples
""""""""""""""""
::

    POST /v1/event/
    Host: 135.248.18.122:8999
    User-Agent: keystoneauth1/2.3.0 python-requests/2.9.1 CPython/2.7.6
    Content-Type: application/json
    Accept: application/json
    X-Auth-Token: 2b8882ba2ec44295bf300aecb2caa4f7


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

Response
""""""""

Status code
~~~~~~~~~~~

-  200 - OK
-  400 - Bad request

Response Body
~~~~~~~~~~~~~

Returns an empty response body if the request was OK.
Otherwise returns a detailed error message (e.g. 'missing time parameter').

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ifat-afek

Testing
=======

The changes will be tested by unit tests and tempest tests.

Documentation Impact
====================
The new api should be documented

References
==========

- https://wiki.opnfv.org/display/doctor/Doctor+Home
- http://artifacts.opnfv.org/doctor/docs/requirements/05-implementation.html
  section 4.5.6
- https://blueprints.launchpad.net/vitrage/+spec/doctor-datasource
