# Linux Login, System Control, Navigation, and Links

This section covers some of the most useful everyday Linux skills:

- Logging into local and remote systems
- Rebooting and shutting down safely
- Finding installed commands
- Navigating directories
- Understanding absolute and relative paths
- Exploring the filesystem
- Understanding hard links and symbolic links
- Moving through a history of directories with `pushd` and `popd`

---

# Logging In and Out

When you log into a Linux system through a text terminal, you will normally see a prompt similar to:

```text
login:
```

Enter your username and press **Enter**. Linux will then ask for your password.

```text
login: student
Password:
```

### Why Doesn't the Password Appear?

When entering a password in a Linux terminal, **nothing is displayed**—not even `*` characters.

This is intentional. It prevents someone watching the screen from learning even the approximate length of your password.

Simply type the password and press **Enter**.

After successful authentication, you receive a shell prompt and can begin working with the system.

> **Important:** A terminal does not show visual feedback while entering a password, but your keystrokes are still being received.

---

# Connecting to Remote Systems with SSH

Once you have a terminal session, you can connect to another Linux machine using **SSH (Secure Shell)**.

For example:

```bash
ssh student@remote-server.com
```

This means:

- `ssh` → the SSH client
- `student` → the username on the remote machine
- `remote-server.com` → the remote machine's hostname

After authentication, your terminal is connected to the remote system.

For example:

```text
$ ssh student@remote-server.com
student@remote-server.com's password:

student@remote-server:~$
```

Commands you now run are executed on the **remote machine**, not your local computer.

## How Does SSH Authenticate You?

SSH commonly uses either:

### Password authentication

You provide the remote user's password:

```bash
ssh student@server.example.com
```

### SSH key authentication

You use a cryptographic key pair instead of entering the account password each time.

Key-based authentication is widely used for servers because it can be more secure and is especially convenient for automation.

> **Important:** SSH provides an encrypted connection between your computer and the remote system, protecting the communication from being transmitted as plain text.

---

# Rebooting and Shutting Down

Linux systems should be shut down or rebooted properly rather than simply turning off the power.

An improper shutdown can cause:

- Unsaved data to be lost
- Filesystem inconsistencies
- Running applications to terminate unexpectedly
- Services to stop without performing their normal cleanup

The `shutdown` command is designed to perform an orderly shutdown or reboot.

## Shut Down the System

A common command is:

```bash
sudo shutdown -h now
```

Here:

- `sudo` → run with administrative privileges
- `shutdown` → initiate a system shutdown
- `-h` → halt/power off
- `now` → perform it immediately

## Reboot the System

```bash
sudo shutdown -r now
```

Here, `-r` means **reboot**.

Modern Linux systems also provide dedicated commands:

```bash
sudo reboot
```

and:

```bash
sudo poweroff
```

These are commonly used for rebooting and powering off the system.

> **Note:** The exact behavior and available options can vary slightly between distributions and systemd configurations.

---

# Warning Users Before Shutdown

On a multi-user system, you should warn users before shutting down or rebooting the machine.

For example:

```bash
sudo shutdown -h 10:00 "Shutting down for scheduled maintenance."
```

This schedules the shutdown for **10:00** and sends the message to logged-in users.

This is particularly useful on shared servers where other users may have active sessions or running work.

> **Production tip:** Before shutting down a server, always consider which services and users may be affected.

---

# Locating Applications and Commands

Linux programs can be installed in many different locations.

Common directories include:

```text
/bin
/usr/bin
/sbin
/usr/sbin
/usr/local/bin
/usr/local/sbin
/opt
```

Users may also keep their own executable programs in directories such as:

```text
/home/student/bin
```

The actual locations depend on the distribution, installation method, and software.

Linux provides several commands for finding out what a command actually refers to.

---

# `which`

The `which` command searches the directories in your `$PATH` and shows the executable that would normally be run.

For example:

```bash
which diff
```

Output:

```text
/usr/bin/diff
```

This tells you that when you run:

```bash
diff
```

the shell will normally find the executable at:

```text
/usr/bin/diff
```

## Why Is `which` Useful?

It is useful when:

- You want to know where an executable is installed
- Multiple versions of a program may exist
- You want to check which executable your `$PATH` will find

For example:

```bash
which python
```

might produce:

```text
/usr/bin/python
```

---

# `whereis`

`whereis` searches for more than just the executable.

For example:

```bash
whereis diff
```

It may return information about:

- The executable
- Source files
- Manual pages

For example:

```text
diff: /usr/bin/diff /usr/share/man/man1/diff.1.gz
```

