# Introduction

Selected topics and scripts related to handling Entropy packages (not ebuilds)
are described.

# Utility scripts in build.git

`bin/bump_misc` is a script that makes it easier to install split packages.
Example usage: `bin/bump_misc sub` or `bin/bump_misc subversion`.

`bin/build-inject-apache.sh` for easier installation of `www-servers/apache`
(one variant of Apache is installed normally and another is injected).

# Stable vs. nonstable packages

It is a matter of the chosen approach (or "subjective" for short). My ideas
were to:

- keep libraries stable unless there is a reason to pick a different version,
- if something was at a stable version, consider bumping to next stable instead
  of newest unstable.

# Cleaning after syncing the Gentoo tree

After the tree gets updated, there are often new masked or removed packages. I
often handled them right away. Useful commands are `equo q revdeps package`
(to check reverse dependencies) and `eix -e package` to see if they are really
gone. I often executed these commands in a shell loop on a list printed by eix
after it finished synchronising.

# Quick look for packages that may depend on one

`grep something /var/db/pkg/*/*/*DEPEND`.

# Useful emerge switches

`-j` and `-A`.

# Creating Firefox l10n packages

After updating the overlay, execute `www-client/install-Firefox-l10n.sh` for
example like this: `www-client/install-Firefox-l10n.sh --eit-inject -- -j 100`.
