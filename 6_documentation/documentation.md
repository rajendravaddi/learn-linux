# Linux Documentation Sources

You will not always remember the exact command, option, or syntax needed to perform a task in Linux. **Consulting documentation is a normal part of Linux administration and development**, even for experienced users.

Linux documentation comes from several sources because a Linux system combines the kernel, GNU tools, distribution-specific software, desktop applications, and other projects. The most useful sources are:

- **Man pages** — detailed reference documentation for commands, configuration files, system calls, libraries, and more.
- **GNU Info** — structured, cross-linked documentation used especially by GNU projects.
- **`--help` and Bash `help`** — quick command-line references for options and shell built-ins.
- **Distribution documentation** — information specific to distributions such as Ubuntu, Fedora, and Gentoo.
- **Package documentation** — README files, changelogs, examples, and other documentation installed with software.
- **Graphical help systems** — desktop help browsers such as GNOME Help and KDE Help.
- **Online resources** — official project documentation, books, wikis, and community resources.

---

## Man Pages

### Overview

**Man pages** (manual pages) are one of the most important and widely available sources of Linux documentation. They provide reference information for:

- Commands and utilities
- Configuration files
- System calls
- Library functions
- Device files
- File formats
- Administrative interfaces

To open a manual page, use:

```bash
man <topic>
```

For example:

```bash
man ls
```

The `man` program finds the requested manual entry, formats it for the terminal, and normally displays it through a **pager**, usually `less`. This lets you read long documentation one screen at a time.

### Navigating a Man Page

Because man pages are commonly displayed with `less`, a few keys are particularly useful:

| Key | Action |
|---|---|
| `↑` / `↓` | Move up or down |
| `Space` | Move forward one screen |
| `b` | Move backward one screen |
| `/keyword` | Search for a keyword |
| `n` | Go to the next search match |
| `N` | Go to the previous search match |
| `q` | Quit and return to the shell |

Learning these `less` shortcuts is useful beyond man pages because `less` is also commonly used to view logs and large text files.

---

## Finding the Right Man Page

A topic can have multiple manual pages, often because the same name is used for different types of interfaces.

### `man -f` / `whatis`

Use:

```bash
man -f <topic>
```

or the equivalent:

```bash
whatis <topic>
```

This lists manual pages whose **names and short descriptions** match the topic.

For example:

```bash
man -f socket
```

### `man -k` / `apropos`

If you know what you want to accomplish but do not know the command name, use:

```bash
man -k <keyword>
```

or:

```bash
apropos <keyword>
```

For example:

```bash
man -k compress
```

This searches the **short descriptions of available man pages** for the keyword and can help you discover relevant commands.

### Selecting a Manual Section

When several man pages have the same name, specify the section:

```bash
man <section> <topic>
```

For example:

```bash
man 2 socket
```

This opens the `socket` manual page from section 2, which documents a system call.

To view all matching pages one after another:

```bash
man -a socket
```

The exact set of available sections and their contents can vary somewhat between systems.

---

## Manual Sections

Man pages are divided into numbered sections. The traditional sections are:

| Section | Contents | Examples |
|---|---|---|
| **1** | User commands and executable programs | `ls`, `cp`, `grep` |
| **2** | System calls provided by the kernel | `open()`, `read()`, `write()` |
| **3** | Library functions | `printf()`, `malloc()` |
| **4** | Special files and device files | `/dev/null` |
| **5** | File formats and configuration files | `fstab(5)`, `crontab(5)` |
| **6** | Games | Games and related programs |
| **7** | Miscellaneous information | Conventions, standards, macro packages |
| **8** | System administration commands | `mount`, `fdisk` |
| **9** | Kernel routines | Non-standard kernel interfaces |

Not every Linux distribution provides the same number of pages in every section. **Section 9, in particular, is often absent or sparsely populated.**

A section number is important when the same topic exists in multiple sections. For example:

```bash
man 1 printf
```

documents the `printf` command, while:

```bash
man 3 printf
```

documents the `printf()` library function.

### Common Man Page References

You may see references such as:

```text
ls(1)
open(2)
printf(3)
fstab(5)
```

The number in parentheses identifies the manual section.

---

# GNU Info

**GNU Info** is another documentation system, particularly important for GNU utilities. It was designed to provide richer, more structured documentation than traditional man pages.

A useful way to think about the difference is:

- **Man page:** primarily a focused reference page.
- **Info manual:** a structured manual containing linked sections, menus, examples, and cross-references.

Info documentation is organized into **nodes**, which are connected to one another like pages in a tree.

### Opening Info

Open the top-level Info directory with:

```bash
info
```

Open documentation for a particular utility with:

```bash
info <command>
```

For example:

```bash
info ls
```

For GNU programs, the Info manual may contain significantly more explanatory material than the corresponding man page.

### Info Navigation

Useful Info keys include:

| Key | Action |
|---|---|
| `Tab` | Move to the next link |
| `Enter` | Follow the selected link |
| `n` | Go to the next node |
| `p` | Go to the previous node |
| `u` | Move to the parent node |
| `l` | Return to the previously visited node |
| `/` | Search |
| `h` | Open the Info tutorial/help |
| `q` | Quit |

