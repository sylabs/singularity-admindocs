.. _whats_new:

###############################
What's New in {Singularity} 4.3
###############################

This section highlights important changes in {Singularity} 4.3 that are of note
to system administrators. See also the "What's New" section in the User Guide
for user-facing changes.

*************
Configuration
*************

 - Support for libsubid. Subid mappings will be retrieved from e.g. LDAP
   according to ``nssswitch.conf`` if Singularity is built with libsubid support
   (default). If built without libsubid support, Singularity will retrieve subid
   directly from ``/etc/subid`` and ``/etc/subgid`` regardless of system
   configuration. Note that ``singularity config fakeroot`` always modifies the
   ``/etc/subid`` and ``/etc/subgid`` files.

************************
Requirements & Packaging
************************

 - Go 1.23.4 or above is now required to build SingularityCE.
 - libsubid headers are now required to build SingularityCE, unless the
   ``--without-libsubid`` flag is passed to ``mconfig``.
 - EL RPM packages are built with libsubid support.
 - Ubuntu deb packages are built without libsubid support.
 - The RPM spec file no longer includes rules for SLES / openSUSE package
   builds, which have been untested / unsupported for some time.
 - Conmon sources are no longer bundled and built with SingularityCE. Install
   the ``conmon`` package from your distribution, or upstream binary, if you
   need to use the ``singularity oci`` commands. Note that conmon is not required
   for ``--oci`` mode.

****************
Removed Features
****************

  - Plugin ``fakerootcallback`` functionality for customizing fakeroot subid
    mappings has been removed. Use the libsubid integration to provide subid
    mappings from a custom source.
