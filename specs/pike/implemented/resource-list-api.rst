..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================
Resource List API
=================

https://blueprints.launchpad.net/vitrage/+spec/resource-list-api

An API to list the resources with specified type or all the resources.

Problem description
===================

Currently Vitrage has the APIs for getting topology and alarms. But user may
want to get specified resources which he cares about.

Proposed change
===============
Add an API to list resources. If user specify the resource type, list the
resources with the given type.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

Resource List
^^^^^^^^^^^^^

Returns resource list

GET /v1/resources/
~~~~~~~~~~~~~~~~~~

Headers
^^^^^^^

-  X-Auth-Token (string, required) - Keystone auth token
-  Accept (string) - application/json
-  User-Agent (String)

Path Parameters
^^^^^^^^^^^^^^^

None.

Query Parameters
^^^^^^^^^^^^^^^^

None

Request Body
^^^^^^^^^^^^

* resource_type - (string, optional) the type of resource. defaults to return all resources.
* all_tenants - (boolean, optional) shows the resources of all tenants (in case the user has the permissions).

Request Examples
^^^^^^^^^^^^^^^^
::

    GET /v1/resources/
    Host: 127.0.0.1:8999
    User-Agent: keystoneauth1/2.3.0 python-requests/2.9.1 CPython/2.7.6
    Accept: application/json
    X-Auth-Token: 2b8882ba2ec44295bf300aecb2caa4f7

Response
~~~~~~~~

Status code
^^^^^^^^^^^

-  200 - OK
-  400 - Bad request

Response Body
^^^^^^^^^^^^^

Returns a list with all the resources requested.

Response Examples
^^^^^^^^^^^^^^^^^

::

  [
    {
      "category": "RESOURCE",
      "is_placeholder": false,
      "is_deleted": false,
      "name": "vm-1",
      "update_timestamp": "2015-12-01T12:46:41Z",
      "state": "ACTIVE",
      "project_id": "0683517e1e354d2ba25cba6937f44e79",
      "type": "nova.instance",
      "id": "dc35fa2f-4515-1653-ef6b-03b471bb395b",
      "vitrage_id": "RESOURCE:nova.instance:dc35fa2f-4515-1653-ef6b-03b471bb395b"
    }
  ]

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

None


Other deployer impact
---------------------

None

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

dong wenjuan <dong.wenjuan@zte.com.cn>


Work Items
----------

* Implement the API and tests
* Implement the client and tests

Future lifecycle
================

None

Dependencies
============

None

Testing
=======

Unit tests and tempest tests need to be added.

Documentation Impact
====================
The new api should be documented

References
==========
None
