.. _installation:

##########################
 Installing {Singularity}
##########################

This section will guide you through the process of installing
{Singularity} {InstallationVersion} via several different methods. (For
instructions on installing earlier versions of {Singularity} please see
`earlier versions of the docs <https://www.sylabs.io/docs/>`_.)

***********************
 Installation on Linux
***********************

{Singularity} can be installed on any modern Linux distribution, on
bare-metal or inside a Virtual Machine. Nested installations inside
containers are not recommended, and require the outer container to be
run with full privilege.

.. _system-requirements:

System Requirements
===================

{Singularity} requires ~163MiB disk space once compiled and installed.

There are no specific CPU or memory requirements at runtime, though 2GB
of RAM is recommended when building from source.

Full functionality of {Singularity} requires that the kernel supports:

-  **OverlayFS mounts** - (minimum kernel >=3.18) Required for full
   flexibility in bind mounts to containers, and to support persistent
   overlays for writable containers.

-  **Unprivileged user namespaces** - (minimum kernel >=3.8, >=3.18 recommended)
   Required to run containers without root or setuid privilege. Required to
   build containers unprivileged in ``--fakeroot`` mode. Required to run
   containers in OCI-mode (``-oci``).

- **FUSE in unprivileged user namespaces** - (minimum kernel >=4.18) Required to
  run containers in OCI-Mode (``-oci``).

-  **Unprivileged overlay** - (minimum kernel >=5.11, >=5.13 recommended)
   Required to use ``--overlay``, to mount a persistent overlay directory onto
   the container, when running without root or setuid.

External Binaries
-----------------

Singularity depends on a number of external binaries for full functionality. The
methods that are used to find these binaries have been standardized as below.

Bundled Utilities
^^^^^^^^^^^^^^^^^

In a standard {Singularity} installation, the following are bundled and
installed into {Singularity}'s ``libexec/bin`` directory. However, at
compilation time ``mconfig`` options can be used to disable building these
tools, in which case they will be searched for on ``$PATH`` at runtime.

- ``squashfuse`` or ``squashfuse_ll`` are used to mount squashfs filesystems
  from OCI-SIF images in OCI-mode.

- ``conmon`` is used to manage monitoring and attaching to non-interactive
  containers started with the ``singularity oci start`` command.

Configurable Paths
^^^^^^^^^^^^^^^^^^

The following binaries are found on ``$PATH`` during build time when
``./mconfig`` is run, and their location is added to the
``singularity.conf`` configuration file. At runtime this configured
location is used. To specify an alternate executable, change the
relevant path entry in ``singularity.conf``.

-  ``cryptsetup`` version 2 with kernel LUKS2 support is required for
   building or executing encrypted containers.

-  ``ldconfig`` is used to resolve library locations / symlinks when
   using the ``-nv`` or ``--rocm`` GPU support.

-  ``nvidia-container-cli`` is used to configure a container for Nvidia
   GPU / CUDA support when running with the experimental ``--nvccli``
   option.

For the following additional binaries, if the ``singularity.conf`` entry
is left blank, then ``$PATH`` will be searched at runtime.

-  ``go`` is required to compile plugins, and must be an identical
   version as that used to build {Singularity}.

-  ``mksquashfs`` from squashfs-tools 4.3+ is used to create the
   squashfs container filesystem that is embedded into SIF container
   images. The ``mksquashfs procs`` and ``mksquashfs mem`` directives in
   ``singularity.conf`` can be used to control its resource usage.

-  ``unsquashfs`` from squashfs-tools 4.3+ is used to extract the
   squashfs container filesystem from a SIF file when necessary.

Searching $PATH
^^^^^^^^^^^^^^^

The following utilities are always found by searching ``$PATH`` at
runtime:

-  ``true``

-  ``mkfs.ext3`` is used to create overlay images.

-  ``cp``

-  ``dd``

-  ``newuidmap`` and ``newgidmap`` are distribution provided setuid
   binaries used to configure subuid/gid mappings for ``--fakeroot`` in
   non-setuid installs, and in OCI-mode.

-  ``crun`` or ``runc`` are OCI runtimes used for the ``singularity oci``
   commands and OCI-mode for ``run / shell / exec``. ``crun`` is preferred over
   ``runc`` if it is available. ``runc`` is provided by a package in all common
   Linux distributions. ``crun`` is packaged in more recent releases of common
   Linux distributions.

-  ``proot`` is an optional dependency that can be used to permit
   limited unprivileged builds without user namespace / subuid
   support. It is packaged in the community repositories for common
   Linux distributions, and is available as a static binary from
   `proot-me.github.io <https://proot-me.github.io>`__.

