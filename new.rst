.. _whats_new:

###############################
What's New in {Singularity} 4.0
###############################

This section highlights important changes in {Singularity} 4.0 that are of note
to system administrators. See also the "What's New" section in the User Guide
for user-facing changes.

*****************
OCI-Compatibility
*****************

- {Singularity}'s `OCI-mode
  <https://docs.sylabs.io/guides/{userversion}/user-guide/oci_runtime.html>`__,
  which was experimental in 3.11, is now expanded and fully supported. It is
  enabled via ``--oci`` on the command line, or by setting ``oci mode = true``
  in ``singularity.conf``.
- OCI-mode runs containers unprivileged, using a low-level OCI runtime rather
  than {Singularity}'s own native runtime. {Singularity}'s setuid starter
  executable is not used on OCI-mode, even when setuid is enabled for the native
  runtime.
- OCI-mode uses `OCI-SIF images
  <https://docs.sylabs.io/guides/{userversion}/user-guide/oci_runtime.html#oci-sif-images>`__,
  a variant of the Singularity Image Format. These images cannot be run using
  earlier versions of {Singularity}.
- OCI-mode supports the `Container Device Interface
  <https://docs.sylabs.io/guides/{userversion}/user-guide/oci_runtime.html#container-device-interface-cdi>`__
  (CDI) standard for enabling access to GPUs and other devices within
  containers.

************
Requirements
************

- {Singularity} uses ``squashfuse_ll`` or ``squashfuse``, which is now built
  from a git submodule unless ``--without-squashfuse`` is specified as an
  argument to ``mconfig``. When built with ``--without-squashfuse``,
  ``squashfuse_ll`` or ``squashfuse`` should be located on ``PATH``. Version
  0.2.0 or later is required.
- OCI-mode requires ``sqfstar`` or ``tar2sqfs`` to be :ref:`installed on the
  system <sqfstar>` in order to create OCI-SIF images.
- OCI-mode requires ``fuse-overlayfs`` to be installed on the system (from a
  distribution package), to fully support unprivileged overlays.
- OCI-mode requires that either ``runc`` or ``crun`` is installed on the system
  (from a distribution package).
- OCI-mode requires that subuid/subgid mappings have been configured for users,
  in the same manner as documented for the :ref:`fakeroot <fakeroot>` feature.

*********
Packaging
*********

- RPM packages now use ``/var/lib/singularity`` (rather than
  ``/var/singularity``) to store local state files.
- Bash completions are now installed to the modern
  ``share/bash-completion/completions`` location, rather than under ``etc``.

***
CLI
***

- The :ref:`keyserver management commands <keyserver>` that were under remote
  have been moved to their own, dedicated keyserver command. Run ``singularity
  help keyserver`` for more information.

*******
Caching
*******

- Caching of OCI blobs is now architecture aware. If older versions of
  {Singularity} are not being used in parallel, users should run ``singularity
  cache clean`` to recover space used by obsolete cached blobs.

*******
Plugins
*******

- Support for image driver plugins, deprecated at 3.11, has been removed.
  Unprivileged kernel overlay is supported without a plugin. In
  ``singularity.conf``, the ``image driver`` directive has been removed, and
  ``enable overlay`` no longer supports the ``driver`` option.