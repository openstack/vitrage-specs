..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Vitrage Alarm Counts API
========================

Extend the Vitrage REST API to support a GET of the Active Alarm Counts, for
each alarm severity level, in Vitrage.


Problem description
===================
Provide REST API access to the Vitrage Active Alarm Counts in support of the
Horizon blueprint, "Vitrage Alarm Banner in Top Navbar".


Proposed change
===============
Support the following REST API to Vitrage for calculating and returning
the Active Alarm Counts for each alarm severity level::

	GET /v1/alarm/count

	Headers
		X-Auth-Token (string, required) - Keystone auth token
		Accept (string) - application/json

	Path Parameters
		None.

	Query Parameters
		None.

	Request Body
		all_tenants - (boolean, optional) shows the alarm counts
                              summed across all tenants (in case the user
			      has the permissions).

	Request Examples
		GET /v1/alarm/count HTTP/1.1
		Host: 135.248.19.18:8999
		X-Auth-Token: 2b8882ba2ec44295bf300aecb2caa4f7
		Accept: application/json

	Response Status code
		200 - OK

	Response Body
		Returns a JSON object containing the alarm counts for the
		different alarm severities.

	Response Examples
		{
		  "critical_alarm_count": 1,
		  "major_alarm_count": 0,
		  "minor_alarm_count": 1,
		  "warning_alarm_count": 3
		}

NOTE: The vitrage CLI and client will be updated for this new API.
e.g. "vitrage alarm count"



Alternatives
------------
For performance reasons, maintain the Active Alarm Counts in Vitrage Entity
Graph, and just return these counts when the REST API command is received.

Although decided against this due to:

- keeping a counter, in addition to the graph, might be buggy (multi threading
  issues etc.),
- calculating the counter means traversing once all of the vertices in the graph,
  get all alarms, and count. It shouldn't be too expensive, it's just like
  'get alarms' api,
- since the result of this api is used for ui query (and not for notification or
  corrective actions for example), the performance is not that critical.

Data model impact
-----------------
None

REST API impact
---------------
Extending REST API with new GET /v1/alarm/count API.

Versioning impact
-----------------
None ... just extending API, no changes.

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
Horizon will use this API to populate the counts in its new "Vitrage
Alarm Banner" in its Top Navbar; a proposed Horizon blueprint.



Implementation
==============

Assignee(s)
-----------
Primary assignee:
  gwaines

Other contributors:
  None

Work Items
----------
- Implement new REST API in Vitrage API:  GET /v1/alarm/count API,
  to calculate and return the Vitrage Active Alarm Counts for each
  alarm severity level,
- Update Vitrage client for new API
- Add the new "vitrage alarm count" CLI command


Dependencies
============
None


Testing
=======
The changes shall be covered by new unit test and tempest test.


Documentation Impact
====================
Update to Vitrage API Documentation; i.e. the new API will be added under
https://github.com/openstack/vitrage/blob/master/doc/source/contributor/vitrage-api.rst


References
==========
None.

