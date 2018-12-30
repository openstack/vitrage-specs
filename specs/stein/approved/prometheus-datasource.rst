..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Prometheus Datasource Labels Mapping
====================================

StoryBoard link (task #28682):
https://storyboard.openstack.org/#!/story/2004682

This blueprint describes the method of mapping Prometheus's alert labels into
Vitrage, in a way that will allow Vitrage to identify the resource that the
alarm was raised on.

Problem description
===================

Vitrage holds an entity graph with resources and alarms. Each alarm should be
connected to its resource in the graph, in order to support alarm correlation.
In Prometheus, an alert is based on a metric, where each metric can have different
``Labels``. The labels contain enough information to identify the
resource, however for each alert Vitrage may need to use one or more different
label(s).

The purpose of this blueprint is to define a way for Vitrage to easily
determine the identification method for every Prometheus alert.

In the use cases that are described below, we would like to create a Prometheus
alarm vertex in Vitrage and connect it with an ``on`` edge to the right
resource in the graph.


Prometheus alert structure
--------------------------

A Prometheus alert contains several fields, including:

* ``annotations`` - annotations including ``title`` and ``description`` of the alert.
* ``labels`` - a list of one or more labels. The labels are generated from alert rule and
  alert metrics that the alert is based on.
* ``status`` - the current status of the alert: ``firing`` or ``resolved``.


An example of a Prometheus alert:

.. code-block:: json

    {
        "annotations": {
            "description": "The average amount of CPU time spent in idle mode, per second, over the last minute (in seconds)",
            "title": "High average CPU time on idle mode"
        },
        "endsAt": "2018-12-30T09:43:52.589431274Z",
        "generatorURL": "http://devstack-rocky-release-4:9090/graph?g0.expr=100+%2A+%281+-+avg+by%28instance%29+%28irate%28node_cpu_seconds_total%7Bjob%3D%22node%22%2Cmode%3D%22idle%22%7D%5B5m%5D%29%29%29+%3E+20&g0.tab=1",
        "labels": {
            "alertname": "AvgCPUTimeOnIdleMode",
            "instance": "135.248.18.109:9100",
            "severity": "warning"
        },
        "receivers": [
            "vitrage"
        ],
        "startsAt": "2018-12-26T15:22:07.589431274Z",
        "status": {
            "inhibitedBy": [],
            "silencedBy": [],
            "state": "active"
        }
    }


A full description of Prometheus alert structure can be found in prometheus_alert_description_

.. _prometheus_alert_description: https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/


Alerts based on libvirt metrics
-------------------------------

All libvirt alerts have the following labels:

* ``instance``: holds the hostname/ip of where the exporter is running
* ``domain``: libvirt name that the exporter is scraping


Alert based on libvirt CPU metrics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Prometheus alert example**

.. code-block:: json


    {
        "annotations": {
            "description": "Test alert to test libvirt exporter.\n",
            "title": "High cpu usage on vm"
        },
        "endsAt": "2018-12-30T09:44:05.91446215Z",
        "generatorURL": "http://devstack-rocky-release-4:9090/graph?g0.expr=rate%28libvirt_domain_info_cpu_time_seconds_total%5B1m%5D%29+%2A+10000+%3E+13&g0.tab=1",
        "labels": {
            "alertname": "HighCpuOnVmAlert",
            "domain": "instance-00000004",
            "instance": "135.248.18.109:9177",
            "job": "libvirt",
            "severity": "critical"
        },
        "receivers": [
            "vitrage"
        ],
        "startsAt": "2018-12-26T15:23:05.91446215Z",
        "status": {
            "inhibitedBy": [],
            "silencedBy": [],
            "state": "active"
        }
    }

**Vitrage resource**

Vitrage resource can be uniquely identified by the ``instance`` and ``domain`` labels.


Alert based on libvirt network metrics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Prometheus alert example**

.. code-block:: json

    {
        "annotations": {
            "description": "Another test alert to test libvirt exporter.\n",
            "title": "High traffic on bridge"
        },
        "endsAt": "2018-12-30T09:43:50.91446215Z",
        "generatorURL": "http://devstack-rocky-release-4:9090/graph?g0.expr=rate%28libvirt_domain_interface_stats_receive_bytes_total%5B5m%5D%29+%3E+0&g0.tab=1",
        "labels": {
            "alertname": "HighTrafficOnBridge",
            "domain": "instance-00000004",
            "instance": "135.248.18.109:9177",
            "job": "libvirt",
            "severity": "critical",
            "source_bridge": "br-int",
            "target_device": "tap456ab233-f4"
        },
        "receivers": [
            "vitrage"
        ],
        "startsAt": "2018-12-26T15:22:05.91446215Z",
        "status": {
            "inhibitedBy": [],
            "silencedBy": [],
            "state": "active"
        }
    }



**Vitrage resource**

* Short term: raise the alarm on the node or instance. Vitrage resource can be
  uniquely identified by the ``instance`` and ``domain`` labels.
* Long term: Vitrage should hold a resource for br-int and the alarm should be
  connected to that resource. Vitrage resource can be uniquely identified by
  the ``instance``, ``domain``, ``source_bridge`` and ``target_device`` labels.


Node metrics
------------

**Prometheus alert**

All Node metrics have a ``instance`` that holds the address of exporter.
The exporter can scrape metrics from the instance it is running on.
In this case ''instance`` label represents the resource address.
Also, It can scrape different metrics not from the instance (e.g. network metrics).
In this case ``instance`` is just an address of the exporter and other labels
indicates to the resource.


Alert based on node CPU metric
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Prometheus alert**

CPU metrics are scraped from the instance so ``instance`` label represents the resource address.

**Prometheus alert example**

.. code-block:: json

    {
        "annotations": {
            "description": "The average amount of CPU time spent in idle mode, per second, over the last minute (in seconds)",
            "title": "High average CPU time on idle mode"
        },
        "endsAt": "2018-12-30T09:43:52.589431274Z",
        "generatorURL": "http://devstack-rocky-release-4:9090/graph?g0.expr=100+%2A+%281+-+avg+by%28instance%29+%28irate%28node_cpu_seconds_total%7Bjob%3D%22node%22%2Cmode%3D%22idle%22%7D%5B5m%5D%29%29%29+%3E+20&g0.tab=1",
        "labels": {
            "alertname": "AvgCPUTimeOnIdleMode",
            "instance": "135.248.18.109:9100",
            "severity": "warning"
        },
        "receivers": [
            "vitrage"
        ],
        "startsAt": "2018-12-26T15:22:07.589431274Z",
        "status": {
            "inhibitedBy": [],
            "silencedBy": [],
            "state": "active"
        }
    }


**Vitrage resource**

Vitrage resource can be uniquely identified by the ``instance`` label.


Alert based on node filesystem metric
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Prometheus alert example**

.. code-block:: json

    {
        "annotations": {
            "description": "\"Consider ssh'ing into the instance and removing files or clean\ntemp files\"\n",
            "device": "/dev/vda1",
            "mount_point": "/",
            "runbook": "troubleshooting/filesystem_alerts_inodes.md",
            "title": "High number of inode usage",
            "value": "92.42%"
        },
        "endsAt": "2018-12-30T09:43:52.589431274Z",
        "generatorURL": "http://devstack-rocky-release-4:9090/graph?g0.expr=node_filesystem_files_free%7Bfstype%3D~%22%28ext.%7Cxfs%29%22%2Cjob%3D%22node%22%7D+%2F+node_filesystem_files%7Bfstype%3D~%22%28ext.%7Cxfs%29%22%2Cjob%3D%22node%22%7D+%2A+100+%3C%3D+100&g0.tab=1",
        "labels": {
            "alertname": "HighInodeUsage",
            "device": "/dev/vda1",
            "fstype": "ext4",
            "instance": "135.248.18.109:9100",
            "job": "node",
            "mountpoint": "/",
            "severity": "critical"
        },
        "receivers": [
            "vitrage"
        ],
        "startsAt": "2018-12-26T15:22:07.589431274Z",
        "status": {
            "inhibitedBy": [],
            "silencedBy": [],
            "state": "active"
        }
    }


**Vitrage resource**

* Short term: raise the alarm on the node or instance. Vitrage resource can be
  uniquely identified by the ``instance`` label.
* Long term: Vitrage should hold a resource for ext4 and the alarm should be
  connected to that resource. Vitrage resource can be uniquely identified by
  the ``instance``, ``device`` and ``fstype`` labels.


Proposed change
===============

A configuration file that maps the Prometheus labels to a corresponding
Vitrage resource with specific properties (id or other unique properties).
The mapping will most likely be defined by the alert name and other fields.


Prometheus configuration file structure
---------------------------------------

The configuration file contains a list of ``alerts``.
Each alert contains ``key`` and ``resource``.

The ``key`` contains ``alertname`` and ``job`` labels which uniquely identify each alert.

The ``resource`` specifies how to identify in Vitrage the resource that the alert is on.
It contains one or more Vitrage property names and corresponding Prometheus alert labels.


**Configuration file example**

.. code-block:: yaml

    alerts:
    - alert:
        key:
          alertname: HighCpuOnVmAlert
          job: libvirt
        resource:
          instance_name: domain
          host_id: instance
    - alert:
        key:
          alertname: HighTrafficOnBridge
          job: libvirt
        resource:
          instance_name: domain
          host_id: instance
    - alert:
        key:
          alertname: AvgCPUTimeOnIdleMode
          job: node
        resource:
          id: instance
    - alert:
        key:
          alertname: HighInodeUsage
          job: node
        resource:
          id: instance


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

TBD

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
  7mode3294 (Muhamad Najjar)


Work Items
----------

* Load configuration file and use it in the Prometheus transformer.
* Documentations and tests.


Dependencies
============

None


Testing
=======

Unit tests, functional tests and tempest tests


Documentation Impact
====================

The new configuration will be documented


References
==========

* Prometheus datasource: https://github.com/openstack/vitrage/tree/master/vitrage/datasources/prometheus
* Prometheus alerting rules: https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/
* Prometheus libvirt exporter: https://github.com/CanonicalLtd/prometheus-openstack-exporter
* Prometheus node exporter: https://github.com/prometheus/node_exporter