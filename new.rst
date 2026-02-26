.. _whats_new:

###############################
What's New in {Singularity} 4.4
###############################

This section highlights important changes in {Singularity} 4.4 that are of note
to system administrators. See also the "What's New" section in the User Guide
for user-facing changes.

***********
OCI Support
***********

- docker-daemon OCI image sources now buffer via a temporary file instead of
  in-memory. Note that the file is created in ``$TMPDIR`` - the dependency involved
  cannot be instructed to use ``$SINGULARITY_TMPDIR`` at this time.

**************
Native Runtime
**************

- Improved support for hosts that have ``/etc/resolv.conf`` pointing to a
  symlink under ``/run``, such as those hosts that are running
  ``systemd-resolved``.  In this case, the symlink is copied into the container
  and the parent directory of the target of the symlink is bind-mounted from the
  host. The result is that even if the target of the symlink is replaced with a
  new file, the container sees the update in ``/etc/resolv.conf``.
- Support for starting fakeroot from setuid mode while in an NFS directory.

************************
Requirements & Packaging
************************

- Go 1.25.6 or above is now required to build SingularityCE.
