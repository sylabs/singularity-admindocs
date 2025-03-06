###################
 Admin Quick Start
###################

This quick start gives an overview of installation of {Singularity} from
source, a description of the architecture of {Singularity}, and pointers
to configuration files. More information, including alternate
installation options and detailed configuration options can be found
later in this guide.

.. _singularity-architecture:

*******************************
 Architecture of {Singularity}
*******************************

{Singularity} is designed to allow containers to be executed as if they
were native programs or scripts on a host system. No daemon is required
to build or run containers, and the security model is compatible with
shared systems.

As a result, integration with clusters and schedulers such as Univa Grid
Engine, Torque, SLURM, SGE, and many others is as simple as running any
other command. All standard input, output, errors, pipes, IPC, and other
communication pathways used by locally running programs are synchronized
with the applications running locally within the container.

Beginning with {Singularity} 4 there are two modes in which to run containers:

- The default native mode uses a runtime that is unique to {Singularity}. This
  is fully compatible with containers built for, and by, {Singularity} versions
  2 and 3.

- The optional OCI-mode uses a standard low-level OCI runtime to execute OCI
  containers natively, for improved compatibility.

{Singularity} favors an 'integration over isolation' approach to containers in
native mode. By default only the mount namespace is isolated for containers, so
that they have their own filesystem view. Default access to user home
directories, ``/tmp`` space, and installation specific mounts makes it simple
for users to benefit from the reproducibility of containerized applications
without major changes to their existing workflows.

In OCI-mode, more isolation is used by default so that behavior is similar to
Docker and other OCI runtimes. However, networking is not virtualized and
{Singularity}'s traditional behavior can be enabled with the ``--no-compat``
option.

In both modes, access to hardware such as GPUs, high speed networks, and shared
filesystems is easy and does not require special configuration.  Where more
complete isolation is important, {Singularity} can use additional Linux
namespaces and other security and resource limits for that purpose.

.. _singularity-security:

************************
 {Singularity} Security
************************

.. note::

   See also the :ref:`security section <security>` of this guide, for more
   detail.

When using the native runtime, a default installation of {Singularity} runs
small amounts of privileged container setup code via a ``starter-setuid``
binary. This is a 'setuid root' binary, used so that {Singularity} can perform
mounts, create namespaces, and enter containers even on older systems that lack
support for fully unprivileged container execution. The setuid flow is the
default mode of operation, but :ref:`can be disabled <install-nonsetuid>` upon
build, or in the ``singularity.conf`` configuration file if required.

If setuid is disabled, or OCI-mode is used, {Singularity} sets up containers
within an unprivileged user namespace. This makes use of features of newer
kernels, as well as user space filesystem mounts (FUSE).

.. note::

   Running {Singularity} in non-setuid mode requires unprivileged user namespace
   support in the operating system kernel and does not support all features.
   This impacts integrity/security guarantees of containers at runtime.

   See the :ref:`non-setuid installation section <install-nonsetuid>`
   for further detail on how to install {Singularity} to run in
   non-setuid mode.


{Singularity} uses a number of strategies to provide safety and
ease-of-use on both single-user and shared systems. Notable security
features include:

   -  When using the default native runtime, with container setup via a setuid
      helper program, the effective user inside a container is the same as the
      user who ran the container. This means access to files and devices from
      the container is easily controlled with standard POSIX permissions.

   -  When using OCI-mode, or an unprivileged installation, subuid/subgid
      mappings allows users access to other uids & gids in the container, which
      map to safe administrator-defined ranges on the host.

   -  Container filesystems are mounted ``nosuid`` and container
      applications run with the prctl ``NO_NEW_PRIVS`` flag set. This means
      that applications in a container cannot gain additional
      privileges. A regular user cannot ``sudo`` or otherwise gain root
      privilege on the host via a container.

   -  The Singularity Image Format (SIF) supports encryption of
      containers, as well as cryptographic signing and verification of
      their content.

   -  SIF containers are immutable and their payload is run directly,
      without extraction to disk. This means that the container can
      always be verified, even at runtime, and encrypted content is not
      exposed on disk.

   -  Restrictions can be configured to limit the ownership, location,
      and cryptographic signatures of containers that are permitted to
      be run.

