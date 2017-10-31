..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================
Add CRUD support for template addition
======================================

https://blueprints.launchpad.net/vitrage/+spec/crud-templates


Adding CRUD support means that templates can be added/removed in real time when Vitrage is up.

Templates added/removed via API will be stored in the database so they remain after restarting Vitrage.
sqlalchemy will be used for DB management.

Problem description
===================

Currently, Vitrage templates are loaded from a specific folder during Vitrage startup.
Adding/removing templates while Vitrage services are running requires restart of the vitrage-graph service.


Proposed change
===============
Templates should be stored in the database instead of specific folder in file system,
In that way they can be modified (add/delete) while vitrage is up.

The evaluator will preform a live update to the entity graph according to actions specified in the added/removed templates.


Template add:

- Validate template.
- Store template in database.
- Notify evaluator.
- Entity graph evaluation with new actions.

Delete:

- Delete template from database.
- Notify evaluator.
- Entity graph evaluation to undo templates actions.


Update - not supported at this stage.
In order to update template use add and delete template.

A few changes should be made in order to implement CRUD support

- Support template Add/Remove commands in vitrageclient (Vitrage API)
- API handler: Store/Delete Templates to and from the Database.
- Graph cloning logic should be extracted to base class.
- Add new DB table called templates

Template DB Table:

::

   +----------------+--------------+------+-----+---------+-------+
   | Field          | Type         | Null | Key | Default | Extra |
   +----------------+--------------+------+-----+---------+-------+
   | created_at     | datetime     | YES  |     | NULL    |       |
   | updated_at     | datetime     | YES  |     | NULL    |       |
   | id             | varchar(64)  | NO   | PRI | NULL    |       |
   | status         | varchar(16)  | YES  |     | NULL    |       |
   | status_details | varchar(128) | YES  |     | NULL    |       |
   | name           | varchar(128) | NO   |     | NULL    |       |
   | file_content   | text         | NO   |     | NULL    |       |
   | type           | varchar(64)  | YES  |     | NULL    |       |
   +----------------+--------------+------+-----+---------+-------+



Alternatives
------------

None

Data model impact
-----------------

A new database table.

REST API impact
---------------

PUT and DELETE methods will be added.


Versioning impact
-----------------

None

Other end user impact
---------------------

New CLI commands added:

Vitrage template add

Vitrage template delete

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

Primary assignee:
  ikinory

Other contributors:
  None

Work Items
----------

- API support for add/remove template
- implement database table via SQLAlchemy.
- implement queries to database.
- Tests as explained in "Testing".


Dependencies
============

None

Testing
=======

API:
 - template add:
    * add all types of templates : standard, equivalence, definition.
    * add corrupted template and check for failed to add.
    * add a folder of templates.

 - template delete:
    * check all types of templates : standard, equivalence, definition.
 - template list.
 - template show:
    * compare cli template content to original file content

e2e:
 evaluate the added/ deleted templates on the entire graph.
  - test evaluator reload templates:

 example:
  1.raise trigger alarm (template is not loaded yet).

  2.add the relevant template.

  3.check action is executed.

  This checks that the evaluators are reloaded and run on all existing vertices.

Documentation Impact
====================

Template add and delete should be added. Modify template validate and list.

Changes should be added to:

API description: vitrage/doc/source/contributor/vitrage-api.rst

CLI description: doc/source/contributor/cli.rst

References
==========

None
