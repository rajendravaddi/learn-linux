# Process Monitoring: `ps`, `pstree`, and `top`

Linux provides several tools for inspecting running processes and system resource usage. The most important command-line tools are:

- **`ps`** — takes a snapshot of processes.
- **`pstree`** — shows processes as a parent-child hierarchy.
- **`top`** — continuously monitors processes and system resources.
- **Graphical system monitors** — provide similar information through a GUI.
- **`vmstat`** — reports system-wide CPU, memory, I/O, and related statistics.

---

# The `ps` Command

## Overview

`ps` stands for **process status**. It displays information about running processes as a **one-time snapshot**.

Unlike `top`, `ps` does not continuously refresh. If the system changes, run `ps` again to obtain another snapshot.

Basic usage:

```bash
ps
```

Example:

```text
 PID TTY          TIME CMD
4102 pts/0    00:00:00 bash
4231 pts/0    00:00:00 ps
```

A simple `ps` normally shows processes associated with the current terminal/session.

For continuous monitoring, use:

```bash
top
```

or an alternative such as `htop` or `btop` if installed.

---

## `ps` Option Styles

One unusual feature of `ps` is that it supports multiple option syntaxes inherited from different UNIX traditions.

The two styles you will encounter most often are:

### System V Style

Options use a leading dash:

```bash
ps -ef
ps -eLf
ps -u student
```

### BSD Style

Options do not use a leading dash:

```bash
ps aux
ps axo pid,ni,user,comm
```

These are different option conventions, not simply two ways of writing the same command.

> **Tip:** Do not write `ps -aux` by habit. The commonly used BSD form is `ps aux`, while `ps -ef` is a System V-style command.

---

# Common `ps` Commands

## Show Current Shell's Processes

```bash
ps
```

Shows processes associated with the current terminal/session.

## Show Processes for a User

```bash
ps -u <username>
```

Example:

```bash
ps -u student
```

## Show All Processes

```bash
ps -e
```

or:

```bash
ps -A
```

Both request processes from the entire system.

## Show All Processes in Full Format

A very common command is:

```bash
ps -ef
```

Example:

```text
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 09:14 ?        00:00:02 /sbin/init
root         842       1  0 09:14 ?        00:00:00 /usr/sbin/sshd -D
student     4102    4098  0 10:05 pts/0    00:00:00 bash
student     4231    4102  0 10:07 pts/0    00:00:00 ps -ef
```

Important columns include:

| Column | Meaning |
|---|---|
| `UID` | User associated with the process |
| `PID` | Process ID |
| `PPID` | Parent Process ID |
| `C` | CPU utilization-related scheduling value |
| `STIME` | Process start time |
| `TTY` | Controlling terminal |
| `TIME` | Accumulated CPU time |
| `CMD` | Command and its arguments |

The exact meaning and formatting of some fields can vary with the `ps` implementation and option set.

---

# Viewing Threads with `ps`

A process can contain multiple threads.

Use:

```bash
ps -eLf
```

The `-L` option requests thread information, resulting in a separate line for each thread.

This is useful when investigating multithreaded applications.

For example:

```text
PID   LWP   NLWP   ...
1000  1000     4   ...
1000  1001     4   ...
1000  1002     4   ...
1000  1003     4   ...
```

Here, `LWP` identifies an individual lightweight process/thread, while `NLWP` indicates the number of threads in the process.

---

# Customizing `ps` Output

You do not have to display every available column.

Use the `-o` option to select specific fields:

```bash
ps -eo pid,ppid,stat,ni,user,comm
```

Example:

```text
 PID  PPID STAT  NI USER     COMMAND
   1     0 Ss     0 root     systemd
 842     1 Ss     0 root     sshd
4102  4098 S      0 student  bash
4231  4102 R      0 student  ps
```

This is especially useful in scripts and troubleshooting because you can request exactly the information you need.

A BSD-style equivalent can also be used:

```bash
ps axo pid,ni,user,comm
```

---

# Inspecting a Specific Process

To inspect a particular PID:

```bash
ps -p <pid>
```

For example:

```bash
ps -p 4820
```

You can combine `-p` with `-o`:

```bash
ps -p 4820 -o pid,ppid,user,stat,ni,%cpu,%mem,comm
```

This provides a compact view of the process's identity, state, priority, and resource usage.

---

# Process Priority with `ps`

`ps` can display a process's priority and nice value.

For example:

```bash
ps -l
```

or:

```bash
ps -o pid,ni,pri,comm
```

The `NI` column shows the **nice value**.

Recall:

