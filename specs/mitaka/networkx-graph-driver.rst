..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
NetworkX Graph driver
==========================

https://blueprints.launchpad.net/vitrage/+spec/networkx-graph-driver

Graph Driver is the defined API for access and manipulation of the underlying graph used for storing the Entity Graph.
This API should be implemented for the NetworkX graph package and possibly for other graph tools, allowing Vitrage a seamless transition between different underlying graph implementations.


Problem description
===================

Vitrage will have a graph data structure that will hold a list of physical and virtual entities and their relationships to one another, in what we call the Entity Graph.
The specific implementation of this graph should be interchangeable allowing a stateless or state-full implementations.

Proposed change
===============

**1. Graph Driver**

Create a Graph Driver, which defines a set of graph methods, to be implemented over NetworkX.
This blueprint describes the addition of the Graph Driver and NetworkX Driver.

::

                                           +-------------------+
                                           |                   |
        +------------------+       +------->   NetworkX Driver |
        |                  |       |       |                   |
        |      Graph       |  Impl |       +-------------------+
        |                  |-------+
        |      Driver      |       |       +-------------------+
        |                  |       |       |                   |
        +------------------+       +------->   Other Drivers   |
                                           |  (BulbFlow,etc..) |
                                           +-------------------+

**Specification of the Graph Driver API:**

Note that the entity graph is a property graph, where edges and vertices can also have a set of key-value properties, which can be added, updated and removed as well.

*Graph CRUD*
 - init
 - num_of_edges
 - num_of_vertices
 - copy //deep copy of the graph

*Vertex CRUD*
 - add_vertex
 - add_vertices
 - get_vertex
 - update_vertex
 - remove_vertex

*Edges CRUD*
 - add_edge
 - get_edge
 - update_edge
 - remove_edge

*Graph algorithms*
 - subgraph_matching (sub-graph isomorphism)
 - BFS
 - DFS

**2. NetworkX Driver**

NetworkX is a pure python library for graphs. It is stateless and suitable for operation on large real world graphs.
The NetworkX Driver will implement Graph Driver


Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

None

Pipeline impact
---------------

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

None


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
None.

Work Items
----------

 - Create the GraphDriver skeleton
 - Implement Graph Driver for NetworkX
 - Testing of GraphDriver over NetworkX


Future lifecycle
================

None

Dependencies
============

None

Testing
=======

This change needs to be tested by unit tests.

Documentation Impact
====================


References
==========