So:

- **`which`** → primarily tells you which executable will be found through `$PATH`
- **`whereis`** → searches for related files such as binaries and manual pages

---

# `type`

The `type` command is especially useful because it tells you **what kind of command** the shell will execute.

For example:

```bash
type diff
```

Output:

```text
diff is /usr/bin/diff
```

This tells you that `diff` is an executable program.

But commands do not always refer to programs stored on disk.

For example:

```bash
type cd
```

may produce:

```text
cd is a shell builtin
```

And:

```bash
type ll
```

may produce:

```text
ll is aliased to 'ls -l --color=auto'
```

Here, `ll` is not a separate program. It is an **alias** that expands to another command.

## Why `type` Is Important

`type` can reveal whether a command is:

- An executable program
- A shell built-in
- An alias
- A shell function

This can explain surprising command behavior.

For example, if you type:

```bash
ll
```

and wonder where the `ll` program is installed, `which ll` may not give you the answer you expect. `type ll` reveals that `ll` is simply an alias.

> **Rule of thumb:** Use `type` when you want to know **what the shell thinks a command is**.

---

# Accessing Directories

When you start a normal terminal session, you will usually begin in your **home directory**.

You can check your current location with:

```bash
pwd
```

`pwd` means **print working directory**.

For example:

```bash
$ pwd
/home/student
```

Your home directory can also be displayed using:

```bash
echo $HOME
```

For example:

```text
/home/student
```

## What Is `$HOME`?

`$HOME` is an **environment variable** containing the path to your home directory.

For a user named `student`, it might contain:

```text
/home/student
```

You can use it in commands:

```bash
cd $HOME
```

This takes you to your home directory.

---

# Your Starting Directory Can Depend on How You Open the Terminal

When launching a terminal from the application menu or using a shortcut such as:

```text
Ctrl + Alt + T
```

you will normally start in your home directory.

However, if you right-click a directory and choose **Open in Terminal**, the terminal may start in that directory.

For example, if you open a terminal from:

```text
/home/student/projects
```

then:

```bash
pwd
```

may show:

```text
/home/student/projects
```

This behavior is controlled by the desktop environment and terminal application.

---

# Essential Directory Navigation Commands

| Command | Purpose |
|---|---|
| `pwd` | Displays the current directory |
| `cd` | Changes directory |
| `cd ~` | Changes to your home directory |
| `cd ..` | Moves to the parent directory |
| `cd -` | Returns to the previous directory |

## `pwd`

Shows where you currently are:

```bash
pwd
```

Example:

```text
/home/student/projects
```

## `cd`

Changes your current directory.

```bash
cd /usr/bin
```

Now:

```bash
pwd
```

produces:

```text
/usr/bin
```

## `cd` or `cd ~`

Both take you to your home directory:

```bash
cd
```

or:

```bash
cd ~
```

The `~` character is a shortcut representing your home directory.

## `cd ..`

Moves one level up to the parent directory.

Suppose you are here:

```text
/home/student/projects
```

Running:

```bash
cd ..
```

takes you to:

```text
/home/student
```

Running it again:

```bash
cd ..
```

takes you to:

```text
/home
```

## `cd -`

Returns to your previous working directory.

For example:

```bash
cd /var/log
cd /usr/bin
```

Now:

```bash
cd -
```

takes you back to:

```text
/var/log
```

This is extremely useful when switching between two directories repeatedly.

---

# Absolute and Relative Paths

Linux paths can be written in two main ways:

- **Absolute paths**
- **Relative paths**

Understanding the difference is essential for working effectively on Linux.

---

# Absolute Paths

An **absolute path** describes a location starting from the root directory:

```text
/
```

It specifies the complete path to the destination.

For example:

```text
/usr/bin
```

and:

```text
/home/student/projects/app.py
```

are absolute paths.

They always begin with `/`.

### Example

Regardless of your current location, this command always attempts to enter the same directory:

```bash
cd /usr/bin
```

Whether you are currently in:

```text
/home/student
```

or:

```text
/tmp
```

the destination is still:

```text
/usr/bin
```

---

# Relative Paths

A **relative path** describes a location starting from your **current working directory**.

Relative paths do **not** begin with `/`.

Suppose you are currently in:

```text
/home/student
```

and the directory you want is:

```text
/home/student/projects
```

You can use:

```bash
cd projects
```

because `projects` is relative to your current location.

You could also use:

```bash
cd ./projects
```

Here `.` means the current directory.

---

# Special Path Shortcuts

Linux provides several useful shortcuts:

