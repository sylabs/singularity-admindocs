.. _plugins:

#########
 Plugins
#########

Sylabs is pleased to provide additional functionality over the open
through installable plugins. In the GitHub official releases these
plugins are packaged as rpm and deb files, that can be installed and
removed using your distribution package manager.

Plugin updates are released with new versions of {Singularity}, and
plugin packages are specific to the matching version of {Singularity}.

.. _log-plugin:

************
 Log Plugin
************

The {Singularity} Log Plugin provides a way to log Singularity
operations via the standard POSIX syslog mechanism. On modern Linux
distributions log messages will be recorded in the system journal
(accessible via ``journalctl``) by default. If you have configured an
alternative syslog daemon, or have setup forwarding to a remote syslog
host, the plugin logs will follow this path.

Installation
============

Download the plugin packages from the GitHub release page.

Install the log plugin package using your distribution package
manager, e.g.:

**Red Hat Enterprise Linux**

.. code::

   $ yum install ./singularity-ce-log-plugin-4.6.0-1.el10.x86_64.rpm

**Ubuntu LTS**

.. code::

   $ apt install ./singularity-ce-log-plugin_4.6.0-resolute_amd64.deb

The plugin will be installed and registered with Singularity, so that it
is enabled.

You can view installed plugins with the command:

.. code::

   $ singularity plugin list
   ENABLED  NAME
       yes  sylabs.io/log-plugin

The root user can temporarily disable / re-enable the installed plugin:

.. code::

   $ sudo singularity plugin disable sylabs.io/log-plugin

   $ singularity plugin list
   ENABLED  NAME
        no  sylabs.io/log-plugin

   $ sudo singularity plugin enable sylabs.io/log-plugin

   $ singularity plugin list
   ENABLED  NAME
       yes  sylabs.io/log-plugin

To permanently remove the plugin please use your package manager to
uninstall the ``singularity-ce-log-plugin`` package.

Source Code
===========

Source code for the log plugin can be found in the ``log-plugin`` directory of
the SingularityCE code.

The plugin can be manually built and managed using the ``singularity plugin``
commands that are documented in the user guide.

Operation
=========

The log plugin is called prior to the initialization of the container
runtime engine. It is intended to log the details of the command being
executed with Singularity, for audit and general system monitoring
purposes.

If you have a standard logging configuration on a system using
systemd-journald you can inspect log entries with the ``journalctl``
command. Use the ``-t`` option to show only ``singularity`` entries,
e.g.:

.. code::

   $ journalctl -t singularity --no-pager
   -- Logs begin at Wed 2020-07-08 13:35:47 CDT, end at Tue 2020-09-29 14:12:06 CDT. --
   Sep 29 14:12:02 ythel.local singularity[91671]: uid=1000 gid=1001 user="dave" group="dave" exe="/usr/bin/singularity" version="3.7-4" cwd="/home/dave" command="run" args=["docker://alpine"] flags=[] image="docker://alpine" sif-uuid="" LOGNAME="dave"

The default logging format lists:

-  ``uid`` - the numeric uid for the user that ran Singularity
-  ``gid`` - the numeric effective gid for the user that ran Singularity
-  ``user`` - alphanumeric username
-  ``group`` - alphanumeric groupname
-  ``exe`` - absolute path of the {Singularity} exectuable that was run
-  ``version`` - version of the {Singularity} executable that was run
-  ``cwd`` - current working directory when {Singularity} was run
-  ``command`` - the {Singularity} command that was called, e.g.
   run/shell
-  ``args`` - any arguments passed to the command
-  ``flags`` - any flags passed to the command
-  ``image`` - the image URL or filename for commands that act on a
   container image
-  ``sif-uuid`` - the UUID of a SIF, if a command was called against a
   SIF container image
-  ``LOGNAME`` - the value of the ``LOGNAME`` environment variable

Configuration
=============

The log plugin is configured using a YAML configuration file and one or
more logging templates found at the path
``/etc/singularity/plugins/sylabs.io/log-plugin/``.

config.yml
----------

The YAML format ``config.yml`` configuration file defines the logging
output to which messages should be sent. The default file configures the
plugin to write log messages to a single syslog destination:

.. code::

   output:
   - name: System logger
     type: syslog
     template: default.tpl

At present only the ``syslog`` type is supported. Please contact
support@sylabs.io if you would like to request the addition of
alternative logging destinations.

The ``template`` field provides the name of a template file that will be
used to set the content and format of log messages. The default logging
format is defined in ``default.tpl``, distributed with the plugin.

default.tpl
-----------

