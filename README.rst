========================
Team and repository tags
========================

.. image:: https://governance.openstack.org/tc/badges/vitrage-specs.svg
    :target: https://governance.openstack.org/tc/reference/tags/index.html

.. Change things from this point on

================================
OpenStack Vitrage Specifications
================================

This git repository is used to hold approved design specifications for additions
to the Vitrage project. Reviews of the specs are done in gerrit, using a similar
workflow to how we review and merge changes to the code itself.

The layout of this repository is::

  specs/<release>/

Where there are two sub-directories:

  specs/<release>/approved: specifications approved but not yet implemented
  specs/<release>/implemented: implemented specifications

This directory structure allows you to see what we thought about doing,
decided to do, and actually got done. Users interested in functionality in a
given release should only refer to the ``implemented`` directory.

You can find an example spec in `doc/source/specs/template.rst`.

Specifications are proposed for a given release by adding them to the
`specs/<release>` directory and posting it for review.  The implementation
status of a blueprint for a given release can be found by looking at the
blueprint in launchpad.  Not all approved blueprints will get fully implemented.

Specifications have to be re-proposed for every release.  The review may be
quick, but even if something was previously approved, it should be re-reviewed
to make sure it still makes sense as written.

Launchpad blueprints::

  http://blueprints.launchpad.net/vitrage

Starting from the Mitaka-1 development milestone Vitrage performs the pilot of
the specs repos approach.

Please note, Launchpad blueprints are still used for tracking the
current status of blueprints. For more information, see::

  https://wiki.openstack.org/wiki/Blueprints

For more information about working with gerrit, see::

  http://docs.openstack.org/infra/manual/developers.html#development-workflow

To validate that the specification is syntactically correct (i.e. get more
confidence in the Jenkins result), please execute the following command::

  $ tox

After running ``tox``, the documentation will be available for viewing in HTML
format in the ``doc/build/`` directory.

To build the document automatically on changes, use the command::

  $ tox -e autobuild

Then open in a browser http://localhost:8000
