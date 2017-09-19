OMERO.server upgrade
====================

The OME team is committed to providing frequent, project-wide upgrades both
with bug fixes and new functionality. We try to make the schedule for these
releases as public as possible. You may want to take a look at the `Trello
boards <https://trello.com/b/4EXb35xQ/getting-started>`_ for exactly what will
go into a release.

See the full details of OMERO |release| features in the :doc:`/users/history`.

This guide aims to be as definitive as possible so please do not be put off by
the level of detail; upgrading should be a straightforward process.

Upgrade check list
------------------

.. contents::
    :local:
    :depth: 1

Check prerequisities
^^^^^^^^^^^^^^^^^^^^

Before starting the upgrade, please ensure that you have reviewed and
satisfied all the :doc:`system requirements <system-requirements>` with
:doc:`correct versions <version-requirements>` for installation. In
particular, ensure that you are running a suitable version of PostgreSQL
to enable successful upgrading of the database, otherwise the upgrade
script aborts with a message saying that your database server version is
less than the OMERO prerequisite. If you are upgrading from a version
earlier than OMERO 5.0 then first review the `5.0 upgrade notes
<https://docs.openmicroscopy.org/omero/5.0.0/sysadmins/server-upgrade.html>`_
regarding previous changes in OMERO.

File limits
^^^^^^^^^^^

You may wish to review the open file limits. Please consult the
:ref:`limitations-openfiles` section for further details.

Password usage
^^^^^^^^^^^^^^

The passwords and logins used here are examples. Please consult the
:ref:`troubleshooting-password` section for explanation. In particular, be
sure to replace the values of **db_user** and **omero_database** with the
actual database user and database name for your installation.

Memoization files invalidation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All cached Bio-Formats memoization files created at import time will be
invalidated by the server upgrade. This means the very first loading of each
image after upgrade will be slower. After re-initialization, a new memoization
file will be automatically generated and OMERO will be able to load images in
a performant manner again.

These files are stored under :file:`BioFormatsCache` in the OMERO data
directory, e.g. :file:`/OMERO/BioFormatsCache`. You may see error messages in
your log files when an old memoization file is found; to avoid these messages
delete everything under this directory before starting the upgraded server.

Troubleshooting
^^^^^^^^^^^^^^^

If you encounter errors during an OMERO upgrade, database upgrade, etc., you
should retain as much log information as possible and notify the OMERO.server
team via the mailing lists available on the :community:`support <>`
page.

Upgrade check
^^^^^^^^^^^^^

All OMERO products check themselves with the OmeroRegistry for update
notifications on startup. If you wish to disable this functionality you should
do so now as outlined on the :doc:`UpgradeCheck` page.

Upgrade steps
-------------

For all users, the basic workflow for upgrading your OMERO.server is listed
below. Please refer to each section for additional details.

.. contents::
    :local:
    :depth: 1

Check ahead for upgrade issues
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is a ``precheck`` SQL script provided that performs various database
checks to verify readiness for upgrade. The precheck script works even
with the OMERO server running so it may be used before downtime for the
actual upgrade is scheduled. Issues that the script reports will need to
be resolved before the upgrade may proceed. The precheck script will
**not** make any changes to the database: it merely performs various
precautionary checks also done by the actual upgrade script.

.. parsed-literal::

    $ cd OMERO.server
    $ psql -h localhost -U **db_user** **omero_database** < sql/psql/|current_dbver|/|previous_dbver|-precheck.sql
    Password for user **db_user**:
    ...
    ...
                               status
    ---------------------------------------------------------------------
                                                                        +
                                                                        +
                                                                        +
    YOUR DATABASE IS READY FOR UPGRADE TO VERSION |current_dbver|           +
                                                                        +
                                                                        +

    (1 row)


.. _back-up-the-db:

Perform a database backup
^^^^^^^^^^^^^^^^^^^^^^^^^

The first thing to do before **any** upgrade activity is to backup your
database.

.. parsed-literal::

    $ pg_dump -h localhost -U **db_user** -Fc -f before_upgrade.db.dump **omero_database**


Copy new binaries
^^^^^^^^^^^^^^^^^

