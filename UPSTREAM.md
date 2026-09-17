# Upstream

Readymacs tracks an upstream project, **Readymax**, maintained by
Gornskew Enterprises (https://github.com/gornskew/readymax), from
which it is periodically merged.  Readymax was formerly published as
**Skewed Emacs**; the old `gornskew/skewed-emacs` address redirects to
it, and its README points here. This file is the one place in this
repository that records the relationship; the product documentation
does not depend on it.

**Shared with upstream, and kept compatible:** the Emacs
configuration and its `skewed-*` elisp namespace (`skewed-install`,
`skewed-icons-*`, `skewed-dashboard-*`), the `lisply-*` backend
namespace and endpoints, the `emacs-user` account, the
`-pre-skewed-emacs` backup suffix, the `~/.config/skewed-emacs/`
configuration path, and the image capability labels. These are
compatibility contracts; renaming them is a versioned behavior
decision made with upstream, never a documentation edit.

**Deliberately different here:** the product name and image
coordinates (`genworks/readymacs`), the attribution, the dashboard
attribution and banner, and the documentation voice.

**Issues and fixes:** file issues against this repository. A fix that
applies to the shared code is carried upstream by the maintainers;
upstream changes are merged here when chosen.

Copyright © 2026 Genworks International; portions copyright © 2026
Gornskew Enterprises. GNU Affero General Public License, version 3
or later.
