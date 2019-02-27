..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================
Service List API
================

StoryBoard link: https://storyboard.openstack.org/#!/story/2004897

This spec adds a new api for listing all vitrage services and their status.

Problem description
===================

In a production cloud environment, Vitrage will have multiple services
deployed on multiple hosts. This will enable an admin to find these services
and get details like:

* what is the node on which vitrage service is running,
* what is the running status of vitrage service.
* How long the vitrage services are running successfully.


Proposed change
===============

A new api command ``vitrage service list`` will be added that will list all
the vitrage services that are currently running, where are they running, their
status and how long are they running.

Example of a response
---------------------

.. code-block:: json

  [
      {
        "Created At": "2019-02-10T11:07:15+00:00",
        "Hostname": "controller-1",
        "Process Id": 23161,
        "Name": "ApiWorker worker(0)"
      },
      {
        "Created At": "2019-02-10T11:07:15+00:00",
        "Hostname": "controller-1",
        "Process Id": 23153,
        "Name": "EvaluatorWorker worker(0)"
      },
      {
        "Created At": "2019-02-10T11:07:15+00:00",
        "Hostname": "controller-1",
        "Process Id": 23155,
        "Name": "EvaluatorWorker worker(1)"
      },
      {
        "Created At": "2019-02-10T11:07:15+00:00",
        "Hostname": "controller-1",
        "Process Id": 23157,
        "Name": "EvaluatorWorker worker(2)"
      },
      {
        "Created At": "2019-02-10T11:07:15+00:00",
        "Hostname": "controller-1",
        "Process Id": 23158,
        "Name": "EvaluatorWorker worker(3)"
      },
      {
        "Created At": "2019-02-10T11:07:33+00:00",
        "Hostname": "controller-1",
        "Process Id": 23366,
        "Name": "MachineLearningService worker(0)"
      },
      {
        "Created At": "2019-02-10T11:07:35+00:00",
        "Hostname": "controller-1",
        "Process Id": 23475,
        "Name": "PersistorService worker(0)"
      },
      {
        "Created At": "2019-02-10T11:07:15+00:00",
        "Hostname": "controller-1",
        "Process Id": 23164,
        "Name": "SnmpParsingService worker(0)"
      },
      {
        "Created At": "2019-02-10T11:14:30+00:00",
        "Hostname": "controller-1",
        "Process Id": 25698,
        "Name": "vitrageuWSGI worker 1"
      },
      {
        "Created At": "2019-02-10T11:14:30+00:00",
        "Hostname": "controller-1",
        "Process Id": 25699,
        "Name": "vitrageuWSGI worker 2"
      },
      {
        "Created At": "2019-02-10T11:07:32+00:00",
        "Hostname": "controller-1",
        "Process Id": 23352,
        "Name": "VitrageNotifierService worker(0)"
      }
  ]



CLI Example
-----------

.. code-block:: console

 +----------------------------------+------------+--------------+---------------------------+
 | Name                             | Process Id | Hostname     | Created At                |
 +----------------------------------+------------+--------------+---------------------------+
 | ApiWorker worker(0)              |      23161 | controller-1 | 2019-02-10T11:07:15+00:00 |
 | EvaluatorWorker worker(0)        |      23153 | controller-1 | 2019-02-10T11:07:15+00:00 |
 | EvaluatorWorker worker(1)        |      23155 | controller-1 | 2019-02-10T11:07:15+00:00 |
 | EvaluatorWorker worker(2)        |      23157 | controller-1 | 2019-02-10T11:07:15+00:00 |
 | EvaluatorWorker worker(3)        |      23158 | controller-1 | 2019-02-10T11:07:15+00:00 |
 | MachineLearningService worker(0) |      23366 | controller-1 | 2019-02-10T11:07:33+00:00 |
 | PersistorService worker(0)       |      23475 | controller-1 | 2019-02-10T11:07:35+00:00 |
 | SnmpParsingService worker(0)     |      23164 | controller-1 | 2019-02-10T11:07:15+00:00 |
 | vitrageuWSGI worker 1            |      25698 | controller-1 | 2019-02-10T11:14:30+00:00 |
 | vitrageuWSGI worker 2            |      25699 | controller-1 | 2019-02-10T11:14:30+00:00 |
 | VitrageNotifierService worker(0) |      23352 | controller-1 | 2019-02-10T11:07:32+00:00 |
 +----------------------------------+------------+--------------+---------------------------+

**Note:**
The cloud operator must pre-install zookeeper or other tooz backend component.
Otherwise, an exception will be raised when users call the service REST API.

If vitrage is running in k8s cluster then this api might be redundant.
Since k8s handles pods health and topology. We might make the service api
communicate with the k8s api in this case to get all vitrage services and
their statuses.

Data model impact
-----------------

None.

REST API impact
---------------

New api will be added to vitrage to list the services.

Versioning impact
-----------------

None.

Other end user impact
---------------------

In order to support the api we will need a backend to store the information.
We will use the tooz library that supports multiple backends see Tooz_.

 .. _Tooz: https://docs.openstack.org/tooz/latest/

Deployer impact
---------------

The deployer must pre-install zookeeper or other tooz backend component
In order to support the API.

When deploying a container then hostname must be changed
so api will be readable.

Developer impact
----------------

We need to think what to do in case of a container deployment.
docker by default has a hostname of the container id but it can be changed.

We might use an optional environment variable (e.g HOST_HOSTNAME) for host name
if exist in case of a container that can be passed to the container.

Horizon impact
--------------

None.



Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Eyal

Work Items
----------

* add tooz support
* add new API to vitrage
* add `service list` to vitrage client
* Documentation and tests

Dependencies
============

Depends on the tooz library with a backend configured.

Testing
=======

Unit tests, functional tests and tempest tests

Documentation Impact
====================

The new api will be documented

References
==========

None
