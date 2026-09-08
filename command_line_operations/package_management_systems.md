# Package Management Systems on Linux

## 1. What Is Package Management?

Most Linux software is distributed as **packages**.

A package is a bundle containing the files and metadata needed to
install and manage a piece of software. Depending on the package, it may
contain:

-   Executable programs
-   Libraries
-   Configuration files
-   Documentation
-   Man pages
-   Metadata such as the package version and architecture
-   Dependency information
-   Installation or removal scripts

For example, installing a Python application may require several Python
libraries. A package manager can identify those dependencies and install
them automatically.

### Why package management matters

Without a package manager, installing software manually would often
require you to:

1.  Download the software.
2.  Find the correct version.
3.  Find all required libraries.
4.  Install those libraries in the correct order.
5.  Keep track of where every file was installed.
6.  Upgrade the software later.
7.  Remove it cleanly when it is no longer needed.

A package manager automates much of this work.

------------------------------------------------------------------------

# 2. Package Management Families

There are two major traditional package formats used across Linux
distributions:

  -----------------------------------------------------------------------
  Package family    Package format    Common low-level  Common
                                      tool              distributions
  ----------------- ----------------- ----------------- -----------------
  Debian family     `.deb`            `dpkg`            Debian, Ubuntu,
                                                        Linux Mint

  RPM family        `.rpm`            `rpm`             Fedora, RHEL,
                                                        CentOS Stream,
                                                        openSUSE
  -----------------------------------------------------------------------

These package formats and their low-level tools are different and are
**not directly interchangeable**.

For example:

``` text
Debian / Ubuntu
      ↓
   .deb
      ↓
   dpkg
      ↑
     apt
```

``` text
Fedora / RHEL
      ↓
   .rpm
      ↓
    rpm
      ↑
    dnf
```

SUSE/openSUSE also uses RPM packages:

``` text
openSUSE
    ↓
  .rpm
    ↓
   rpm
    ↑
 zypper
```

Other Linux distributions use other package systems, such as Alpine
Linux's `apk` or Arch Linux's `pacman`.

------------------------------------------------------------------------

# 3. Two Levels of Package Management

A useful way to understand Linux package management is to divide it into
**low-level** and **high-level** tools.

## Low-level tools

Low-level tools work directly with individual package files.

Examples:

-   `dpkg` --- Debian packages
-   `rpm` --- RPM packages

They handle operations such as:

-   Installing a package file
-   Removing a package
-   Querying package information
-   Listing files belonging to an installed package

However, they generally do not provide the complete
dependency-resolution experience that high-level package managers
provide.

------------------------------------------------------------------------

## High-level tools

High-level package managers work with repositories and dependency
relationships.

Examples:

-   `apt` --- Debian/Ubuntu
-   `dnf` --- Fedora/RHEL family
-   `zypper` --- SUSE/openSUSE

They can:

-   Search repositories
-   Download packages
-   Resolve dependencies
-   Install multiple packages
-   Upgrade packages
-   Remove packages
-   Manage repositories
-   Update package metadata

The high-level tool normally uses the low-level package system
underneath.

Conceptually:

``` text
             High-level package manager
           apt / dnf / zypper
                    │
       ┌────────────┴────────────┐
       │                         │
 dependency resolution       repositories
       │
       ↓
Low-level package system
   dpkg / rpm
       │
       ↓
Install / remove / manage files
```

### Why dependency resolution is important

Suppose package `A` requires packages `B` and `C`:

``` text
Application A
   ├── requires B
   └── requires C
```

A high-level package manager can normally determine this automatically:

``` text
Install A
   ↓
Find B and C
   ↓
Download them
   ↓
Install dependencies
   ↓
Install A
```

A single package can sometimes bring in dozens or even hundreds of
dependencies.

------------------------------------------------------------------------

# 4. Common High-Level Package Managers

## `apt`

`apt` is the standard high-level package manager on Debian-based systems
such as:

-   Debian
-   Ubuntu
-   Linux Mint

It works with `.deb` packages and uses `dpkg` underneath.

