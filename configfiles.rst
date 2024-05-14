.. _singularity_configfiles:

###################################
 {Singularity} Configuration Files
###################################

As a {Singularity} Administrator, you will have access to various
configuration files, that will let you manage container resources, set
security restrictions and configure network options etc, when
installing {Singularity} across the system. All of these files can be
found in ``/usr/local/etc/singularity`` by default for installations
from source (though the location may differ based on options passed
during the installation). For installations from RPM or Deb packages
you will find the configuration files in ``/etc/singularity``. This
section will describe the configuration files and the various
parameters contained by them.

******************
 singularity.conf
******************

Most of the configuration options are set using the file
``singularity.conf`` that defines the global configuration for
{Singularity} across the entire system. Using this file, system
administrators can influence the behavior of {Singularity} and
restrict the functionality that users can access. As a security
measure, for ``setuid`` installations of {Singularity},
``singularity.conf`` must be owned by root and must not be writable by
users or {Singularity} will refuse to run. This is not the case for
``non-setuid`` installations that will only ever execute with user
privilege and thus do not require such limitations.

The options set via ``singularity.conf`` are listed below. Options are
grouped together based on relevance. The actual order of options within
``singularity.conf`` may differ.

Setuid and Capabilities
=======================

``allow setuid``: To use all features of {Singularity} containers,
{Singularity} will need to have access to some privileged system calls.
{Singularity} achieves this by using a helper binary with the ``setuid``
bit enabled. The ``allow-setuid`` option lets you enable/disable users
ability to utilize these binaries within {Singularity}. By default, it
is set to "yes", but when disabled, various {Singularity} features will
not function. Please see :ref:`Unprivileged Installations
<userns-limitations>` for more information about running {Singularity}
without ``setuid`` enabled.

``root default capabilities``: In its default, non-OCI-mode, {Singularity}
allows the specification of capabilities kept by default when the root user runs
a container. Options include:

-  full: all capabilities are maintained, this gives the same behavior
   as the ``--keep-privs`` option.
-  file: only capabilities granted for root in
   ``etc/singularity/capability.json`` are maintained.
-  no: no capabilities are maintained, this gives the same behavior as
   the ``--no-privs`` option.

The root user can modify the capabilities granted to individual containers when
they are launched through the ``--add-caps`` and ``drop-caps`` flags.

In OCI-mode, {Singularity} follows the behavior of other OCI runtimes, and will
always grant a default set of capabilities to the container. The ``root default
capabilities`` option does not apply.

Please see `Linux Capabilities
<https://sylabs.io/guides/{userversion}/user-guide/security_options.html#linux-capabilities>`_
in the user guide for more information.

Loop Devices
============

{Singularity} uses loop devices to facilitate the mounting of container file
systems from SIF and other images.

``max loop devices``: This option allows an admin to limit the total
number of loop devices {Singularity} will consume at a given time.

``shared loop devices``: This allows containers running the same image
to share a single loop device. This minimizes loop device usage and
helps optimize kernel cache usage. Enabling this feature can be
particularly useful for MPI jobs.

.. _sec:ocimode:

Default runtime (native vs. OCI)
================================

Starting with version 4.0, {Singularity} includes a fully-supported OCI-mode, allowing
you to run OCI containers using ``crun`` or ``runc`` as the low-level runtime,
for true OCI runtime compatibility. (See the `OCI-mode section
<https://sylabs.io/guides/{userversion}/user-guide/oci_runtime.html>`_ in the user
guide for more information.)

By default, {Singularity} will use its native runtime unless the ``run`` /
``shell`` / ``exec`` / ``pull`` commands are invoked with the ``--oci`` flag.
However, administrators who wish to configure their installation of
{Singularity} to use OCI-mode by default can do so by adding ``oci mode = yes``
to their configuration file:

``oci mode``: Enable OCI-mode by default. (default: ``no``)

.. note::

   By default, OCI-mode will attempt to use OCI-SIF images to locally store and
   mount containers. (See the `discussion of OCI-SIF
   <https://sylabs.io/guides/{userversion}/user-guide/oci_runtime.html#oci-sif-images>`_
   in the user guide for more information.) If system does not meet the
   requirements for using OCI-SIF, OCI mode will fall back to a filesystem-based
   strategy: the OCI container will be unpacked into a temporary sandbox dir and
   run from there.

   Administrators who wish to disable this fallback behavior can do so via the
   ``tmp sandbox = no`` option discussed :ref:`below <sec:tmpsandbox>`.

Namespace Options
=================

``allow pid ns``: This option determines if users can leverage the PID
namespace when running their containers through the ``--pid`` flag.

.. note::

   Using the PID namespace can confuse the process tracking of some
   resource managers, as well as some MPI implementations.

``allow uts ns``: Likewise, this option determines whether users can make use
of the UTS namespace when running their containers. If sets to ``no``, it
disables the ``--uts`` and ``--hostname`` flags.

Configuration Files
===================

{Singularity} can automatically create or modify several system files
within containers to ease usage.

.. note::

   These options will have no effect if the file does not exist within
   the container, or overlay or underlay support are enabled.

``config passwd``: This option determines if {Singularity} should
automatically append an entry to ``/etc/passwd`` for the user running
the container.

``config group``: This option determines if {Singularity} should
automatically append the calling user's group entries to the containers
``/etc/group``.

``config resolv_conf``: This option determines if {Singularity} should
automatically bind the host's ``/etc/resolv.conf`` within the container.

Session Directory and System Mounts
===================================