- ``sqfstar`` or ``tar2sqfs`` are used in the creation of OCI-SIF images from
  OCI sources, in OCI-mode (``--oci``).

- ``fuse-overlayfs`` is used in OCI-mode to setup overlay filesystems when the
  kernel does not support unprivileged overlay or the required overlay
  configuration.

- ``fusermount3`` or ``fusermount`` is used to unmount FUSE filesystems safely,
  in OCI-mode and other flows.

Bootstrap Utilities
^^^^^^^^^^^^^^^^^^^

The following utilities are required to bootstrap containerized
distributions using their native tooling:

-  ``mount``, ``umount``, ``pacstrap`` for Arch Linux.
-  ``mount``, ``umount``, ``mknod``, ``debootstrap`` for Debian based
   distributions.
-  ``dnf`` or ``yum``, ``rpm``, ``curl`` for EL derived RPM based
   distributions.
-  ``uname``, ``zypper``, ``SUSEConnect`` for SLES derived RPM based
   distributions.

Installing sqfstar / tar2sqfs for OCI-mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you intend to use the  `OCI mode
<https://sylabs.io/guides/{userversion}/user-guide/oci_runtime.html>`_ of
{Singularity}, your system must provide either:

* ``squashfs-tools`` / ``squashfs`` >= 4.5, which provides the ``sqfstar``
  utility. Note that older versions of these packages, provided by many
  distributions, do not include ``sqfstar``.
* ``squashfs-tools-ng``, which provides the ``tar2sqfs`` utility. This is not
  packaged by all distributions.

Below are instructions on how to obtain one of these two utilities on various
distributions.

Debian / Ubuntu
"""""""""""""""

On Debian/Ubuntu, ``squashfs-tools-ng`` is available in the distribution
repositories. No further action is necessary.

Fedora
""""""

On Fedora, the ``squashfs-tools`` package, available in the repositories,
includes `sqfstar`. No further action is necessary.

RHEL / Alma Linux / Rocky Linux / CentOS
""""""""""""""""""""""""""""""""""""""""

On RHEL and derivatives, a COPR is available at:
https://copr.fedorainfracloud.org/coprs/dctrud/squashfs-tools-ng/

This COPR provides ``squashfs-tools-ng``, which will not replace any standard EL
or EPEL packages. To use it:

**EL 8 / 9:**

.. code::

  sudo dnf install dnf-plugins-core
  sudo dnf copr enable dctrud/squashfs-tools-ng
  sudo dnf install squashfs-tools-ng

**EL 7:**

.. code::

  sudo yum install yum-plugin-copr
  sudo yum copr enable dctrud/squashfs-tools-ng
  sudo yum install squashfs-tools-ng

SLES / openSUSE Leap
""""""""""""""""""""

On SLES/openSUSE, follow the instructions at the `filesystems
project <https://software.opensuse.org//download.html?project=filesystems&package=squashfs>`_
to obtain a more recent ``squashfs`` package, which provides ``sqfstar``.

Non-standard ldconfig / Nix & Guix Environments
-----------------------------------------------

If {Singularity} is installed under a package manager such as Nix or
Guix, but on top of a standard Linux distribution (e.g. CentOS or
Debian), it may be unable to correctly find the libraries for ``--nv``
and ``--rocm`` GPU support. This issue occurs as the package manager
supplies an alternative ``ldconfig``, which does not identify GPU
libraries installed from host packages.

To allow {Singularity} to locate the host (i.e. CentOS / Debian) GPU
libraries correctly, set ``ldconfig path`` in ``singularity.conf`` to
point to the host ``ldconfig``. I.E. it should be set to
``/sbin/ldconfig`` or ``/sbin/ldconfig.real`` rather than a Nix or Guix
related path.

Filesystem support / limitations
--------------------------------

{Singularity} supports most filesystems, but there are some limitations
when installing {Singularity} on, or running containers from, common
parallel / network filesystems. In general:

-  We strongly recommend installing {Singularity} on local disk on each
   compute node.

-  If {Singularity} is installed to a network location, a
   ``--localstatedir`` should be provided on each node, and Singularity
   configured to use it.

-  The ``--localstatedir`` filesystem should support overlay mounts.

-  ``TMPDIR`` / ``SINGULARITY_TMPDIR`` should be on a local filesystem
   wherever possible.

.. note::

   Set the ``--localstatedir`` location by by providing
   ``--localstatedir my/dir`` as an option when you configure your
   {Singularity} build with ``./mconfig``.

   Disk usage at the ``--localstatedir`` location is negligible (<1MiB).
   The directory is used as a location to mount the container root
   filesystem, overlays, bind mounts etc. that construct the runtime
   view of a container. You will not see these mounts from a host shell,
   as they are made in a separate mount namespace.