You may also encounter related tools such as:

-   `apt-get`
-   `apt-cache`

For normal interactive use, `apt` is generally the preferred
command-line interface.

Examples:

``` bash
sudo apt update
sudo apt install curl
sudo apt remove curl
sudo apt search nginx
```

------------------------------------------------------------------------

## `dnf`

`dnf` is the modern high-level package manager used by distributions in
the Red Hat family, including Fedora and current RHEL-based systems.

Examples:

``` bash
sudo dnf install curl
sudo dnf remove curl
sudo dnf upgrade
sudo dnf search nginx
```

------------------------------------------------------------------------

## `zypper`

`zypper` is the high-level package manager used by SUSE/openSUSE.

Examples:

``` bash
sudo zypper install curl
sudo zypper remove curl
sudo zypper update
sudo zypper search nginx
```

------------------------------------------------------------------------

# 5. Low-Level Package Management with `dpkg`

`dpkg` directly manages Debian `.deb` packages.

It is useful for inspecting and manipulating individual packages,
especially local `.deb` files.

## Install a `.deb` file

``` bash
sudo dpkg --install foo.deb
```

Short form:

``` bash
sudo dpkg -i foo.deb
```

Be aware that `dpkg` does not provide the same automatic
repository-based dependency resolution as `apt`.

If dependencies are missing, you may need to use `apt` to resolve them.

------------------------------------------------------------------------

## Remove a package

``` bash
sudo dpkg --remove foo
```

This removes the package itself while generally preserving its
configuration files.

------------------------------------------------------------------------

## List installed packages

``` bash
dpkg --list
```

Because the output can be long:

``` bash
dpkg --list | less
```

You can also search the output:

``` bash
dpkg --list | grep bzip2
```

------------------------------------------------------------------------

## Show package information

``` bash
dpkg --status bzip2
```

or:

``` bash
dpkg -s bzip2
```

This can show information such as:

-   Version
-   Architecture
-   Description
-   Installation status
-   Dependencies

------------------------------------------------------------------------

## List files installed by a package

``` bash
dpkg --listfiles bzip2
```

or:

``` bash
dpkg -L bzip2
```

This is useful when you want to know exactly which files a package
installed.

------------------------------------------------------------------------

## Find which package owns a file

``` bash
dpkg --search /usr/bin/something
```

or:

``` bash
dpkg -S /usr/bin/something
```

This asks:

> Which installed Debian package owns this file?

------------------------------------------------------------------------

# 6. Low-Level Package Management with `rpm`

`rpm` directly manages RPM package files.

## Query all installed packages

``` bash
rpm -qa
```

Here:

-   `-q` means query
-   `-a` means all

Because the result can be long:

``` bash
rpm -qa | less
```

You can search it with `grep`:

``` bash
rpm -qa | grep bzip2
```

------------------------------------------------------------------------

## Show information about a package

``` bash
rpm -qi bzip2
```

This displays information such as the package version, architecture,
description, and other metadata.

------------------------------------------------------------------------

## List files installed by a package

``` bash
rpm -ql bzip2
```

The `-l` option means list the files belonging to the package.

You can combine this with other commands:

``` bash
ls -l $(rpm -ql bzip2)
```

Here `$()` is **command substitution**.

The shell first runs:

``` bash
rpm -ql bzip2
```

and substitutes its output into the surrounding command.

Conceptually:

``` text
rpm -ql bzip2
      ↓
list of paths
      ↓
ls -l path1 path2 path3 ...
```

------------------------------------------------------------------------

## Remove a package

``` bash
sudo rpm -e bzip2
```

The `-e` option means erase.

However, removing a package with `rpm` can fail if other installed
packages depend on it.

You can test the operation without actually removing anything:

``` bash
sudo rpm -e --test bzip2
```

If dependencies would be broken, `rpm` reports them.

------------------------------------------------------------------------

## Find packages that require another package

``` bash
rpm -q --whatrequires bzip2
```

This can help answer:

> Which installed packages depend on `bzip2`?

