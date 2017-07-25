..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================
Snmp Parsing Service
====================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/snmp-support

This blueprint describes the implementation of SNMP parsing service for transforming SNMP alarm
messages to alarm details and distributing them to corresponding datasource.

Problem description
===================

The following use case should be supported:

Vitrage datasources module provides the ability to handle alarms from part of monitored systems,
but currently there is no system that reports alarms by SNMP communication.

Proposed change
===============
A SNMP service module is presented here, which provides service to parse alarms reported from SNMP
managed system and sends them to the OpenStack message bus, for further processing by the datasources.

Since snmp service is a common service for alarm datasource, the service powers on just after api,
graph and notifier service. After successfully powered on, SNMP parsing service can receive and
decode alarm messages. Decoded alarm details are made up of alarm/object info and corresponding value,
e.g. alarm_code. The SNMP parsing service parses alarm datasource info according to corresponding
OID, then constructs message after marking datasource information and distributes messages to the
RabbitMQ queue. According to the values of OID, the alarm datasource can extract information by decoded
alarm details.

The configuration for snmp service:
[snmp_parsing]

# snmp listening port (integer value)
snmp_listening_port = xxx

# traps oid mapping yaml file path(string value)
#oid_mapping = /etc/vitrage/snmp_parsing_conf.yaml

An example of config for snmp_parsing_conf.yaml:

- oid: 1.3.6.1.4.1.3902.4101.1.3.1.12  # for example
  system: iaas_platform  # for example
  datasource: new_datasource
- oid: xxxx
  system: xxx
  datasource: xxx


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

None

Developer impact
----------------

The snmp parsing service does not support lost notifications at the moment. If one
needs the solution in the future, the service should be enhanced.

Horizon impact
--------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  xupeipei

Work Items
----------

* Add a new SNMP parsing service


Dependencies
============

None

Testing
=======

The implementation will be covered by unit tests and tempest tests.

Documentation Impact
====================

The new SNMP configuration should be documented

References
==========

None

