# Introduction

In this document it is shown how to check for new versions and modifications of
ebuilds from the Gentoo tree. It can be done periodically for the purpose of
synchronising ebuilds in Sabayon with Gentoo changes.

# Checking for new ebuilds

1. Have a git clone of the Gentoo tree. I use a "bare" clone.
2. Run a script that processes log from the last fetched commit up to the
   newest one.
3. Pick from the log (commit range) commits with changes in ebuilds of
   interest.
4. Based on this, check if some ebuilds in Sabayon need to be updated.

# The code snippet

Includes list of ebuilds that are up to date for my purpose at the moment of
writing.

```sh
upd_gn() {
	cd "$1" || return

	local prev_ref
	prev_ref=$(git log -1 --format=%H) || return

	git fetch

	local w="
		net-firewall/ufw
		net-firewall/ufw-frontends
		dev-vcs/git
		dev-vcs/subversion
		media-sound/jack-audio-connection-kit
		net-irc/quassel
		net-p2p/transmission
		www-client/firefox
		www-servers/apache
		app-crypt/pinentry
		app-text/poppler
		net-analyzer/nmap
		x11-misc/lightdm
		www-client/conkeror
		media-sound/clementine
		net-proxy/privoxy
		app-backup/tsm
		lxde-base/lxpanel
		media-gfx/darktable
		x11-terms/pangoterm
		dev-perl/Bio-SamTools
	"

	local s=""
	local a
	for a in $w; do
		s="$s$a "
	done

	s=${s% }

	git --no-pager log --name-status "$prev_ref..FETCH_HEAD" -- $s
}
```
