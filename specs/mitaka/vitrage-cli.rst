..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========
Vitrage CLI
===========

https://blueprints.launchpad.net/vitrage/+spec/vitrage-cli

Vitrage Project introduces a Root Cause Analysis (RCA) engine
for organizing, analyzing and expanding OpenStack alarms & events.

In order to communicate with Vitrage a command line utility can be used
that will issue some REST command to the API service.

The CLI will use a ``python-vitrageclient`` which is a client library built
on the Vitrage API.

::

 +-----------------+                     +-----------------+
 |      *CLI*      |                     |                 |
 |                 |                     |                 |
 |  RCA            |                     |                 |
 |                 |   HTTP/Vitrage API  |  vitrage api    |
 |  CRUD Templates |+-------------------->                 |
 |                 |                     |    service      |
 |  Topology       |                     |                 |
 |                 |                     |                 |
 +-----------------+                     +-----------------+

Problem description
===================

As a user I would like to be able to see the root cause of any alerts or events in the system.
A command line utility will be used to communicate with Vitrage API service.
The CLI will 3 types of commands:

#. RCA - find the root cause for an alert/event

#. CRUD Templates -  Create/Read/Update/Delete Templates

#. Topology - get the topology of the system


Proposed change
===============

The CLI and the vitrage client is part of a new project for Root Cause Analysis
called vitrage

Alternatives
------------
None

Data model impact
-----------------

No data is stored or cached.

REST API impact
---------------

TBD

Versioning impact
-----------------

Discuss how your change affects versioning and backward compatibility:

None

Other end user impact
---------------------

The User will be able to interact using any HTTP rest client.
The User will also have a UI.

Deployer impact
---------------

A new project called Vitrage will deploy the vitrage client and CLI

Developer impact
----------------

None

Horizon impact
--------------

A new UI will be added to Horizon to support the Vitrage project
a separate blueprint will be supplied.


Implementation
==============

Assignee(s)
-----------

None

Work Items
----------

None


Dependencies
============

None


Testing
=======

All code will be tested.

Documentation Impact
====================

None


References
==========

`Vitrage project <https://wiki.openstack.org/wiki/Vitrage>`_
