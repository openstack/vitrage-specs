..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================
httppost Registration
=====================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/httppost-notifications

The Evaluator performs root cause analysis on the Vitrage Graph and may
determine that an alarm should be created, deleted or otherwise updated.
Other components are notified of such changes by the Vitrage Notifier service.
To regirster to the httppost notifications, one needs to supply the details
of the registration, and a regex for the wanted alarms.

This blueprint describes the implementation of registering for httppost
notifications on Vitrage.


Problem description
===================

Vitrage should support registering for httppost notifications with a regex
to support tailored notifications per target.


Proposed change
===============

Vitrage evaluator may perform raise alarm or set state actions, and the
httppost notifier will send notifications on any changes / alarms.

The notifications will be sent to all destinations specified in the DB.

We will add an httppost registration API which will handle the following :

- Address : HTTP post recipient address

- (optional) Certification properties : Used for headers purpose
  example: User, password  ....... The registered can specify any properties
  he wants to see in the notification header.

- Regex : A set of properties / values to filter the relevant data. Example :

  - vitrage_type : nova.instance, nova.host

  - aggregated_state :.....

  - category : alarm, resource

  - name : "%kinori%"

Each registration will be saved in a different row in the DB.

There can be multiple registrations with the same address and different regex(es)

Validation will be made, during registration that same recipient + regex
do not exist.

First stage will only support Add and Delete.

We will also add UI to enable manually adding registration requests.

Alternatives
------------

None

Data model impact
-----------------

New table in the DB

REST API impact
---------------

`http post notification <https://github.com/openstack/vitrage-specs/blob/master/specs/pike/httppost-notifications.rst>`_


Versioning impact
-----------------

None

Other end user impact
---------------------

None

Deployer impact
---------------

We need a new DB table, to represent registration details
Preliminary columns :

- ID

- Date

- Address

- Certificate data

- Filter properties


Developer impact
----------------

We need to implement a DB an infrastructure for working with database.
For example sqlalchemy. The spec for this will be written in a different
document.

Horizon impact
--------------

New Vitrage Configuration Tab :

Will currently only have a list of registrations, with options to delete and add.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  danoffek

Work Items
----------

- Create new table

- Create registraion add / delete api

- Create filtering in httppost notifications

- Create unit test for registration

  - test that the registration works

  - test filter works properly by creating alarms

- Create tempest tests for registration

  - test registration success: url matches, properties match, filter matches

  - test additional, same and different registration

  - test delete registration success

Dependencies
============

None

Testing
=======

This blueprint requires both unit tests and tempest tests.
Test Add, Remove registrations, and test that they have the same address,
properties, and filters,

Documentation Impact
====================

The usage of notifier registration will be documented


References
==========

`http post registration request <https://github.com/openstack/vitrage/blob/master/doc/source/notifier-httppost-plugin.rst>`_
