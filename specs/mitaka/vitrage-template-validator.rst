..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================
Template Validator
==================

Launchpad blueprint:

https://blueprints.launchpad.net/vitrage/+spec/template-validator

Vitrage Evaluator serves as workflow manager controlling the analysis and activation of templates and execution of template actions.

Template Validator ensures that a new template is correct. Meaning, it conforms with Vitrage Template Standard.


Problem description
===================

Templates do not always meet the Vitrage Template Standard. For example, unsupported action, invalid alarm name, incorrect graph template and etc.

Proposed change
===============

Template validator is a part of Vitrage Evaluator. It receives a template, runs over it and checks its correctness.
If the template is valid, it notify the Evaluator Engine which inserts the template into the template DB. Otherwise, insertion is failed.

Alternatives
------------
A template editor that prevent invalid templates.

Data model impact
-----------------
None

REST API impact
---------------
The validator runs when a new template is added through API create call.

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
TBD

Work Items
----------
None

Dependencies
============
None

Testing
=======
TBD

Documentation Impact
====================
TBD


References
==========
TBD