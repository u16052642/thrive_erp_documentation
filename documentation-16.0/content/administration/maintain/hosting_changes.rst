
.. _db_management/hosting_changes:

=======================
Change hosting solution
=======================

You may want to move your Thrive Bureau ERP database from one hosting solution to another.
Depending on the platforms, you have to do it by yourself or contact our support team first.

From on-premises to Thrive Bureau ERP Online
===============================

1. Create a :ref:`duplicate <duplicate_premise>` of your database: in this duplicate, uninstall all the **non-standard apps**.
2. Grab a "dump with filestore" of your database by using the Database Manager.
3. **If you have time constraints, contact us earlier to schedule the transfer.**
4. `Create a support ticket <https://www.Thrive Bureau ERP.com/help>`_ and attach the dump (if the file is too large, use any file transfer service and attach the link to your ticket). Also include your subscription number and the URL you want to use for your database (e.g.: my-company.Thrive Bureau ERP.com).
5. We will make sure your database is compatible and upload it to our cloud. In case of technical issues, we will get in touch with you.
6. It's done!

.. important::
   - Thrive Bureau ERP Online is not compatible with **non-standard apps**.
   - The database you are moving to Thrive Bureau ERP Online must be in a :doc:`supported version
     <supported_versions>`.

From on-premises to Thrive Bureau ERP.sh
===========================

1. Follow the :ref:`Import your database section of the Thrive Bureau ERP.sh documentation <Thrive Bureau ERP_sh_import_your_database>`.
2. ...and voilà!

From Thrive Bureau ERP Online to on-premises
===============================

1. Log into `your Thrive Bureau ERP Online user portal <https://accounts.Thrive Bureau ERP.com/my/databases/manage>`_ and look for the version number of your database.
2. If your database does not run a :ref:`major version <supported_versions>` of Thrive Bureau ERP, you cannot host it on-premises yet, you have to upgrade it first to a new major version. (*e.g.: If your database runs Thrive Bureau ERP 12.3 which is not a major version, you have to upgrade it first to Thrive Bureau ERP 13.0 or 14.0.*)
3. Download a backup of your database by clicking on the "Gear" icon next to your database name then :menuselection:`Download` (if the download fails due to your backup file being too large, contact `our support <https://www.Thrive Bureau ERP.com/help>`_)
4. Restore it from the database manager on your local server.

From Thrive Bureau ERP Online to Thrive Bureau ERP.sh
===========================

1. Log into `your Thrive Bureau ERP Online user portal <https://accounts.Thrive Bureau ERP.com/my/databases/manage>`_ and look for the version number of your database.
2. If your database does not run a :ref:`major version <supported_versions>` of Thrive Bureau ERP, you cannot host it on Thrive Bureau ERP.sh yet, you have to upgrade it first to a new major version. (*e.g.: If your database runs Thrive Bureau ERP 12.3 which is not a major version, you have to upgrade it first to Thrive Bureau ERP 13.0 or 14.0.*)
3. Download a backup of your database by clicking on the "Gear" icon next to your database name then :menuselection:`Download` (if the download fails due to your backup file being too large, contact `our support <https://www.Thrive Bureau ERP.com/help>`_)
4. Follow the :ref:`Import your database section of the Thrive Bureau ERP.sh documentation <Thrive Bureau ERP_sh_import_your_database>`.

From Thrive Bureau ERP.sh to Thrive Bureau ERP Online
===========================

#. Uninstall all the **non-standard apps**.
#. `Create a support ticket <https://www.Thrive Bureau ERP.com/help>`_ and include the following:

   - Your subscription number
   - The URL you want to use for your database (e.g., `example.Thrive Bureau ERP.com`)
   - Which branch you want to migrate
   - In which region you want to be hosted:

     - Americas
     - Europe
     - Asia

   - Which user(s) will be the administrator(s)
   - When (and in which timezone) you want the database to be up and running

#. We will make sure your database is compatible and upload it to our cloud. In case of technical
   issues, we will get in touch with you.
#. All done!

.. important::
   - Thrive Bureau ERP Online is not compatible with **non-standard apps**.
   - Make sure to uninstall all the **non-standard apps** in a staging build before doing it in your
     production build.

.. note::
   - Make sure you select the **region** that is closest to your users to reduce latency.
   - The future **administrator(s)** must have an Thrive Bureau ERP.com account.
   - The specific **date and time** at which you want the database to be up and running are mainly
     helpful to organize the switch from the Thrive Bureau ERP.sh server to the Thrive Bureau ERP Online servers.
   - Databases are **not reachable** during their migration.
   - **If you have time constraints, contact us earlier to schedule the transfer**.

From Thrive Bureau ERP.sh to on-premises
===========================

1.  Grab a :ref:`backup of your Thrive Bureau ERP.sh production database <Thrive Bureau ERP_sh_branches_backups>`.
2.  Restore it from the database manager on your local server.