``sessiondir max size``: In order for the {Singularity} runtime to run
a container it needs to create a temporary in-memory ``sessiondir`` as
a location to assemble various components of the container, including
mounting filesystems over the base image. In addition, this sessiondir
will hold files that are written to the container when
``--writable-tmpfs`` is used, plus isolated temporary filesystems in
``--contain`` mode. The default value from {Singularity} 3.11 is
64MiB. If users commonly run containers with ``--writable-tmpfs``,
``--contain``, or in ``--oci`` mode, this value should be increased to
accommodate their workflows. The tmpfs will grow to the specified
maximum size, as required. Memory is not allocated ahead of usage.

``mount proc``: This option determines if {Singularity} should
automatically bind mount ``/proc`` within the container.

``mount sys``: This option determines if {Singularity} should
automatically bind mount ``/sys`` within the container.

``mount dev``: Should be set to "YES", if you want {Singularity} to
automatically bind mount a complete ``/dev`` tree within the container.
If set to ``minimal``, then only ``/dev/null``, ``/dev/zero``,
``/dev/random``, ``/dev/urandom``, and ``/dev/shm`` will be included.

``mount devpts``: This option determines if {Singularity} will mount a
new instance of ``devpts`` when there is a ``minimal`` ``/dev``
directory as explained above, or when the ``--contain`` option is
passed.

.. note::

   This requires either a kernel configured with
   ``CONFIG_DEVPTS_MULTIPLE_INSTANCES=y``, or a kernel version at or
   newer than ``4.7``.

``mount home``: When this option is enabled, {Singularity} will
automatically determine the calling user's home directory and attempt to
mount it into the container.

``mount tmp``: When this option is enabled, {Singularity} will
automatically bind mount ``/tmp`` and ``/var/tmp`` into the container
from the host. If the ``--contain`` option is passed, {Singularity} will
create both locations within the ``sessiondir`` or within the directory
specified by the ``--workdir`` option if that is passed as well.

``mount hostfs``: This option will cause {Singularity} to probe the host
for all mounted filesystems and bind those into containers at runtime.

``mount slave``: {Singularity} automatically mounts a handful host
system directories to the container by default. This option determines
if filesystem changes on the host should automatically be propagated to
those directories in the container.

.. note::

   This should be set to ``yes`` when autofs mounts occurring on the host
   system should be reflected up in the container.

``memory fs type``: This option allows admins to choose the temporary
filesystem used by {Singularity}. Temporary filesystems are primarily
used for system directories like ``/dev`` when the host system directory
is not mounted within the container.

.. note::

   For Cray CLE 5 and 6, up to CLE 6.0.UP05, there is an issue (kernel
   panic) when Singularity uses tmpfs, so on affected systems it's
   recommended to set this value to ``ramfs`` to avoid a kernel panic.

Bind Mount Management
=====================

``bind path``: This option is used to define a list of files or
directories to automatically be made available when {Singularity} runs a
container. In order to successfully mount listed paths the file or
directory must exist within the container, or {Singularity} must be
configured with either overlay or underlay support enabled.

.. note::

   This option is ignored when containers are invoked with the
   ``--contain`` option.

You can define the a bind point where the source and destination are
identical:

.. code::

   bind path = /etc/localtime

Or you can specify different source and destination locations using:

.. code::

   bind path = /etc/singularity/default-nsswitch.conf:/etc/nsswitch.conf

``user bind control``: This allows admins to decide if users can define
bind points at runtime. By Default, this option is set to ``YES``, which
means users can specify bind points, scratch and tmp locations.

Limiting Container Execution
============================

.. warning::

   If unprivileged user namespace creation is allowed on a system, a user can
   supply and use their own unprivileged installation of Singularity or another
   container runtime to bypass container limits. They may also be able to use
   standard system tools such as ``unshare``, ``nsenter``, and FUSE mounts to
   access / execute arbitrary containers without installing any runtime.

   The 'limit container' and 'allow container' directives are not effective if
   unprivileged user namespaces are enabled. They are only effectively applied
   when Singularity is running using the native runtime in setuid mode, and
   unprivileged container execution is not possible on the host.

   **You must disable unprivileged user namespace creation on the host if you
   rely on the these directives to limit container execution.** This will
   disable OCI mode, which is unprivileged and cannot enforce these limits.

There are several ways to limit container execution as an admin listed
below. If stricter controls are required, check out the :ref:`Execution
Control List <execution_control_list>`.

``limit container owners``: This restricts container execution to only
allow containers that are owned by the specified user.

.. note::

   This feature will only apply when {Singularity} is running in SUID
   mode and the user is non-root. By default this is set to ``NULL``.

``limit container groups``: This restricts container execution to only
allow containers that are owned by the specified group.

.. note::

   This feature will only apply when {Singularity} is running in SUID
   mode and the user is non-root. By default this is set to ``NULL``.

``limit container paths``: This restricts container execution to only
allow containers that are located within the specified path prefix.

.. note::

   This feature will only apply when {Singularity} is running in SUID
   mode and the user is non-root. By default this is set to ``NULL``.

``allow container ${type}``: This option allows admins to limit the
types of image formats that can be leveraged by users with
{Singularity}.

-  ``allow container sif`` permits / denies execution of unencrypted SIF
   containers.
-  ``allow container encrypted`` permits / denies execution of SIF
   containers with an encrypted root filesystem.
-  ``allow container squashfs`` permits / denies execution of bare
   SquashFS image files. E.g. Singularity 2.x images.
