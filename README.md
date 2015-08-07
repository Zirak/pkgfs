## NAME

pkg - package handling pseudo-filesystem

## DESCRIPTION
The pkg filesystem is a pseudo-filesystem which provides an interface to package management.
Its goal is to provide a unified abstraction over package management which is easy to use, for humans and programs alike.
Packages can be listed, installed, deleted, upgraded, introspected, and so forth using just the filesystem.

## MOUNT OPTIONS
¯\\(°_o)/¯

## Files and directories

Files and directories found under the mount path, assumed to be `/pkg/`.

### Root directory
* `index/`
All packages available on the remote repository.
* `installed/`
Directory containing all installed packages.
* `oudated/`
Contains packages which can be upgraded. Currently unspecified.
* `sync`
Executable which synchronises the local package repository with the remote one (think `apt-get update` or `pacman -Sy`)
* `upgrade`
Executable which does a full system upgrade (think `apt-get upgrade` or `pacman -Su`)

### Package files and directories
Files and directories found under both `/pkg/index/[name]` and `/pkg/installed/[name]`.

* `name`
* `description`
* `size`
Note that this is the installed size - how much disk space (in bytes) the package occupies/will occupy.
* `dependencies/`
A directory of relative symlinks to the package's dependencies. For example, if `foobin` depends on `libfoo`, this directory will contain a single symlink pointing to `../../libfoo`, thus pointing to `/pkg/(index|installed)/libfoo`.

### Index package files and directories
Files and directories found only under `/pkg/index/[name]`

* `install`
Executable which installs the package.
Flags:
    * `-y`, `--yes` - Automatic "yes" to prompts, unless asked a potentially destructive question (like conflict resolution).

### Installed package files and directories
Files and directories found only under `/pkg/installed/[name]`

* `uninstall`
Executable which uninstalls the package.

* `files/`
A directory of the package's files which it created in the directory tree. Each leaf is a symlink to the actual file. For example:

```
/pkg/installed/zsh/files:
etc
usr

/pkg/installed/zsh/files/etc:
zsh

/pkg/installed/zsh/files/etc/zsh:
zprofile -> /etc/zsh/zprofile

/pkg/installed/zsh/files/usr:
bin
lib
share

/pkg/installed/zsh/files/usr/bin:
zsh -> /usr/bin/zsh
zsh-5.0.8 -> /usr/bin/zsh-5.0.8

...
```

## TODOs

### Root directory
1. dpkg/apt separate between a regular `upgrade` and a `distro-upgrade`; which one should `/pkg/upgrade` run? Is there an equivalent separation in other managers?
2. `outdated/` directory: Spec it. Some suggestions below. To decide which is the better way, some research is needed to find out whether upgrading a package is the same as installing the new version, cross package manager. Anyway, suggestions:
	1. Symlinks to `/pkg/index`
	2. Directory with two symlinks, one to the installed package and the other to the index, with an executable to upgrade.
3. `search/` directory: Spec it, and figure out what should it search (the index? installed? both? how can the user tell which should be searched?)

### Installed directory
1. When uninstalling a package, should we also remove the to-be orphaned packages it depended on? That is, if `libfoo` was installed purely as a dependency of `foobin` and we uninstall `foobin`, should `libfoo` also be uninstalled? Is this opt-out or opt-in? How does the user decide?
	1. As a sub-issue, how do we deal with orphaned packages? Too much of an edge case to be handled by the user explicitly, or maybe a different directory under `/pkg`?
2. Find out if `files/` can be implemented across all package managers.

### Assorted pains in the side
1. Do we need to handle cases where the same package is named differently in different repos? For instance, it's `foobin` in Debian Stable, but `foo-bin` in the Centos6 repos.
	1. On the one hand it'd be pretty cool, on the other it's a never-ending rabbit hole where we can't cover 100% of cases.

## Implementations
* [dpkg-fs](https://github.com/ralt/dpkg-fs) for Debian (apt/dpkg).
* [pacman-fs](https://github.com/Zirak/pacman-fs) for Arch (pacman/alpm).
* We'd love to see more!

Based off of an older [google doc file](https://docs.google.com/document/d/1Fi1ebe_rAq4v-JNW8i2IbT4iUHIPro-wbVT86tBhW14) before it grew too unwieldy.