```text
Lower nice value → higher ordinary scheduling priority
Higher nice value → lower ordinary scheduling priority
```

For example:

```bash
renice +5 -p 4820
```

increases the nice value to `5`, making the process less CPU-favored under ordinary scheduling.

Changing a process's priority can require elevated privileges when increasing its scheduling priority.

---

# The `pstree` Command

## Overview

`ps` presents processes as a mostly flat list. **`pstree`** displays them as a hierarchy, making parent-child relationships easier to understand.

Run:

```bash
pstree
```

Example:

```text
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─gnome-shell───{gnome-shell}
        ├─sshd───sshd───bash───pstree
        └─systemd───(sd-pam)
```

This makes relationships such as:

```text
sshd
 └── sshd
      └── bash
           └── pstree
```

immediately visible.

### Why Use `pstree`?

It is particularly useful when you want to answer:

- Which process launched this process?
- What processes are children of a service?
- What does a process hierarchy look like?
- Which shell or service is responsible for a process?

---

## Threads in `pstree`

`pstree` can represent threads using braces:

```text
application─┬─{application}
            ├─{application}
            └─{application}
```

These entries represent threads belonging to the parent process.

`pstree` may also collapse identical sibling processes:

```text
2*[process]
```

means that two similar child entries have been collapsed into one display entry.

---

# The `top` Command

## Overview

`top` provides a **continuously updating, interactive view** of processes and system resources.

Start it with:

```bash
top
```

It periodically refreshes the display until you quit.

Press:

```text
q
```

to exit.

A useful way to remember the difference is:

```text
ps   → snapshot
top  → live monitoring
```

`top` is similar in purpose to Task Manager on Windows or Activity Monitor on macOS, but it works directly in a terminal and is available on systems without a graphical desktop.

---

# Understanding the `top` Display

A typical `top` display has:

1. A **summary area** containing system-wide statistics.
2. A **process list** containing information about individual processes.

Example:

```text
top - 14:32:07 up 3 days,  2:15,  2 users,  load average: 0.45, 0.17, 0.12
Tasks: 215 total,   1 running, 214 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.7 sy,  0.0 ni, 96.8 id,  0.2 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7842.5 total,   3120.8 free,   2450.1 used,   2271.6 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   5045.9 avail Mem
```

---

# `top` Summary Area

## First Line — Uptime and Load

Example:

```text
top - 14:32:07 up 3 days, 2:15, 2 users, load average: 0.45, 0.17, 0.12
```

It shows:

- Current time
- System uptime
- Number of logged-in users
- 1-minute load average
- 5-minute load average
- 15-minute load average

Load average represents runnable work and certain uninterruptible waits. It should be interpreted relative to the number of logical CPUs.

---

## Second Line — Task Counts

Example:

```text
Tasks: 215 total, 1 running, 214 sleeping, 0 stopped, 0 zombie
```

This summarizes processes/tasks by state.

The categories include:

- Total tasks
- Running tasks
- Sleeping tasks
- Stopped tasks
- Zombie tasks

A large number of processes is not automatically a problem. Modern systems can legitimately have hundreds or thousands of processes.

---

## Third Line — CPU Usage

Example:

```text
%Cpu(s): 2.3 us, 0.7 sy, 0.0 ni, 96.8 id, 0.2 wa, 0.0 hi, 0.0 si, 0.0 st
```

| Field | Meaning |
|---|---|
| `us` | CPU time spent running user-space processes |
| `sy` | CPU time spent running kernel/system code |
| `ni` | CPU time spent running processes with modified nice values |
| `id` | Idle CPU time |
| `wa` | Time waiting for I/O |
| `hi` | Time servicing hardware interrupts |
| `si` | Time servicing software interrupts |
| `st` | CPU time stolen from a virtual machine by the hypervisor |

For example:

```text
96.8 id
```

means the CPUs were idle for roughly 96.8% of the sampled CPU time.

A high `wa` value can indicate significant I/O waiting, although it should be investigated alongside disk and application metrics.

---

# Memory Information in `top`

The fourth line describes physical memory:

```text
MiB Mem :   7842.5 total,   3120.8 free,   2450.1 used,   2271.6 buff/cache
```

It includes information such as:

- Total RAM
- Free memory
- Used memory
- Buffer/cache memory

The fifth line describes swap:

```text
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   5045.9 avail Mem
```

It shows:

- Total swap
- Free swap
- Used swap
- Available memory

### Do Not Treat Linux `free` Memory as "Wasted"

Linux intentionally uses otherwise-unused RAM for filesystem caches and buffers.

Therefore:

```text
Low "free" memory
```

does not automatically mean:

```text
Low available memory
```

The `avail Mem` value is generally more useful for judging how much memory can be made available to applications without significant swapping.

### Swap

Swap provides disk-backed storage that can be used when memory pressure occurs.

Because storage is much slower than RAM, heavy swapping can significantly reduce performance.

However, **having swap usage does not automatically indicate a problem**. The important questions are whether the system is under memory pressure and whether active swapping is affecting performance.

---

# Reading the `top` Process List

Example:

```text
 PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
1892 student   20   0  982340 145820  78200 S   4.7   1.8   1:12.44 firefox
 842 root      20   0   72180   6420   5210 S   0.3   0.1   0:02.18 sshd
4231 student   20   0   11260   3980   3120 R   0.3   0.0   0:00.05 top
```

| Column | Meaning |
|---|---|
| `PID` | Process ID |
| `USER` | User owning the process |
| `PR` | Scheduler priority shown by `top` |
| `NI` | Nice value |
| `VIRT` | Virtual memory size |
| `RES` | Resident memory currently in RAM |
| `SHR` | Shared memory portion |
| `S` | Process state |
| `%CPU` | CPU usage |
| `%MEM` | Percentage of physical memory |
| `TIME+` | Accumulated CPU time |
| `COMMAND` | Command/process name |

### `VIRT`, `RES`, and `SHR`

These memory columns are often misunderstood.

- **VIRT** — virtual address-space size, which can include mapped files, libraries, allocated-but-not-resident memory, and other virtual mappings.
- **RES** — resident memory currently held in physical RAM.
- **SHR** — memory that may be shared with other processes.

Therefore, `VIRT` should not be interpreted as the amount of RAM the process is actually consuming.

---

# Useful `top` Interactive Keys

While `top` is running, single-key commands can change the display or perform actions.

| Key | Function |
|---|---|
| `h` / `?` | Display help |
| `P` | Sort by CPU usage |
| `M` | Sort by memory usage |
| `T` | Sort by cumulative CPU time |
| `t` | Toggle CPU/task summary display |
| `m` | Toggle memory summary display |
| `1` | Toggle individual CPU statistics |
| `d` | Change refresh interval |
| `r` | Change a process's nice value |
| `k` | Send a signal to a process |
| `q` | Quit |

Keys are **case-sensitive**.

### Find CPU-Hungry Processes

Start:

```bash
top
```

Then press:

```text
P
```

This sorts the process list by CPU usage.

### Find Memory-Hungry Processes

Press:

```text
M
```

This sorts the list by memory usage.

> **Tip:** `P` and `M` are two of the most useful `top` keys to memorize.

### View Individual CPU Cores

Press:

```text
1
```

This toggles between aggregate CPU statistics and statistics for individual logical CPUs.

This is useful when one CPU is heavily loaded while the others are relatively idle.

---

# Changing Process Priority from `top`

`top` can also change a process's nice value.

Press:

```text
r
```

and follow the prompts to select a PID and enter the new nice value.

The same permission rules that apply to `renice` apply here. An ordinary user generally cannot increase the CPU priority of another user's process or arbitrarily lower the nice value.

---

# Sending Signals from `top`

Press:

```text
k
```

to send a signal to a selected process.

You will normally be asked for:

1. The PID.
2. The signal number.

For example, signal `15` is `SIGTERM`:

```text
15
```

Signal `9` is `SIGKILL`:

```text
9
```

Prefer `SIGTERM` when possible and use `SIGKILL` only when necessary.

---

# Batch Mode with `top`

`top` can produce a non-interactive snapshot using batch mode:

```bash
top -b -n 1
```

For example, to show only the first three lines:

```bash
top -b -n 1 | head -3
```

Options:

- `-b` — batch mode, suitable for scripts/pipelines
- `-n 1` — perform one iteration

This is useful when you want to capture `top` output without entering its interactive interface.

---

# Alternatives to `top`

Several tools provide friendlier or more feature-rich process monitors.

## `htop`

`htop` provides:

- Interactive process management
- Easier scrolling
- Colorful terminal interface
- Mouse support on many terminals
- Per-CPU visualization

Install on Debian/Ubuntu:

```bash
sudo apt install htop
```

Fedora/RHEL-based systems:

```bash
sudo dnf install htop
```

openSUSE:

```bash
sudo zypper install htop
```

## `btop`

`btop` provides a modern terminal interface with graphical CPU, memory, disk, and network views.

It may not be installed by default.

## `atop`

`atop` provides detailed system and process monitoring and can be useful for performance analysis and historical monitoring when configured appropriately.

### Why Learn `top`?