This is useful before removing an important package.

------------------------------------------------------------------------

# 7. Why High-Level Tools Are Usually Preferred

Imagine package `A` depends on:

``` text
A → B → C → D
```

If you manually use a low-level tool, you may have to determine and
install the dependencies yourself.

A high-level package manager can normally calculate the dependency graph
and handle the required packages automatically.

Therefore:

``` text
Low-level
dpkg / rpm
     ↓
Direct package operations

High-level
apt / dnf / zypper
     ↓
Repositories
     ↓
Dependency resolution
     ↓
Low-level package operations
```

### Practical rule

For everyday software installation and upgrades:

> **Prefer the high-level package manager.**

Use `dpkg` or `rpm` when you specifically need to work with an
individual local package or inspect package details.

------------------------------------------------------------------------

# 8. High-Level Package Management with `apt`

`apt` is the main command-line package manager you will commonly use on
Debian and Ubuntu.

Most package-management commands require administrator privileges, so
you will commonly use `sudo`.

------------------------------------------------------------------------

## Refresh package information

Before upgrading packages on Debian/Ubuntu, run:

``` bash
sudo apt update
```

This **does not upgrade your installed software**.

It downloads the latest package information from configured repositories
and updates the local package lists.

Think of it as:

``` text
Repository
    ↓
apt update
    ↓
Updated local package information
```

Then:

``` bash
sudo apt upgrade
```

uses that information to upgrade installed packages.

### Important distinction

``` bash
apt update
```

means:

> Refresh information about what packages and versions are available.

``` bash
apt upgrade
```

means:

> Upgrade installed packages to newer available versions.

This distinction is one of the most important things to remember about
Debian/Ubuntu package management.

------------------------------------------------------------------------

# 9. Searching for Packages with `apt`

You can search package names and descriptions:

``` bash
apt search wget2
```

For example:

``` bash
apt search nginx
```

This searches the package information available to APT.

------------------------------------------------------------------------

# 10. Installing Packages with `apt`

To install a package:

``` bash
sudo apt install wget2
```

APT automatically determines required dependencies and proposes them for
installation.

You will normally see a summary similar to:

``` text
The following additional packages will be installed:
    dependency1
    dependency2
    dependency3

Need to get ... MB of archives.
After this operation ... MB of additional disk space will be used.

Do you want to continue? [Y/n]
```

Pressing `Enter` usually accepts the displayed default choice.

### Important

On Debian/Ubuntu:

``` bash
sudo apt install package-name
```

does two useful things:

-   Installs the package if it is not installed.
-   If it is already installed, it can upgrade it to the newest version
    available from the configured repositories.

------------------------------------------------------------------------

# 11. Removing Packages with `apt`

To remove a package:

``` bash
sudo apt remove wget2
```

APT checks dependencies and determines what can safely be removed.

If other packages were automatically installed as dependencies and are
no longer required, you can clean them up with:

``` bash
sudo apt autoremove
```

### `remove` vs `purge`

`apt remove` generally removes the package but can leave its
configuration files.

If you want to remove the package and its system-wide configuration
files:

``` bash
sudo apt purge wget2
```

You can then optionally clean up automatically installed dependencies:

``` bash
sudo apt autoremove
```

Be careful with `purge`, especially on software whose configuration you
may want to preserve.

------------------------------------------------------------------------

# 12. Listing Installed Packages with `apt`

``` bash
apt list --installed
```

Because this can produce a lot of output:

``` bash
apt list --installed | less
```

You can search the result:

``` bash
apt list --installed | grep nginx
```

------------------------------------------------------------------------

# 13. High-Level Package Management with `dnf`

The same general concepts apply to `dnf`.

## Install

``` bash
sudo dnf install foo
```

## Remove

``` bash
sudo dnf remove foo
```

## Upgrade a specific package

``` bash
sudo dnf upgrade foo
```

## Upgrade the system

``` bash
sudo dnf upgrade
```

## Search

``` bash
dnf search foo
```

## List installed packages

``` bash
dnf list installed
```