-  ``allow container extfs`` permits / denies execution of bare extfs
   image files.
-  ``allow container dir`` permits / denies execution of sandbox directory
   containers. Also applies to SIF / squashfs / extfs images when mounted to a
   directory by FUSE binaries run as the user, or automatically extracted to a
   directory.

.. note::

   These limitations do not apply to the root user.

.. note::

   When a SIF / squashfs / extfs container image is mounted using FUSE, or
   extracted to disk, the ``allow container dir`` setting applies. In contrast
   to kernel mounts, FUSE mounted container images are mounted at a directory
   under the full control of the user, who may also manipulate the behavior of
   the FUSE binary.

.. _sec:kernelmounts:

Disabling Kernel Filesystem Mounts
==================================

When running in setuid mode, {Singularity} will mount extfs and squashfs
filesystems using the kernel's filesystem drivers. These mounts are performed
for standalone or SIF container images, overlay images or partitions, that use
extfs or squashfs formats.

Options in ``singularity.conf`` allow these mounts to be disabled, to e.g. work
around a kernel vulnerability that cannot be patched in a timely manner.
Singularity will then attempt to use ``squashfuse`` or ``fuse2fs`` to mount
container images in user space. If it is not possible to perform a FUSE mount, a
container image will be extracted to a sandbox directory for execution.

Note that disabling kernel mounts will result in a significant loss of
functionality in setuid mode. Container execution restrictions cannot be
effectively applied, and not all overlay configurations are supported.

``allow kernel squashfs``: Defaults to yes. When set to no, {Singularity} will
not mount squashfs filesystems using the kernel squashfs driver. If possible,
``squashfuse_ll`` will be used to mount a squashfs container image in user
space. If ``squashfuse_ll`` is not available, or fails, the image will be
extracted to a directory for execution.

``allow kernel extfs``: Defaults to yes. When set to no, {Singularity} will not
mount extfs filesystems using the kernel extfs driver. If possible, ``fuse2fs``
will be used to mount an extfs container image in user space. If ``fuse2fs`` is
not available, or fails, the image will be extracted to a directory for
execution.

Networking Options
==================

The ``--network`` option can be used to specify a CNI networking
configuration that will be used when running a container with `network
virtualization
<https://sylabs.io/guides/{userversion}/user-guide/networking.html>`_.
Unrestricted use of CNI network configurations requires root privilege,
as certain configurations may disrupt the host networking environment.

{Singularity} 3.8 allows specific users or groups to be granted the
ability to run containers with administrator specified CNI
configurations.

``allow net users``: Allow specified root administered CNI network
configurations to be used by the specified list of users. By default
only root may use CNI configuration, except in the case of a fakeroot
execution where only 40_fakeroot.conflist is used. This feature only
applies when {Singularity} is running in SUID mode and the user is
non-root.

``allow net groups``: Allow specified root administered CNI network
configurations to be used by the specified list of users. By default
only root may use CNI configuration, except in the case of a fakeroot
execution where only 40_fakeroot.conflist is used. This feature only
applies when {Singularity} is running in SUID mode and the user is
non-root.

``allow net networks``: Specify the names of CNI network configurations
that may be used by users and groups listed in the allow net users /
allow net groups directives. Thus feature only applies when
{Singularity} is running in SUID mode and the user is non-root.

GPU Options
===========

{Singularity} provides integration with GPUs in order to facilitate GPU
based workloads seamlessly. Both options listed below are particularly
useful in GPU only environments. For more information on using GPUs with
{Singularity} checkout :ref:`GPU Library Configuration
<gpu_library_configuration>`.

``always use nv``: Enabling this option will cause every action command
(``exec/shell/run/instance``) to be executed with the ``--nv`` option
implicitly added.

``always use rocm``: Enabling this option will cause every action
command (``exec/shell/run/instance``) to be executed with the ``--rocm``
option implicitly added.

Supplemental Filesystems
========================

``enable fusemount``: This will allow users to mount fuse filesystems
inside containers using the ``--fusemount`` flag.

``enable overlay``: This option will allow {Singularity} to create bind
mounts at paths that do not exist within the container image. This
option can be set to ``try``, which will try to use an overlayfs. If it
fails to create an overlayfs in this case the bind path will be silently
ignored.

``enable underlay``: This option will allow {Singularity} to create bind
mounts at paths that do not exist within the container image, just like
``enable overlay``, but instead using an underlay. This is suitable for
systems where overlay is not possible or not working. If the overlay
option is available and working, it will be used instead.

CNI Configuration and Plugins
=============================

``cni configuration path``: This option allows admins to specify a
custom path for the CNI configuration that {Singularity} will use for
`Network Virtualization
<https://sylabs.io/guides/{userversion}/user-guide/networking.html>`_.

``cni plugin path``: This option allows admins to specify a custom path
for {Singularity} to access CNI plugin executables. Check out the
`Network Virtualization
<https://sylabs.io/guides/{userversion}/user-guide/networking.html>`_
section of the user guide for more information.

External Binaries
=================

{Singularity} calls a number of external binaries for full
functionality. The paths for certain critical binaries can be set in
``singularity.conf``. At build time, ``mconfig`` will set initial values
for these, by searching on the ``$PATH`` environment variable. You can
override which external binaries are called by changing the value in
``singularity.conf``.

``cryptsetup path``: Path to the cryptsetup executable, used to work
with encrypted containers. Must be owned by root for security reasons.

``ldconfig path``: Path to the ldconfig executable, used to find GPU
libraries. Must be owned by root for security reasons.