*******************
 OCI Compatibility
*******************

{Singularity} allows users to run, and build from, the majority of OCI
containers created with tools such as Docker. Beginning with {Singularity} 3.11,
there are two modes of operation that support OCI containers in different ways.

{Singularity}'s *native runtime*, used by default, supports all features that
are exposed via the ``singularity`` command. It builds and runs containers in
{Singularity}'s own on-disk formats. When an OCI container is pulled or built
into a {Singularity} image, a translation step occurs. While most OCI images are
supported as-is, there are some limitations and compatibility options may be
required.

{Singularity} 4's OCI-mode, enabled with the ``--oci`` flag,
runs containers using a low-level OCI runtime - either ``crun`` or ``runc``. The
container is executed from a native OCI format on-disk. Not all CLI features are
currently implemented, but OCI containers using the ``USER`` directive or which
are otherwise incompatible with {Singularity}'s native runtime are better
supported. Note that OCI-mode has additional :ref:`system requirements <system-requirements>`.

***********************
 Version Compatibility
***********************

Up to and including version 4, the major version number of {Singularity} was
increased only when a significant change or addition was made to the container
format:

- v1 packaged applications in a different manner than later versions, and was
  not widely deployed.
- v2 used extfs or squashfs bare image files for the container root filesystem.
- v3 introduced and switched to the Singularity Image Format (SIF).
- v4 added OCI-SIF images, a variant of SIF encapsulating OCI containers
  directly. These are used by the new OCI-mode.

Minor versions, e.g. within the 3.x series, frequently introduced changes to
existing behavior not related to the basic container format.

Beginning with version 4, {Singularity} aims to follow `semantic versioning
<https://semver.org/>`__ where breaking changes to the CLI or runtime behavior
will also be limited to a new major version. New features that do not modify
existing behavior may be introduced in minor version updates.

Backward Compatibility
======================

Execution of container images from 2 prior major versions is supported.
{Singularity} 4 can run container images created with versions 2 and 3. Except
where documented in the project changelog, differences in behaviour when
running v2 or v3 containers using the native runtime in setuid mode are
considered bugs.

{Singularity} 4's OCI-mode cannot perfectly emulate the behavior of the native
runtime in setuid mode. Although most workflows are supported, complex
containers created with {Singularity} 2 or 3 may not run as expected in OCI-mode.

Forward Compatibility
=====================

{Singularity} 4 can build SIF container images that can be run with version 3.
The scope of this forward compatibility depends on the features used when
creating the container, and the 3.x minor version used to run the container:

- The OCI-SIF format (OCI-mode) is not supported before v4.
- The SIF DSSE signature format (key / certificate based signing) was introduced
  at v3.11.0.
- The SIF PGP signature format was changed at v3.6.0, therefore older versions
  cannot verify newer signatures.
- Container / host environment handling was modified at v3.6.0.
- LUKS2 encrypted containers are not supported prior to v3.4.0

{Singularity} 4.1 will build container images that can be run with version 4.0,
with the exception of multi-layer OCI-SIF images. A multi-layer OCI-SIF created
with the 4.1 ``--keep-layers`` option must be executed using version 4.1 or
later. 

**************************
 Installation from Source
**************************

{Singularity} can be installed from source directly, or by building an
RPM package from the source. Linux distributions may also package
{Singularity}, but their packages may not be up-to-date with the
upstream version on GitHub.

To install {Singularity} directly from source, follow the procedure
below. Other methods are discussed in the :ref:`Installation
<installation>` section.

.. Note::

   This quick-start that you will install as ``root`` using ``sudo``, so
   that {Singularity} uses the default ``setuid`` workflow, and all
   features are available. See the :ref:`non-setuid installation
   <install-nonsetuid>` section of this guide for detail of how to
   install as a non-root user, and how this affects the functionality of
   {Singularity}.

