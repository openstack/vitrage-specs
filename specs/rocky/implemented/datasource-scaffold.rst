..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
Datasource Scaffold
===================

https://blueprints.launchpad.net/vitrage/+spec/datasource-scaffold

A command line tool to generate skeleton of new datasource. A skeleton contains
stubs of required classes and methods as described in the `design specs`_,
without detail implementation. It aims to bootstrapping the development of new
datasource.

Problem description
===================

The `design specs`_ has given detail instructions on how to add a new data
source. However, there is much overhead to create it from scratch. Developers
used to copy from existing datasource as a start. It is sometimes out of date
and always contains many specific codes.

Proposed change
===============

Create templates for the datasource skeleton with placeholders of names and
render the Python source file on demand.

Example template in `Jinja2`_::

  from oslo_config import cfg
  from vitrage.common.constants import UpdateMethod

  {{ name|upper }}_DATASOURCE = '{{ name }}'

  # define needed options
  OPTS = [
      # Transformer with the path to your transformer classes
      cfg.StrOpt('transformer',
                 default='vitrage.datasources.{{ name }}_datasource.transformer.'
                         '{{ name|capitalize }}Transformer',
                 help='{{ name|capitalize }} transformer class path',
                 required=True),

Providing ``name=foo``, it will generate the skeleton source file in Python::

  from oslo_config import cfg
  from vitrage.common.constants import UpdateMethod

  FOO_DATASOURCE = 'foo'

  # define needed options
  OPTS = [
      # Transformer with the path to your transformer classes
      cfg.StrOpt('transformer',
                 default='vitrage.datasources.foo_datasource.transformer.'
                         'FooTransformer',
                 help='Foo transformer class path',
                 required=True),

Alternatives
------------

Create and maintain a sample datasource to allow user to modify as base. In this
way, developer is likely to miss some string replacement somewhere as we
experienced in the `abandoned patch set`_.

Data model impact
-----------------

None

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

None

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

- Create datasource skeleton template
  - driver
  - transformer
- Create unit test skeleton template
  - test driver
  - test transformer
  - mock configuration
  - mock driver
  - trace generator
- Templates for datasource with different update methods

Dependencies
============

None

Testing
=======

The changes shall be covered by new unit test.

Documentation Impact
====================

How to use the generator will be documented.

References
==========

.. _design specs: http://docs.openstack.org/developer/vitrage/add-new-datasource.html
.. _Jinja2: http://jinja.pocoo.org
.. _abandoned patch set: https://review.openstack.org/#/c/396974