``nvidia-container-cli path``: Path to the nvidia-container-cli
executable, used to find GPU libraries and configure the container when
running with the ``--nvccli`` option. Must be owned by root for security
reasons.

For the following additional binaries, if the ``singularity.conf`` entry
is left blank, then ``$PATH`` will be searched at runtime.

``go path``: Path to the go executable, used to compile plugins.

``mksquashfs path``: Path to the mksquashfs executable, used to create
SIF and SquashFS containers.

``mksquashfs procs``: Allows the administrator to specify the number of
CPUs that mksquashfs may use when building an image. The fewer
processors the longer it takes. To use all available CPU's set this to
0.

``mksquashfs mem``: Allows the administrator to set the maximum amount
of memory that mksquashfs nay use when building an image. e.g. 1G for
1gb or 500M for 500mb. Restricting memory can have a major impact on the
time it takes mksquashfs to create the image. NOTE: This functionality
did not exist in squashfs-tools prior to version 4.3. If using an
earlier version you should not set this.

``unsquashfs path``: Path to the unsquashfs executable, used to extract
SIF and SquashFS containers.

Concurrent Downloads
====================

{Singularity} 3.9 and above will pull ``library://`` container images
using multiple concurrent downloads of parts of the image. This speeds
up downloads vs using a single stream. The defaults are generally
appropriate for the Sylabs Cloud, but may be tuned for your network
conditions, or if you are pulling from a different library server.

``download concurrency``: specifies how many concurrent streams when
downloading (pulling) an image from cloud library.

``download part size``: specifies the size of each part (bytes) when
concurrent downloads are enabled.

``download buffer size``: specifies the transfer buffer size (bytes)
when concurrent downloads are enabled.

Cgroups Options
===============

``systemd cgroups``: specifies whether to use systemd to manage container
cgroups. Required (with cgroups v2) for unprivileged users to apply resource
limits on containers. If set to ``no``, {Singularity} will directly manage
cgroups via the cgroupfs.

.. _sec:tmpsandbox:

Disabling temporary sandbox dirs
================================

Some execution flows will extract the contents of an image into a temporary
local sandbox dir prior to execution. Examples include: using a user namespace
in native mode when FUSE is not available, as well as using OCI-mode in an
environment that does not support OCI-SIF (see the discussion of OCI-mode
:ref:`above <sec:ocimode>`).

Administrators who wish to disable this behavior, and prevent {Singularity} from
extracting the contents of images to temporary sandbox dirs, may do so by adding
``tmp sandbox = no`` to their configuration file:

``tmp sandbox``: Allow extraction of image contents to temporary sandbox dir.
(default: ``yes``)

Deprecated Experimental Options
===============================

``sif fuse``: If set to ``yes``, always attempt to mount a SIF image using
``squashfuse`` when running in unprivileged / user namespace flows. This is
equivalent to always specifying the experimental `--sif-fuse` flag. **Deprecated
in {Singularity} 4.1**, as a FUSE mount will be attempted by default in these
circumstances. The option has no effect in 4.1, and is retained only for
configuration file compatibility with prior versions.


Updating Configuration Options
==============================

In order to manage this configuration file, {Singularity} has a ``config
global`` command group that allows you to get, set, reset, and unset
values through the CLI. It's important to note that these commands must
be run with elevated privileges because the ``singularity.conf`` can
only be modified by an administrator.

Example
-------

In this example we will changing the ``bind path`` option described
above. First we can see the current list of bind paths set within our
system configuration:

.. code::

   $ sudo singularity config global --get "bind path"
   /etc/localtime,/etc/hosts

Now we can add a new path and verify it was successfully added:

.. code::

   $ sudo singularity config global --set "bind path" /etc/resolv.conf
   $ sudo singularity config global --get "bind path"
   /etc/resolv.conf,/etc/localtime,/etc/hosts

From here we can remove a path with:

.. code::

   $ sudo singularity config global --unset "bind path" /etc/localtime
   $ sudo singularity config global --get "bind path"
   /etc/resolv.conf,/etc/hosts

If we want to reset the option to the default at installation, then we
can reset it with:

.. code::

   $ sudo singularity config global --reset "bind path"
   $ sudo singularity config global --get "bind path"
   /etc/localtime,/etc/hosts

And now we are back to our original option settings. You can also test
what a change would look like by using the ``--dry-run`` option in
conjunction with the above commands. Instead of writing to the
configuration file, it will output what would have been written to the
configuration file if the command had been run without the ``--dry-run``
option:

.. code::

   $ sudo singularity config global --dry-run --set "bind path" /etc/resolv.conf
   # SINGULARITY.CONF
   # This is the global configuration file for Singularity. This file controls
   [...]
   # BIND PATH: [STRING]
   # DEFAULT: Undefined
   # Define a list of files/directories that should be made available from within
   # the container. The file or directory must exist within the container on
   # which to attach to. you can specify a different source and destination
   # path (respectively) with a colon; otherwise source and dest are the same.
   # NOTE: these are ignored if singularity is invoked with --contain.
   bind path = /etc/resolv.conf
   bind path = /etc/localtime
   bind path = /etc/hosts
   [...]
   $ sudo singularity config global --get "bind path"
   /etc/localtime,/etc/hosts

Above we can see that ``/etc/resolv.conf`` is listed as a bind path in
the output of the ``--dry-run`` command, but did not affect the actual
bind paths of the system.


.. _cgroups_toml:

**************
 cgroups.toml
**************

