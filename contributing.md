---
layout: page
title: Contributing
---

## Kernel

For kernel contributions to the ieee802514 and 6lowpan subsystems we follow the
usual kernel development approach. In a nutshell you would write a patch, send
it to the development mailing list via git send-email, wait for review and
feedback, re-spin if necesassary and wait for the patch to be applied to one of
our git trees.

We run two different git trees. One for bugfixes which should go directly into
the current kernel in development (we also consider stable backports if marked
accordingly), [wpan](https://git.kernel.org/pub/scm/linux/kernel/git/sschmidt/wpan.git).

The second git tree is for new features and bigger developments that are
scheduled for the next merge window, [wpan-next](https://git.kernel.org/pub/scm/linux/kernel/git/sschmidt/wpan-next.git).

All patches should be send to [linux-wpan@vger.kernel.org](mailto:linux-wpan@vger.kernel.org).

## Wpan-tools

For wpan-tools checkout the [wpan-tools](https://github.com/linux-wpan/wpan-tools)
repository. Patches should also be send to [linux-wpan@vger.kernel.org](mailto:linux-wpan@vger.kernel.org).
Adding wpan-tools into the patch prefix helps us to filter them out from kernel
patches.

If in doubt, do not worry to much about all the details, send your contribution
and we will figure out together where to put it. :-)
