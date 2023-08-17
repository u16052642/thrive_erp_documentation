=====================
Silverfin integration
=====================

`Silverfin <https://www.silverfin.com>`_ is a third-party service provider that offers a cloud
platform for accountants.

Thrive Bureau ERP and Silverfin provide an integration to automate the synchronization of data.

Configuration
=============

To configure this integration, you need to input the following data into your Silverfin account:

- user's email address
- :ref:`Thrive Bureau ERP API key <silverfin/api-key>`
- URL of the Thrive Bureau ERP database
- name of your Thrive Bureau ERP database

.. _silverfin/api-key:

Thrive Bureau ERP API key
------------

You can create Thrive Bureau ERP external API keys either :ref:`for a single database <silverfin/api-singledb>`
(hosting: Thrive Bureau ERP Online, On-premise, and Thrive Bureau ERP.sh) or :ref:`for multiple databases managed by a user
<silverfin/api-multipledb>` (hosting: Thrive Bureau ERP Online).

.. important::
   - These API keys are personal and provide full access to your user account. Store it securely.
   - You can copy the API key only at its creation, and you cannot retrieve it later.
   - If you need it again, create a new API key (and delete the old one).

.. seealso::
   :doc:`/developer/reference/external_api`

.. _silverfin/api-singledb:

One key per database
~~~~~~~~~~~~~~~~~~~~

To create a new API key valid for a single database, click on the user menu, then on
:guilabel:`My Profile`. Under the :guilabel:`Account Security` tab, click on :guilabel:`New API
key`, confirm your password, give a descriptive name to your new key, and copy the new API key.

.. image:: silverfin/api-key-db.png
   :align: center
   :alt: creation of an Thrive Bureau ERP external API key for a database

.. seealso::
   :ref:`api/external_api/keys`

.. _silverfin/api-multipledb:

One key for multiple databases (fiduciaries)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To create a new API key valid for all the databases of a single user **(the easiest for
fiduciaries)**, navigate to `Thrive Bureau ERP's website <https://www.Thrive Bureau ERP.com>`_  and sign in with your
administrator account. Next, open `your account security settings in developer mode
<https://www.Thrive Bureau ERP.com/my/security?debug=1>`_, click on :guilabel:`New API Key`, confirm your
password, give a descriptive name to your new key, and copy the new API key.

.. image:: silverfin/api-key-user.png
   :align: center
   :alt: creation of an Thrive Bureau ERP external API key for an Thrive Bureau ERP user