The cgroups (control groups) functionality of the Linux kernel allows
you to limit and meter the resources used by a process, or group of
processes. Using cgroups you can limit memory and CPU usage. You can
also rate limit block IO, network IO, and control access to device
nodes.

There are two versions of cgroups in common use. Cgroups v1 sets
resource limits for a process within separate hierarchies per resource
class. Cgroups v2, the default in newer Linux distributions, implements
a unified hierarchy, simplifying the structure of resource limits on
processes.

-  v1 documentation:
   https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt
-  v2 documentation:
   https://www.kernel.org/doc/Documentation/cgroup-v2.txt

{Singularity} 3.9 and above can apply resource limitations to systems
configured for both cgroups v1 and the v2 unified hierarchy. Resource
limits are specified using a TOML file that represents the ``resources``
section of the OCI runtime-spec:
https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md#control-groups

On a cgroups v1 system the resources configuration is applied directly.
On a cgroups v2 system the configuration is translated and applied to
the unified hierarchy.

Under cgroups v1, access restrictions for device nodes are managed
directly. Under cgroups v2, the restrictions are applied by attaching
eBPF programs that implement the requested access controls.

Examples
========

To apply resource limits to a container, use the ``--apply-cgroups``
flag, which takes a path to a TOML file specifying the cgroups
configuration to be applied:

.. code::

   $ singularity shell --apply-cgroups /path/to/cgroups.toml my_container.sif

.. note::

   The ``--apply-cgroups`` option requires cgroups v2 to be used without root
   privileges.

Limiting memory
---------------

To limit the amount of memory that your container uses to 500MB
(524288000 bytes), set a ``limit`` value inside the ``[memory]`` section
of your cgroups TOML file:

.. code::

   [memory]
       limit = 524288000

Start your container, applying the toml file, e.g.:

.. code::

   $ singularity run --apply-cgroups path/to/cgroups.toml library://alpine

Limiting CPU
------------

CPU usage can be limited using different strategies, with limits
specified in the ``[cpu]`` section of the TOML file.

**shares**

This corresponds to a ratio versus other cgroups with cpu shares.
Usually the default value is ``1024``. That means if you want to allow
to use 50% of a single CPU, you will set ``512`` as value.

.. code::

   [cpu]
       shares = 512

A cgroup can get more than its share of CPU if there are enough idle CPU
cycles available in the system, due to the work conserving nature of the
scheduler, so a contained process can consume all CPU cycles even with a
ratio of 50%. The ratio is only applied when two or more processes
conflicts with their needs of CPU cycles.

**quota/period**

You can enforce hard limits on the CPU cycles a cgroup can consume, so
contained processes can't use more than the amount of CPU time set for
the cgroup. ``quota`` allows you to configure the amount of CPU time
that a cgroup can use per period. The default is 100ms (100000us). So if
you want to limit amount of CPU time to 20ms during period of 100ms:

.. code::

   [cpu]
       period = 100000
       quota = 20000

**cpus/mems**

You can also restrict access to specific CPUs (cores) and associated
memory nodes by using ``cpus/mems`` fields:

.. code::

   [cpu]
       cpus = "0-1"
       mems = "0-1"

Where container has limited access to CPU 0 and CPU 1.

.. note::

   It's important to set identical values for both ``cpus`` and
   ``mems``.

Limiting IO
-----------

To control block device I/O, applying limits to competing container, use
the ``[blockIO]`` section of the TOML file:

.. code::

   [blockIO]
       weight = 1000
       leafWeight = 1000

``weight`` and ``leafWeight`` accept values between ``10`` and ``1000``.

``weight`` is the default weight of the group on all the devices until
and unless overridden by a per device rule.

``leafWeight`` relates to weight for the purpose of deciding how heavily
to weigh tasks in the given cgroup while competing with the cgroup's
child cgroups.

To apply limits to specific block devices, you must set configuration
for specific device major/minor numbers. For example, to override
``weight/leafWeight`` for ``/dev/loop0`` and ``/dev/loop1`` block
devices, set limits for device major 7, minor 0 and 1:

.. code::

   [blockIO]
       [[blockIO.weightDevice]]
           major = 7
           minor = 0
           weight = 100
           leafWeight = 50
       [[blockIO.weightDevice]]
           major = 7
           minor = 1
           weight = 100
           leafWeight = 50

You can also limit the IO read/write rate to a specific absolute value,
e.g. 16MB per second for the ``/dev/loop0`` block device. The ``rate``
is specified in bytes per second.

.. code::

   [blockIO]
       [[blockIO.throttleReadBpsDevice]]
           major = 7
           minor = 0
           rate = 16777216
       [[blockIO.throttleWriteBpsDevice]]
           major = 7
           minor = 0
           rate = 16777216

Other limits
------------

{Singularity} can apply all resource limits that are valid in the OCI
runtime-spec ``resources`` section, including ``unified`` cgroups v2
constraints. It is most compatible, however, to use the cgroups v1 limits,
which will be translated to v2 format when applied on a cgroups v2 system.

See
https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md#control-groups
for information about the available limits. Note that {Singularity} uses
TOML format for the configuration file, rather than JSON.

.. _execution_control_list:

**********
 ecl.toml
**********

The execution control list that can be used to restrict the execution of
SIF files by signing key is defined here. You can authorize the
containers by validating both the location of the SIF file in the
filesystem and by checking against a list of signing entities.

