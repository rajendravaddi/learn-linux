# Linux Command Line and Terminal Basics

Linux system administrators spend a lot of time at the command line because it gives direct control over the system. A common Unix saying captures the idea well:

> **Graphical interfaces make easy tasks easier; the command line makes difficult tasks possible.**

Linux was designed around the command line, so terminal-based tools remain powerful even when a graphical desktop is installed.

## Why Learn the Command Line?

Working from the command line lets you:

- **Avoid graphical overhead:** A terminal uses far fewer resources than a full desktop environment.
- **Access almost every system function:** You can perform tasks directly without navigating through multiple menus and windows.
- **Automate repetitive work:** Commands can be combined into shell scripts so that tasks can run automatically.
- **Manage remote systems:** Tools such as SSH allow you to administer a remote Linux server as if you were working on it locally.
- **Launch applications quickly:** You can start programs directly by typing their command instead of searching through menus.

### Why Command-Line Skills Transfer Well

Desktop environments and graphical tools can differ significantly between Linux distributions. The command line is much more consistent.

For example, the graphical settings application may look different on Ubuntu and openSUSE, but commands such as `ls`, `cd`, `cat`, `grep`, and `systemctl` work in familiar ways across most Linux systems.

**Learning the command line on one Linux distribution gives you skills that are useful on many others.**

---

# Terminal Emulators

A **terminal emulator** is a graphical application that provides a window where you can interact with a shell using text commands.

Although the terminal is displayed inside a normal graphical window, it behaves much like the traditional text-only terminals used on Unix and Linux systems.

Most modern terminal emulators support:

- Multiple tabs
- Multiple terminal windows
- Copy and paste
- Search
- Custom fonts and colors

Common terminal emulators include:

- `gnome-terminal` — commonly used with GNOME
- `konsole` — commonly used with KDE Plasma
- `xterm` — a lightweight, traditional terminal emulator
- `terminator` — supports multiple terminal panes and layouts

> **Important:** The terminal emulator is the window/application. The **shell** running inside it is what interprets the commands you type. Bash is one of the most common Linux shells.

---

# Opening a Terminal

The exact method depends on your desktop environment and distribution.

### Using the application search

On modern GNOME-based desktops:

1. Press the **Super/Windows key** or select **Activities**.
2. Type `terminal`.
3. Press **Enter**.

### Using the keyboard shortcut

On Ubuntu, **Ctrl + Alt + T** opens a terminal directly.

This is one of the most useful shortcuts to remember if you frequently work from the command line.

### Opening a terminal from a directory

Many desktop environments provide an **Open in Terminal** option when you right-click a directory.

This is particularly useful when you want the terminal to start in a specific directory.

> **Tip:** If you use the terminal frequently, pin it to your desktop's favorites or dock.

---

# Basic Command-Line Utilities

You will encounter a few commands repeatedly while learning Linux.

| Command | Purpose | Example |
|---|---|---|
| `cat` | Displays file contents | `cat notes.txt` |
| `head` | Displays the beginning of a file | `head notes.txt` |
| `tail` | Displays the end of a file | `tail notes.txt` |
| `man` | Opens documentation for a command | `man ls` |

### `cat`

Displays the contents of a file:

```bash
cat notes.txt
```

It can also combine multiple files:

```bash
cat file1.txt file2.txt
```

### `head`

Displays the first few lines:

```bash
head notes.txt
```

By default, `head` normally shows the first 10 lines. You can specify the number:

```bash
head -n 5 notes.txt
```

### `tail`

Displays the last few lines:

```bash
tail notes.txt
```

This is especially useful for checking log files:

```bash
tail /var/log/syslog
```

You can also follow a log as new lines are added:

```bash
tail -f application.log
```

### `man`

The `man` command displays a command's manual page:

```bash
man ls
```

For example:

```bash
man systemctl
```

Manual pages are one of the most important built-in sources of Linux documentation.

---

# Pipes: Connecting Commands

The **pipe operator** `|` sends the output of one command to another command as its input.

For example:

```bash
ls -l | head
```

Here:

1. `ls -l` produces a list of files.
2. `|` sends that output to `head`.
3. `head` displays only the beginning of the list.

You can chain several commands:

```bash
cat application.log | grep ERROR | head
```

This means:

> Read the log → find lines containing `ERROR` → show the first few matches.

Pipes are one of the most powerful features of the Linux shell because they allow small commands to be combined into useful workflows.

---

# Understanding Command Structure

A typical shell command can be understood as three main parts:

```text
command  options  arguments
```

For example:

```bash
ls -l /home/raj
```

- `ls` → **command**
- `-l` → **option**
- `/home/raj` → **argument**

## 1. Command

The **command** identifies the program, utility, or script you want to execute.

Example:

```bash
ls
```

Here, `ls` is the command.

## 2. Options

**Options** modify how a command behaves.

Short options usually use a single dash:

```bash
ls -l
```