Overlay support
^^^^^^^^^^^^^^^

Various features of {Singularity}, such as the ``--writable-tmpfs`` and
``--overlay``, options use the Linux ``overlay`` filesystem driver to
construct a container root filesystem that combines files from different
locations. Not all filesystems can be used with the ``overlay`` driver,
so when containers are run from these filesystems some {Singularity}
features may not be available.

Overlay support has two aspects:

-  ``lowerdir`` support for a filesystem allows a directory on that
   filesystem to act as the 'base' of a container. A filesystem must
   support overlay ``lowerdir`` for you be able to run a Singularity
   sandbox container on it, while using functionality such as
   ``--writable-tmpfs`` / ``--overlay``.

-  ``upperdir`` support for a filesystem allows a directory on that
   filesystem to be merged on top of a ``lowerdir`` to construct a
   container. If you use the ``--overlay`` option to overlay a directory
   onto a container, then the filesystem holding the overlay directory
   must support ``upperdir``.

Note that any overlay limitations mainly apply to sandbox (directory)
containers only. A SIF container is mounted into the ``--localstatedir``
location, which should generally be on a local filesystem that supports
overlay.

Fakeroot & OCI-Mode subuid/gid mapping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When {Singularity} is run using the :ref:`fakeroot <fakeroot>` option, or in
OCI-Mode, it creates a user namespace for the container, and UIDs / GIDs in that
user namespace are mapped to different host UID / GIDs.

Most local filesystems (ext4/xfs etc.) support this uid/gid mapping in a
user namespace.

Most network filesystems (NFS/Lustre/GPFS etc.) *do not* support this
uid/gid mapping in a user namespace. Because the fileserver is not aware
of the mappings it will deny many operations, with 'permission denied'
errors. This is currently a generic problem for rootless container
runtimes.

{Singularity} cache / atomic rename
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

{Singularity} will cache SIF container images generated from remote
sources, and any OCI/docker layers used to create them. The cache is
created at ``$HOME/.singularity/cache`` by default. The location of the
cache can be changed by setting the ``SINGULARITY_CACHEDIR`` environment
variable.

The directory used for ``SINGULARITY_CACHEDIR`` should be:

-  A unique location for each user. Permissions are set on the cache so
   that private images cached for one user are not exposed to another.
   This means that ``SINGULARITY_CACHEDIR`` cannot be shared.

-  Located on a filesystem with sufficient space for the number and size
   of container images anticipated.

-  Located on a filesystem that supports atomic rename, if possible.

In {Singularity} version 3.6 and above the cache is concurrency safe.
Parallel runs of {Singularity} that would create overlapping cache
entries will not conflict, as long as the filesystem used by
``SINGULARITY_CACHEDIR`` supports atomic rename operations.

Support for atomic rename operations is expected on local POSIX
filesystems, but varies for network / parallel filesystems and may be
affected by topology and configuration. For example, Lustre supports
atomic rename of files only on a single MDT. Rename on NFS is only
atomic to a single client, not across systems accessing the same NFS
share.

If you are not certain that your ``$HOME`` or ``SINGULARITY_CACHEDIR``
filesystems support atomic rename, do not run ``singularity`` in parallel
using remote container URLs. Instead use ``singularity pull`` to create
a local SIF image, and then run this SIF image in a parallel step. An
alternative is to use the ``--disable-cache`` option, but this will
result in each {Singularity} instance independently fetching the
container from the remote source, into a temporary location.

NFS
^^^

NFS filesystems support overlay mounts as a ``lowerdir`` only, and do
not support user-namespace (sub)uid/gid mapping.

-  Containers run from SIF files located on an NFS filesystem do not
   have restrictions.

-  You cannot use ``--overlay mynfsdir/`` to overlay a directory onto a
   container when the overlay (upperdir) directory is on an NFS
   filesystem.

-  When using ``--fakeroot`` to build or run a container, your
   ``TMPDIR`` / ``SINGULARITY_TMPDIR`` should not be set to an NFS
   location.

-  You should not run a sandbox container with ``--fakeroot`` from an
   NFS location.

Lustre / GPFS / PanFS
^^^^^^^^^^^^^^^^^^^^^

Lustre, GPFS, and PanFS do not have sufficient ``upperdir`` or
``lowerdir`` overlay support for certain {Singularity} features, and
do not support user-namespace (sub)uid/gid mapping.

- You cannot use ``--overlay`` or ``--writable-tmpfs`` with a sandbox
  container that is located on a Lustre, GPFS, or PanFS
  filesystem. SIF containers on Lustre, GPFS, and PanFS will work
  correctly with these options.

- You cannot use ``--overlay`` to overlay a directory onto a
  container, when the overlay (upperdir) directory is on a Lustre,
  GPFS, or PanFS filesystem.