.. warning::

   If unprivileged user namespace creation is allowed on a system, a user can
   supply and use their own unprivileged installation of Singularity or another
   container runtime to bypass container limits. They may also be able to use
   standard system tools such as ``unshare``, ``nsenter``, and FUSE mounts to
   access / execute arbitrary containers without installing any runtime.

   The ECL is not effective if unprivileged user namespaces are enabled. It is
   only effectively applied when Singularity is running using the native runtime
   in setuid mode, and unprivileged container execution is not possible on the
   host.

   **You must disable unprivileged user namespace creation on the host if you
   rely on the ECL limit container execution.** This will disable OCI mode,
   which is unprivileged and cannot enforce these limits.

.. warning::

   The ECL configuration applies to SIF container images only. To lock
   down execution fully you should disable execution of other container
   types (squashfs/extfs/dir) via the ``singularity.conf`` file ``allow
   container`` settings.

.. code::

   [[execgroup]]
     tagname = "group2"
     mode = "whitelist"
     dirpath = "/tmp/containers"
     keyfp = ["7064B1D6EFF01B1262FED3F03581D99FE87EAFD1"]

Only the containers running from and signed with above-mentioned path
and keys will be authorized to run.

Three possible list modes you can choose from:

**Whitestrict**: The SIF must be signed by all of the keys mentioned.

**Whitelist**: As long as the SIF is signed by one or more of the keys,
the container is allowed to run.

**Blacklist**: Only the containers whose keys are not mentioned in the
group are allowed to run.

.. note::

   The ECL checks will use the new signature format introduced in
   {Singularity} 3.6.0. Containers signed with older versions of
   Singularity {Singularity} will not pass ECL checks.

   To temporarily permit the use of legacy insecure signatures, set
   ``legacyinsecure = true`` in ``ecl.toml``.

Managing ECL public keys
========================

Since {Singularity} 3.7.0 a global keyring is used for ECL signature
verification. This keyring can be administered using the ``--global``
flag for the following commands:

-  ``singularity key import`` (root user only)
-  ``singularity key pull`` (root user only)
-  ``singularity key remove`` (root user only)
-  ``singularity key export``
-  ``singularity key list``

.. note::

   For security reasons, it is not possible to import private keys into
   this global keyring because it must be accessible by users and is
   stored in the file ``SYSCONFDIR/singularity/global-pgp-public``.

.. _gpu_library_configuration:

***************************
 GPU Library Configuration
***************************

When a container includes a GPU enabled application, {Singularity} (with
the ``--nv`` or ``--rocm`` options) can properly inject the required
Nvidia or AMD GPU driver libraries into the container, to match the
host's kernel. The GPU ``/dev`` entries are provided in containers run
with ``--nv`` or ``--rocm`` even if the ``--contain`` option is used to
restrict the in-container device tree.

Compatibility between containerized CUDA/ROCm/OpenCL applications and
host drivers/libraries is dependent on the versions of the GPU compute
frameworks that were used to build the applications. Compatibility and
usage information is discussed in the 'GPU Support' section of the `user
guide <https://www.sylabs.io/guides/{userversion}/user-guide/>`__

NVIDIA GPUs / CUDA
==================

The ``nvliblist.conf`` configuration file is used to specify libraries
and executables that need to be injected into the container when running
{Singularity} with the ``--nv`` Nvidia GPU support option. The provided
``nvliblist.conf`` is suitable for CUDA 11, but may need to be modified
if you need to include additional libraries, or further libraries are
added to newer versions of the Nvidia driver/CUDA distribution.

When adding new entries to ``nvliblist.conf`` use the bare filename of
executables, and the ``xxxx.so`` form of libraries. Libraries are
resolved via ``ldconfig -p``, and exectuables are found by searching
``$PATH``.

Experimental nvidia-container-cli Support
-----------------------------------------

The `nvidia-container-cli
<https://github.com/NVIDIA/libnvidia-container>`_ tool is Nvidia's
officially support method for configuring containers to use a GPU. It is
targeted at OCI container runtimes.

{Singularity} 3.9 introduces an experimental ``--nvccli`` option, which
will call out to ``nvidia-container-cli`` for container GPU setup,
rather than use the ``nvliblist.conf`` approach.

To use ``--nvccli`` a root-owned ``nvidia-container-cli`` binary must be
present on the host. The binary that is run is controlled by the
``nvidia-container-cli`` directive in ``singularity.conf``. During
installation of {Singularity}, the ``./mconfig`` step will set the
correct value in ``singularity.conf`` if ``nvidia-container-cli`` is
found on the ``$PATH``. If the value of ``nvidia-container-cli path`` is
empty, {Singularity} will look for the binary on ``$PATH`` at runtime.

.. note::

   To prevent use of ``nvidia-container-cli`` via the ``--nvccli`` flag,
   you may set ``nvidia-container-cli path`` to ``/bin/false`` in
   ``singularity.conf``.

``nvidia-container-cli`` is run as the ``root`` user during setuid
operation of {Singularity}. The container starter process grants a
number of Linux capabilities to ``nvidia-container-cli``, which are
required for it to configure the container for GPU operation. The
operations performed by ``nvidia-container-cli`` are broadly similar to
those which {Singularity} carries out when setting up a GPU container
from ``nvliblist.conf``.

AMD Radeon GPUs / ROCm
======================

The ``rocmliblist.conf`` file is used to specify libraries and
executables that need to be injected into the container when running
{Singularity} with the ``--rocm`` Radeon GPU support option. The
provided ``rocmliblist.conf`` is suitable for ROCm 4.0, but may need to
modified if you need to include additional libraries, or further
libraries are added to newer versions of the ROCm distribution.