Install Dependencies
====================

On Debian-based systems, including Ubuntu:

.. code::

   # Ensure repositories are up-to-date
   sudo apt-get update
   # Install debian packages for dependencies
   sudo apt-get install -y \
      autoconf \
      automake \
      cryptsetup \
      fuse \
      fuse2fs \
      git \
      libfuse-dev \
      libglib2.0-dev \
      libseccomp-dev \
      libtool \
      pkg-config \
      runc \
      squashfs-tools \
      squashfs-tools-ng \
      uidmap \
      wget \
      zlib1g-dev

On versions 8 or later of RHEL / Alma Linux / Rocky Linux, as well as on Fedora:

.. code::

   # Install basic tools for compiling
   sudo dnf groupinstall -y 'Development Tools'
   # Install RPM packages for dependencies
   sudo dnf install -y \
      autoconf \
      automake \
      crun \
      cryptsetup \
      fuse \
      fuse3 \
      fuse3-devel \
      git \
      glib2-devel \
      libseccomp-devel \
      libtool \
      squashfs-tools \
      wget \
      zlib-devel

.. note::

   You can build {Singularity} without ``cryptsetup`` available, but you will
   not be able to use encrypted containers without it installed on your system.

   If you will not use the ``singularity oci`` commands, or OCI-mode, ``crun`` /
   ``runc`` is not required.

Install sqfstar / tar2sqfs for OCI-mode
=======================================

If you intend to use the ``--oci`` execution mode of SingularityCE, your system
must provide either:

- ``squashfs-tools / squashfs`` >= 4.5, which provides the ``sqfstar`` utility.
  Older versions packaged by many distributions do not include ``sqfstar``.
- ``squashfs-tools-ng``, which provides the ``tar2sqfs`` utility. This is not
  packaged by all distributions.

Debian / Ubuntu
---------------

On Debian/Ubuntu ``squashfs-tools-ng`` is available in the distribution
repositories. It has been included in the "Install system dependencies" step
above. No further action is necessary.

RHEL / Alma Linux / Rocky Linux
-------------------------------

On RHEL and derivatives, the ``squashfs-tools-ng`` package is now
available in the EPEL repositories.

Follow the `EPEL Quickstart <https://docs.fedoraproject.org/en-US/epel/#_quickstart>`__
for you distribution to enable the EPEL repository. Install ``squashfs-tools-ng`` with
``dnf``.

.. code::

   sudo dnf install squashfs-tools-ng

Install Go
==========

{Singularity} is written in Go, and you will need Go installed to
compile it from source. Versions of Go packaged by your distribution may not be
new enough to build {Singularity}.

{SingularityCE} aims to maintain support for the two most recent stable versions
of Go. This corresponds to the Go `Release Maintenance
Policy <https://github.com/golang/go/wiki/Go-Release-Cycle#release-maintenance>`_
and `Security Policy <https://golang.org/security>`_, ensuring critical bug fixes
and security patches are available for all supported language versions.

The method below is one of several ways to `install and configure Go
<https://golang.org/doc/install>`_.

.. note::

   If you have previously installed Go from a download, rather than an
   operating system package, you should remove your ``go`` directory,
   e.g. ``rm -r /usr/local/go`` before installing a newer version.
   Extracting a new version of Go over an existing installation can lead
   to errors when building Go programs, as it may leave old files, which
   have been removed or replaced in newer versions.

Visit the `Go download page <https://golang.org/dl/>`_ and pick a
package archive to download. Copy the link address and download with
wget. Then extract the archive to ``/usr/local`` (or use other
instructions on go installation page).

.. code::

   $ export VERSION={GoVersion} OS=linux ARCH=amd64 && \
       wget https://dl.google.com/go/go$VERSION.$OS-$ARCH.tar.gz && \
       sudo tar -C /usr/local -xzvf go$VERSION.$OS-$ARCH.tar.gz && \
       rm go$VERSION.$OS-$ARCH.tar.gz

Finally, add ``/usr/local/go/bin`` to the ``PATH`` environment variable:

