..
This work is licensed under a Creative Commons Attribution 3.0 Unported
License.

http://creativecommons.org/licenses/by/3.0/legalcode

========================
Integration with Mistral
========================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/integration-with-mistral

Support executing Mistral workflows from Vitrage.

Problem description
===================

Vitrage provides insights about the state of the cloud, but is not meant to be
a policy engine. In order to take corrective actions, for example, we need to
integrate an external engine like Mistral - the OpenStack workflow engine.

Proposed change
===============

It will be possible to define in Vitrage templates that under certain
conditions, a Mistral workflow should be executed. This gives the user the
power to decide, for example, that different corrective actions should be taken
based on the root cause of the problem (as identified by Vitrage).

Note that this blueprint is based on the external-actions blueprint, that
handles the more general case.

Examples
--------

.. code-block:: yaml

 - scenario:
    condition: host_down_alarm_on_host
    actions:
     - action:
        action_type: execute_mistral
        properties:
         workflow: wf1


Alternatives
------------
Discussed in the external-actions blueprint.

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
ifat-afek

Work Items
----------

* Implement the Mistral notifier
* Update the documentation

Dependencies
============

None

Testing
=======

The implementation will be covered by unit tests and tempest tests.

Documentation Impact
====================

The new action should be documented

References
==========

None
