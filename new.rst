.. _whats_new:

###############################
What's New in {Singularity} 4.5
###############################

SingularityCE 4.5.0 contains mostly internal code changes and defense-in-depth
hardening. The majority of the changes made since release 4.4.2 do not alter
behaviour, with the exception of specific points highlighted below.

Like many other open source projects, SingularityCE is increasingly the target
of LLM driven analysis. The changes in 4.5.0 aim to minimise false positives,
reduce maintainer burden, and provide defense-in-depth in areas where it is
appropriate.

If you are a security researcher working on SingularityCE, please see the new
AGENTS.md and SECURITY.md content, in the sylabs/singularity repository.

If you are a developer, intending to contribute to SingularityCE, please review
the LLM policy in CONTRIBUTING.md, in the sylabs/singularity repository.

*****************
Behaviour Changes
*****************

- In setuid mode, root-ownership checks on ``singularity.conf`` and the capabilities
  / ecl configuration now assert that these files are not writable except by the
  root owner. Management of these files by an administrator group is no longer
  possible. The files cannot be relocated by symlink.

- External helper binaries executed with elevated privileges must also be
  root-owned, regular executable files that are not writable by group or others.

- The majority of files that may be created by SingularityCE (e.g. remote
  configuration, pulled images), can no longer be created through a dangling
  symlink.

- If ``ecl.toml`` is missing, SIF execution is rejected rather than assuming an
  inactive ECL configuration. The default install ships an activated = false
  template, so standard installations are unaffected; sites with custom or
  partial installs must ensure ``ecl.toml`` is present and valid.
