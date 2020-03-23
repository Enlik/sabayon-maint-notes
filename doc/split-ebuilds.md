# Introduction

This document presents various ways of how split ebuilds are handled. I
introduced several ways of doing that whilst trying to find ways that are
easier to do correctly, and in case of one of them, to avoid rebuilding core
parts for each or most variants (useful when I had a slow computer). I will
start from general ideas first.

## General ideas

In order to improve quality and not pile up potential problems across version
bumps, I tried to follow Gentoo exactly, e.g. by making sure that I did not
miss any dependency or configure switch (either added or removed) in a new
version, and try to check if all files installed by the new version are
installed by one of split ebuilds as well. The latter is done by merging a
version from Gentoo with all relevant USE flags first and collecting installed
file list.

There are two ways to do that, and each of them is basically performing a 3-way
merge. The first way is: apply changes done in Gentoo to Sabayon. Second is
doing the inverse. In the most cases I preferred the former because usually
delta of changes between versions is smaller than delta introduced by
modifications for Sabayon.

I often was doing it manually. However, I tried also interactive GUI tools and
git merge with rerere (where `git merge` was done on a local branch with
Sabayon ebuilds and the merged branch contained pristine Gentoo versions), and
some of that helps a bit, but none avoided conflicts in the most cases (also:
semantic conflicts can appear too). Coutermeasures I tried: reduce number of
code blocks to handle by moving away common code, or make modified ebuilds as
similar to Gentoo ones as possible.

I also stored on my system each used ebuild from Gentoo to have a reliable base
for a future version bump.

I usually added a new version and remove one so that `git show` produces a
readable diff of the commit.

**While introducing new versions it is a good idea to first fetch the sources
(`emerge -f`) and then introduce ebuilds, so that checksums match whatever was
used in Gentoo.**

## Variants

### Plain split ebuilds

Several ebuilds modified to split the package with nothing fancy. Example:
`dev-vcs/subversion` [1].

### Ebuilds that use sab-split.eclass

`dev-vcs/git` uses this way. It's about adding `sab-split` to IUSE and using
`dev-util/dirstr` [2] to "clean up." This idea is inspired by package managers
that have built in support for split packages where it works roughly like this:
package is built only once and files are distributed across split packages.

This way is the easiest when it comes to modification of ebuilds: they are
almost the same, with the main difference being different flags passed to
`dirstr.py`. The idea is described in more detail in `sabayon-distro.git`,
`dev-vcs/README.git` so here I'll only mention what it takes to introduce a new
version. The downside of ebuilds based solution is that the whole code (or at
least common parts) is built multiple times.

1. Prepare new ebuilds. Again, changes between them are minor so there is no
   need to solve conflicts more than once.
2. Prepare a "spec" file like `dev-vcs/git/files/git-2.24.1-spec`. It describes
   what file goes to which package. This might be the more time consuming part.
   `dev-vcs/mkspec-git.sh` *can* be used for that (see comments inside the file
   for usage).

This solution requires that in package.use `sab-split` and common flags are
enabled for every package, and naturally specific flags are enabled for split
packages.

### Ebuilds that avoid building common parts for each split ebuild

`net-p2p/transmission` installs .a and uses it. It is a trick for reducing
build times and not making it easier to maintain, and could be undone if causes
problems given that this package does not take particularly long to build on
present computers.

### Ebuilds that use dedicated eclasses with common parts

By moving common parts to one place, number of code blocks to merge is reduced.
Again it's `net-p2p/transmission`. Example follows how building for different
variants is done.

```sh
if _transmission_is base; then
	export ac_cv_header_xfs_xfs_h=$(usex xfs)
	econfargs+=(
		--disable-cli
		--disable-daemon
		--without-gtk
		$(use_enable lightweight)
	)   
elif _transmission_is cli; then
	econfargs+=(
		--enable-cli
		--disable-daemon
		--without-gtk
	)
# (...)
```

### Ebuilds that use "generator" scripts

It applies to `l10n` ebuilds, for example `www-client/firefox-l10n-pl`.
`www-client/ebuilds-new-version-Firefox-l10n.sh` is the script to introduce new
ebuilds. It uses Manifest file from Gentoo to make sure checksums match
whatever was used in Gentoo, and to avoid needless fetching of the various
locale files that would otherwise be required to calculate the checksums. (Side
note, `app-office/do-libreoffice-bump.sh` does not do this by default, but has
a commented out code to make it work this way.)

What I do whilst bumping Firefox localisation ebuilds follows.

1. Make sure the Gentoo tree is synchronised.
2. Check whether `MOZ_ESR` is set or not in a new version.
3. Check for changes of `eclass/mozlinguas-v2.eclass` in Gentoo git repository
   to see if there are changes that need to be applied. Good thing is that it
   happens very rarely.
4. Execute `www-client/ebuilds-new-version-Firefox-l10n.sh` (it is an
   interactive script). It should not fetch anything. If it does, probably
   `SRC_URI` or `MOZ_ESR` is wrong.

Note: `www-client/ebuilds-new-version-Firefox-l10n.sh` uses pquery from
`sys-apps/pkgcore`.

See [entropy-server-packages.md](entropy-server-packages.md) for how to create
binary packages from these.

### Ebuilds generated from a template file

`net-dns/avahi` ebuilds are supposed to be generated from
`net-dns/avahi.ebuild.jinja2` using `net-dns/avahi-generate.sh`.

1. Update `net-dns/avahi.ebuild.jinja2`.
2. Generate the ebuilds.


[1] In the names used as examples I usually mention only "base" packages.

[2] https://github.com/Enlik/dirstr
