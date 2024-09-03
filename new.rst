.. _whats_new:

###############################
What's New in {Singularity} 4.2
###############################

This section highlights important changes in {Singularity} 4.2 that are of note
to system administrators. See also the "What's New" section in the User Guide
for user-facing changes.

If you are upgrading from a 3.x version of {Singularity} we recommend also
reviewing the `"What's New" section for 4.0 <https://docs.sylabs.io/guides/{adminversion}/admin-guide/new.html>`__.

*************
Configuration
*************

- Additional directives in ``singularity.conf`` allow the use of namespaces to
  be restricted in native mode. See :ref:`sec:nsoptions`.
- The new ``--netns-path`` flag takes a path to a network namespace to join when
  starting a container. The root user may join any network namespace. An
  unprivileged user can only join a network namespace specified in the new
  allowed ``netns paths directive`` in ``singularity.conf``, if they are also
  listed in ``allow net users`` / ``allow net groups``. Not currently supported
  with ``--fakeroot``, or in ``--oci`` mode. See :ref:`sec:networkoptions`.

************
Requirements
************

- Go 1.22.5 or above is now required to build {Singularity} due to indirect
  dependencies.
- OCI-Mode now supports embedded writable overlays, which can be added to
  OCI-SIF files with ``singularity overlay create``. This functionality requires
  ``fuse2fs`` version 1.46.6 or above.
- {Singularity} 4.2 does not support EL 7 and SLES 12. The mainstream
  end-of-life dates for these distributions are 2024-06-30 and 2024-10-31
  respectively.