When adding new entries to ``rocmlist.conf`` use the bare filename of
executables, and the ``xxxx.so`` form of libraries. Libraries are
resolved via ``ldconfig -p``, and exectuables are found by searching
``$PATH``.

GPU liblist format
==================

The ``nvliblist.conf`` and ``rocmliblist`` files list the basename of
executables and libraries to be bound into the container, without path
information.

Binaries are found by searching ``$PATH``:

.. code::

   # put binaries here
   # In shared environments you should ensure that permissions on these files
   # exclude writing by non-privileged users.
   rocm-smi
   rocminfo

Libraries should be specified without version information, i.e.
``libname.so``, and are resolved using ``ldconfig``.

.. code::

   # put libs here (must end in .so)
   libamd_comgr.so
   libcomgr.so
   libCXLActivityLogger.so

If you receive warnings that binaries or libraries are not found, ensure
that they are in a system path (binaries), or available in paths
configured in ``/etc/ld.so.conf`` (libraries).

*****************
 capability.json
*****************

Native runtime / non-OCI-Mode
=============================

In {Singularity}'s default configuration, without ``--oci``, a container started
by root receives all capabilities, while a container started by a non-root user
receives no capabilities.

Additionally, {Singularity} provides support for granting and revoking Linux
capabilities on a user or group basis. The ``capability.json`` file is
maintained by {Singularity} in order to manage these capabilities. The
``capability`` command group allows you to ``add``, ``drop``, and
``list`` capabilities for users and groups.

.. warning::

   In {Singularity}'s default setuid and non-OCI mode, containers are only
   isolated in a mount namespace. A user namespace, which limits the scope of
   capabilities, is not used by default.

   Therefore, it is extremely important to recognize that **granting users Linux
   capabilities with the** ``capability`` **command group is usually
   identical to granting those users root level access on the host
   system**. Most if not all capabilities will allow users to "break
   out" of the container and become root on the host. This feature is
   targeted toward special use cases (like cloud-native architectures)
   where an admin/developer might want to limit the attack surface
   within a container that normally runs as root. This is not a good
   option in multi-tenant HPC environments where an admin wants to grant
   a user special privileges within a container. For that and similar
   use cases, the :ref:`fakeroot feature <fakeroot>` is a better option.

For example, let us suppose that we have decided to grant a user (named
``pinger``) capabilities to open raw sockets so that they can use
``ping`` in a container where the binary is controlled via capabilities.

To do so, we would issue a command such as this:

.. code::

   $ sudo singularity capability add --user pinger CAP_NET_RAW

This means the user ``pinger`` has just been granted permissions
(through Linux capabilities) to open raw sockets within {Singularity}
containers.

We can check that this change is in effect with the ``capability list``
command.

.. code::

   $ sudo singularity capability list --user pinger
   CAP_NET_RAW

To take advantage of this new capability, the user ``pinger`` must also
request the capability when executing a container with the
``--add-caps`` flag. ``pinger`` would need to run a command like this:

.. code::

   $ singularity exec --add-caps CAP_NET_RAW \
     library://sylabs/tests/ubuntu_ping:v1.0 ping -c 1 8.8.8.8
   PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
   64 bytes from 8.8.8.8: icmp_seq=1 ttl=52 time=73.1 ms

   --- 8.8.8.8 ping statistics ---
   1 packets transmitted, 1 received, 0% packet loss, time 0ms
   rtt min/avg/max/mdev = 73.178/73.178/73.178/0.000 ms

If we decide that it is no longer necessary to allow the user ``pinger``
to open raw sockets within {Singularity} containers, we can revoke the
appropriate Linux capability like so:

.. code::

   $ sudo singularity capability drop --user pinger CAP_NET_RAW

Now if ``pinger`` tries to use ``CAP_NET_RAW``, {Singularity} will not
give the capability to the container and ``ping`` will fail to create a
socket:

.. code::

   $ singularity exec --add-caps CAP_NET_RAW \
     library://sylabs/tests/ubuntu_ping:v1.0 ping -c 1 8.8.8.8
   WARNING: not authorized to add capability: CAP_NET_RAW
   ping: socket: Operation not permitted

The ``capability add`` and ``drop`` subcommands will also accept the
case insensitive keyword ``all`` to grant or revoke all Linux
capabilities to a user or group.

For more information about individual Linux capabilities check out the
`man pages <http://man7.org/linux/man-pages/man7/capabilities.7.html>`_
or use the ``capability avail`` command to output available capabilities
with a description of their behaviors.

OCI-Mode
========

When containers are run in OCI-mode, by a non-root user, initialization is
always performed inside a user namespace. The capabilities granted to a
container are specific to this user namespace. For example, ``CAP_SYS_ADMIN``
granted to an OCI-mode container does not give the user the ability to mount a
filesystem outside of the container's user namespace.

Because of this isolation of capabilities users can add and drop capabilities,
using ``--add-caps`` and ``--drop-caps``, without the need for the administrator
to have granted permission to do so with the ``singularity capabilities``
command. The ``capability.json`` file is not consulted.

OCI-mode containers do not inherit the user's own capabilities, but instead run
with a default set of capabilities that matches other OCI runtimes.

-  CAP_NET_RAW
-	CAP_NET_BIND_SERVICE
-	CAP_AUDIT_READ
-	CAP_AUDIT_WRITE
-	CAP_DAC_OVERRIDE
-	CAP_SETFCAP
-	CAP_SETPCAP
-	CAP_SETGID
-	CAP_SETUID
-	CAP_MKNOD
-	CAP_CHOWN
-	CAP_FOWNER
-	CAP_FSETID
-	CAP_KILL
-	CAP_SYS_CHROOT