Logging templates are written in the Go language's template syntax.
Templates can access a range of information collected by the log plugin
and use it to construct log messages. The Go template syntax allows use
of logic (if / else) to modify the log message written. For example, you
may wish to only log MPI jobs at the rank 0 invocation of Singularity
(see example below).

Reference documentation for the Go template syntax can be found at:
https://golang.org/pkg/text/template/

The ``default.tpl`` provided with the plugin has the content:

.. code::

   {{- printf "uid=%d gid=%d user=%q group=%q exe=%q version=%q cwd=%q command=%q args=%q flags=%q image=%q sif-uuid=%q LOGNAME=%q" .UID .GID .User .Group .Executable .Version .CWD .Command .Args .Flags .Image .SIFUUID .Env.LOGNAME -}}

Note that:

-  A ``printf`` statement is used to format the log message into a
   single line.

-  Information exposed by the plugin is accessed as ``.NAME``, and
   environment variables as ``.Env.NAME``.

-  The ``%q`` 'quoted' format specifier is generally used for strings,
   so that any paths etc. are properly quoted in the output log message.

-  The start ``{{-`` and end ``-}}`` markers of the template syntax have
   hyphen characters inside the braces. This ensures that any whitespace
   before and after is removed, preventing blank log lines.

Log Variables
-------------

The log plugin exposes the following information as variables to the
template:

.. code::

   // UID is the numeric user id of the account that ran singularity
   UID int

   // GID is the numeric group id of the account that ran singularity
   GID int

   // User is the text user name of the account that ran singularity
   User string

   // Group is the text group name of the account that ran singularity
   Group string

   // Executable is the full path to the singularity executable that was run
   Executable string

   // Version is the version number of the singularity executable that was run
   Version string

   // CWD is the current working directory
   CWD string

   // Command is the top level command passed to singularity, e.g. run or build
   Command string

   // Args is a list of arguments passed to the command
   Args []string

   // Flags is a list of flags passed to the command
   Flags []string

   // Image is the path / URL of the container image, if applicable
   Image string

   // SIFUUID is the unique identifier (UUID) of the SIF image, if applicable
   SIFUUID string

   // Env gives access to log arbitrary environment variables.
   // Map of NAME -> VALUE
   Env map[string]string

Note the types listed, which may affect the format strings used in a
``printf`` statement in the log template.

Complex Example / Template Logic
--------------------------------

The following example shows how we can:

-  Use logic to alter the log message based on the value of variables.
-  Report batch scheduler information in log messages.

The goal of the template is to:

-  Only raise a log message for the RANK 0 process in MPI jobs, and
   always if the job is not an MPI job.
-  Show the job and task IDs where the batch system is GridEngine.

.. code::

   {{- $rank := "0" -}}
   {{- if .Env.OMPI_COMM_WORLD_RANK -}}
     {{- $rank = .Env.OMPI_COMM_WORLD_RANK -}}
   {{- end -}}
   {{- if .Env.PMI_RANK -}}
     {{- $rank =.Env.PMI_RANK -}}
   {{- end -}}
   {{- if eq $rank "0" -}}
     {{- printf "uid=%d gid=%d user=%q group=%q exe=%q version=%q cwd=%q command=%q args=%q flags=%q image=%q sif-uuid=%q LOGNAME=%q JOB_ID=%q SGE_TASK_ID=%q" .UID .GID .User .Group .Executable .Version .CWD .Command .Args .Flags .Image .SIFUUID .Env.LOGNAME .Env.JOB_ID .Env.SGE_TASK_ID -}}
   {{- end -}}

Here we:

-  Begin by setting a template variable ``$rank`` to 0.

-  Reset ``$rank`` to the OpenMPI RANK if it is set in the environment.

-  Reset ``$rank`` to the MPICH/Intel MPI RANK if it is set in the
   environment.

-  Only ``printf`` a log message when the ``$rank`` is 0.

-  Include ``JOB_ID`` and ``SGE_TASK_ID`` from the environment
   (``.Env.``) in our log message output.

-  Use start ``{{-`` and end ``-}}`` markers of the template syntax
   which have hyphen characters inside the braces. This ensures that any
   whitespace before and after is removed, preventing blank log lines.

Troubleshooting
---------------

If the ``config.yml`` file or the log template are invalid then a
warning will be displayed pointing to the line and location of the
error:

.. code::

   $ singularity run docker://alpine
   WARNING: Plugin sylabs.io/log-plugin: cannot log to syslog (System logger): error parsing template file: template: default.tpl:9: bad character U+003D '='
   singularity>

