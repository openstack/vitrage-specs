..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================
Template not operator Support
=============================

We would like the templates language to support the "not" operator in addition
to "and" and "or" operators.

Problem description
===================

Today the templates language support the "and" and "or" operators, but this is
not enough.
Many scenarios can't be described by those two operators, and thus we would
like to add a support for the "not" operator.

Proposed change
===============

Remark: positive vertex = vertex that has an edge with "is_deleted"=False property.
        negative vertex = the opposite of positive vertex. Meaning that the
                          vertex has only edges with "is_deleted"=True property.

The following changes needs to be done to support the "not" operator.

1. In the template validation. Check that the "Not" operator can appear only
   in the following way:

   "Not" operator can appear before edges.

2. Scenario Evaluator:

   check if "not" operator appears on an element, and add a property named
   "negative_condition" on it, and updated it's "is_delete" property to True.

3. Networkx Algorithm:

   In the subgraph_matching method. If the match is not a "negative condition"
   then perform regular subgraph_matching.
   Otherwise, if the match is a "negative condition" then perform
   subgraph_matching on a negative edges.


Performance/Scalability Impacts
-------------------------------

The performance of the subgraph matching algorithm is a bit slower due to
the steps above.


Other deployer impact
---------------------

None

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
    alexey_weyl <alexey.weyl@nokia.com>

Work Items
----------

None

Future lifecycle
================

None

Dependencies
============

None

Testing
=======

Added tests that are checking different uses of the "not" operator.

Documentation Impact
====================

Added documentation in Vitrage:
    https://github.com/openstack/vitrage/blob/master/doc/source/not_operator_support.rst

References
==========

None
