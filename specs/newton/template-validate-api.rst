..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Vitrage Get Topology API
========================

https://blueprints.launchpad.net/vitrage/+spec/template-validate-api

An API for validating templates

Problem description
===================

We would like to be able to validate a single template (or several templates)
through api before uploading it to Vitrage.

Proposed change
===============
Create API to validate Vitrage templates in terms of content and syntax.

#. By given a full path to template file, validate a single template.
#. By given a full path to directory, validate all template files inside it.

The template validate API returns a result that contains the following fields:

#. status - validation succeeded/failed
#. file path - the full path to the template file
#. description
#. message - error message
#. status code

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

Template Validate
^^^^^^^^^^^^^

Validate Vitrage template(s)

GET /
~~~~~

Headers
^^^^^^^

-  X-Auth-Token (string, required) - Keystone auth token
-  Accept (string) - application/json
-  User-Agent (String)
-  Content-Type (String): application/json

Path Parameters
^^^^^^^^^^^^^^^

None.

Query Parameters
^^^^^^^^^^^^^^^^
-  path (string(255), required) - the path to template file or directory


Request Body
^^^^^^^^^^^^

None.

Request Examples
^^^^^^^^^^^^^^^^
::

    POST /v1/template/?path=[file/dir path]
    Host: 135.248.18.122:8999
    User-Agent: keystoneauth1/2.3.0 python-requests/2.9.1 CPython/2.7.6
    Content-Type: application/json
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

Returns a JSON object that is a list of results.
Each result describes the full validation (syntax and content) of one template file.

Response Examples
^^^^^^^^^^^^^^^^^

::

{
  "results": [
    {
      "status": "validation failed",
      "file path": "/tmp/templates/basic_no_meta.yaml",
      "description": "Template syntax validation",
      "message": "metadata is a mandatory section.",
      "status code": 62
    },
    {
      "status": "validation OK",
      "file path": "/tmp/templates/basic.yaml",
      "description": "Template validation",
      "message": "Template validation is OK",
      "status code": 4
    }
  ]
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

liat har-tal <liat.har-tal@nokia.com>


Work Items
----------

None

Future lifecycle
================

None

Dependencies
============

Depends on the template validation bluprints

Testing
=======

Tempest tests also need to be added in order to test:

#. Validate single template
#. Validate several templates


Documentation Impact
====================
The new api should be documented

References
==========
None
