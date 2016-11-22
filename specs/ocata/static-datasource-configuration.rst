..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Static Data Source Configuration
==========================================

https://blueprints.launchpad.net/vitrage/+spec/static-datasource-config-format

The configuration of static data source has a lot in common with entity and
relationships definition in evaluator template. This blueprint proposes a
refactoring to reuse the template format and parsing methods in static data
source. By doing so, we may reduce the work in maintenance and bring in new
features more easily.

Problem description
===================

Currently the configuration of static data source use a dedicated format, which
has a lot of overlapping with the evaluator templates.

In static configuration, there are ``entities`` and their ``relationships``

.. highlight:: yaml

  - entities
    - {entity}
    - {entity}

In each entity

.. highlight:: yaml

  - name:
    id:
    relationship:
      - {relationship}
      - {relationship}

In evaluator templates we define: ``entities``, ``relationship`` and
``scenarios``. Each scenario has a condition and actions.

.. highlight:: yaml

  - definitions
    - entities
      - {entity}
      - {entity}
    - relationships
      - {relationship}
      - {relationship}

Though serving different purpose, they both

#. Describe ``entities`` and ``relationships``
#. Use a dedicated key (id/template_id) to reference the items
#. Include a source entity and target entity in relationship

The main differences between the two are

- Evaluator templates defines a topology and scenarios based on it
- Static config defines a topology and **adds** it to the graph

We may define the static configuration using the same format as the evaluator
templates. And then simulate an entity discovery from the same file.

By reusing the template parsing engine and workflow, we may reduce the work
in maintenance and bring in new features more easily.

Proposed change
===============

Refactoring the static data source template and use the same parser as in
evaluator template.

Discover entities from the static data source template.

For backward compatibility, static data source will take over the control of the
default configuration folder ``/etc/vitrage/static_datasource`` which was used
by static physical datasource.

Both legacy format and new format will be placed in the same folder. static
datasource parse the file and check the existence of ``meta`` to decide which
engine to use. If not found, proxy the job to static physical datasource and
print a deprecation warning.

Static physical datasource will be disabled by default and throws exception if
running standalone.

Alternatives
------------

Fix the issues found in static data source without refactoring the format. This
will keep best back-compatibility but will cause redundant work with scenario
evaluator.

Data model impact
-----------------

None

REST API impact
---------------

None

Versioning impact
-----------------

- Backward compatibility with old format will be kept.
- This introduces a new feature and a minor version incremental is required.

Other end user impact
---------------------

New format of static data source configuration should be applied by the end
user.

Deployer impact
---------------

Old parser will be kept but a deprecated warning will be prompt.

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
  yujunz

Other contributors:
  None

Work Items
----------

- Reuse the parser of evaluator template in static data source configuration.
- Discover entities from the configuration.
- Add deprecated warning on old format.

Dependencies
============

None

Testing
=======

The changes shall be covered by new unit test.

Documentation Impact
====================

New format of the template shall be documented.

References
==========

- `http://lists.openstack.org/pipermail/openstack-dev/2016-September/102678.html`_
