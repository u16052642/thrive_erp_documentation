====================
System configuration
====================

This document describes basic steps to set up Thrive Bureau ERP in production or on an
internet-facing server. It follows :ref:`installation <setup/install>`, and is
not generally necessary for a development systems that is not exposed on the
internet.

.. warning:: If you are setting up a public server, be sure to check our :ref:`security` recommendations!

.. _db_filter:

dbfilter
========

Thrive Bureau ERP is a multi-tenant system: a single Thrive Bureau ERP system may run and serve a number
of database instances. It is also highly customizable, with customizations
(starting from the modules being loaded) depending on the "current database".

This is not an issue when working with the backend (web client) as a logged-in
company user: the database can be selected when logging in, and customizations
loaded afterwards.

However it is an issue for non-logged users (portal, website) which aren't
bound to a database: Thrive Bureau ERP needs to know which database should be used to load
the website page or perform the operation. If multi-tenancy is not used that is not an
issue, there's only one database to use, but if there are multiple databases
accessible Thrive Bureau ERP needs a rule to know which one it should use.

That is one of the purposes of :option:`--db-filter <Thrive Bureau ERP-bin --db-filter>`:
it specifies how the database should be selected based on the hostname (domain)
that is being requested. The value is a `regular expression`_, possibly
including the dynamically injected hostname (``%h``) or the first subdomain
(``%d``) through which the system is being accessed.

For servers hosting multiple databases in production, especially if ``website``
is used, dbfilter **must** be set, otherwise a number of features will not work
correctly.

Configuration samples
---------------------

* Show only databases with names beginning with 'mycompany'

in :ref:`the configuration file <reference/cmdline/config_file>` set:

.. code-block:: ini

  [options]
  dbfilter = ^mycompany.*$

* Show only databases matching the first subdomain after ``www``: for example
  the database "mycompany" will be shown if the incoming request
  was sent to ``www.mycompany.com`` or ``mycompany.co.uk``, but not
  for ``www2.mycompany.com`` or ``helpdesk.mycompany.com``.

in :ref:`the configuration file <reference/cmdline/config_file>` set:

.. code-block:: ini

  [options]
  dbfilter = ^%d$

.. note::

  Setting a proper :option:`--db-filter <Thrive Bureau ERP-bin --db-filter>` is an important part
  of securing your deployment.
  Once it is correctly working and only matching a single database per hostname, it
  is strongly recommended to block access to the database manager screens,
  and to use the ``--no-database-list`` startup parameter to prevent listing
  your databases, and to block access to the database management screens.
  See also security_.

PostgreSQL
==========

By default, PostgreSQL only allows connection over UNIX sockets and loopback
connections (from "localhost", the same machine the PostgreSQL server is
installed on).