Long options commonly use two dashes:

```bash
ls --all
```

Some commands allow multiple short options to be combined:

```bash
ls -la
```

This is equivalent to using `-l` and `-a` together.

## 3. Arguments

An **argument** tells the command what it should operate on.

Example:

```bash
cat report.txt
```

- `cat` → command
- `report.txt` → argument

Another example:

```bash
cp report.txt backup.txt
```

Here, `report.txt` and `backup.txt` are arguments specifying the source and destination.

> **Note:** Not every command requires options or arguments. For example, `ls` can be used by itself.

---

# `sudo`: Running Commands with Elevated Privileges

`sudo` allows an authorized user to execute a command with another user's privileges. It is most commonly used to run commands as the **root** user.

For example:

```bash
sudo apt update
```

The command runs with administrative privileges.

A useful comparison for Windows users is **Run as Administrator**.

macOS also provides the `sudo` command because both macOS and Linux have Unix-like foundations.

## Why Is `sudo` Needed?

Normal users do not have permission to modify important system files or perform certain administrative operations.

For example:

```bash
rm /important-system-file
```

may fail because the user does not have sufficient permissions.

An administrator may instead run:

```bash
sudo rm /important-system-file
```

However, **sudo should not be treated as a way to bypass permissions casually**. Running the wrong command with root privileges can damage the system.

---

# `sudo` Configuration

The default administrative configuration varies between Linux distributions.

### Ubuntu

During a typical Ubuntu installation, the first user account is automatically given `sudo` access.

Ubuntu also normally does not enable direct root login with a password by default.

Therefore, an administrator can usually run:

```bash
sudo command
```

without separately configuring sudo for the initial user.

### Other distributions

On some distributions and installation configurations, administrative access may need to be configured explicitly for additional users.

The exact configuration depends on the distribution and its security policies.

---

# Becoming Root

Linux provides several ways to obtain a root shell.

On systems where the root account has a usable password:

```bash
su
```

After entering the root password, the prompt may change from:

```text
$
```

to:

```text
#
```

The `#` prompt is commonly used to indicate that you are operating as root.

On Ubuntu, a safer/common approach is:

```bash
sudo -i
```

This starts an interactive root shell using your existing administrative privileges.

> **Security tip:** Prefer `sudo command` when you only need elevated privileges for one operation. Staying in a root shell for a long time increases the chance of accidentally executing a destructive command with full system privileges.

### Password Input

When Linux asks for a password in the terminal, **nothing is displayed while you type**—not even `*` characters.

This is normal. Type the password and press **Enter**.

---

# Granting a User `sudo` Access

If you need to give another user administrative access, use the distribution's recommended administrative mechanism.

A common approach is to add the user to an administrative group.

For example, on Ubuntu:

```bash
sudo usermod -aG sudo username
```

The user generally needs to log out and back in before the new group membership takes effect.

You can then verify group membership with:

```bash
groups username
```

On distributions such as RHEL/CentOS, the administrative group is commonly `wheel`:

```bash
sudo usermod -aG wheel username
```

> **Important:** Group names and sudo policies vary by distribution. Check the distribution's documentation before changing administrative access.

## Editing `sudoers`

For more advanced or customized sudo policies, use `visudo` rather than directly editing `/etc/sudoers`.

For example:

```bash
sudo visudo
```

`visudo` validates the syntax before installing the configuration. This is important because a syntax error in the sudo configuration can prevent users from obtaining administrative access.

A custom rule can also be placed under `/etc/sudoers.d/`, for example:

```bash
sudo visudo -f /etc/sudoers.d/username
```

A basic rule such as:

```text
username ALL=(ALL) ALL
```

allows `username` to run commands as any user through sudo, subject to the sudo configuration.

> **Important:** Do not casually edit `/etc/sudoers` with commands such as `echo ... > /etc/sudoers`. A mistake can break administrative access. `visudo` is designed specifically to reduce this risk.

---

# GUI vs. Command Line

A Linux graphical desktop is **optional**. Linux can run with a complete desktop environment or entirely without one.

For example, Linux can be installed as:

- A desktop system with GNOME or KDE Plasma
- A server without a graphical desktop
- A minimal cloud server containing only the services and tools required by the application

## Why Do Servers Often Avoid a GUI?

Production servers commonly run without a graphical desktop because removing unnecessary components can:

- Reduce resource usage
- Reduce the number of installed components that need maintenance
- Reduce the potential attack surface
- Make remote administration simpler

A typical Linux server might therefore be managed entirely through SSH and command-line tools.

---

# Virtual Terminals (VTs)

A **Virtual Terminal (VT)** is a full-screen text-based login session that operates separately from the graphical desktop.

Linux can provide multiple VTs simultaneously. Only one is normally visible at a time.

For example:

```text
VT1 → text terminal
VT2 → text terminal
VT3 → text terminal
VT4 → text terminal
...
```

