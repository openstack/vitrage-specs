..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================================
Heat Datasource Driver - get_all implementation
===============================================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/heat-datasource-get-all

This blueprint describes the Heat driver for Vitrage Datasource, and its
implementation for get_all Stacks and Stack Resources.

Problem description
===================

Heat stacks should be added to Vitrage Graph via Vitrage Datasources.
This requires writing a Datasource driver for heat.

The driver should support two modes:

* get_all: query all Heat Stacks.
* notifications: notify the datasource upon a change in a stack definition or state

This blueprint refers to get_all implementation.

Proposed change
===============

Heat driver will be configured with:

* Poll interval in seconds (default: 600)

Every poll-interval seconds, Heat Driver will call Heat list all stacks API. The stacks will be converted to Vitrage datasource events and passed to Vitrage Graph queue.


Alternatives
------------

None

Data model impact
-----------------

Heat event will be sent to Vitrage Graph queue with the following properties:

+------------------+----------------------------------------------------------+-----------------------------------------------------+
| Field            | Description                                              | Examples                                            |
+==================+==========================================================+=====================================================+
| stack_id         | The stack UUID                                           | e47e1be6-3598-46e6-bb63-0cc9a4e35ad7                |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| name             | The stack name                                           | mystack                                             |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| description      | The stack description                                    | mydescription                                       |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| user_project_id  | The ID of the tenant that owns the alarm                 | 5542b27142154f30b32dea6238aa81aa                    |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| owner            | The ID of the user that owns the alarm                   | 5555527142154f30b32dea6238aa81aa                    |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| status           | The stack status                                         | in progress / failed                                |
+------------------+----------------------------------------------------------+-----------------------------------------------------+
| status_reason    | The stack status reason                                  | Stack create started ....                           |
+------------------+----------------------------------------------------------+-----------------------------------------------------+




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

Heat driver should be configured

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
  dan-offek

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

Datasource main blueprint:
https://github.com/openstack/vitrage-specs/blob/master/specs/mitaka/vitrage-datasource.rst