.. code::

   echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
   source ~/.bashrc


Download {Singularity} from a GitHub release
============================================

You can download {Singularity} from one of the releases. To see a full
list, visit `the GitHub release page
<https://github.com/sylabs/singularity/releases>`_. After deciding on a
release to install, you can run the following commands to proceed with
the installation.

.. code::

   $ export VERSION={InstallationVersion} && # adjust this as necessary \
       wget https://github.com/sylabs/singularity/releases/download/v${VERSION}/singularity-ce-${VERSION}.tar.gz && \
       tar -xzf singularity-ce-${VERSION}.tar.gz && \
       cd singularity-ce-${VERSION}

Compile & Install {Singularity}
===============================

{Singularity} uses a custom build system called ``makeit``. ``mconfig``
is called to generate a ``Makefile`` and then ``make`` is used to
compile and install.

.. code::

   $ ./mconfig && \
       make -C ./builddir && \
       sudo make -C ./builddir install

By default {Singularity} will be installed in the ``/usr/local``
directory hierarchy. You can specify a custom directory with the
``--prefix`` option, to ``mconfig``:

.. code::

   $ ./mconfig --prefix=/opt/singularity

This option can be useful if you want to install multiple versions of
Singularity, install a personal version of {Singularity} on a shared
system, or if you want to remove {Singularity} easily after installing
it.

For a full list of ``mconfig`` options, run ``mconfig --help``. Here are
some of the most common options that you may need to use when building
{Singularity} from source.

-  ``--sysconfdir``: Install read-only config files in sysconfdir. This
   option is important if you need the ``singularity.conf`` file or
   other configuration files in a custom location.

-  ``--localstatedir``: Set the state directory where containers are
   mounted. This is a particularly important option for administrators
   installing {Singularity} on a shared file system. The
   ``--localstatedir`` should be set to a directory that is present on
   each individual node.

-  ``-b``: Build {Singularity} in a given directory. By default this is
   ``./builddir``.

- ``--without-conmon``: Do not build ``conmon``, a container monitor that is
  used by the ``singularity oci`` commands. ``conmon`` is bundled with
  {Singularity} and will be built and installed by default. Use
  ``--without-conmon`` if you wish to use a version of ``conmon`` >=2.0.24 that
  is provided by your distribution rather than the bundled version. You can also
  specify ``--without-conmon`` if you know you will not use the ``singularity
  oci`` commands.


************************************
 Installation from RPM/Deb Packages
************************************

Sylabs provides ``.rpm`` packages of {Singularity}, for
mainstream-supported versions of RHEL and derivatives (e.g. Alma Linux
/ Rocky Linux). We also provide ``.deb`` packages for current Ubuntu
LTS releases.

These packages can be downloaded from the `GitHub release
page <https://github.com/sylabs/singularity/releases>`_ and installed
using your distribution's package manager.

The packages are provided as a convenience for users of the open
source project, and are built in our public CircleCI workflow. They are not
signed, but SHA256 sums are provided on the release page.

***************
 Configuration
***************

{Singularity} is configured using files under ``etc/singularity`` in your
``--prefix``, or ``--syconfdir`` if you used that option with ``mconfig``. In a
default installation from source without a ``--prefix`` set you will find them
under ``/usr/local/etc/singularity``. In a default installation from RPM or Deb
packages you will find them under ``/etc/singularity``.

You can edit these files directly, or using the ``{Singularity} config
global`` command as the root user to manage them.

``singularity.conf`` contains the majority of options controlling the
runtime behavior of {Singularity}. Additional files control security,
network, and resource configuration. Head over to the
:ref:`Configuration files <singularity_configfiles>` section where the
files and configuration options are discussed.

********************
 Test {Singularity}
********************

You can run a quick test of {Singularity} using a container in the
Sylabs Container Library:

.. code::

   $ singularity exec library://alpine cat /etc/alpine-release
   3.9.2

See the `user guide
<https://www.sylabs.io/guides/{userversion}/user-guide/>`__ for more
information about how to use {Singularity}.