| Symbol | Meaning |
|---|---|
| `/` | Root directory |
| `.` | Current directory |
| `..` | Parent directory |
| `~` | Current user's home directory |
| `-` | Previous working directory when used with `cd` |

### Example

Suppose you are in:

```text
/home/student/projects
```

Then:

```text
.
```

means:

```text
/home/student/projects
```

and:

```text
..
```

means:

```text
/home/student
```

---

# Absolute vs. Relative: Which Should You Use?

Neither is always better.

Use whichever makes the command clearer and less error-prone.

Suppose you are currently in:

```text
/home/student
```

and want to reach:

```text
/usr/bin
```

You could use the absolute path:

```bash
cd /usr/bin
```

Or navigate relatively:

```bash
cd ../../usr/bin
```

The absolute version is clearly better here because it is shorter and easier to understand.

However, if you are already in:

```text
/home/student
```

and want to reach:

```text
/home/student/projects
```

then:

```bash
cd projects
```

is much simpler than:

```bash
cd /home/student/projects
```

> **Tip:** If you ever become unsure about your location, run `pwd`. It shows your complete absolute path.

---

# Multiple Slashes in Paths

Linux generally accepts multiple consecutive `/` characters in a path.

For example:

```bash
cd ////usr///bin
```

is interpreted in practice as:

```text
/usr/bin
```

The extra slashes between path components do not create additional directories.

Although this is valid, **avoid unnecessary slashes in normal commands** because `/usr/bin` is clearer and easier to read.

---

# Exploring the Filesystem

Linux filesystems form a tree-like hierarchy.

At the top is the root directory:

```text
/
```

Directories branch out from it:

```text
/
├── home
│   └── student
├── etc
├── usr
│   ├── bin
│   └── lib
├── var
└── tmp
```

Several commands are useful for exploring this structure.

---

# `cd /`

To move to the root directory:

```bash
cd /
```

Verify your location:

```bash
pwd
```

Output:

```text
/
```

---

# `ls`

The `ls` command lists the contents of a directory.

```bash
ls
```

For example:

```text
bin  boot  dev  etc  home  lib  opt  tmp  usr  var
```

By default, it lists the current directory.

You can also specify another directory:

```bash
ls /var/log
```

---

# `ls -a`

Files and directories whose names begin with `.` are normally hidden from a standard `ls` listing.

Use:

```bash
ls -a
```

to display them.

For example:

```text
.  ..  .bashrc  .config  Documents  Downloads
```

Here:

- `.` → current directory
- `..` → parent directory
- `.bashrc` → a hidden configuration file
- `.config` → a hidden configuration directory

> **Important:** "Hidden" does not mean secure or inaccessible. In Linux, a filename beginning with `.` is simply treated as hidden by many directory-listing tools.

---

# `tree`

The `tree` command provides a visual representation of directories and their contents.

For example:

```bash
tree
```

might display:

```text
.
├── Documents
│   ├── report.txt
│   └── notes.txt
├── Downloads
│   └── installer.deb
└── projects
    ├── app
    └── website
```

If you only want to see directories:

```bash
tree -d
```

This can be useful when you want a quick overview of a directory structure.

### Installing `tree`

`tree` may not be installed by default.

On Ubuntu/Debian:

```bash
sudo apt install tree
```

Then:

```bash
tree -d
```

---

# Hard Links

Linux supports a concept called a **hard link**.

A hard link is not an independent copy of a file. Instead, it is **another filename referring to the same underlying filesystem object and data**.

Suppose:

```text
file1
```

already exists.

You can create another hard link to it:

```bash
ln file1 file2
```

Now both names refer to the same underlying file data.

Conceptually:

```text
file1 ──┐
        ├──> same inode/data
file2 ──┘
```

---

# Inodes and Hard Links

Every file on a Unix/Linux filesystem is associated with an **inode**.

An inode stores metadata about the file, while the file's directory entry associates a filename with that inode.

You can inspect inode numbers using:

```bash
ls -li file1 file2
```

You may see something similar to:

```text
123456 -rw-r--r-- 2 student student 100 Sep 8 10:00 file1
123456 -rw-r--r-- 2 student student 100 Sep 8 10:00 file2
```

Notice:

- Both files have the same inode number: `123456`
- The link count is `2`

This confirms that the two names refer to the same underlying file.

> **Important:** The exact inode number, ownership, permissions, size, and timestamps will differ on your system.

---

# What Happens When You Delete a Hard Link?

Suppose:

```text
file1 ──┐
        ├──> same inode/data
file2 ──┘
```

If you run:

```bash
rm file1
```