The main advantage of `top` is availability. It is commonly installed by default on Linux systems, making it a dependable tool when working on a remote server.

---

# Graphical System Monitoring

Linux desktop environments often provide graphical system-monitoring applications.

For example, GNOME systems commonly provide:

```bash
gnome-system-monitor
```

KDE Plasma provides a system monitor application commonly associated with:

```bash
plasma-systemmonitor
```

A graphical system monitor can provide:

- CPU graphs
- Memory and swap graphs
- Network activity
- Filesystem usage
- Process lists
- Process termination
- Process priority controls

The exact application and command depend on the desktop environment and distribution.

---

# `vmstat`

`vmstat` provides a compact system-wide view of CPU, memory, processes, paging, and I/O activity.

A basic command is:

```bash
vmstat
```

For repeated samples:

```bash
vmstat 2
```

This reports a new sample every two seconds.

For a finite number of samples:

```bash
vmstat 2 10
```

This collects ten samples at two-second intervals.

`vmstat` is particularly useful when you want to correlate CPU activity with memory pressure, swapping, and I/O rather than focusing on individual processes.

---

# Choosing the Right Monitoring Tool

| Goal | Tool |
|---|---|
| Take a process snapshot | `ps` |
| Inspect one process | `ps -p` |
| See parent-child relationships | `pstree` |
| Monitor processes continuously | `top` |
| Quickly find CPU-heavy processes | `top` + `P` |
| Quickly find memory-heavy processes | `top` + `M` |
| Inspect individual CPU usage | `top` + `1` |
| Get a friendly interactive process monitor | `htop` / `btop` |
| Monitor system-wide CPU/memory/I/O | `vmstat` |
| Monitor from a graphical desktop | System Monitor application |

---

# Practical Troubleshooting Workflow

When a system appears slow, start with a broad view and narrow the investigation.

## 1. Check load and uptime

```bash
uptime
```

Ask:

```text
Is the load unusually high compared with the number of CPUs?
```

## 2. Check live CPU and memory activity

```bash
top
```

Press:

```text
P
```

to identify CPU-heavy processes.

Press:

```text
M
```

to identify memory-heavy processes.

## 3. Inspect a suspicious process

```bash
ps -p <pid> -o pid,ppid,user,stat,ni,%cpu,%mem,comm
```

## 4. Inspect its process hierarchy

```bash
pstree -p
```

This can reveal which service or process launched it.

## 5. Check system-wide resource behavior

```bash
vmstat 2 5
```

Look for evidence of:

- CPU saturation
- Memory pressure
- Swapping
- I/O activity

## 6. Take action only after identifying the cause

Depending on the problem, you might:

```bash
kill <pid>
```

adjust priority:

```bash
renice 10 -p <pid>
```

or investigate the parent service, disk subsystem, memory usage, or application itself.

> **Important:** Do not automatically kill the process using the most CPU. High CPU usage may be completely normal for a legitimate workload such as compilation, encoding, or scientific computation.

---

# `ps` vs. `top` vs. `pstree`

A simple comparison:

```text
                 Process monitoring
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
         ps           top          pstree
      snapshot       live view     hierarchy
          │             │             │
       details      resources      relationships
```

### `ps`

Best when you need a precise snapshot or want output suitable for scripts:

```bash
ps -eo pid,ppid,stat,ni,%cpu,%mem,comm
```

### `top`

Best when you need to watch the system as it changes:

```bash
top
```

### `pstree`

Best when you need to understand process relationships:

```bash
pstree -p
```

---

# Key Takeaways

- `ps` means **process status** and provides a point-in-time process snapshot.
- `top` provides a continuously updating process and system-resource view.
- `pstree` displays processes as a **parent-child hierarchy**.
- `ps` supports multiple option styles, including System V (`-ef`) and BSD (`aux`) syntax.
- `ps -ef` is a common way to inspect all processes in full format.
- `ps -eLf` can display individual threads.
- `ps -o` lets you choose exactly which columns to display.
- `pstree` makes process relationships easier to understand than a flat `ps` listing.
- In `top`, use `P` to sort by CPU and `M` to sort by memory.
- Press `1` in `top` to inspect individual CPU statistics.
- `top -b -n 1` is useful for non-interactive snapshots and scripts.
- `VIRT` in `top` is not the same as physical RAM usage; `RES` is more directly related to resident memory.
- Linux's low `free` memory value is not automatically a problem because the kernel uses RAM for caches.
- `vmstat` provides a compact system-wide view of CPU, memory, paging, and I/O activity.
- Use multiple metrics together when troubleshooting performance rather than relying on a single number.