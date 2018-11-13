..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
kubernetes Datasource Driver - get_all implementation
=====================================================

launchpad blueprint:
https://blueprints.launchpad.net/vitrage/+spec/k8s-datasource

This blueprint describes the kubernetes datasource, and its
implementation for get_all nodes (VM's).

Problem description
===================

Kubernetes nodes should be added to Vitrage Graph via Vitrage Datasources.
This requires writing a Datasource for kubernetes.

The datasource should support get_all: periodic query all kubernetes nodes.


Proposed change
===============

Kubernetes datasource will be configured with:

* Poll interval in seconds (default: 600)

Every poll-interval seconds, Kubernetes Driver will call kubernetes client to retrieve all nodes.
The nodes will be converted to Vitrage datasource events and passed to Vitrage
Graph queue.

Example of kubernetes client return call for list_nodes() ::

    apiVersion: v1
    items:
    - apiVersion: v1
      kind: Node
      metadata:
        annotations:
          node.alpha.kubernetes.io/ttl: "0"
          volumes.kubernetes.io/controller-managed-attach-detach: "true"
        creationTimestamp: 2017-11-29T07:24:59Z
        labels:
          beta.kubernetes.io/arch: amd64
          beta.kubernetes.io/os: linux
          failure-domain.beta.kubernetes.io/region: regionOne
          failure-domain.beta.kubernetes.io/zone: zone0
          is_control: "true"
          kubernetes.io/hostname: bcmt-control-01
        name: bcmt-control-01
        namespace: ""
        resourceVersion: "10714282"
        selfLink: /api/v1/nodes/bcmt-control-01
        uid: 68011206-d4d6-11e7-9c63-fa163e2e2123
      spec:
        externalID: 41c40aab-80e9-4bb6-a280-27976bfc811f
        providerID: openstack:///41c40aab-80e9-4bb6-a280-27976bfc811f
        taints:
        - effect: NoExecute
          key: is_control
          timeAdded: null
          value: "true"
      status:
        addresses:
        - address: 172.16.1.12
          type: InternalIP
        - address: 10.5.138.49
          type: InternalIP
        - address: bcmt-control-01
          type: Hostname
        allocatable:
          cpu: "2"
          memory: 3779500Ki
          pods: "110"
        capacity:
          cpu: "2"
          memory: 3881900Ki
          pods: "110"
        conditions:
        - lastHeartbeatTime: 2018-02-15T12:08:58Z
          lastTransitionTime: 2018-02-14T13:39:53Z
          message: kubelet has sufficient disk space available
          reason: KubeletHasSufficientDisk
          status: "False"
          type: OutOfDisk
        - lastHeartbeatTime: 2018-02-15T12:08:58Z
          lastTransitionTime: 2018-02-14T13:39:53Z
          message: kubelet has sufficient memory available
          reason: KubeletHasSufficientMemory
          status: "False"
          type: MemoryPressure
        - lastHeartbeatTime: 2018-02-15T12:08:58Z
          lastTransitionTime: 2018-02-14T13:39:53Z
          message: kubelet has no disk pressure
          reason: KubeletHasNoDiskPressure
          status: "False"
          type: DiskPressure
        - lastHeartbeatTime: 2018-02-15T12:08:58Z
          lastTransitionTime: 2018-02-14T13:39:53Z
          message: kubelet is posting ready status
          reason: KubeletReady
          status: "True"
          type: Ready
        daemonEndpoints:
          kubeletEndpoint:
            Port: 10250
        images:
        - names:
          - 172.16.1.4:5000/gcr.io/google-containers/hyperkube-amd64
          - 172.16.1.4:5000/gcr.io/google-containers/hyperkube-amd64:v1.7.4
          sizeBytes: 615424570
        nodeInfo:
          architecture: amd64
          bootID: 883c98a9-17ea-40f9-af7d-a448ad817249
          containerRuntimeVersion: docker://1.12.6
          kernelVersion: 3.10.0-514.21.1.el7.x86_64
          kubeProxyVersion: v1.7.4
          kubeletVersion: v1.7.4
          machineID: 10783ea106f742728fede153a98b035d
          operatingSystem: linux
          osImage: Red Hat Enterprise Linux Server 7.3 (Maipo)
          systemUUID: 41C40AAB-80E9-4BB6-A280-27976BFC811F

Relevant data will be extracted:
 - Creation Timestamp.
 - Name.
 - Addresses (IP's)
 - kubernetes ID (uid)
 - provider
 - providerID


Alternatives
------------

None

Data model impact
-----------------
New vertices will be added to the entity graph. This might be duplicate vertices.
(VM's from Nova and kubernetes).
Proposed solution is resource equivalence. (planned for future work)

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

Kubernetes driver should be configured.(get access to master node)

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
  Idan-kinory

Work Items
----------

None

Dependencies
============

None

Testing
=======

This blueprint requires unit tests.

Documentation Impact
====================

Datasource configuration.

References
==========

Datasource main blueprint:
https://blueprints.launchpad.net/vitrage/+spec/k8s-datasource