the underlying data is **not immediately removed**, because `file2` still refers to the same inode.

You can still access the data through:

```bash
cat file2
```

The filesystem removes the underlying file data only when there are no remaining directory entries (hard links) referring to that inode and no processes still have the file open.

This is why a hard link is better understood as **another name for the same file**, rather than a copy.

---

# Hard Links and Editing Files

Be careful when using hard links for files that you intend to edit.

Some modern editors use a technique called an **atomic save**.

Instead of modifying the original file directly, an editor may:

1. Create a new temporary file.
2. Write the new contents to it.
3. Rename the new file over the original.

If this happens to a hard-linked file, the edited filename may now point to a **new inode**, while the other hard-link name continues pointing to the original inode.

For example, before editing:

```text
file1 ──┐
        ├──> inode A
file2 ──┘
```

After an atomic save through `file1`:

```text
file1 ──> inode B
file2 ──> inode A
```

The two names are no longer linked to the same data.

The exact behavior depends on the editor and how it saves files.

> **Practical advice:** Do not use hard links as a way of maintaining synchronized copies of files you expect to edit regularly.

---

# Symbolic Links (Soft Links)

A **symbolic link**, commonly called a **symlink**, is different from a hard link.

Create one using:

```bash
ln -s file1 file3
```

A symbolic link stores a **path to another file or directory**.

Conceptually:

```text
file3 ──> path of file1 ──> target
```

It behaves similarly to a shortcut.

---

# Inspecting a Symbolic Link

Run:

```bash
ls -li file1 file3
```

You may see:

```text
123456 -rw-r--r-- 1 student student 100 Sep 8 10:00 file1
789012 lrwxrwxrwx 1 student student   5 Sep 8 10:01 file3 -> file1
```

Notice the differences:

- The symlink has a **different inode number**
- The `l` at the beginning of the permissions field indicates a symbolic link
- `-> file1` shows the link's target

Unlike a hard link, the symlink does not share the target's inode.

---

# Why Use Symbolic Links?

Symbolic links are extremely useful because they are flexible.

## 1. Cross-Filesystem Links

Hard links normally cannot cross filesystem boundaries.

For example, you cannot generally create a hard link from one filesystem to a file on another filesystem.

Symbolic links can point across:

- Different partitions
- Different disks
- Mounted filesystems
- Network filesystems
- Other filesystem locations

Example:

```bash
ln -s /mnt/storage/project /home/student/project
```

The link in your home directory points to a location on another mounted filesystem.

---

## 2. Very Little Storage

A symbolic link mainly stores the target path.

For example:

```bash
ln -s /var/log/application/production.log latest.log
```

`latest.log` does not contain a second copy of the log file.

It simply points to:

```text
/var/log/application/production.log
```

Therefore, a symlink usually requires very little additional storage.

---

## 3. Convenient Stable Names

Symlinks can provide a short or stable name for a changing location.

For example, suppose applications are installed in versioned directories:

```text
/opt/myapp-1.0
/opt/myapp-2.0
/opt/myapp-3.0
```

You could create:

```text
/opt/myapp -> /opt/myapp-3.0
```

Applications can then use:

```text
/opt/myapp
```

without needing to know the exact version directory.

When version 4.0 is installed, the symlink can be changed to point to:

```text
/opt/myapp-4.0
```

This pattern is common in software deployments.

---

# Dangling Symbolic Links

Because a symlink points to a **path**, the target can disappear.

For example:

```text
file1
  ↑
  |
file3
```

If `file1` is deleted:

```bash
rm file1
```

then `file3` still exists, but its target no longer does.

This is called a **dangling symlink**.

You may see:

```text
file3 -> file1
```

even though `file1` does not exist.

The same thing can happen when:

- A target is moved
- A target is renamed
- A filesystem containing the target is unmounted
- A mounted directory is temporarily unavailable

---

# Hard Link vs. Symbolic Link

| Feature | Hard Link | Symbolic Link |
|---|---|---|
| Points to | Same inode/data | A path |
| Own inode? | No separate inode | Yes |
| Can cross filesystems? | Usually no | Yes |
| Can point to directories? | Generally no | Yes |
| Can become dangling? | No, as long as the inode remains referenced | Yes |
| Uses much extra storage? | No copy of data | Very little |
| Multiple names for same data? | Yes | Indirectly |
| Common use | Additional name for the same file | Shortcut/reference to another path |

### Simple way to remember

**Hard link:**

> "Another name for the same file."

**Symbolic link:**

> "A path that points to another file or directory."

---

# Navigating Directory History