{Singularity} will continue to run, but no log message will be sent to
syslog. You should address any errors as soon as possible.

.. _enterprise-plugin:

*******************
 Enterprise Plugin
*******************

The {Singularity} Enterprise plugin is a management tool for Singularity
Enterprise installations. It adds an ``enterprise`` command to the
{Singularity} CLI, which allows a user or administrator to query or work
with the Singularity Enterprise installation that is set as their
current remote.

It can currently:

-  ``get`` lists and detail of various objects in the Enterprise
   installation.
-  ``describe`` objects in the library, showing full detail about them.
-  ``delete`` *tokens*, to revoke them.
-  Show the ``status`` of the Enterprise services.
-  Produce output in a variety of formats, so that it can be used to
   create e.g. usage reports.

Requirements
============

The enterprise plugin is designed for use with Singularity Enterprise
2.0.1 or greater.

Some functionality is compatible with Singularity Enterprise 1.x, but
not all features will be available.

.. note::

   The plugin supports entity / collection / container objects for
   Singularity Enterprise 1.x,


Installation
============

Download the plugin packages from the GitHub release page.

Install the enterprise-plugin package using your distribution
package manager:

**Red Hat Enterprise Linux**

.. code::

   $ yum install ./singularity-ce-enterprise-plugin-4.6.0-1.el10.x86_64.rpm

**Ubuntu LTS**

.. code::

   $ apt install ./singularity-ce-enterprise-plugin_4.6.0-resolute_amd64.deb

The plugin will be installed and registered with Singularity, so that it
is enabled.

The plugin will be installed and registered with Singularity, so that it
is enabled.

You can view installed plugins with the command:

.. code::

   $ singularity plugin list
   ENABLED  NAME
       yes  sylabs.io/-enterprise-plugin

The root user can temporarily disable / re-enable the installed plugin:

.. code::

   $ sudo singularity plugin disable sylabs.io/enterprise-plugin

   $ singularity plugin list
   ENABLED  NAME
        no  sylabs.io/enterprise-plugin

   $ sudo singularity plugin enable sylabs.io/enterprise-plugin

   $ singularity plugin list
   ENABLED  NAME
       yes  sylabs.io/enteprise-plugin

To permanently remove the plugin please use your package manager to
uninstall the ``singularity-ce-enterprise-plugin`` package.

Remote Configuration
====================

The ``singularity enterprise`` command runs against the currently
configured {Singularity} remote endpoint for your account. If you have
not logged in to any Enterprise installation, or the public Sylabs
cloud, the command will not run.

From a fresh installation of {Singularity} you can use the ``singularity
remote login`` command to login to the Sylabs cloud, which will allow
you to explore the plugin functionality:

.. code::

   $ singularity remote login
   Generate an access token at https://cloud.sylabs.io/auth/tokens, and paste it here.
   Token entered will be hidden for security.
   Access Token:
   INFO:    Access Token Verified!
   INFO:    Token stored in /home/dtrudg/.singularity/remote.yaml

   $ singularity enterprise get images library/default/ubuntu
   ID                       Tags                   Arch    Description                Size     Signed Encrypted Uploaded
   5baba99494feb900016ea434 [14.04]                amd64   No description             59.7 MiB false  false     true
   5baba9b994feb900016ea436 [16.04]                amd64   No description             35.3 MiB false  false     true
   5baba9d494feb900016ea438 []                     amd64   No description             26.7 MiB false  false     true
   5baba9fb94feb900016ea43a [18.10]                amd64   No description             26.8 MiB false  false     true
   5babaa1594feb900016ea43c []                     amd64   No description             26.7 MiB false  false     true
   5ce86bc44fb0942d12f3d20d []                     amd64   No Description             35.4 MiB true   false     true
   5ea0557e290c48bae3392b2e []                     amd64   No Description             55.4 MiB false  false     true
   5ea055c8f0f8eb90a8a7b913 [19.10 eoan]           amd64   No Description             55.0 MiB false  false     true
   5ea1e528c7eca47f070b1f22 []                     amd64   No Description             55.4 MiB false  false     true
   5ea6eba8d0ff9c878fea57c2 []                     amd64   No Description             52.1 MiB false  false     true
   61084029bc16537b1321c703 [bionic 18.04]         amd64   Ubuntu 18.04 LTS (bionic)  34.7 MiB true   false     true
   6108406cff2db5ba27b5b875 [20.04 focal]          amd64   Ubuntu 20.04 LTS (focal)   35.0 MiB true   false     true
   610840aebc16537b1321c705 [latest hirsute 21.04] amd64   Ubuntu 21.04 (hirsute)     35.5 MiB true   false     true
   610861aebc16537b1321c71e [bionic 18.04]         ppc64le Ubuntu 18.04 LTS (bionic)  36.9 MiB true   false     true
   6108653abc16537b1321c722 [focal 20.04]          ppc64le Ubuntu 20.04 LTS (focal)   38.9 MiB true   false     true
   6108659bff2db5ba27b5b89c [18.04 bionic]         arm64   Ubuntu 18.04 LTS (bionic)  30.8 MiB true   false     true
   610865c9ff2db5ba27b5b89e [focal 20.04]          arm64   Ubuntu 20.04 LTS (focal)   33.2 MiB true   false     true
   610865f4d63fe43757fac6d9 [21.04 latest hirsute] arm64   Ubuntu 21.04 (hirsute)     33.9 MiB true   false     true
   610866dfbc16537b1321c726 [latest 21.04 hirsute] ppc64le Ubuntu 21.04 (hirsute)     40.1 MiB true   false     true