When the container is entered as the root user (e.g. with ``--fakeroot``), these
default capabilities are added to the effective, permitted, and bounding sets.

When the container is entered as a non-root user, these default capabilities are
added to the bounding set.

******************
 seccomp-profiles
******************

Secure Computing (seccomp) Mode is a feature of the Linux kernel that
allows an administrator to filter system calls being made from a
container. Profiles made up of allowed and restricted calls can be
passed to different containers. *Seccomp* provides more control than
*capabilities* alone, giving a smaller attack surface for an attacker to
work from within a container.

You can set the default action with ``defaultAction`` for a non-listed
system call. Example: ``SCMP_ACT_ALLOW`` filter will allow all the
system calls if it matches the filter rule and you can set it to
``SCMP_ACT_ERRNO`` which will have the thread receive a return value of
*errno* if it calls a system call that matches the filter rule. The file
is formatted in a way that it can take a list of additional system calls
for different architecture and {Singularity} will automatically take
syscalls related to the current architecture where it's been executed.
The ``include``/``exclude``-> ``caps`` section will include/exclude the
listed system calls if the user has the associated capability.

Use the ``--security`` option to invoke the container like:

.. code::

   $ sudo singularity shell --security seccomp:/home/david/my.json my_container.sif

For more insight into security options, network options, cgroups,
capabilities, etc, please check the `Userdocs
<https://www.sylabs.io/guides/{userversion}/user-guide/>`_ and it's
`Appendix
<https://www.sylabs.io/guides/{userversion}/user-guide/appendix.html>`_.

*************
 remote.yaml
*************

System-wide remote endpoints are defined in a configuration file
typically located at ``/usr/local/etc/singularity/remote.yaml`` (this
location may vary depending on installation parameters) and can be
managed by administrators with the ``remote`` command group.

Remote Endpoints
================

Sylabs introduced the online `Sylabs Cloud
<https://cloud.sylabs.io/home>`_ to enable users to `Create
<https://cloud.sylabs.io/builder>`_, `Secure
<https://cloud.sylabs.io/keystore?sign=true>`_, and `Share
<https://cloud.sylabs.io/library/guide#create>`_ their container images
with others.

{Singularity} allows users to login to an account on the Sylabs Cloud,
or configure {Singularity} to use an API compatible container service
such as a local installation of {Singularity} Enterprise, which provides
an on-premise private Container Library, Remote Builder and Key Store.

.. note::

   A fresh installation of {Singularity} is automatically configured to
   connect to the public `Sylabs Cloud <https://cloud.sylabs.io>`__
   services.

**Examples**

Use the ``remote`` command group with the ``--global`` flag to create a
system-wide remote endpoint:

.. code::

   $ sudo singularity remote add --global company-remote https://enterprise.example.com
   INFO:    Remote "company-remote" added.
   INFO:    Global option detected. Will not automatically log into remote.

Conversely, to remove a system-wide endpoint:

.. code::

   $ sudo singularity remote remove --global company-remote
   INFO:    Remote "company-remote" removed.

.. note::

   Once users log in to a system wide endpoint, a copy of the endpoint
   will be listed in a their ``~/.singularity/remote.yaml`` file. This
   means modifications or removal of the system-wide endpoint will not
   be reflected in the users configuration unless they remove the
   endpoint themselves.

Exclusive Endpoint
------------------

{Singularity} 3.7 introduces the ability for an administrator to make a
remote the only usable remote for the system by using the
``--exclusive`` flag:

.. code::

   $ sudo singularity remote use --exclusive company-remote
   INFO:    Remote "company-remote" now in use.
   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME            URI                     ACTIVE  GLOBAL  EXCLUSIVE  INSECURE
   SylabsCloud     cloud.sylabs.io         NO      YES     NO         NO
   company-remote  enterprise.example.com  YES     YES     YES        NO
   myremote        enterprise.example.com  NO      NO      NO         NO

   Keyservers
   ==========

   URI                       GLOBAL  INSECURE  ORDER
   https://keys.example.com  YES     NO        1*

   * Active cloud services keyserver

Insecure (HTTP) Endpoints
-------------------------

From {Singularity} 3.9, if you are using a endpoint that exposes its
service discovery file over an insecure HTTP connection only, it can be
added by specifying the ``--insecure`` flag:

.. code::

   $ sudo singularity remote add --global --insecure test http://test.example.com
   INFO:    Remote "test" added.
   INFO:    Global option detected. Will not automatically log into remote.

This flag controls HTTP vs HTTPS for service discovery only. The
protocol used to access individual library, build and keyserver URLs is
set by the service discovery file.

Additional Information
----------------------

For more details on the ``remote`` command group and managing remote
endpoints, please check the `Remote Userdocs
<https://www.sylabs.io/guides/{userversion}/user-guide/endpoint.html>`_.

.. _keyserver:

Keyserver Configuration
=======================

By default, {Singularity} will use the keyserver correlated to the active cloud
service endpoint. This behavior can be changed or supplemented via the
``keyserver add`` and ``keyserver remove`` commands. These commands allow an
administrator to create a global list of key servers used to verify container
signatures by default.

For more details on the ``keyserver`` command group and managing keyservers,
please see the `Keyserver Management section
<https://www.sylabs.io/guides/{userversion}/user-guide/keyserver.html>`_ of the
user guide.