Depending on the distribution and display manager, one VT is also used for the graphical session.

## Switching to a Virtual Terminal

From a graphical desktop, you can commonly switch to a VT using:

```text
Ctrl + Alt + F3
```

This usually takes you to VT3.

Other function keys can select other terminals:

```text
Ctrl + Alt + F2
Ctrl + Alt + F3
Ctrl + Alt + F4
```

The exact VT used by the graphical session varies by distribution and configuration.

From another text VT, `Alt + F3` is often sufficient.

## Why Are Virtual Terminals Useful?

VTs are especially useful when the graphical desktop freezes.

For example:

1. Your desktop becomes unresponsive.
2. You press `Ctrl + Alt + F3`.
3. Linux switches to a text login screen.
4. You log in.
5. You inspect processes, logs, memory usage, or services.
6. You can troubleshoot the problem without immediately rebooting.

To return to the graphical session, the correct function key depends on which VT the graphical session is using.

> **Important:** VTs are not the same as terminal emulator windows. A terminal emulator runs inside the graphical desktop, while a VT is a separate full-screen system console.

---

# Switching Between Graphical and Text-Only Modes

Modern Linux distributions generally use **systemd** to manage system startup and operating modes.

Instead of the older SysV **runlevels**, systemd uses **targets**.

Two important targets are:

- `multi-user.target` → text-based multi-user system
- `graphical.target` → graphical system with the desktop environment

## Switch to Text-Only Mode

```bash
sudo systemctl isolate multi-user.target
```

This changes the currently running system to the multi-user, non-graphical target.

Conceptually, this is similar to the traditional **runlevel 3**.

## Return to Graphical Mode

```bash
sudo systemctl isolate graphical.target
```

This switches the system to the graphical target.

Conceptually, this is similar to the traditional **runlevel 5**.

### What Does `isolate` Mean?

`isolate` tells systemd to activate the requested target and stop units that are not required by that target.

This is different from simply starting or stopping one individual service.

---

# The Older `telinit` Command

Older Linux documentation may use:

```bash
sudo telinit 3
```

to switch to a text-oriented runlevel and:

```bash
sudo telinit 5
```

to switch to a graphical runlevel.

On many modern systemd-based distributions, these commands are supported through compatibility behavior.

However, for modern systems, prefer:

```bash
sudo systemctl isolate multi-user.target
```

and:

```bash
sudo systemctl isolate graphical.target
```

because these commands directly express the systemd concepts being used.

---

# Display Managers

A **display manager** is responsible for providing the graphical login screen and starting graphical desktop sessions.

Examples include:

- `gdm` — GNOME Display Manager
- `sddm` — commonly used with KDE Plasma
- `lightdm` — lightweight display manager used by various desktop environments

You may encounter commands such as:

```bash
sudo systemctl stop gdm
```

and:

```bash
sudo systemctl start gdm
```

These commands stop or start the **display manager service**.

## Display Manager vs. System Target

These two approaches are related but not identical.

### Stopping the display manager

```bash
sudo systemctl stop gdm
```

This specifically stops the graphical login/display manager service.

### Switching to a text-only target

```bash
sudo systemctl isolate multi-user.target
```

This changes the system's overall active target and stops units that are no longer required.

The screen may look similar after both operations, but they represent different levels of system management.

> **Rule of thumb:** Use `systemctl isolate` when you want to change the system's overall operating target. Manage the display manager directly only when you specifically need to control that service.

---

# Quick Summary

| Concept | Meaning |
|---|---|
| **Terminal emulator** | Graphical application that provides a terminal window |
| **Shell** | Program that interprets commands, such as Bash |
| **Command** | Program or utility being executed |
| **Option** | Modifies how a command behaves |
| **Argument** | Data or target supplied to a command |
| **Pipe `\|`** | Sends one command's output to another command |
| **`sudo`** | Runs a command with elevated/another user's privileges |
| **Root** | Linux's superuser with extensive system privileges |
| **VT** | Full-screen virtual text terminal |
| **`systemctl`** | Tool for managing systemd services and targets |
| **`multi-user.target`** | Typical non-graphical multi-user system target |
| **`graphical.target`** | Graphical system target |
| **Display manager** | Provides graphical login and starts desktop sessions |

## Key Commands to Remember

```bash
# Open documentation
man ls

# Display a file
cat file.txt

# Show first lines
head file.txt

# Show last lines
tail file.txt

# Combine commands with a pipe
ls -l | head

# Run a command with administrative privileges
sudo command

# Start a root shell
sudo -i

# Switch to text-only system target
sudo systemctl isolate multi-user.target

# Switch back to graphical target
sudo systemctl isolate graphical.target
```

The most important idea is that **the Linux command line is not merely an alternative to the GUI**. It is a fundamental way of interacting with Linux. Once you become comfortable with commands, options, arguments, pipes, permissions, and system-management tools, you can administer both local machines and remote servers efficiently.