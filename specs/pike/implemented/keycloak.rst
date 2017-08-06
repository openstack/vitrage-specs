..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================
Keycloak support
================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/keycloak-support

As part of an on going effort to make vitrage to be able to work also in a non
OpenStack environment (in addition to the default OpenStack environment).
We should be able to make vitrage work with a different authorization server
instead of keystone. An optional authorization server can be Keycloak which is
an open source Identity and Access Management solution aimed at modern
applications and services


Problem description
===================

Vitrage at the moment can only work in an OpenStack environment because it needs
Keystone for authorization. We should support other authorization such as Keycloak.



Proposed change
===============

New auth_mode in api section in Vitrage config file::

 [api]
 auth_mode = keycloak

New keycloak section with the auth_url in Vitrage config::

 [keycloak]
 auth_url = http://[keycloak server]:[keycloak port]/auth

The Vitrage server will use a new middleware which will authenticate with the
Keycloak server once an api request is received.

A new auth plugin will be added to the vitrage client which will get the token
from the Keycloak server and sent it with the api request.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------
When using the client we should use the keycloak-plugin

Versioning impact
-----------------

None

Other end user impact
---------------------

None

Deployer impact
---------------

To use the Keycloak Authorization there is a need to define it in the
Vitrage config file.

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
  eyalb1

Work Items
----------

- Create Keycloak plugin in client

- Create Keycloak plugin in server

Dependencies
============

None

Testing
=======

This blueprint requires unit tests.

Documentation Impact
====================

The usage of the KeyCloak authorization will be documented


References
==========

`keycloak-config.rst <https://github.com/openstack/vitrage/blob/master/doc/source/contributor/keycloak-config.rst>`_
