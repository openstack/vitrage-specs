..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================
httppost Notifications
======================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/httppost-notifications

The Evaluator performs root cause analysis on the Vitrage Graph and may
determine that an alarm should be created, deleted or otherwise updated.
Other components are notified of such changes by the Vitrage Notifier service.
Among others, Vitrage Notifier is responsible for sending http post
notifications on Vitrage deduced alarms.

This blueprint describes the implementation of Vitrage Notifier for
http post notifications on Vitrage deduced alarms.


Problem description
===================

Vitrage should support registering for http post notifications, and sending them
on raised alarms, deactivated alarms, state changes, RCA or other to any
registered targets.
Furthermore any registered recipient should supply a regex to filter the alarms
sent to that recipient.


Proposed change
===============

Needed definitions in Vitrage config file::

 [DEFAULT]
 notifiers = httppost

SSL Client certificate file for REST notifier. (string value) :
 rest_notifier_certificate_file =

SSL Client private key file for REST notifier. (string value)
 rest_notifier_certificate_key =

SSL CA_BUNDLE certificate for REST notifier (string value)
 rest_notifier_ca_bundle_certificate_path = <None>

Whether to verify the SSL Server certificate when calling alarm action (boolean value)
 rest_notifier_ssl_verify = true

Number of retries for REST notifier (integer value)
 rest_notifier_max_retries = 0

Timeout seconds for HTTP requests. Set it to None to disable timeout (integer value)
 http_timeout = 600

Vitrage listener will get the alarm events from the message bus and the http post
notifier will send http post notifications on raised deduced alarms and deleted deduced alarms.

The filtered notifications will be sent to destinations specified in registered yaml file or
other (for now, only one option will be developed), which can be updated manually or via
API requests.

The notifications will be sent only on alarms which meet the regex filter specified in the
yaml file / via registration. This section will be specified in a different spec.

The format of sent notifications will be hard coded.

The http post notifier is pluggable, you can implement your own http post sender and use
it (it must inherit from the base class), when there is a default implementation.

Alternatives
------------

None

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

To use http post notifier one needs to define it in the Vitrage config
file, and in addition create additional yaml files and define them in Vitrage config file.

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
  danoffek

Work Items
----------

- Create http post notifier

  - Create http post sender

- Create unit test for http post sender

  - test http post notifier
  - test http post sender
  - test http post regex filtering

Dependencies
============

None

Testing
=======

This blueprint requires unit tests.

Documentation Impact
====================

The usage of the http post notifier will be documented


References
==========

Example on http post notifications in AODH
`http post request <https://github.com/openstack/aodh/blob/master/aodh/notifier/rest.py#L60-L109>`_