To use the plugin with a local Singularity Enterprise installation you
should use ``singularity remote add`` to configure the endpoint and
login to it.

Using the Plugin
================

The ``singularity enterprise`` plugin interface is designed to be
similar in operation to ``kubectl``, which most administrators should be
familiar with from deployment and management of Singularity Enterprise
under Kubernetes.

A ``status`` and ``version`` command provide basic information about
your Enterprise installation, and the ``enterprise`` tool respectively.

For all other operations you interact by running a *command* against a
*type*, and optionally specifying one or more *names*. E.g. running the
``describe`` command against the ``container`` type, with
``user123/linux/alpine`` as the name.

``version`` command
-------------------

The ``version`` command shows detailed version information for the
``singularity enterprise`` plugin itself.

.. code::

   $ singularity enterprise version
   Version:  unknown
   By:       SingularityCE 4.6.0
   Runtime:  go1.27.1 (linux/amd64)

``status`` command
------------------

The ``status`` command contacts each Enterprise service, and shows
version and status information for them:

.. code::

   $ singularity enterprise status

   Status for Enterprise installation

   SERVICE         STATUS VERSION
   library         OK     v1.3.3-3-g055eabc-dirty
   build server    OK     v1.4.9-0-g785f7150
   build manager   OK     v1.4.9-0-g785f7150
   key service     OK     v1.18.2-0-g0d26399
   consent service OK     v1.4.14-0-g541ed1d
   token service   OK     v1.4.14-0-g541ed1d

   Auth token is valid

``get`` command
---------------

The ``get`` command displays information about items in the Singularity
Enterprise installation.

Types
^^^^^

You must run the ``get`` command against a specific *type* of item in
Enterprise:

.. code::

   Plural        Singular     Short     Item
   =======================================================================
   projects      project      prj     - Library projects
   repositories  repository   rep     - Library repositories
   images        image        img     - Library images

   builds        build        bld     - Remote builder builds
   build-agents  build-agent  age     - Remote builder build agents

   keys          key                  - Key service keys

   users         user         usr     - User accounts
   tokens        token        tok     - User authentication tokens

You can use the plural, singular or short forms of each type
interchangeably. ``get projects`` / ``get project`` / ``get prj`` will
have the same effect.

There is limited support for interrogating a Singularity Enterprise
1.x library using the following types:

.. code::

   entities      entity       ent     - Library entities
   collections   collection   col     - Library collections
   containers    container    con     - Library containers
   images        image        img     - Library images


Output
^^^^^^

By default, the ``get`` command lists items in a short tabular format,
which includes only the most commonly useful attributes for the type.

You can retrieve more detailed information by specifying an different
``--output`` format:

-  ``--output short`` is the default short tabular format
-  ``--output long`` shows all attributes in a tabular format
-  ``--output csv`` shows all attributes in a CSV format, for
   redirection to file and import into spreadsheet software etc.
-  ``--output json`` dumps all detail about items in JSON format.
-  ``--output yaml`` dumps all detail about items in YAML format.

Any log / warning / error messages are written to STDERR, and the
``get`` output to STDOUT, so you can collect information to a file
easily, e.g.:

.. code::

   $ singularity enterprise get --output csv projects > projects.csv

   $ singularity enterprise get --output json projects > projects.json

