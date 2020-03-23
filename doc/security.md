# Introduction

Handling security bumps is described. Normally GLSA can be used but my
observation is that it is for "stable" packages, so a different solution had to
be found.

# Security bumps handling

One time:

1. Create a search filter in bugs.gentoo.org: resolution: everything, assignee:
   security`@`gentoo.org. Make sure results are sorted by changed date.

When processing:

1. Read saved date of when bugs were processed previously.
2. Look for bugs since that time, and check one by one if a security version
   bump is available (or available with a stable version).
3. When done, save the "changed" date of the last bug that was processed to
   have a starting point for the next time.

Undoubtly doing this takes some time but not as much as it may seem; in most
cases, `eix -e package` for a package from bug description is enough, so
opening the bug is even not necessary (although it is a good idea to do that
and check "Package list" in some cases).