Before copying the new binaries, stop the existing server::

    $ cd OMERO.server
    $ bin/omero web stop
    $ bin/omero admin stop

Your OMERO configuration is stored using :file:`config.xml` in the
:file:`etc/grid` directory under your OMERO.server directory. Assuming you
have not made any file changes within your OMERO.server distribution
directory, you are safe to follow the following upgrade procedure:

.. parsed-literal::

    $ cd ..
    $ mv OMERO.server OMERO.server-old
    $ unzip OMERO.server-|release|-ice3x-byy.zip
    $ ln -s OMERO.server-|release|-ice3x-byy OMERO.server
    $ cp OMERO.server-old/etc/grid/config.xml OMERO.server/etc/grid

.. note::
    ``ice3x`` and ``byy`` **need to be replaced** by the appropriate Ice
    version and build number of OMERO.server.

.. _upgradedb:

Upgrade your database
^^^^^^^^^^^^^^^^^^^^^

.. only:: point_release

    .. warning::
        This section only concerns users upgrading from a |previousversion| or
        earlier server. If upgrading from a |version| server, you do not need
        to upgrade the database.

Ensure Unicode character encoding
"""""""""""""""""""""""""""""""""

Versions of OMERO from 5.1.0 onwards require a Unicode-encoded database;
without it, the upgrade script aborts with a message warning how the ``OMERO
database character encoding must be UTF8``. From :command:`psql`::

  # SELECT datname, pg_encoding_to_char(encoding) FROM pg_database;
    datname   | pg_encoding_to_char
  ------------+---------------------
   template1  | UTF8
   template0  | UTF8
   postgres   | UTF8
   omero      | UTF8
  (4 rows)

Alternatively, simply run :command:`psql -l` and check the output. If
your OMERO database is not Unicode-encoded with ``UTF8`` then it must be
re-encoded.

If you have the :command:`pg_upgradecluster` command available then its
``--locale`` option can effect the change in encoding. Otherwise,
create a Unicode-encoded dump of your database: dump it :ref:`as before
<back-up-the-db>` but to a different dump file and with an additional
``-E UTF8`` option. Then, create a Unicode-encoded database for
OMERO and restore that dump into it with :command:`pg_restore`,
similarly to :ref:`effecting a rollback <restore-the-db>`. If required
to achieve this, the ``-E UTF8`` option is accepted by both
:command:`initdb` and :command:`createdb`.

Run the upgrade script
""""""""""""""""""""""

You **must** use the same username and password you have defined during
:doc:`unix/server-installation`. For a large production system you
should plan for the fact that the upgrade script may take several hours
to run.

.. parsed-literal::

    $ cd OMERO.server
    $ psql -h localhost -U **db_user** **omero_database** < sql/psql/|current_dbver|/|previous_dbver|.sql
    Password for user **db_user**:
    ...
    ...
                               status
    ---------------------------------------------------------------------
                                                                        +
                                                                        +
                                                                        +
    YOU HAVE SUCCESSFULLY UPGRADED YOUR DATABASE TO VERSION |current_dbver| +
                                                                        +
                                                                        +

    (1 row)


If you are upgrading from a server earlier than |previousversion| then
it suffices to run the earlier upgrade scripts in sequence before the
one above. There is no need to download and run the server from an
intermediate major release.

.. note::

   If you perform the database upgrade using *SQL shell*, make sure you are
   connected to the database using **db_user** before running the script. See
   :forum:`this forum thread <viewtopic.php?f=5&t=7778>` for more information.

Delete certain annotations (optional)
"""""""""""""""""""""""""""""""""""""

For various reasons, production databases may accumulate non-sharable
annotations that are orphaned. These are :doc:`structured annotations
</developers/Model/StructuredAnnotations>` that are 'basic' (`Boolean`,
`Timestamp`, `Term`), 'numeric' (`Double`, `Long`), or `Comment`, and
that are *not* annotating an object. An illustrative example is that
deleting a rating in OMERO.insight 5.2 may have left behind the
corresponding `Long` annotation that captured the rating's number of
stars. Non-sharable annotations, like comments and ratings, cannot be
viewed from OMERO.insight or OMERO.web after they have been orphaned
because they are no longer associated with any model object such as an
image. The deletion script does *not* delete annotations that have a
custom/non-OME namespace (ns) set.