Info navigation is different from the `less` interface used by most man pages, so the shortcuts are worth learning if you use Info regularly.

---

# The `--help` Option

For a quick reminder of a command's syntax and options, try:

```bash
<command> --help
```

For example:

```bash
ls --help
```

This usually prints a short usage summary directly in the terminal and immediately returns you to the shell.

### When to Use `--help`

Use `--help` when:

- You already know the command.
- You need to remember an option or syntax.
- You want a quick overview rather than detailed documentation.

Use a **man page** when you need a more complete reference, detailed behavior, descriptions of arguments, examples, or related information.

### Do Not Assume `-h` Means Help

The short option `-h` is **not standardized as a help option**.

For example:

```bash
ls -h
```

usually means **human-readable sizes**, not help.

Therefore, when you specifically want command help, prefer:

```bash
ls --help
```

The behavior of `--help` is common but not guaranteed for every program; some programs use different conventions.

---

# The Bash `help` Command

Some commands are not separate executable programs. They are **shell built-ins**, meaning Bash implements them directly.

Examples include:

```bash
cd
echo
pwd
```

For Bash built-ins, Bash provides the `help` command:

```bash
help
```

This lists the available Bash built-ins.

To get information about a specific built-in:

```bash
help cd
```

This is especially useful because a built-in may not have its own standalone executable or may behave differently from a similarly named external command.

### `help` vs `--help`

For shell built-ins, `help` is generally the reliable way to access Bash's built-in documentation.

For example:

```bash
help echo
```

is preferable to assuming:

```bash
echo --help
```

will display documentation. In Bash, `echo --help` can simply treat `--help` as text to print, depending on the implementation and options involved.

### Is a Command a Built-in?

Use:

```bash
type <command>
```

For example:

```bash
type cd
type ls
```

You may see output indicating that `cd` is a Bash built-in, while `ls` is an external program.

---

# Distribution-Specific Documentation

Linux distributions provide documentation for features that are specific to that distribution, including:

- Installation and setup
- Package management
- System configuration
- Distribution-specific tools
- Security configuration
- Networking
- Desktop environments
- Release-specific changes

Examples include:

- [Ubuntu Community Help Wiki](https://help.ubuntu.com/community/CommunityHelpWiki)
- [Fedora Documentation](https://docs.fedoraproject.org/en-US/fedora/latest/)
- [Gentoo Documentation](https://www.gentoo.org/support/documentation/)

When instructions differ between distributions, prefer the documentation for the distribution you are actually using.

---

# Graphical Help Systems

Linux desktop environments commonly provide graphical help applications.

### GNOME

GNOME provides **Help**, commonly associated with:

```bash
yelp
```

and also available through the `gnome-help` command on many systems.

### KDE

KDE provides:

```bash
khelpcenter
```

Graphical help systems can provide documentation for the desktop environment and, depending on the application, may integrate or present other documentation sources.

Many graphical Linux applications also support:

```text
F1
```

to open application help, although this depends on the individual application.

---

# Package Documentation

Software packages often include documentation in addition to their executable files.

A common location is:

```text
/usr/share/doc/
```

For example:

```text
/usr/share/doc/bash/
```

A package's documentation directory may contain:

- `README` files
- Changelogs
- Copyright and license information
- Examples
- Sample configuration files
- Distribution-specific notes

This documentation can be particularly valuable when you need information that is not covered by a command's man page.

For example:

```bash
ls /usr/share/doc/bash/
```

---

# Online Documentation

When local documentation is not enough, online resources can provide additional context and current information.

Useful sources include:

1. **Official project documentation** — usually the best source for project-specific behavior.
2. **Distribution documentation** — useful for distribution-specific configuration.
3. **Linux community wikis and forums** — useful for practical troubleshooting and real-world examples.
4. **Books and tutorials** — useful when learning concepts rather than looking up a single command.

A useful free learning resource is **The Linux Command Line** by William Shotts, available from [LinuxCommand.org](https://linuxcommand.org/tlcl.php).

You can also browse Linux man pages online at [man7.org](https://man7.org/linux/man-pages/).

---

# Which Documentation Source Should You Use?

A simple decision guide:

| Situation | Best starting point |
|---|---|
| Quickly remember a command option | `command --help` |
| Learn a Bash built-in | `help command` |
| Need detailed command reference | `man command` |
| Same name exists in multiple sections | `man <section> command` |
| Do not know which command to use | `man -k keyword` / `apropos keyword` |
| Need GNU-specific, detailed documentation | `info command` |
| Need distribution-specific instructions | Distribution documentation |
| Need examples, README, or changelog | `/usr/share/doc/<package>/` |
| Need current project-specific information | Official online documentation |

## A Practical Documentation Workflow

When working in Linux, a useful workflow is:

```text
Know the command, need a quick reminder
        ↓
    command --help

Need detailed reference
        ↓
       man

Need Bash built-in documentation
        ↓
    help command

Don't know the command name
        ↓
  man -k keyword

Need GNU's detailed manual
        ↓
      info

Need distro/package-specific information
        ↓
Distribution docs / /usr/share/doc

Still need context or current information
        ↓
Official online documentation
```

The goal is not to memorize every Linux command. **Learn how to find reliable documentation quickly.** That skill remains useful even as you become an experienced Linux user.