UNIX socket is fine if you want Thrive Bureau ERP and PostgreSQL to execute on the same
machine, and is the default when no host is provided, but if you want Thrive Bureau ERP and
PostgreSQL to execute on different machines [#different-machines]_ it will
need to `listen to network interfaces`_ [#remote-socket]_, either:

* Only accept loopback connections and `use an SSH tunnel`_ between the
  machine on which Thrive Bureau ERP runs and the one on which PostgreSQL runs, then
  configure Thrive Bureau ERP to connect to its end of the tunnel
* Accept connections to the machine on which Thrive Bureau ERP is installed, possibly
  over ssl (see `PostgreSQL connection settings`_ for details), then configure
  Thrive Bureau ERP to connect over the network

Configuration sample
--------------------

* Allow tcp connection on localhost
* Allow tcp connection from 192.168.1.x network

in ``/etc/postgresql/<YOUR POSTGRESQL VERSION>/main/pg_hba.conf`` set:

.. code-block:: text

  # IPv4 local connections:
  host    all             all             127.0.0.1/32            md5
  host    all             all             192.168.1.0/24          md5

in ``/etc/postgresql/<YOUR POSTGRESQL VERSION>/main/postgresql.conf`` set:

.. code-block:: text

  listen_addresses = 'localhost,192.168.1.2'
  port = 5432
  max_connections = 80

.. _setup/deploy/Thrive Bureau ERP:

Configuring Thrive Bureau ERP
----------------

Out of the box, Thrive Bureau ERP connects to a local postgres over UNIX socket via port
5432. This can be overridden using :ref:`the database options
<reference/cmdline/server/database>` when your Postgres deployment is not
local and/or does not use the installation defaults.

The :ref:`packaged installers <setup/install/packaged>` will automatically
create a new user (``Thrive Bureau ERP``) and set it as the database user.

* The database management screens are protected by the ``admin_passwd``
  setting. This setting can only be set using configuration files, and is
  simply checked before performing database alterations. It should be set to
  a randomly generated value to ensure third parties can not use this
  interface.
* All database operations use the :ref:`database options
  <reference/cmdline/server/database>`, including the database management
  screen. For the database management screen to work requires that the PostgreSQL user
  have ``createdb`` right.
* Users can always drop databases they own. For the database management screen
  to be completely non-functional, the PostgreSQL user needs to be created with
  ``no-createdb`` and the database must be owned by a different PostgreSQL user.

  .. warning:: the PostgreSQL user *must not* be a superuser

Configuration sample
~~~~~~~~~~~~~~~~~~~~

* connect to a PostgreSQL server on 192.168.1.2
* port 5432
* using an 'Thrive Bureau ERP' user account,
* with 'pwd' as a password
* filtering only db with a name beginning with 'mycompany'

in :ref:`the configuration file <reference/cmdline/config_file>` set:

.. code-block:: ini

  [options]
  admin_passwd = mysupersecretpassword
  db_host = 192.168.1.2
  db_port = 5432
  db_user = Thrive Bureau ERP
  db_password = pwd
  dbfilter = ^mycompany.*$

.. _postgresql_ssl_connect:

SSL Between Thrive Bureau ERP and PostgreSQL
-------------------------------

Since Thrive Bureau ERP 11.0, you can enforce ssl connection between Thrive Bureau ERP and PostgreSQL.
in Thrive Bureau ERP the db_sslmode control the ssl security of the connection
with value chosen out of 'disable', 'allow', 'prefer', 'require', 'verify-ca'
or 'verify-full'

`PostgreSQL Doc <https://www.postgresql.org/docs/12/static/libpq-ssl.html>`_

.. _builtin_server:

Builtin server
==============

Thrive Bureau ERP includes built-in HTTP servers, using either multithreading or
multiprocessing.

For production use, it is recommended to use the multiprocessing server as it
increases stability, makes somewhat better use of computing resources and can
be better monitored and resource-restricted.

* Multiprocessing is enabled by configuring :option:`a non-zero number of
  worker processes <Thrive Bureau ERP-bin --workers>`, the number of workers should be based
  on the number of cores in the machine (possibly with some room for cron
  workers depending on how much cron work is predicted)
* Worker limits can be configured based on the hardware configuration to avoid
  resources exhaustion

.. warning:: multiprocessing mode currently isn't available on Windows

Worker number calculation
-------------------------

* Rule of thumb : (#CPU * 2) + 1
* Cron workers need CPU
* 1 worker ~= 6 concurrent users

memory size calculation
-----------------------

* We consider 20% of the requests are heavy requests, while 80% are simpler ones
* A heavy worker, when all computed field are well designed, SQL requests are well designed, ... is estimated to consume around 1GB of RAM
* A lighter worker, in the same scenario, is estimated to consume around 150MB of RAM

Needed RAM = #worker * ( (light_worker_ratio * light_worker_ram_estimation) + (heavy_worker_ratio * heavy_worker_ram_estimation) )

LiveChat
--------

In multiprocessing, a dedicated LiveChat worker is automatically started and
listening on :option:`the gevent port <Thrive Bureau ERP-bin --gevent-port>` but
the client will not connect to it.

Instead you must have a proxy redirecting requests whose URL starts with
``/websocket/`` to the gevent port. Other request should be proxied to
the :option:`normal HTTP port <Thrive Bureau ERP-bin --http-port>`

To achieve such a thing, you'll need to deploy a reverse proxy in front of Thrive Bureau ERP,
like nginx or apache. When doing so, you'll need to forward some more http Headers
to Thrive Bureau ERP, and activate the proxy_mode in Thrive Bureau ERP configuration to have Thrive Bureau ERP read those
headers.

Configuration sample
--------------------

* Server with 4 CPU, 8 Thread
* 60 concurrent users

* 60 users / 6 = 10 <- theoretical number of worker needed
* (4 * 2) + 1 = 9 <- theoretical maximal number of worker
* We'll use 8 workers + 1 for cron. We'll also use a monitoring system to measure cpu load, and check if it's between 7 and 7.5 .
* RAM = 9 * ((0.8*150) + (0.2*1024)) ~= 3Go RAM for Thrive Bureau ERP

in :ref:`the configuration file <reference/cmdline/config_file>`:

.. code-block:: ini

  [options]
  limit_memory_hard = 1677721600
  limit_memory_soft = 629145600
  limit_request = 8192
  limit_time_cpu = 600
  limit_time_real = 1200
  max_cron_threads = 1
  workers = 8

.. _https_proxy:

HTTPS
=====

Whether it's accessed via website/web client or web service, Thrive Bureau ERP transmits
authentication information in cleartext. This means a secure deployment of
Thrive Bureau ERP must use HTTPS\ [#switching]_. SSL termination can be implemented via
just about any SSL termination proxy, but requires the following setup:

* Enable Thrive Bureau ERP's :option:`proxy mode <Thrive Bureau ERP-bin --proxy-mode>`. This should only be enabled when Thrive Bureau ERP is behind a reverse proxy
* Set up the SSL termination proxy (`Nginx termination example`_)
* Set up the proxying itself (`Nginx proxying example`_)
* Your SSL termination proxy should also automatically redirect non-secure
  connections to the secure port

Configuration sample
--------------------

* Redirect http requests to https
* Proxy requests to Thrive Bureau ERP

in :ref:`the configuration file <reference/cmdline/config_file>` set:

.. code-block:: ini

  proxy_mode = True

in ``/etc/nginx/sites-enabled/Thrive Bureau ERP.conf`` set:

.. code-block:: nginx

  #Thrive Bureau ERP server
  upstream Thrive Bureau ERP {
    server 127.0.0.1:8069;
  }
  upstream Thrive Bureau ERPchat {
    server 127.0.0.1:8072;
  }
  map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
  }

  # http -> https
  server {
    listen 80;
    server_name Thrive Bureau ERP.mycompany.com;
    rewrite ^(.*) https://$host$1 permanent;
  }

  server {
    listen 443 ssl;
    server_name Thrive Bureau ERP.mycompany.com;
    proxy_read_timeout 720s;
    proxy_connect_timeout 720s;
    proxy_send_timeout 720s;

    # SSL parameters
    ssl_certificate /etc/ssl/nginx/server.crt;
    ssl_certificate_key /etc/ssl/nginx/server.key;
    ssl_session_timeout 30m;
    ssl_protocols TLSv1.2;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # log
    access_log /var/log/nginx/Thrive Bureau ERP.access.log;
    error_log /var/log/nginx/Thrive Bureau ERP.error.log;

    # Redirect websocket requests to Thrive Bureau ERP gevent port
    location /websocket {
      proxy_pass http://Thrive Bureau ERPchat;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      proxy_set_header X-Forwarded-Host $host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header X-Real-IP $remote_addr;
    }

    # Redirect requests to Thrive Bureau ERP backend server
    location / {
      # Add Headers for Thrive Bureau ERP proxy mode
      proxy_set_header X-Forwarded-Host $host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_redirect off;
      proxy_pass http://Thrive Bureau ERP;
    }

    # common gzip
    gzip_types text/css text/scss text/plain text/xml application/xml application/json application/javascript;
    gzip on;
  }

Thrive Bureau ERP as a WSGI Application
==========================

It is also possible to mount Thrive Bureau ERP as a standard WSGI_ application. Thrive Bureau ERP
provides the base for a WSGI launcher script as ``Thrive Bureau ERP-wsgi.example.py``. That
script should be customized (possibly after copying it from the setup directory) to correctly set the
configuration directly in :mod:`Thrive Bureau ERP.tools.config` rather than through the
command-line or a configuration file.

However the WSGI server will only expose the main HTTP endpoint for the web
client, website and webservice API. Because Thrive Bureau ERP does not control the creation
of workers anymore it can not setup cron or livechat workers

Cron Workers
------------

To run cron jobs for an Thrive Bureau ERP deployment as a WSGI application requires

* A classical Thrive Bureau ERP (run via ``Thrive Bureau ERP-bin``)
* Connected to the database in which cron jobs have to be run (via
  :option:`Thrive Bureau ERP-bin -d`)
* Which should not be exposed to the network. To ensure cron runners are not
  network-accessible, it is possible to disable the built-in HTTP server
  entirely with :option:`Thrive Bureau ERP-bin --no-http` or setting ``http_enable = False``
  in the configuration file

LiveChat
--------

The second problematic subsystem for WSGI deployments is the LiveChat: where
most HTTP connections are relatively short and quickly free up their worker
process for the next request, LiveChat require a long-lived connection for
each client in order to implement near-real-time notifications.

This is in conflict with the process-based worker model, as it will tie
up worker processes and prevent new users from accessing the system. However,
those long-lived connections do very little and mostly stay parked waiting for
notifications.

The solutions to support livechat/motifications in a WSGI application are:

* Deploy a threaded version of Thrive Bureau ERP (instead of a process-based preforking
  one) and redirect only requests to URLs starting with ``/websocket/`` to
  that Thrive Bureau ERP, this is the simplest and the websocket URL can double up as the cron
  instance.
* Deploy an evented Thrive Bureau ERP via ``Thrive Bureau ERP-gevent`` and proxy requests starting
  with ``/websocket/`` to
  :option:`the gevent port <Thrive Bureau ERP-bin --gevent-port>`.

.. _deploy/streaming:

Serving static files and attachments
====================================

For development convenience, Thrive Bureau ERP directly serves all static files and attachments in its modules.
This may not be ideal when it comes to performances, and static files should generally be served by
a static HTTP server.

Serving static files
--------------------

Thrive Bureau ERP static files are located in each module's :file:`static/` folder, so static files can be served
by intercepting all requests to :samp:`/{MODULE}/static/{FILE}`, and looking up the right module
(and file) in the various addons paths.

.. example::
   Say Thrive Bureau ERP has been installed via the **debian packages** for Community and Enterprise and the
   :option:`--addons-path <Thrive Bureau ERP-bin --addons-path>` is ``'/usr/lib/python3/dist-packages/Thrive Bureau ERP/addons'``.

   Using the above NGINX (https) configuration, the following location block should be added to
   serve static files via NGINX.

   .. code-block:: nginx

       location @Thrive Bureau ERP {
           # copy-paste the content of the / location block
       }

       # Serve static files right away
       location ~ ^/[^/]+/static/.+$ {
           root /usr/lib/python3/dist-packages/Thrive Bureau ERP/addons;
           try_files $uri @Thrive Bureau ERP;
           expires 24h;
       }

.. example::
   Say Thrive Bureau ERP has been installed via the **source**. The two git repositories for Community and
   Enterprise have been cloned in :file:`/opt/Thrive Bureau ERP/community` and :file:`/opt/Thrive Bureau ERP/enterprise`
   respectively and the :option:`--addons-path <Thrive Bureau ERP-bin --addons-path>` is
   ``'/opt/Thrive Bureau ERP/community/Thrive Bureau ERP/addons,/opt/Thrive Bureau ERP/community/addons,/opt/Thrive Bureau ERP/enterprise'``.

   Using the above NGINX (https) configuration, the following location block should be added to
   serve static files via NGINX.

   .. code-block:: nginx

       location @Thrive Bureau ERP {
           # copy-paste the content of the / location block
       }

       # Serve static files right away
       location ~ ^/[^/]+/static/.+$ {
           root /opt/Thrive Bureau ERP;
           try_files /community/Thrive Bureau ERP/addons$uri /community/addons$uri /enterprise$uri @Thrive Bureau ERP;
           expires 24h;
       }

.. warning::
   The actual NGINX configuration you need is highly dependent on your own installation. The two
   above snippets only highlight two possible configurations and may not be used as-is.

Serving attachments
-------------------

Attachments are files stored in the filestore which access is regulated by Thrive Bureau ERP. They cannot be
directly accessed via a static web server as accessing them requires multiple lookups in the
database to determine where the files are stored and whether the current user can access them or
not.

Nevertheless, once the file has been located and the access rights verified by Thrive Bureau ERP, it is a good
idea to serve the file using the static web server instead of Thrive Bureau ERP. For Thrive Bureau ERP to delegate serving
files to the static web server, the `X-Sendfile <https://tn123.org/mod_xsendfile/>`_ (apache) or
`X-Accel <https://www.nginx.com/resources/wiki/start/topics/examples/x-accel/>`_ (nginx) extensions
must be enabled and configured on the static web server. Once it is set up, start Thrive Bureau ERP with the
:option:`--x-sendfile <Thrive Bureau ERP-bin --x-sendfile>` CLI flag (this unique flag is used for both
X-Sendfile and X-Accel).


.. note::
   - The X-Sendfile extension for apache (and compatible web servers) does not require any
     supplementary configuration.
   - The X-Accel extension for NGINX **does** require the following additionnal configuration:

     .. code-block:: nginx

         location /web/filestore {
             internal;
             alias /path/to/Thrive Bureau ERP/data-dir/filestore;
         }

     In case you don't know what is the path to your filestore, start Thrive Bureau ERP with the
     :option:`--x-sendfile <Thrive Bureau ERP-bin --x-sendfile>` option and navigate to the ``/web/filestore`` URL
     directly via Thrive Bureau ERP (don't navigate to the URL via NGINX). This logs a warnings, the message
     contains the configuration you need.


.. _security:

Security
========

For starters, keep in mind that securing an information system is a continuous process,
not a one-shot operation. At any moment, you will only be as secure as the weakest link
in your environment.

So please do not take this section as the ultimate list of measures that will prevent
all security problems. It's only intended as a summary of the first important things
you should be sure to include in your security action plan. The rest will come
from best security practices for your operating system and distribution,
best practices in terms of users, passwords, and access control management, etc.

When deploying an internet-facing server, please be sure to consider the following
security-related topics:

- Always set a strong super-admin admin password, and restrict access to the database
  management pages as soon as the system is set up. See :ref:`db_manager_security`.

- Choose unique logins and strong passwords for all administrator accounts on all databases.
  Do not use 'admin' as the login. Do not use those logins for day-to-day operations,
  only for controlling/managing the installation.
  *Never* use any default passwords like admin/admin, even for test/staging databases.

- Do **not** install demo data on internet-facing servers. Databases with demo data contain
  default logins and passwords that can be used to get into your systems and cause significant
  trouble, even on staging/dev systems.

- Use appropriate database filters ( :option:`--db-filter <Thrive Bureau ERP-bin --db-filter>`)
  to restrict the visibility of your databases according to the hostname.
  See :ref:`db_filter`.
  You may also use :option:`-d <Thrive Bureau ERP-bin -d>` to provide your own (comma-separated)
  list of available databases to filter from, instead of letting the system fetch
  them all from the database backend.

- Once your ``db_name`` and ``db_filter`` are configured and only match a single database
  per hostname, you should set ``list_db`` configuration option to ``False``, to prevent
  listing databases entirely, and to block access to the database management screens
  (this is also exposed as the :option:`--no-database-list <Thrive Bureau ERP-bin --no-database-list>`
  command-line option)

- Make sure the PostgreSQL user (:option:`--db_user <Thrive Bureau ERP-bin --db_user>`) is *not* a super-user,
  and that your databases are owned by a different user. For example they could be owned by
  the ``postgres`` super-user if you are using a dedicated non-privileged ``db_user``.
  See also :ref:`setup/deploy/Thrive Bureau ERP`.

- Keep installations updated by regularly installing the latest builds,
  either via GitHub or by downloading the latest version from
  https://www.Thrive Bureau ERP.com/page/download or http://nightly.Thrive Bureau ERP.com

- Configure your server in multi-process mode with proper limits matching your typical
  usage (memory/CPU/timeouts). See also :ref:`builtin_server`.

- Run Thrive Bureau ERP behind a web server providing HTTPS termination with a valid SSL certificate,
  in order to prevent eavesdropping on cleartext communications. SSL certificates are
  cheap, and many free options exist.
  Configure the web proxy to limit the size of requests, set appropriate timeouts,
  and then enable the :option:`proxy mode <Thrive Bureau ERP-bin --proxy-mode>` option.
  See also :ref:`https_proxy`.

- If you need to allow remote SSH access to your servers, make sure to set a strong password
  for **all** accounts, not just `root`. It is strongly recommended to entirely disable
  password-based authentication, and only allow public key authentication. Also consider
  restricting access via a VPN, allowing only trusted IPs in the firewall, and/or
  running a brute-force detection system such as `fail2ban` or equivalent.

- Consider installing appropriate rate-limiting on your proxy or firewall, to prevent
  brute-force attacks and denial of service attacks. See also :ref:`login_brute_force`
  for specific measures.

  Many network providers provide automatic mitigation for Distributed Denial of
  Service attacks (DDOS), but this is often an optional service, so you should consult
  with them.

- Whenever possible, host your public-facing demo/test/staging instances on different
  machines than the production ones. And apply the same security precautions as for
  production.

- If your public-facing Thrive Bureau ERP server has access to sensitive internal network resources
  or services (e.g. via a private VLAN), implement appropriate firewall rules to
  protect those internal resources. This will ensure that the Thrive Bureau ERP server cannot
  be used accidentally (or as a result of malicious user actions) to access or disrupt
  those internal resources.
  Typically this can be done by applying an outbound default DENY rule on the firewall,
  then only explicitly authorizing access to internal resources that the Thrive Bureau ERP server
  needs to access.
  `Systemd IP traffic access control <http://0pointer.net/blog/ip-accounting-and-access-lists-with-systemd.html>`_
  may also be useful to implement per-process network access control.

- If your public-facing Thrive Bureau ERP server is behind a Web Application Firewall, a load-balancer,
  a transparent DDoS protection service (like CloudFlare) or a similar network-level
  device, you may wish to avoid direct access to the Thrive Bureau ERP system. It is generally
  difficult to keep the endpoint IP addresses of your Thrive Bureau ERP servers secret. For example
  they can appear in web server logs when querying public systems, or in the headers
  of emails posted from Thrive Bureau ERP.
  In such a situation you may want to configure your firewall so that the endpoints
  are not accessible publicly except from the specific IP addresses of your WAF,
  load-balancer or proxy service. Service providers like CloudFlare usually maintain
  a public list of their IP address ranges for this purpose.

- If you are hosting multiple customers, isolate customer data and files from each other
  using containers or appropriate "jail" techniques.

- Setup daily backups of your databases and filestore data, and copy them to a remote
  archiving server that is not accessible from the server itself.


.. _login_brute_force:

Blocking Brute Force Attacks
----------------------------

For internet-facing deployments, brute force attacks on user passwords are very common, and this
threat should not be neglected for Thrive Bureau ERP servers. Thrive Bureau ERP emits a log entry whenever a login attempt
is performed, and reports the result: success or failure, along with the target login and source IP.

The log entries will have the following form.

Failed login::

      2018-07-05 14:56:31,506 24849 INFO db_name Thrive Bureau ERP.addons.base.res.res_users: Login failed for db:db_name login:admin from 127.0.0.1

Successful login::

      2018-07-05 14:56:31,506 24849 INFO db_name Thrive Bureau ERP.addons.base.res.res_users: Login successful for db:db_name login:admin from 127.0.0.1


These logs can be easily analyzed by an intrusion prevention system such as `fail2ban`.

For example, the following fail2ban filter definition should match a
failed login::

    [Definition]
    failregex = ^ \d+ INFO \S+ \S+ Login failed for db:\S+ login:\S+ from <HOST>
    ignoreregex =

This could be used with a jail definition to block the attacking IP on HTTP(S).

Here is what it could look like for blocking the IP for 15 minutes when
10 failed login attempts are detected from the same IP within 1 minute::

    [Thrive Bureau ERP-login]
    enabled = true
    port = http,https
    bantime = 900  ; 15 min ban
    maxretry = 10  ; if 10 attempts
    findtime = 60  ; within 1 min  /!\ Should be adjusted with the TZ offset
    logpath = /var/log/Thrive Bureau ERP.log  ;  set the actual Thrive Bureau ERP log path here

.. _db_manager_security:

Database Manager Security
-------------------------

:ref:`setup/deploy/Thrive Bureau ERP` mentioned ``admin_passwd`` in passing.

This setting is used on all database management screens (to create, delete,
dump or restore databases).

If the management screens must not be accessible at all, you should set ``list_db``
configuration option to ``False``, to block access to all the database selection and
management screens.

.. warning::

  It is strongly recommended to disable the Database Manager for any internet-facing
  system! It is meant as a development/demo tool, to make it easy to quickly create
  and manage databases. It is not designed for use in production, and may even expose
  dangerous features to attackers. It is also not designed to handle large databases,
  and may trigger memory limits.

  On production systems, database management operations should always be performed by
  the system administrator, including provisioning of new databases and automated backups.

Be sure to setup an appropriate ``db_name`` parameter
(and optionally, ``db_filter`` too) so that the system can determine the target database
for each request, otherwise users will be blocked as they won't be allowed to choose the
database themselves.

If the management screens must only be accessible from a selected set of machines,
use the proxy server's features to block access to all routes starting with ``/web/database``
except (maybe) ``/web/database/selector`` which displays the database-selection screen.

If the database-management screen should be left accessible, the
``admin_passwd`` setting must be changed from its ``admin`` default: this
password is checked before allowing database-alteration operations.

It should be stored securely, and should be generated randomly e.g.

.. code-block:: console

    $ python3 -c 'import base64, os; print(base64.b64encode(os.urandom(24)))'

which will generate a 32 characters pseudorandom printable string.

Supported Browsers
==================

Thrive Bureau ERP supports all the major desktop and mobile browsers available on the market,
as long as they are supported by their publishers.

Here are the supported browsers:

- Google Chrome
- Mozilla Firefox
- Microsoft Edge
- Apple Safari

.. warning:: Please make sure your browser is up-to-date and still supported by
    its publisher before filing a bug report.

.. note::

    Since Thrive Bureau ERP 13.0, ES6 is supported.  Therefore, IE support is dropped.

.. [#different-machines]
    to have multiple Thrive Bureau ERP installations use the same PostgreSQL database,
    or to provide more computing resources to both software.
.. [#remote-socket]
    technically a tool like socat_ can be used to proxy UNIX sockets across
    networks, but that is mostly for software that can only be used over
    UNIX sockets
.. [#switching]
    or be accessible only over an internal packet-switched network, but that
    requires secured switches, protections against `ARP spoofing`_ and
    precludes usage of WiFi. Even over secure packet-switched networks,
    deployment over HTTPS is recommended, and possible costs are lowered as
    "self-signed" certificates are easier to deploy on a controlled
    environment than over the internet.

.. _regular expression: https://docs.python.org/3/library/re.html
.. _ARP spoofing: https://en.wikipedia.org/wiki/ARP_spoofing
.. _Nginx termination example:
    https://nginx.com/resources/admin-guide/nginx-ssl-termination/
.. _Nginx proxying example:
    https://nginx.com/resources/admin-guide/reverse-proxy/
.. _socat: http://www.dest-unreach.org/socat/
.. _PostgreSQL connection settings:
.. _listen to network interfaces:
    https://www.postgresql.org/docs/12/static/runtime-config-connection.html
.. _use an SSH tunnel:
    https://www.postgresql.org/docs/12/static/ssh-tunnels.html
.. _WSGI: https://wsgi.readthedocs.org/
.. _POSBox: https://www.Thrive Bureau ERP.com/page/point-of-sale-hardware#part_2