- When using ``--fakeroot`` to build or run a container, your
  ``TMPDIR/SINGULARITY_TMPDIR`` should not be a Lustre, GPFS, or
  PanFS location.

- You should not run a sandbox container with ``--fakeroot`` from a
  Lustre, GPFS, or PanFS location.

Install from Provided RPM / Deb Packages
========================================

Sylabs provides ``.rpm`` packages of {Singularity}, for
mainstream-supported versions of RHEL and derivatives (e.g. Alma Linux
/ Rocky Linux). We also provide ``.deb`` packages for current Ubuntu
LTS releases.

These packages can be downloaded from the `GitHub release
page <https://github.com/sylabs/singularity/releases>`_
and installed using your distribution's package manager.

The packages are provided as a convenience for users of the open
source project, and are built in our public CircleCI workflow. They are not
signed, but SHA256 sums are provided on the release page.

.. _install-dependencies:

Install from Source
===================

To use the latest version of {Singularity} from GitHub you will need to
build and install it from source. This may sound daunting, but the
process is straightforward, and detailed below.

If you have an earlier version of {Singularity} installed, you should
:ref:`remove it <remove-an-old-version>` before executing the
installation commands. You will also need to install some dependencies
and install `Go <https://golang.org/>`_.

Install Dependencies
--------------------

On Red Hat Enterprise Linux or CentOS install the following
dependencies:

.. code:: sh

   # Install basic tools for compiling
   sudo yum groupinstall -y 'Development Tools'
   # Install RPM packages for dependencies
   sudo yum install -y \
      libseccomp-devel \
      glib2-devel \
      squashfs-tools \
      cryptsetup \
      runc

On Ubuntu or Debian install the following dependencies:

.. code:: sh

   # Ensure repositories are up-to-date
   sudo apt-get update
   # Install debian packages for dependencies
   sudo apt-get install -y \
      build-essential \
      libseccomp-dev \
      libglib2.0-dev \
      pkg-config \
      squashfs-tools \
      cryptsetup \
      runc

.. note::

   You can build {Singularity} without ``cryptsetup`` available,
   but will not be able to use encrypted containers without it installed
   on your system.

   If you will not use the ``singularity oci`` commands, ``runc`` is not
   required.

.. _install-go:

Install Go
----------

{Singularity} is written in Go, and aims to maintain support for the two most
recent stable versions of Go. This corresponds to the Go Release Maintenance
Policy and Security Policy, ensuring critical bug fixes and security patches are
available for all supported language versions.

