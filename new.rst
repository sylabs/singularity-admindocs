.. _whats_new:

###############################
What's New in {Singularity} 4.6
###############################

SingularityCE now includes the following functionality that was previously
limited to SingularityPRO:

- A customizable log-plugin using templates. See additional information in the
  :ref:`log-plugin` section.

- An enterprise-plugin to query / manage Singularity Enterprise instances. See
  additional information in the :ref:`enterprise-plugin` section.

- Native hooks functionality, to allow configurable external tasks to be run on
  container startup. See additional information in the :ref:`native-hooks`
  section.

- A 'trusted bind paths' configuration option, which if set to 'yes' disables
  the remount of bind mounts configured in `singularity.conf`, so that they
  maintain the same mount flags as on the host. `MS_NOSUID` and `MS_NODEV` will
  not be forced. setuid execution is still blocked by `PR_SET_NO_NEW_PRIVS`. See
  the :ref:`singularity_configfiles` section.