## List available packages

``` bash
dnf list available
```

`dnf` handles repository access and dependency resolution for you.

------------------------------------------------------------------------

# 14. High-Level Package Management with `zypper`

On openSUSE/SUSE systems, the equivalent high-level tool is `zypper`.

## Install

``` bash
sudo zypper install foo
```

## Remove

``` bash
sudo zypper remove foo
```

## Update packages

``` bash
sudo zypper update
```

## Search

``` bash
zypper search foo
```

## List installed packages

``` bash
zypper search --installed-only
```

The exact behavior and available commands can vary by distribution and
version, so use:

``` bash
man zypper
```

when needed.

------------------------------------------------------------------------

# 15. Command Comparison

The following table gives a practical overview.

  ----------------------------------------------------------------------------------------------------
  Operation         Debian/Ubuntu (`apt`)    Fedora/RHEL (`dnf`)    openSUSE (`zypper`)
  ----------------- ------------------------ ---------------------- ----------------------------------
  Refresh           `apt update`             `dnf check-update` /   `zypper refresh`
  repository                                 metadata handled       
  metadata                                   automatically as       
                                             needed                 

  Install           `apt install foo`        `dnf install foo`      `zypper install foo`

  Remove            `apt remove foo`         `dnf remove foo`       `zypper remove foo`

  Upgrade packages  `apt upgrade`            `dnf upgrade`          `zypper update`

  Search            `apt search foo`         `dnf search foo`       `zypper search foo`

  List installed    `apt list --installed`   `dnf list installed`   `zypper search --installed-only`
  ----------------------------------------------------------------------------------------------------

Run administrative operations with `sudo` when required.

------------------------------------------------------------------------

# 16. Low-Level vs High-Level: Quick Comparison

  -------------------------------------------------------------------------------------------------------------
  Task           Debian low-level     Debian high-level          RPM low-level       RPM high-level
  -------------- -------------------- -------------------------- ------------------- --------------------------
  Tool           `dpkg`               `apt`                      `rpm`               `dnf` / `zypper`

  Install local  `dpkg -i file.deb`   `apt install ./file.deb`   `rpm -i file.rpm`   `dnf install ./file.rpm`
  package                                                                            

  Remove         `dpkg -r foo`        `apt remove foo`           `rpm -e foo`        `dnf remove foo`

  Query/list     Yes                  Yes                        Yes                 Yes
  packages                                                                           

  Repository     No                   Yes                        No                  Yes
  support                                                                            

  Dependency     Limited              Yes                        Limited             Yes
  resolution                                                                         
  -------------------------------------------------------------------------------------------------------------

A particularly useful point is that modern high-level managers can often
install a local package **and resolve its dependencies**:

``` bash
sudo apt install ./foo.deb
```

or:

``` bash
sudo dnf install ./foo.rpm
```

This is often preferable to directly invoking `dpkg` or `rpm` for local
packages.

------------------------------------------------------------------------

# 17. Package Repositories

A **package repository** is a server or collection of servers that
provides packages and metadata for a Linux distribution.

Instead of manually downloading software from random websites, the
package manager can use trusted repositories.

Conceptually:

``` text
Your computer
     │
     │ apt / dnf / zypper
     ↓
Package repository
     │
     ├── package files
     ├── versions
     ├── dependencies
     └── metadata
```

Repositories make it easier to:

-   Find software
-   Install software
-   Resolve dependencies
-   Upgrade software
-   Verify package metadata
-   Keep software versions consistent

Linux distributions normally configure official repositories by default,
and administrators can add additional repositories when necessary.

------------------------------------------------------------------------

# 18. A Typical Debian/Ubuntu Workflow

A common workflow looks like this:

### Step 1: Refresh package information

``` bash
sudo apt update
```

### Step 2: Search for software

``` bash
apt search nginx
```

### Step 3: Install it

``` bash
sudo apt install nginx
```

### Step 4: Upgrade installed packages

``` bash
sudo apt upgrade
```

### Step 5: Remove software when no longer needed

