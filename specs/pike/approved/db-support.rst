..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========
DB Support
==========

https://blueprints.launchpad.net/vitrage/+spec/db-support

There is a need in Vitrage for persistent DB. The main reasons are history and
high availability, but there are additional uses for persistent DB, such as
saving the registered rest notifications recipients, and improving performance
using DB capabilities.
We will use sqlalchemy for DB management, same as Nova, Heat, Aodh and other
projects.


Problem description
===================

Vitrage should support persistent db management.


Proposed change
===============

The development will happen in a few stages. To enable faster development
of Vitrage History / HA / Vitrage notifications, in the first stage only basic
sqlalchemy support will be developed.

Note : A good example of sqlalchemy implementation can be found in :

https://github.com/openstack/aodh/blob/master/aodh/storage/impl_sqlalchemy.py

|

Basic configuration in vitrage.conf :

# The SQLAlchemy connection string to use to connect to the database.

# Example:

# connection = mysql://root:pass@127.0.0.1:8999/vitrage

#connection = <None>

|

# The SQL mode to be used for MySQL sessions. This option, including the

# default, overrides any server-set SQL mode. To use whatever SQL mode is set

# by the server configuration, set this to no value. Example: mysql_sql_mode=

# (string value)

# Additonal modes and their functionality :

# https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sql-mode-combo

#mysql_sql_mode = TRADITIONAL

|

# Timeout before idle SQL connections are reaped. (integer value)

#idle_timeout = 3600

|

# Minimum number of SQL connections to keep open in a pool. (integer value)

#min_pool_size = 1

|

# Maximum number of SQL connections to keep open in a pool. Setting a value of

# 0 indicates no limit. (integer value)

#max_pool_size = 5

|

# Maximum number of database connection retries during startup. Set to -1 to

# specify an infinite retry count. (integer value)

#max_retries = 10

|

# Interval between retries of opening a SQL connection. (integer value)

#retry_interval = 10

|

# If set, use this value for max_overflow with SQLAlchemy. (integer value)

#max_overflow = 50

|

# Verbosity of SQL debugging information: 0=None, 100=Everything. (integer

# value)

# Minimum value: 0

# Maximum value: 100

#connection_debug = 0

|

# Add Python stack traces to SQL as comment strings. (boolean value)

#connection_trace = false

|

# If set, use this value for pool_timeout with SQLAlchemy. (integer value)

#pool_timeout = <None>

|

# Enable the experimental use of database reconnect on connection lost.

# (boolean value)

#use_db_reconnect = false

|

# Seconds between retries of a database transaction. (integer value)

#db_retry_interval = 1

|

# If True, increases the interval between retries of a database operation up to

# db_max_retry_interval. (boolean value)

#db_inc_retry_interval = true

|

# If db_inc_retry_interval is set, the maximum seconds between retries of a

# database operation. (integer value)

#db_max_retry_interval = 10

|

# Maximum retries in case of connection error or deadlock error before error is

# raised. Set to -1 to specify an infinite retry count. (integer value)

#db_max_retries = 20


------
Step 1
------

Build a basic sqlalchemy implementation for connecting to the DB,
Create a Vitrage schema in the db and required tables (from Vitrage history
spec and Vitrage httppost registration spec). Each developer should add
and test his own tables to the schema.

The schema and tables should be created dynamically from Vitrage, each time
Vitrage loads. Vitrage will test existence of the database tables and if they
do not exist, create them, using a "model" data representation.
Testing validity of existing tables or even fixing them is not required in the
first stage.

Cinder Data Model example :
https://github.com/openstack/cinder/blob/master/cinder/db/sqlalchemy/models.py

Create DAO that will perform hard coded CRUD on the tables.

------
Step 2
------

- Upgrade / versioning with alembic.

- Generic sql support.

- Pagination for big queries, such as all of the entity graph or events table.


Alternatives
============

Not today.

Data model impact
=================

This whole design is one big Data Model impact.

REST API impact
===============

None

Security impact
===============

None

Pipeline impact
===============

None

Other end user impact
=====================

None

Performance/Scalability Impacts
===============================

Proper Indexes should be applied, and, according to performance testing
during development, multiply table data to smaller indexed tables for better data
polling performance.


Other deployer impact
=====================

None

Developer impact
================

None


Implementation
==============

Assignee(s)
===========

Primary assignees:
danoffek.
alexey_weyl.

Work Items
==========

- Create a basic SQLAlchemy implementation to connect to the DB.

- Create Data Model for events and other tables.

- Create simple CRUD methods for event storage.

- CRUD templates

- Tests as explained in "Testing".


Future lifecycle
================

See "Step 2"

Dependencies
============

None

Testing
=======

Unit tests:

- Data selection queries

- Data update

- Table create with indexes

- (should not run in the gate or regularly) Performance tests on large table data with
  multiple inserts per seconds over a period of an hour.

|

Tempest tests: Each developer should create his own DB tempest tests.
Example: Create alarms / resources events, and poll the system afterwards. In case the data
wasn't stored in the events table properly, errors should be issued.

Documentation Impact
====================
The DB configuration should be documented in an .rst file, and there should be a
link to it from
https://github.com/openstack/vitrage/blob/master/doc/source/installation-and-configuration.rst

References
==========