Building {Singularity} may require a newer version of Go than is available in
the repositories of your distribution. We recommend installing the latest
version of Go from the [official binaries](https://golang.org/dl/).

This is one of several ways to `install and configure Go
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

Then, set up your environment for Go.

.. code::

   $ echo 'export GOPATH=${HOME}/go' >> ~/.bashrc && \
       echo 'export PATH=/usr/local/go/bin:${PATH}:${GOPATH}/bin' >> ~/.bashrc && \
       source ~/.bashrc

Download {Singularity} from a release
-------------------------------------

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

Checkout Code from Git
----------------------

The following commands will install {Singularity} from the `GitHub repo
<https://github.com/sylabs/singularity>`_ to ``/usr/local``. This method
will work for >=v{InstallationVersion}. To install an older tagged
release see `older versions of the docs <https://www.sylabs.io/docs/>`_.

When installing from source, you can decide to install from either a
**tag**, a **release branch**, or from the **main branch**.

-  **tag**: GitHub tags form the basis for releases, so installing from
   a tag is the same as downloading and installing a `specific release
   <https://github.com/sylabs/singularity/releases>`_. Tags are expected
   to be relatively stable and well-tested.

-  **release branch**: A release branch represents the latest version of
   a minor release with all the newest bug fixes and enhancements (even
   those that have not yet made it into a point release). For instance,
   to install v3.10 with the latest bug fixes and enhancements checkout
   ``release-3.10``. Release branches may be less stable than code in a
   tagged point release.

-  **main branch**: The ``main`` branch contains the latest,
   bleeding edge version of {Singularity}. This is the default branch
   when you clone the source code, so you don't have to check out any
   new branches to install it. The ``main`` branch changes quickly and
   may be unstable.

To ensure that the {Singularity} source code is downloaded to the
appropriate directory use these commands.

.. code::

   $ git clone --recurse-submodules https://github.com/sylabs/singularity.git && \
       cd singularity && \
       git checkout --recurse-submodules v{InstallationVersion}

Compile Singularity
-------------------

{Singularity} uses a custom build system called ``makeit``. ``mconfig``
is called to generate a ``Makefile`` and then ``make`` is used to
compile and install.

To support the SIF image format, automated networking setup etc., and
older Linux distributions without user namespace support, Singularity
must be ``make install``ed as root or with ``sudo``, so it can install
the ``libexec/singularity/bin/starter-setuid`` binary with root
ownership and setuid permissions for privileged operations. If you need
to install as a normal user, or do not want to use setuid functionality
:ref:`see below <install-nonsetuid>`.

.. code::

   $ ./mconfig && \
       make -C ./builddir && \
       sudo make -C ./builddir install

By default {Singularity} will be installed in the ``/usr/local``
directory hierarchy. You can specify a custom directory with the
``--prefix`` option, to ``mconfig`` like so:

.. code::

   $ ./mconfig --prefix=/opt/singularity

This option can be useful if you want to install multiple versions of
{Singularity}, install a personal version of {Singularity} on a shared
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

-  ``--without-conmon``: Do not build the ``conmon`` OCI container monitor. Use
   this option if you are certain you will not use the ``singularity oci``
   commands, or wish to use conmon >=2.0.24 provided by your distribution, and
   available on ``$PATH``.

- ``--reproducible``: Enable support for reproducible builds. Ensures
   that the compiled binaries do not include any temporary paths, the
   source directory path, etc. This disables support for building plugins.

.. _install-nonsetuid:

Unprivileged (non-setuid) Installation
--------------------------------------

If you need to install {Singularity} as a non-root user, or do not wish
to allow the use of a setuid root binary, you can configure
{Singularity} with the ``--without-suid`` option to mconfig:

.. code::

   $ ./mconfig --without-suid --prefix=/home/dave/singularity-ce && \
       make -C ./builddir && \
       make -C ./builddir install

If you have already installed {Singularity} you can disable the setuid
flow by setting the option ``allow setuid = no`` in
``etc/singularity/singularity.conf`` within your installation directory.

When {Singularity} does not use setuid all container execution will use
a user namespace. This requires support from your operating system
kernel, and imposes some limitations on functionality. You should review
the :ref:`requirements <userns-requirements>` and :ref:`limitations
<userns-limitations>` in the :ref:`user namespace <userns>` section of
this guide.

Relocatable Installation
------------------------

Since {Singularity} 3.8, an unprivileged (non-setuid) installation is
relocatable. As long as the structure inside the installation directory
(``--prefix``) is maintained, it can be moved to a different location
and {Singularity} will continue to run normally.

Relocation of a default setuid installation is not supported, as
restricted location / ownership of configuration files is important to
security.

Source bash completion file
---------------------------

To enjoy bash shell completion with {Singularity} commands and options,
source the bash completion file:

.. code::

   $ . /usr/local/etc/bash_completion.d/singularity

Add this command to your ``~/.bashrc`` file so that bash completion
continues to work in new shells. (Adjust the path if you installed
{Singularity} to a different location.)

.. _install-rpm:

Build and install an RPM
========================

If you use RHEL, CentOS or SUSE, building and installing a Singularity
RPM allows your {Singularity} installation be more easily managed,
upgraded and removed. In {Singularity} >=v3.0.1 you can build an RPM
directly from the `release tarball
<https://github.com/sylabs/singularity/releases>`_.

.. note::

   Be sure to download the correct asset from the `GitHub releases page
   <https://github.com/sylabs/singularity/releases>`_. It should be
   named ``singularity-ce-<version>.tar.gz``.

After installing the :ref:`dependencies <install-dependencies>` and
installing :ref:`Go <install-go>` as detailed above, you are ready to
download the tarball and build and install the RPM.

.. code::

   $ export VERSION={InstallationVersion} && # adjust this as necessary \
       wget https://github.com/sylabs/singularity/releases/download/v${VERSION}/singularity-ce-${VERSION}.tar.gz && \
       rpmbuild -tb singularity-ce-${VERSION}.tar.gz && \
       sudo rpm -ivh ~/rpmbuild/RPMS/x86_64/singularity-ce-$VERSION-1.el7.x86_64.rpm && \
       rm -rf ~/rpmbuild singularity-ce-$VERSION*.tar.gz

If you encounter a failed dependency error for golang but installed it
from source, build with this command:

.. code::

   rpmbuild -tb --nodeps singularity-ce-${VERSION}.tar.gz

Options to ``mconfig`` can be passed using the familiar syntax to
``rpmbuild``. For example, if you want to force the local state
directory to ``/mnt`` (instead of the default ``/var``) you can do the
following:

.. code::

   rpmbuild -tb --define='_localstatedir /mnt' singularity-ce-$VERSION.tar.gz

.. note::

   It is very important to set the local state directory to a directory
   that physically exists on nodes within a cluster when installing
   {Singularity} in an HPC environment with a shared file system.

Build an RPM from Git source
----------------------------

Alternatively, to build an RPM from a branch of the Git repository you
can clone the repository, directly ``make`` an rpm, and use it to
install Singularity:

.. code::

   $ ./mconfig && \
   make -C builddir rpm && \
   sudo rpm -ivh ~/rpmbuild/RPMS/x86_64/singularity-ce-{InstallationVersion}.el7.x86_64.rpm # or whatever version you built

To build an rpm with an alternative install prefix set ``RPMPREFIX`` on
the make step, for example:

.. code::

   $ make -C builddir rpm RPMPREFIX=/usr/local

For finer control of the rpmbuild process you may wish to use ``make
dist`` to create a tarball that you can then build into an rpm with
``rpmbuild -tb`` as above.

.. _remove-an-old-version:

Remove an old version
=====================

In a standard installation of {Singularity} 3.0.1 and beyond (when
building from source), the command ``sudo make install`` lists all the
files as they are installed. You must remove all of these files and
directories to completely remove {Singularity}.

.. code::

   $ sudo rm -rf \
       /usr/local/libexec/singularity \
       /usr/local/var/singularity \
       /usr/local/etc/singularity \
       /usr/local/bin/singularity \
       /usr/local/bin/run-singularity \
       /usr/local/etc/bash_completion.d/singularity

If you anticipate needing to remove {Singularity}, it might be easier to
install it in a custom directory using the ``--prefix`` option to
``mconfig``. In that case {Singularity} can be uninstalled simply by
deleting the parent directory. Or it may be useful to install
{Singularity} :ref:`using a package manager <install-rpm>` so that it
can be updated and/or uninstalled with ease in the future.

Testing & Checking the Build Configuration
==========================================

After installation you can perform a basic test of Singularity
functionality by executing a simple container from the Sylabs Cloud
library:

.. code::

   $ singularity exec library://alpine cat /etc/alpine-release
   3.10.0

See the `user guide
<https://www.sylabs.io/guides/{userversion}/user-guide/>`__ for more
information about how to use {Singularity}.

singularity buildcfg
--------------------

Running ``singularity buildcfg`` will show the build configuration of an
installed version of {Singularity}, and lists the paths used by
{Singularity}. Use ``singularity buildcfg`` to confirm paths are set
correctly for your installation, and troubleshoot any 'not-found' errors
at runtime.

.. code::

   $ singularity buildcfg
   PACKAGE_NAME=singularity
   PACKAGE_VERSION={InstallationVersion}
   BUILDDIR=/home/dtrudg/Sylabs/Git/singularity/builddir
   PREFIX=/usr/local
   EXECPREFIX=/usr/local
   BINDIR=/usr/local/bin
   SBINDIR=/usr/local/sbin
   LIBEXECDIR=/usr/local/libexec
   DATAROOTDIR=/usr/local/share
   DATADIR=/usr/local/share
   SYSCONFDIR=/usr/local/etc
   SHAREDSTATEDIR=/usr/local/com
   LOCALSTATEDIR=/usr/local/var
   RUNSTATEDIR=/usr/local/var/run
   INCLUDEDIR=/usr/local/include
   DOCDIR=/usr/local/share/doc/singularity
   INFODIR=/usr/local/share/info
   LIBDIR=/usr/local/lib
   LOCALEDIR=/usr/local/share/locale
   MANDIR=/usr/local/share/man
   SINGULARITY_CONFDIR=/usr/local/etc/singularity
   SESSIONDIR=/usr/local/var/singularity/mnt/session

Note that the ``LOCALSTATEDIR`` and ``SESSIONDIR`` should be on local,
non-shared storage.

The list of files installed by a successful ``setuid`` installation of
{Singularity} can be found in the :ref:`appendix, installed files
section <installed-files>`.

Test Suite
----------

The {Singularity} codebase includes a test suite that is run during
development using CI services.

If you would like to run the test suite locally you can run the test
targets from the ``builddir`` directory in the source tree:

-  ``make check`` runs source code linting and dependency checks

-  ``make unit-test`` runs basic unit tests

-  ``make integration-test`` runs integration tests

-  ``make e2e-test`` runs end-to-end tests, which exercise a large
   number of operations by calling the {Singularity} CLI with different
   execution profiles.

.. note::

   Running the full test suite requires a ``docker`` installation, and
   ``nc`` in order to test docker and instance/networking functionality.

   {Singularity} must be installed in order to run the full test suite,
   as it must run the CLI with setuid privilege for the ``starter-suid``
   binary.

.. warning::

   ``sudo`` privilege is required to run the full tests, and you should
   not run the tests on a production system. We recommend running the
   tests in an isolated development or build environment.

********************************
 Installation on Windows or Mac
********************************

Linux container runtimes like {Singularity} cannot run natively on
Windows or Mac because of basic incompatibilities with the host kernel.
(Contrary to a popular misconception, macOS does not run on a Linux
kernel. It runs on a kernel called Darwin originally forked from BSD.)

To run {Singularity} on a Windows or macOS computer, a Linux virtual machine
(VM) is required. There are various ways to configure a VM on both Windows and
macOS. On WIndows, we recommend the Windows Subsystem for Linux (WSL2), and
macOS, we recommend Lima.

Windows
=======

Recent builds of Windows 10, and all builds of Windows 11, include version 2 of
the Windows Subsystem for Linux. WSL2 provides a Linux virtual machine that is
tightly integrated with the Windows environment. The default Linux distribution
used by WSL2 is Ubuntu. It is straightforward to install {Singularity} inside
WSL2 Ubuntu, and use all of its features.

Follow the `WSL2 installation instructions
<https://docs.microsoft.com/en-us/windows/wsl/install>`__ to enable WSL2 with
the default Ubuntu 22.04 environment. On Windows 11 and the most recent builds
of Windows 10 this is as easy as opening an administrator command prompt or
Powershell window and entering:

.. code::

  wsl --install

Follow the prompts. A restart is required, and when you open the 'Ubuntu' app
for the first time you'll be asked to set a username and password for the Linux
environment.

You can install SingularityCE from source, or from the Ubuntu packages at the
GitHub releases page. To quickly install the 4.0.0 package use the following
commands inside the WSL2 Ubuntu window:

.. code::

  $ wget https://github.com/sylabs/singularity/releases/download/v4.0.0/singularity-ce_4.0.0-jammy_amd64.deb
  $ sudo apt install ./singularity-ce_4.0.0-jammy_amd64.deb

The ``singularity`` command will now be available in your WSL2 environment:

.. code::

  $ singularity exec library://ubuntu echo "Hello World!"
  INFO:    Downloading library image
  28.4MiB / 28.4MiB [=================================================================================] 100 % 5.6 MiB/s 0s
  Hello World!

GPU Support
-----------

WSL2 supports using an NVIDIA GPU from the Linux environment. To use a GPU from
{Singularity} in WSL2, you must first install ``libnvidia-container-tools``,
following the instructions in the `libnvidia-container documentation
<https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html>`__:

.. code::

  curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list \
  sudo apt-get update
  sudo apt-get install -y nvidia-container-toolkit

Once this process has been completed, GPU containers can be run under WSL2 using
the ``--nv`` and ``--nvccli`` flags together:

.. code::

  $ singularity pull docker://tensorflow/tensorflow:latest-gpu

  $  singularity run --nv --nvccli tensorflow_latest-gpu.sif
  INFO:    Setting 'NVIDIA_VISIBLE_DEVICES=all' to emulate legacy GPU binding.
  INFO:    Setting --writable-tmpfs (required by nvidia-container-cli)
  ________                               _______________
  ___  __/__________________________________  ____/__  /________      __
  __  /  _  _ \_  __ \_  ___/  __ \_  ___/_  /_   __  /_  __ \_ | /| / /
  _  /   /  __/  / / /(__  )/ /_/ /  /   _  __/   _  / / /_/ /_ |/ |/ /
  /_/    \___//_/ /_//____/ \____//_/    /_/      /_/  \____/____/|__/
  You are running this container as user with ID 1000 and group 1000,
  which should map to the ID and group for your user on the Docker host. Great!
  Singularity> python
  Python 3.8.10 (default, Nov 26 2021, 20:14:08)
  [GCC 9.3.0] on linux
  Type "help", "copyright", "credits" or "license" for more information.
  >>> import tensorflow as tf
  >>> tf.config.list_physical_devices('GPU')
  2022-03-25 11:42:25.672088: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:922] could not open file to read NUMA node: /sys/bus/pci/devices/0000:01:00.0/numa_node
  Your kernel may have been built without NUMA support.
  2022-03-25 11:42:25.713295: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:922] could not open file to read NUMA node: /sys/bus/pci/devices/0000:01:00.0/numa_node
  Your kernel may have been built without NUMA support.
  2022-03-25 11:42:25.713892: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:922] could not open file to read NUMA node: /sys/bus/pci/devices/0000:01:00.0/numa_node
  Your kernel may have been built without NUMA support.
  [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]