``` bash
sudo apt remove nginx
```

### Step 6: Clean unused dependencies

``` bash
sudo apt autoremove
```

This gives you a basic package-management cycle:

``` text
Refresh
   ↓
Search
   ↓
Install
   ↓
Upgrade
   ↓
Remove
   ↓
Clean unused dependencies
```

------------------------------------------------------------------------

# 19. Important Safety Practices

Package managers can modify critical parts of the operating system, so
use them carefully.

### Review what will be installed or removed

Before confirming a large transaction, read the package manager's
summary.

Pay attention to:

-   Packages being installed
-   Packages being removed
-   Packages being upgraded
-   Additional disk space required
-   Dependency changes

### Prefer official/trusted repositories

Avoid installing random packages from untrusted sources.

Third-party repositories can be useful, but adding one means you are
trusting software and metadata from another source.

### Be careful with `sudo`

Commands such as:

``` bash
sudo apt remove ...
```

have administrator privileges.

A mistake can remove important system components.

### Avoid mixing package systems unnecessarily

Do not manually install the same software through unrelated methods
without understanding how they interact.

For example, if a distribution package manager already manages a system
library, manually replacing its files can make future upgrades
difficult.

------------------------------------------------------------------------

# 20. Useful Package-Management Questions

When troubleshooting Linux software, these questions are often useful:

### Is the package installed?

Debian/Ubuntu:

``` bash
dpkg -s package-name
```

or:

``` bash
apt list --installed | grep package-name
```

RPM-based:

``` bash
rpm -q package-name
```

### What files did it install?

Debian/Ubuntu:

``` bash
dpkg -L package-name
```

RPM-based:

``` bash
rpm -ql package-name
```

### Which package owns this file?

Debian/Ubuntu:

``` bash
dpkg -S /path/to/file
```

RPM-based:

``` bash
rpm -qf /path/to/file
```

### Which installed packages depend on this package?

RPM:

``` bash
rpm -q --whatrequires package-name
```

For Debian/Ubuntu, dependency inspection is generally easier with APT
tools such as:

``` bash
apt depends package-name
```

and reverse-dependency information can be explored with:

``` bash
apt rdepends package-name
```

------------------------------------------------------------------------

# 21. Package Management vs Application Distribution

It is useful to distinguish traditional distribution packages from other
Linux application formats.

Traditional distribution packages:

``` text
.deb → dpkg → apt
.rpm → rpm → dnf/zypper
```

Other application distribution systems include:

-   Snap
-   Flatpak
-   AppImage
-   Language-specific package managers such as `pip`, `npm`, and `cargo`

These solve somewhat different problems.

For example, `apt` primarily manages software integrated with the Linux
distribution, while Flatpak and Snap are designed to make applications
more self-contained and portable across distributions.

A software project can therefore be distributed through multiple
channels.

------------------------------------------------------------------------

# Key Takeaways

-   Linux software is commonly distributed as **packages**.
-   A package contains software files plus metadata needed to install
    and manage that software.
-   Debian-based systems commonly use `.deb` packages and `dpkg`.
-   RPM-based systems use `.rpm` packages and `rpm`.
-   `apt`, `dnf`, and `zypper` are **high-level package managers**.
-   `dpkg` and `rpm` are **low-level package-management tools**.
-   High-level package managers are normally preferred for everyday work
    because they can use repositories and resolve dependencies
    automatically.
-   `apt update` refreshes package information; it does **not** upgrade
    installed software.
-   `apt upgrade` upgrades installed packages using the current package
    information.
-   `apt install foo` installs `foo`, and can upgrade it if `foo` is
    already installed and a newer version is available.
-   `apt remove foo` removes the package but may leave configuration
    files and automatically installed dependencies.
-   `apt purge foo` also removes the package's configuration files.
-   `apt autoremove` removes automatically installed packages that are
    no longer required.
-   `find`, `grep`, and package-manager query commands can help
    investigate which files and packages are installed.
-   Always review package-manager transactions carefully, especially
    when using `sudo`.