``get projects``
""""""""""""""""

The ``get`` command for ``projects`` can be called as:

-  ``singularity enterprise get projects`` to list all projects in the
   Enterprise container library.

-  ``singularity enterprise get projects <project ref>...`` to list
   detail about the specified projects in the Enterprise container
   library.

``get repositories``
""""""""""""""""""

The ``get`` command for ``repositories`` can be called as:

-  ``singularity enterprise get repositories <repository path>`` to list
   all repositories that begin with the specified path. E.g. specify
   ``user1/hpc`` to list all repositories in the ``user1`` project,
   that have a repository name beginning ``linux/``.

-  ``singularity enterprise get containers <repository ref>...`` to list
   detail about the specified repositories in the Enterprise container
   library.

``get images``
""""""""""""""

The ``get`` command for ``images`` can be called as:

-  ``singularity enterprise get images <repository ref>`` to list all
   images in the specified repository in the Enterprise container
   library.

-  ``singularity enterprise get images <image ref>...`` to list detail
   about the specified images in the Enterprise container library.

``get builds``
""""""""""""""

The ``get`` command for ``builds`` can be called as:

-  ``singularity enterprise get builds`` to list remote builds that have
   been performed.
-  ``singularity enterprise get builds <build id>...`` to list details
   of specific remote builds, specified by ID.

By default ``singularity enterprise get builds`` will show builds
performed by the current user, in the past 24 hours.

To see builds performed during a different time period use the
``--from`` and ``--to`` flags.

Administrators can retrieve builds performed by another user with the
``--user`` flag, or all users with the ``--all`` flag. The ``--user``
flag accepts either a user's BSON ID, or their username.

``get build-agents`` (admin only)
"""""""""""""""""""""""""""""""""

The ``get`` command for ``build-agents`` can be called as:

-  ``singularity enterprise get build-agents default`` to list build
   agents (VMs) that are part of the ``default`` pool.

``get keys``

The ``get`` command for ``keys`` can be called as:

-  ``singularity enterprise get keys`` to list keys belong to a user.

-  ``singularity enterprise get keys <search term>`` to search for keys
   matching a term. Currently returns HKP server output, not formatted
   TSV/CSV/JSON/YAML.

By default ``singularity enterprise get keys`` will show keys submitted
by the current user.

Administrators can retrieve keys submitted by another user with the
``--user`` flag. The ``--user`` flag accepts either a user's BSON ID, or
their username.

``get users`` (admin only)
""""""""""""""""""""""""""

The ``get`` command for ``users`` can be called as:

-  ``singularity enterprise get users`` to list user accounts in
   Enterprise.
-  ``singularity enterprise get users <user id>...`` to list details of
   specific user accounts, specified by ID or username.

``get tokens``
""""""""""""""

The ``get`` command for ``tokens`` can be called as:

-  ``singularity enterprise get tokens`` to list the authenticated
   user's authentication tokens.
-  ``singularity enterprise get tokens <token id>...`` to list details
   of specific authentication tokens, specified by ID.

Administrators can retrieve keys submitted by another user with the
``--user`` flag. The ``--user`` flag acceptes either a user's BSON ID,
or their username.

``describe`` command
--------------------

The ``describe`` command displays information about a single item in the
Singularity Enterprise installation, in a readable format for
interactive use.

You must run the ``describe`` command against a specific *type* of item
in Enterprise:

.. code::

   Plural        Singular     Short     Item
   =======================================================================
   projects      project      prj     - Library projects
   repositories  repository   rep     - Library repositories
   images        image        img     - Library images

   builds        build        bld     - Remote builder builds
   build-agents  build-agent  age     - Remote builder build agents

   keys          key                  - Key service keys
   users         user         usr     - User accounts
   tokens        token        tok     - User authentication tokens

You must specify the reference / id of the item to display. E.g.:

.. code::

   $ singularity enterprise describe repository library/linux/alpine

   $ singularity enterprise describe build 60d382812cde530fbbe5867e

delete command
--------------

The ``delete`` command deletes, revokes, or invalidates items in the
Singularity Enterprise installation. The CLI currently supports
``delete`` for authentication tokens only.

``delete tokens``
^^^^^^^^^^^^^^^^^

The ``delete`` command for ``tokens`` can be called as:

-  ``singularity enterprise delete tokens <token id>...`` to revoke
   specified tokens by ID.
-  ``singularity enterprise delete tokens --all`` to revoke all tokens
   for the currently authenticated user.

Administrators may additionally use:

-  ``singularity enterprise delete tokens --all --user <user>`` to
   revoke all tokens for the specified user ID or username.

-  ``singularity enterprise delete tokens --all --user all`` to revoke
   all tokens, for all users, *except* the administrator running this
   command (to avoid locking out the adminitstrator). To revoke the
   administrator's tokens also you must run ``singularity enterprise
   delete tokens --all`` separately.
