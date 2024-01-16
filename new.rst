.. _whats_new:

###############################
What's New in {Singularity} 4.1
###############################

This section highlights important changes in {Singularity} 4.1 that are of note
to system administrators. See also the "What's New" section in the User Guide
for user-facing changes.

If you are upgrading from a 3.x version of {Singularity} we recommend also
reviewing the `"What's New" section for 4.0 <https://docs.sylabs.io/guides/{adminversion}/admin-guide/new.html>`__.

********
OCI-Mode
********

- {Singularity} will now build OCI-SIF images from Dockerfiles, if the
  ``--oci`` flag is used with the build command. Dockerfile builds are performed
  by ``buildkitd``. If a ``buildkitd`` instance is not running on the host,
  {Singularity} will use its own customized ephemeral
  ``singularity-buildkitd``. To disable the customized ephemeral buildkitd,
  remove ``libexec/singularity/bin/singularity-buildkitd``, or remove execute
  permissions from this binary.
- A new ``--keep-layers`` flag, for the ``pull`` and ``run/shell/exec/instance
  start`` commands, allows individual layers to be preserved when an OCI-SIF
  image is created from an OCI source. Multi layer OCI-SIF images can be run
  with {Singularity} 4.1 and later, and are not supported with earlier
  versions.

************
Requirements
************

- ``fuse2fs`` is required to mount legacy extfs container images via FUSE when
  kernel extfs mounts are disabled or unavailable. If ``fuse2fs`` is not
  available, {Singularity} will fall back to extracting containers to a
  temporary sandbox directory for execution.
- Kernel 4.18 or above is required for Dockerfile builds. EL 7 does not support
  Dockerfile builds, and continues to have limited support for other OCI-mode
  features. SLES 12 has no support for OCI-Mode, including Dockerfile builds.
- {Singularity} 4.1 is the final version that will support EL 7 and SLES 12. The
  mainstream end-of-life dates for these distributions are 2024-06-30 and
  2024-10-31 respectively.

*************
Configuration
*************

- The ``sif fuse`` option in ``singularity.conf``, which previously enabled
  experimental FUSE mounts of container images in some execution flows, is
  deprecated. When kernel mounts are disabled or unavailable, {Singularity}
  4.1 now uses FUSE by default, with fall-back to extraction to a temporary
  sandbox directory.
- A new ``tmp sandbox`` option in ``singularity.conf`` can be used to disable
  fall-back extraction of container images to temporary sandbox directories when
  kernel or FUSE mounts are unavailable. This option may be useful to avoid
  excessive filesystem load from implicit extraction of very large container
  images.
