# Common Linux Filesystem Types

A **filesystem** provides the structure Linux uses to store, organize, and retrieve data.

### Common Types

- **Conventional Disk Filesystems:** Used on hard drives and Solid-State Drives (SSDs), such as `ext4`, `XFS`, and `Btrfs`.
- **Flash Storage Filesystems:** Designed for raw flash memory, especially in embedded systems, such as `UBIFS` and `YAFFS`.
- **Special-Purpose Filesystems:** Usually exist in memory and expose system information, such as `procfs` (`/proc`) and `tmpfs`.
- **Database Filesystems:** Organize data as database-like objects or rows, focusing on fast searching, rich metadata, and data integrity.

# Partitions and Filesystems

It is important to distinguish between **storage hardware** and its **logical organization**.

- **Partition:** A defined section of a storage device. Example: `/dev/sda1`.
- **Filesystem:** The logical structure used to organize and access files within a partition, such as `ext4` or `XFS`.
- **Formatting:** Installing a filesystem on a partition.
- **Mounting:** Attaching a filesystem to a directory called a **mount point**.

For example:

```text
/dev/sdb1 → /home
```

After mounting, files saved under `/home` are stored on `/dev/sdb1`.

Unlike Windows, Linux does not use drive letters such as `C:` or `D:`. All filesystems are integrated into a **single directory tree**.

## Windows vs Linux Storage

| Feature | Windows | Linux |
|---|---|---|
| Partition Identifier | Disk 1 | `/dev/sda1` |
| Filesystem | NTFS / VFAT | ext4 / XFS / Btrfs |
| Mounting | Drive Letter (`D:`) | Mount Point (directory) |
| Root | `C:\` | `/` |

# Filesystem Hierarchy Standard (FHS)

The **Filesystem Hierarchy Standard (FHS)** defines the conventional organization of directories in Linux.

The current version is **FHS 3.0**. Most major distributions, including Fedora, Debian, Ubuntu, and Red Hat Enterprise Linux (RHEL), follow it closely.

The FHS makes Linux systems more predictable by giving directories standardized purposes.

## Key FHS Directories

| Directory | Purpose |
|---|---|
| `/` | Root of the entire filesystem tree. Every file and directory exists below it. |
| `/home` | Personal directories for regular users, e.g. `/home/student`. |
| `/root` | Home directory of the `root` superuser. |
| `/etc` | System-wide configuration files. |
| `/bin`, `/usr/bin` | Essential and general user commands such as `ls`, `cp`, and `bash`. On modern systems, `/bin` is usually a symbolic link to `/usr/bin`. |
| `/sbin`, `/usr/sbin` | System administration programs. |
| `/lib`, `/usr/lib` | Shared libraries required by system programs. |
| `/var` | Variable data such as logs, mail spools, and print queues. |
| `/tmp` | Temporary files; contents are not guaranteed to survive a reboot. |
| `/dev` | Device files representing hardware such as disks, terminals, and USB devices. |
| `/proc` | Virtual filesystem exposing real-time process and kernel information. |
| `/sys` | Virtual filesystem providing information and interfaces for the kernel and hardware. |
| `/boot` | Files required to boot the system, including the Linux kernel. |
| `/run` | Runtime data created since the last boot, such as Process ID (PID) and lock files. |

## Removable Media

Removable devices such as USB drives are also accessed through the filesystem tree.

Modern Linux systems commonly mount them under:

```text
/run/media/<username>/<disklabel>/
```

Example:

```text
/run/media/student/FEDORA/README.txt
```

Older distributions may use `/media`.

> The **Filesystem Hierarchy Standard (FHS)** is a "trailing standard": it documents established practices rather than strictly enforcing them. Distributions can have minor differences.

# Case Sensitivity

Linux is **case-sensitive**:

```text
/boot
/Boot
/BOOT
```

These are three different directories.

Windows and macOS are generally **case-insensitive but case-preserving**: they remember capitalization but normally treat `Documents`, `documents`, and `DOCUMENTS` as the same location.

Therefore, Linux paths must be typed exactly:

```bash
cd /home
```

works, while:

```bash
cd /Home
```

may fail because `/Home` and `/home` are different paths.

# The `/usr` Directory

Linux separates files needed for **basic boot/recovery** from the broader operating system software.

- `/` contains the minimum required for booting and basic recovery.
- `/usr` contains most general-purpose programs, libraries, and resources.

The name `/usr` originally meant **Unix System Resources**.

## Key `/usr` Directories

| Directory | Contains |
|---|---|
| `/usr/bin` | Most user-facing commands and programs. |
| `/usr/sbin` | System administration tools not required during early boot. |
| `/usr/lib` | Shared libraries for `/usr/bin` and `/usr/sbin`. |
| `/usr/share` | Architecture-independent data such as documentation and icons. |
| `/usr/local` | Software manually installed outside the distribution's package manager. |

### `/bin` vs `/usr/bin`

On most modern Linux distributions:

```text
/bin → /usr/bin
```

`/bin` is usually a **symbolic link** to `/usr/bin`. The same applies to directories such as `/sbin` and `/lib` on many modern systems.

The historical separation existed partly because older systems used smaller disks. Modern systems generally no longer need this separation.

### Practical Takeaway

When looking for programs:

```text
/usr/bin        → Distribution-provided programs
/usr/local/bin  → Manually installed or compiled programs
```