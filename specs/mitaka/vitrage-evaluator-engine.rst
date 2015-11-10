..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================
Evaluator Engine
================

Launchpad blueprint:

https://blueprints.launchpad.net/vitrage/+spec/evalutor-engine

Vitrage Evaluator serves as workflow manager controlling the analysis and activation of templates and execution of template actions.

Evaluator engine is the main core of Vitrage evaluator which responsible for managing the templates, executing them over Vitrage Graph and running their action.

::

 +---------------------+
 |                     |
 |                     <------------------+
 |   Vitrage Graph     |                  |
 |                     |          +-------+-------------------+
 |                     |          | Vitrage Evaluator         |
 +--------------+------+          |                           |
                |                 |                  +------+ |
                |                 |                  | RCA  | |
                +----------------->                  |      | |
                                  |                  +------+ |
                                  |                  +------+ |
 +---------------------+          |                  |Deduce| |
 |                     <----------+                  |Alarm | |
 |   Vitrage Notifier  |          |                  +------+ |
 |                     |          |                           |
 |                     |          +---------------------------+
 |                     |
 +---------------------+


Problem description
====================

Vitrage requires a component that is responsible for managing and executing templates, which are the basis for the different algorithms used in Vitrage, such as RCA.

**Use cases:**
 #. When a new **instance** is added/removed/updated in the Vitrage Graph, find the templates relevant to this change and compare the pattern specified in each template to the new graph structure. For each pattern match found, execute the actions specified in the template.

 #. When a new **alarm** is added/removed/updated in the Vitrage Graph, find the templates relevant to this change and compare the pattern specified in each template to the new graph structure. For each pattern match found, execute the actions specified in the template.


Proposed change
===============

**Managing Templates**

Users can perform CRUD actions on templates, in order to make changes to the use cases Vitrage supports. Specifically, when adding a template it should be added and stored in graph format and saved in NetworkX (Graph-based in-memory DB). Similarly, this template can be modified or deleted as well. The functionality added here should have a corresponding set of API calls for users to perform these actions.

**Executing Templates**

When a change takes place in the Vitrage Graph, it informs the Evaluator Engine of the change. The engine will then search for templates relevant to this change and compare the pattern specified in each template to the new graph structure. For each pattern match found, execute the actions specified in the template.


Alternatives
------------
None

Data model impact
-----------------
The templates are saved in NetworkX graph-base in memeory DB.

REST API impact
---------------
The functionality added here for **managing** templates should have a corresponding set of API calls for users to perform these actions.

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


Dependencies
============

* API with Vitrage Graph
* API with Vitrage Notifier


Testing
=======
TBD


Documentation Impact
====================
TBD


References
==========
TBD