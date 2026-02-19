# archlive

Build script for ArchLinux-based live images.

## What is archlive

This is a fairly simple wrapper for `mkarchroot`, `mkarchiso`, `mkchrootpkg`,
and `arch-nspawn` that attempts to simplify the processes of building and
maintaining an ArchLinux package repository and building Arch-based live
images.

The first iteration of this script was called `caelestia-build` and was
tailored specifically for building the `caelestia-live` image, but it had
many issues related to having various things hard-coded that made it difficult
to work on images outside of the build server. When I finally decided to fix
those issues, I opted to break away from Caelestia entirely and make the script
a generic Arch build system.

## Installation

No installation is required (though a PKGBUILD is planned). Simply clone the
repository and run the `archlive` script. archlive uses a config file stored at
`~/.config/archliverc` to determine necessary paths and a few image-specific
options. This file is created on first run if it can't be found. With no
additional adjustment, running a build with the default configs results in
a clone of the official Arch install image... so you probably want to take
the time to configure it to your needs.

## Configuration

Configuring `archlive` is pretty straightforward, and the config file is
well-documented. The stock config file looks like this:

```bash
#!/bin/bash
#
# Config file for archlive
#
# Copyright (C) 2026 Caelestia Packager <packager@caelestiashell.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the \"Software\"), to
# deal in the Software without restriction, including without limitation the
# rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
# sell copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED \"AS IS\", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
# IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
# CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT
# OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR
# THE USE OR OTHER DEALINGS IN THE SOFTWARE.


# Path to the package repo. This is a pacman-compatible repository.
REPO_OUT=./repo/archlive/x86_64

# Name for the package repo. If this is a local build, this MUST be "custom".
REPO_NAME=custom

# Path to the build profile. If no profile is set, will use the
# default Arch install image profile.
BUILD_PROFILE=

# Name for the ISO image.
IMAGE_NAME=archlive

# Path for built ISO files and related signature/verification files.
ISO_OUT=./repo/iso

# Path where PKGBUILD files are stored. Think of this as a local
# version of the AUR, specifically for packages we care about.
PKGBUILD_DIR=./build/packages

# Path to the archroot used for building pretty much everything.
CHROOT_DIR=./build/chroot

# Path to a custom pacman.conf file for the archroot. Leave empty
# to use the ArchLinux default.
PACMAN_CONF=

# The gpg key to use for signing packages and images. Leave empty
# to skip signing.
SIGNING_KEY=

# Path to the directory used for temporary storage during the build
# process. Ideally, this should be somewhere on tmpfs as the build
# scripts currently only clean up when a new build is initiated.
TMP_DIR=/tmp
```

## Known Issues

While `archlive` is in active use for the `caelestia-live` image, it is far
from flawless. There are a few known bugs that, while not critical, can be
an annoyance. The currently known issues are outlined below.

* The original script did a check for signature files before copying a new
  or updated package to the public repository. Unfortunately, the way
  `archlive` processes packages no longer allows for easy parsing of the
  specific package name. As a result, a forced removal of pre-existing
  package content before pushing an updated version no longer works as it
  used to (e.g.: if you have `gnome-icon-theme` and `gnome-icon-theme-symbolic`
  in your repo, when it tries to clean up `gnome-icon-theme` it removes both
  packages as they both fit the pattern). Due to this limitation, the automatic
  removal of pre-existing package files before an update is temporarily
  disabled. This results in a confirmation prompt when a package is rebuilt
  without a version bump and the signature file is pushed.
* Occasionally, the script fails to properly sync the chroot when building a
  package, resulting in a build failure. At the moment, I'm not sure what's
  causing this, but it's simple enough to fix by simply re-running the build.
* If a build fails, the script currently doesn't clean up after itself well.
  This doesn't impact much of anything other than a rebuild has to be run
  as an update instead of a new build.