.. parsed-literal::

    $ cd OMERO.server
    $ psql -h localhost -U **db_user** **omero_database** < sql/psql/|current_dbver|/delete-ns-orphans.sql

This script may be used during some maintenance window subsequent to the
actual upgrade as long as it runs on a |current_dbver| database. If at
upgrade time you have questions about the script then you may perform
further research before :ref:`backing up the database again
<back-up-the-db>` then running the script. There is no requirement to
ever use it.

Optimize an upgraded database (optional)
""""""""""""""""""""""""""""""""""""""""

After you have run the upgrade script, you may want to optimize your
database which can both save disk space and speed up access times.

.. parsed-literal::

    $ psql -h localhost -U **db_user** **omero_database** -c 'VACUUM FULL VERBOSE ANALYZE;'

Reset ROI shape color (optional)
""""""""""""""""""""""""""""""""

Regions of interest generated using OMERO.insight did not follow the
OME model specification. The color of shapes is now handled according
to the data model, using RGBA rather than ARGB format. 
A script is provided to upgrade the color settings of shapes created using OMERO.insight
or a different client that saved them in the ARGB format. If you only wish to apply the changes
to a subset of ROIs, edit the ``TRUE`` in the script below before running it:

.. parsed-literal::

    $ psql -h localhost -U **db_user** **omero_database** < sql/psql/|current_dbver|/shape_color_argb_to_rgba.sql

If you need to roll back the changes, a reverse script ``reverse_shape_color_argb_to_rgba.sql``
is also available in the same folder. Edit the ``TRUE`` if you wish to roll back the changes to a
subset of ROIs.

Move annotation from Image to Well (optional)
"""""""""""""""""""""""""""""""""""""""""""""

Since 5.3, users can annotate Wells in both OMERO.web and OMERO.insight. Previously they could only
annotate Images linked to WellSamples.
The official script `Move_Annotations.py <https://github.com/ome/scripts/blob/develop/omero/util_scripts/Move_Annotations.py>`_
can be run by users to move annotations from Images to Wells but an administrator can run the
script on any user's data.


.. _upgrademergescript:

Merge script changes
^^^^^^^^^^^^^^^^^^^^

If any new official scripts have been added under ``lib/scripts`` or if
you have modified any of the existing ones, then you will need to backup
your modifications. Doing this, however, is not as simple as copying the
directory over since the core developers will have also improved these
scripts. In order to facilitate saving your work, we have turned the
scripts into a Git submodule which can be found at
`<https://github.com/ome/scripts>`_.

For further information on managing your scripts, refer to
:doc:`installing-scripts`. If you require help, please contact the OME
developers.

Update your environment variables and memory settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Environment variables
"""""""""""""""""""""

If you changed the directory name where the |release| server code resides,
make sure to update any system environment variables. Before restarting
the server, make sure your PATH and PYTHONPATH system environment
variables are pointing to the new locations.

JVM memory settings
"""""""""""""""""""

Your memory settings should be copied along with :file:`etc/grid/config.xml`,
but you can check the current settings by running :program:`omero admin jvmcfg`.
See :ref:`jvm_memory_settings` for more information.

Restart your server
^^^^^^^^^^^^^^^^^^^

-  Following a successful database upgrade, you can start the server.

   .. parsed-literal::

       $ cd OMERO.server
       $ bin/omero admin start

-  If anything goes wrong, please send the output of
   :program:`omero admin diagnostics` to
   ome-users@lists.openmicroscopy.org.uk.

.. _restore-the-db:

Restore a database backup
^^^^^^^^^^^^^^^^^^^^^^^^^

If the upgraded database or the new server version do not work for you,
or you otherwise need to rollback to a previous database backup, you may
want to restore a database backup. To do so, create a new database,

.. parsed-literal::

    $ createdb -h localhost -U postgres -E UTF8 -O **db_user** omero_from_backup

restore the previous archive into this new database,

::

    $ pg_restore -Fc -d omero_from_backup before_upgrade.db.dump

and configure your server to use it.

::

    $ bin/omero config set omero.db.name omero_from_backup
