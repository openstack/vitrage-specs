..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================================================
Include template definitions from an external file
===================================================

https://blueprints.launchpad.net/vitrage/+spec/definition-templates

Define special template files that contain only definitions
(entities / relationships and not scenarios), that can then be included within
other template files and used there to create new scenarios.

Problem description
===================

A lot of templates were redefining the same entities and relationships for use
in their scenarios.

Proposed change
===============

Add a new file (or a set of files) into a "def_templates" directory. These
files are the same as templates but do not contain scenarios or an "include"
section (only definitions):

.. code-block:: yaml

 metadata:
  name: alarm_on_host_defs
  description: basic def_template example
 definitions:
  entities:
   - entity:
      category: ALARM
      type: nagios
      name: host_problem
      template_id: alarm
   - entity:
      category: RESOURCE
      type: nova.host
      template_id: resource
  relationships:
   - relationship:
      source: alarm
      target: resource
      relationship_type: on
      template_id : alarm_on_host


Add an "include" section within templates, which states the name that should be
included at it appears in the metadata of the definition template. Multiple
definition templates can be added:

.. code-block:: yaml

 metadata:
  name: alarm_on_host_scenario
  description: basic template with an include section example
 definitions:
  entities:
   - entity:
     ...
  relationships:
   - relationship:
      ...
 include:
  - name: alarm_on_host_defs
  - name: ...
 scenarios:
  - scenario:
     condition: alarm_on_host
     actions:
      - action:
         action_type: set_state
         properties:
          state: SUBOPTIMAL
         action_target:
          target: resource11

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None.
Should be addressed in a future template CRUD implementation

Versioning impact
-----------------

None - Old template formats will still be supported. Introduces an alternative
version.

Other end user impact
---------------------

None

Deployer impact
---------------

None

Developer impact
----------------

None

Horizon impact
--------------

Template UI should be changed to show definition template files.

Topology view
^^^^^^^^^^^^^

No impact

RCA view
^^^^^^^^

No impact


Entity graph
^^^^^^^^^^^^

No impact

Summary
^^^^^^^

No impacts

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  nivolas

Other contributors:
  None

Work Items
----------
In scope:

- Loading definition templates.
- Validating definition templates.
- Tests
- Documentation

The following items are not in scope:

- Definition templates with scenarios.
- Recursive includes (a definition template can not include other definition
  templates).

Dependencies
============

None

Testing
=======

The implementation will be covered by additional unit tests and tempest tests.

Documentation Impact
====================

Documentation on how to define definition template files and when to use them

References
==========

None