Note that the ``--nvccli`` flag is required to enable container setup using the
``nvidia-container-cli`` utility. {Singularity}'s simpler library binding
approach (``--nv`` only) is not sufficient for GPU support under WSL2.

Mac
===

To install {Singularity} on macOS, we recommend using the `lima <https://github.com/lima-vm/lima>`__ VM platform, available on `Homebrew <https://brew.sh/>`__.

If you don't already have Homebrew installed, you can install it as follows:

.. code::

   $ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

Follow the instructions at the end of the installation process. In particular,
make sure to add the relevant lines to your shell configuration:

.. code::

   $ (echo; echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"') >> $HOME/.profile
   $ eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

Once Homebrew is installed, install lima:

.. code::

   $ brew install lima

As part of the {Singularity} distribution (starting with version 4), we have
provided an example template for using {Singularity} with lima. The example
is available under the ``examples/lima`` directory in the {Singularity} source
bundle, and can also be downloaded `directly from the code repository
<https://raw.githubusercontent.com/sylabs/singularity/main/examples/lima/singularity-ce.yml>`_.

The template is named ``singularity-ce.yml``, and:

* Is based on AlmaLinux 9.
* Supports both Intel and Apple Silicon (ARM64) Macs.
* Installs the latest stable release of SingularityCE that has been published to
  the Fedora EPEL repositories.

Once you have obtained the template file, use it to start a lima VM:

.. code::

   $ limactl start ./singularity-ce.yml

You will be presented with an interactive menu:

.. code::

   $ limactl start ./singularity-ce.yml
   ? Creating an instance "singularity-ce"  [Use arrows to move, type to filter]
   > Proceed with the current configuration
     Open an editor to review or modify the current configuration
     Choose another template (docker, podman, archlinux, fedora, ...)
     Exit

Choose the ``Proceed with the current configuration`` option, and lima will
proceed to configure the VM according to the specifications in the template
file. This can take a couple of minutes.

Once lima is done with the configuration step, you can enter the VM
interactively and run {Singularity} commands:

.. code::

   $ limactl shell singularity-ce
   [myuser@lima-singularity-ce myuser]$ singularity run library://alpine
   INFO:    Downloading library image
   2.8MiB / 2.8MiB [==========================================================================================] 100 % 0.0 b/s 0s
   Singularity> cat /etc/os-release
   NAME="Alpine Linux"
   ID=alpine
   VERSION_ID=3.15.5
   PRETTY_NAME="Alpine Linux v3.15"
   HOME_URL="https://alpinelinux.org/"
   BUG_REPORT_URL="https://bugs.alpinelinux.org/"
   Singularity>

Your home directory is shared into the lima VM by default. However, since
macOS places home directories under ``/Users`` (rather than ``/home``),
{Singularity} will not mount your home directory in the container unless you
explicitly specify your macOS homedir, as shown here:

.. code::

   $ limactl shell singularity-ce
   [myuser@lima-singularity-ce myuser]$ singularity run -H /Users/myuser library://alpine
   INFO:    Using cached image
   Singularity> ls
   Applications Documents    Library      Music        Public
   Desktop      Downloads    Movies       Pictures

You can also run {Singularity} using lima directly from the macOS
command-line:

.. code::

   $ limactl shell singularity-ce singularity run library://alpine
   INFO:    Using cached image
   Singularity>

Or, with homedir mounting:

.. code::

   $ limactl shell singularity-ce singularity run -H /Users/myuser library://alpine
   INFO:    Using cached image
   Singularity>

To stop the lima VM:

.. code::

   $ limactl stop singularity-ce

To delete the lima VM:

.. code::

   $ limactl delete singularity-ce

{Singularity} Docker Image
==========================

It is also possible to run {Singularity} inside Docker, or another compatible
OCI container runtime. This may be convenient if you have Docker Desktop, or a
similar solution, already installed on your PC or Mac.

Docker containers for {Singularity} are maintained at
https://quay.io/repository/singularity/singularity. 

.. note::

  These containers are maintained by a third party. They are not part of the
  {Singularity} project, nor are they reviewed by Sylabs.

An example of a suitable ``compose.yaml`` file to start up {Singularity} in a
Docker container is given below. Note that privileged operation is needed to
successfully run {Singularity} nested inside of Docker. Change the version
number on the ``image:`` line to your preferred release.

.. code::

   services:
     singularity:
       image: quay.io/singularity/singularity:v3.11.4-slim
       stdin_open: true
       tty: true
       privileged: true
       volumes:
         - .:/root
       entrypoint: ["/bin/sh"]

Singularity in Docker can have various disadvantages, but basic
container operations will work. Currently, the intended use case is
continuous integration, meaning that you should be able to build a
Singularity container using this Docker Compose file. For more
information see `issue#5
<https://github.com/sylabs/singularity-admindocs/issues/5#issuecomment-852307931>`_
and the image's source `repo
<https://github.com/singularityhub/singularity-docker#use-cases>`_