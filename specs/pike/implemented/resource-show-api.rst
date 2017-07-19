..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================
Resource show API
=================

https://blueprints.launchpad.net/vitrage/+spec/resource-show-api

An API to show the details of the specified resource.

Problem description
===================

As a user, I want to get the details of a specified resource.

Proposed change
===============

Add an API to show the details of specified resource.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

Resource show
^^^^^^^^^^^^^

Returns details of the resource

GET /v1/resources/vitrage_id
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Headers
^^^^^^^

-  X-Auth-Token (string, required) - Keystone auth token
-  Accept (string) - application/json
-  User-Agent (String)

Path Parameters
^^^^^^^^^^^^^^^

- vitrage_id

Query Parameters
^^^^^^^^^^^^^^^^

None

Request Body
^^^^^^^^^^^^

None

Request Examples
^^^^^^^^^^^^^^^^
::

    GET /v1/resources/`<vitrage_id>`
    Host: 127.0.0.1:8999
    User-Agent: keystoneauth1/2.3.0 python-requests/2.9.1 CPython/2.7.6
    Accept: application/json
    X-Auth-Token: 2b8882ba2ec44295bf300aecb2caa4f7

Response
~~~~~~~~

Status code
^^^^^^^^^^^

-  200 - OK
-  404 - Not Found

Response Body
^^^^^^^^^^^^^

Returns details of the requested resource.

Response Examples
^^^^^^^^^^^^^^^^^

::

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
