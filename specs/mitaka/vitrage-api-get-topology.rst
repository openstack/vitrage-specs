..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Vitrage Get Topology API
========================

https://blueprints.launchpad.net/vitrage/+spec/vitrage-api-get-topology

Vitrage needs to supply an api for listing a full or partial resources and alarms topology.

Problem description
===================

We would like to be able to retrieve the Vitrage topology, which will include both the resources and their appropriate alarms


Proposed change
===============

Create API to retrieve the resources and alarms topology :

#.	Retrieve the full topology.
#.	List partial topology with filter :

    #.   Full connected components of a given element
    #.   Connected component of a given element, filtered according to edges of specific type(s) (i.e., labels and properties)



Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

List Versions
^^^^^^^^^^^^^

Lists the supported versions of the Vitrage API.

GET /
~~~~~

Headers
^^^^^^^

-  X-Auth-Token (string, required) - Keystone auth token
-  Accept (string) - application/json

Path Parameters
^^^^^^^^^^^^^^^

None.

Query Parameters
^^^^^^^^^^^^^^^^

None.

Request Body
^^^^^^^^^^^^

None.

Request Examples
^^^^^^^^^^^^^^^^

::

    GET / HTTP/1.1
    Host: 135.248.19.18:8999
    X-Auth-Token: 2b8882ba2ec44295bf300aecb2caa4f7
    Accept: application/json

Response
~~~~~~~~

Status code
^^^^^^^^^^^

-  200 - OK

Response Body
^^^^^^^^^^^^^

Returns a JSON object with a 'links' array of links of supported versions.

Response Examples
^^^^^^^^^^^^^^^^^

::

    {
        "versions": [
            {
               "id": "v1.0",
              "links": [
                    {
                     "href": "http://135.248.19.18:8999/v1/",
                    "rel": "self"
                   }
              ],
              "status": "CURRENT",
              "updated": "2015-11-29"
            }
        ]

    }


Get  topology
^^^^^^^^^^^^^

Get the topology for the node.
Its possible to filter the edges vertices and depth of the
graph


GET /v1.0/topology/
~~~~~~~~~~~~~~~~~~~

Headers
^^^^^^^

-  X-Auth-Token (string, required) - Keystone auth token
-  Accept (string) - application/json

Path Parameters
^^^^^^^^^^^^^^^

None.

Query Parameters
^^^^^^^^^^^^^^^^

-  edges (string(255), optional) - name of edges to filter.
-  vertices (string, optional) - name of vertices to filter.
-  depth (int, optional) - the depth of graph required.
-  graph type ([tree,graph], optional) - The type of graph required
   defaults to graph

Request Body
^^^^^^^^^^^^

None.

Request Examples
^^^^^^^^^^^^^^^^

::

    GET /v1/topology/?graph_type=tree&depth=4&edges=vm&edges=host&edges=zone
    &vertices=contains
    Host: 135.248.19.18:8999
    Content-Type: application/json
    X-Auth-Token: 2b8882ba2ec44295bf300aecb2caa4f7

Response
~~~~~~~~

Status Code
^^^^^^^^^^^

-  200 - OK
-  400 - Bad request

Response Body
^^^^^^^^^^^^^

Returns a JSON object that describes a graph with nodes
and links. If a tree representation is asked then returns
a Json tree with nodes and children.

An error of cannot represent as a tree will be return if the
graph is not a tree. (400 - Bad request)

Response Examples
^^^^^^^^^^^^^^^^^

::

    {
    "children": [
        {
            "children": [
                {
                    "children": [
                        {
                            "id": 16,
                            "name": "vm5",
                            "state": "ERROR"
                        },
                        {
                            "id": 23,
                            "name": "vm12",
                            "state": "SUBOPTIMAL"
                        }
                    ],
                    "id": 8,
                    "name": "host4",
                    "state": "SUBOPTIMAL"
                },
                {
                    "children": [
                        {
                            "id": 26,
                            "name": "vm15",
                            "state": "SUBOPTIMAL"
                        },
                        {
                            "id": 19,
                            "name": "vm8",
                            "state": "ERROR"
                        },
                        {
                            "id": 12,
                            "name": "vm1",
                            "state": "RUNNING"
                        }
                    ],
                    "id": 5,
                    "name": "host1",
                    "state": "SUBOPTIMAL"
                }
            ],
            "id": 1,
            "name": "zone0",
            "state": "ERROR"
        },
        {
            "children": [
                {
                    "children": [
                        {
                            "id": 21,
                            "name": "vm10",
                            "state": "RUNNING"
                        },
                        {
                            "id": 14,
                            "name": "vm3",
                            "state": "SUBOPTIMAL"
                        }
                    ],
                    "id": 10,
                    "name": "host6",
                    "state": "ERROR"
                },
                {
                    "children": [
                        {
                            "id": 20,
                            "name": "vm9",
                            "state": "SUBOPTIMAL"
                        },
                        {
                            "id": 13,
                            "name": "vm2",
                            "state": "ERROR"
                        }
                    ],
                    "id": 4,
                    "name": "host0",
                    "state": "ERROR"
                },
                {
                    "children": [
                        {
                            "id": 24,
                            "name": "vm13",
                            "state": "RUNNING"
                        },
                        {
                            "id": 17,
                            "name": "vm6",
                            "state": "SUBOPTIMAL"
                        }
                    ],
                    "id": 7,
                    "name": "host3",
                    "state": "ERROR"
                }
            ],
            "id": 2,
            "name": "zone1",
            "state": "SUBOPTIMAL"
        },
    ],
        "id": 0,
        "name": "node1",
        "state": "RUNNING"
    }


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
	dan-ofek <dan.ofek@alcatel-lucent.com>

Work Items
----------

None

Future lifecycle
================

None

Dependencies
============

Depends on the blueprints of the graph and the synchronizer

Testing
=======

Tempest tests also need to be added to test "get full topology" and "get partial topology" API.


Documentation Impact
====================
The new api should be documented

References
==========
None
