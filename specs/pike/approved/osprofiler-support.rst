..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================
OSProfiler Support
==================

https://blueprints.launchpad.net/vitrage/+spec/osprofiler-support

`OSProfiler`_ is an OpenStack cross-project profiling library. It allows user
to generate 1 trace per request which is processed in multiple services.

Problem description
===================

It is quite common that Vitrage is integrated in a large system to provide a
complete solution e.g. as inspector in `OPNFV/Doctor`_ for fault management.

When performance is a critical issue, it is very complicated to analyse the
event processing workflow and locate the bottleneck in case something works
slowly.

Proposed change
===============

Add osprofiler support in vitrage and vitrage-client.

OSProfiler will help to generate a tree of calls (see `example`_) which is
intuitive to understand what exactly is going on.

Alternatives
------------

Explained in `why not cprofile and etc`_

.. _why not cprofile and etc: https://osprofiler.readthedocs.io/en/latest/#why-not-cprofile-and-etc

Data model impact
-----------------

None

REST API impact
---------------

Additional HTTP header inserted by profiler should be checked when it is
enabled. Besides that, there is no impact on the business logic in api handler.

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

- add configuration options for osprofiler
- add initialization of osprofiler in service startup
- add osprofiler wsgi middleware to trace HTTP calls
- trace RPC calls

Refer to the changes proposed in `similar topic in ironic`_ for an overview.

Dependencies
============

None

Testing
=======

Covered by unit test.

Documentation Impact
====================

Add to developer guide on how to use osprofiler.

References
==========

.. _OSProfiler: https://docs.openstack.org/developer/osprofiler/index.html
.. _OPNFV/Doctor: https://wiki.opnfv.org/display/doctor
.. _similar topic in ironic: https://review.openstack.org/#/q/topic:bug/1560704
.. _example: http://doctor.surge.sh/