Normally, `cd` only makes it easy to return to your **immediately previous directory**:

```bash
cd -
```

But what if you want to remember several directories?

Linux shells provide a **directory stack** using:

- `pushd`
- `popd`
- `dirs`

---

# `pushd`

`pushd` changes the current directory and saves the previous directory on a stack.

Suppose you start in:

```text
/usr/local
```

Run:

```bash
pushd /tmp
```

You move to `/tmp`, while `/usr/local` is saved on the directory stack.

Then:

```bash
pushd /boot
```

moves to `/boot` and adds another directory to the stack.

Then:

```bash
pushd /
```

moves to `/` and adds another entry.

Conceptually, the stack now contains several previous locations.

---

# `dirs`

Use:

```bash
dirs
```

to display the directory stack.

For example, you might see:

```text
/ /boot /tmp /usr/local
```

The exact formatting depends on your shell.

---

# `popd`

`popd` removes the most recently saved directory from the stack and changes to it.

For example, if you used:

```bash
pushd /tmp
pushd /boot
pushd /
```

then:

```bash
popd
```

takes you back to:

```text
/boot
```

Another:

```bash
popd
```

takes you to:

```text
/tmp
```

Another:

```bash
popd
```

takes you to:

```text
/usr/local
```

The directory stack follows **last in, first out (LIFO)** behavior.

This is similar to a stack data structure in programming:

```text
push → add an item
pop  → remove the most recently added item
```

---

# `cd -` vs. `pushd`/`popd`

These commands solve related but different problems.

### `cd -`

Useful when switching between **two directories**:

```bash
cd /var/log
cd /etc
cd -
```

You return to `/var/log`.

### `pushd` and `popd`

Useful when you need to remember **several directories**:

```bash
pushd /tmp
pushd /boot
pushd /var/log
```

You can then use:

```bash
popd
```

to move backward through the saved locations.

> **Tip:** Use `cd -` for quick back-and-forth navigation. Use `pushd`/`popd` when working across several directories and you want an explicit navigation history.

---

# Practical Navigation Example

Imagine you are starting in:

```text
/home/student
```

You want to inspect several system directories and eventually return to your original location.

You could do:

```bash
pushd /tmp
pushd /var/log
pushd /etc
```

Check the stack:

```bash
dirs
```

Explore the current directory:

```bash
ls
```

Then return through the saved directories:

```bash
popd
popd
popd
```

You have now worked your way back through the directory stack.

---

# Quick Reference

## Login and Remote Access

```bash
# Connect to a remote machine
ssh student@remote-server.com
```

## System Shutdown

```bash
# Shut down immediately
sudo shutdown -h now

# Reboot immediately
sudo shutdown -r now

# Reboot
sudo reboot

# Power off
sudo poweroff
```

## Find Commands

```bash
# Find executable selected through PATH
which diff

# Find binary, source, and manual pages
whereis diff

# Determine what kind of command it is
type diff
type cd
type ll
```

## Directory Navigation

```bash
# Show current directory
pwd

# Go to home directory
cd
cd ~

# Go to root
cd /

# Go to parent
cd ..

# Go to previous directory
cd -
```

## Explore Directories

```bash
# List contents
ls

# Include hidden files
ls -a

# Show directory tree
tree

# Show directories only
tree -d
```

## Links

```bash
# Create a hard link
ln file1 file2

# Create a symbolic link
ln -s file1 file3

# Show inode numbers
ls -li file1 file2 file3
```

## Directory Stack

```bash
# Save current location and move
pushd /tmp

# Show directory stack
dirs

# Return through saved locations
popd
```

---

# Key Takeaways

1. **Passwords are intentionally invisible** when entered in a Linux terminal.
2. **SSH** lets you securely access and administer remote systems from the command line.
3. Use **`shutdown`**, **`reboot`**, or **`poweroff`** for orderly system management instead of abruptly cutting power.
4. **`which`** finds the executable that would normally be selected through `$PATH`.
5. **`type`** tells you what a command really is—such as an executable, alias, built-in, or function.
6. **`pwd`** tells you where you are, while **`cd`** moves you around the filesystem.
7. **Absolute paths** start from `/`; **relative paths** start from your current directory.
8. **`.`**, **`..`**, and **`~`** are important shortcuts for navigating paths.
9. A **hard link** is another name for the same underlying inode/data.
10. A **symbolic link** points to another path and can cross filesystem boundaries.
11. Symbolic links can become **dangling** when their targets disappear.
12. **`pushd`**, **`popd`**, and **`dirs`** provide a directory stack for navigating through multiple locations.