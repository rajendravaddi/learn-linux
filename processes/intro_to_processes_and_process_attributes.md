# Linux Processes

## What Is a Process?

A **process** is a running instance of a program. When you start a program, the operating system creates a process to execute it and gives that process its own execution context and resources.

A process can contain one or more **threads**. Threads are execution units within a process that share the process's memory and other resources.

A **program** is executable code stored on disk, while a **process** is that program while it is running.

For example:

```bash
firefox
```

The `firefox` executable is a program. When launched, Linux creates one or more processes to run it.

Processes use resources such as:

- CPU time
- Memory
- Files and file descriptors
- Network connections
- Devices
- Other kernel-managed resources

The Linux kernel manages these resources and schedules runnable tasks so multiple programs can share the system.

> **Key idea:** Program = code stored on disk. Process = running instance of that program.

---

## Process Types

Processes can be classified by how they interact with users and the system.

| Type | Description | Examples |
|---|---|---|
| **Interactive** | Started directly or indirectly by a user and typically interacts with a terminal or graphical interface. | `bash`, `firefox`, `top`, LibreOffice |
| **Batch** | Performs work automatically, often according to a schedule, without requiring interactive input. | `updatedb`, maintenance scripts, log rotation |
| **Daemon** | A background service that waits for requests or events and may start during system boot or on demand. | `sshd`, `cupsd`, `systemd-resolved` |
| **Thread** | An execution unit inside a process. Threads share the process's memory and resources but can be scheduled independently. | Threads inside browsers, servers, and desktop applications |
| **Kernel thread** | A kernel-managed task used for low-level operating-system work. | `kthreadd`, `ksoftirqd`, `migration` |

### Interactive Processes

An interactive process is associated with user activity. It may be started from a terminal:

```bash
top
```

or through a graphical interface.

It can remain attached to the terminal or be placed in the background.

### Batch Processes

Batch processes perform work automatically rather than waiting for direct user interaction. They are commonly scheduled with tools such as **cron** or **systemd timers**.

Batch processing does **not** inherently mean FIFO ordering; the scheduling system determines when and how jobs run.

### Daemons

A **daemon** is a background service that normally waits for requests, events, or other work.

Examples:

```text
sshd              → accepts SSH connections
cupsd             → manages printing
systemd-resolved  → provides DNS-related services
```

Many daemons start during system boot, although some are started on demand.

### Threads

A process can contain multiple threads.

Threads belonging to the same process normally share:

- Address space
- Open files
- Other process resources

Each thread nevertheless has its own execution state and scheduling identity.

For example, a web server might use multiple threads to handle client requests concurrently.

### Kernel Threads

**Kernel threads** are execution tasks created and managed by the Linux kernel rather than ordinary user applications.

They perform internal work such as kernel housekeeping, scheduling-related operations, and I/O-related tasks.

They may appear in process listings with names such as:

```text
[kthreadd]
[ksoftirqd/0]
[migration/0]
```

---

## Process Scheduling and States

A CPU core can execute only one thread at a time. Modern systems have multiple cores, so several threads can execute simultaneously, but there can still be more runnable tasks than available CPU cores.

The Linux **scheduler** decides which runnable tasks should execute on each CPU.

A task can broadly be:

- **Running/runnable** — currently executing or ready to execute.
- **Sleeping** — waiting for an event or resource.
- **Stopped** — suspended and not currently eligible to run.
- **Zombie** — finished execution but still has a process-table entry until its parent collects its exit status.

The kernel maintains scheduling structures such as **run queues** for runnable tasks. A task waiting for I/O or another event can sleep instead of consuming CPU time.

### Process States with `ps`

Use:

```bash
ps -eo pid,ppid,stat,comm
```

Example:

```text
 PID   PPID STAT COMMAND
   1      0 Ss   systemd
 842      1 Ss   sshd
1290   1187 R+   ps
1305   1187 S    sleep
```

Primary process states:
| State | Name                  | Meaning                                                                                                              |
| ----- | --------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `R`   | Running / Runnable    | The process is currently running on a CPU or is ready and waiting to be scheduled.                                   |
| `S`   | Interruptible Sleep   | The process is sleeping and waiting for an event; it can normally be interrupted by signals.                         |
| `D`   | Uninterruptible Sleep | The process is waiting for a condition that generally cannot be interrupted, commonly certain kernel I/O operations. |
| `T`   | Stopped               | The process has been stopped, usually by a job-control signal such as `SIGSTOP` or `SIGTSTP`.                        |
| `t`   | Tracing Stop          | The process is stopped by a debugger or tracer such as `ptrace`.                                                     |
| `Z`   | Zombie                | The process has finished execution, but its parent has not yet collected its exit status.                            |
| `X`   | Dead                  | The process is dead. This state is normally not visible in `ps` output because it exists only briefly.               |
| `I`   | Idle Kernel Thread    | An idle kernel thread. This state is mainly seen for kernel threads.                                                 |


ps STAT modifiers

The STAT column can contain additional characters after the primary state. For example, Ss, R+, or Sl.
| Modifier | Meaning                                                              |
| -------- | -------------------------------------------------------------------- |
| `<`      | High-priority process                                                |
| `N`      | Low-priority process (nice value is positive)                        |
| `L`      | Process has pages locked in memory                                   |
| `s`      | Session leader                                                       |
| `l`      | Multi-threaded process                                               |
| `+`      | Process is in the foreground process group                           |
| `C`      | Process has a significant amount of CPU usage (Linux `ps` modifier)  |
| `E`      | Process is running in an elevated scheduling class (where supported) |


### Zombie Processes

A zombie is a process that has **already finished execution**, but its parent has not yet collected its exit status.

A zombie is not actively executing and does not consume CPU time. It remains in the process table so its parent can retrieve its termination status.

Normally, the parent eventually performs a `wait()`-family operation and the zombie disappears.

> A zombie is not a process that is still running. It is a completed process whose process-table entry has not yet been cleaned up.

---

## Process IDs

Linux identifies processes using numeric identifiers.

### PID — Process ID

A **PID (Process ID)** identifies a process during its lifetime.

For example:

```bash
pgrep firefox
```

might return:

```text
4820
```

PIDs are commonly allocated from an increasing range, but you should **not assume they are always strictly sequential**. After a process exits, its PID can eventually be reused.

### PPID — Parent Process ID

The **PPID (Parent Process ID)** identifies the process that created the current process.

For example:

```text
bash
 ├── firefox
 └── sleep
```

If Bash launches `sleep`, the `sleep` process normally has Bash as its parent.

Inspect both IDs with:

```bash
ps -o pid,ppid,comm
```

### Orphan Processes

If a parent exits while its child is still running, the child becomes an **orphan**.

Linux reparents the orphan to an appropriate process. Traditionally, PID 1 (`init`/`systemd`) adopts orphaned processes, although Linux also supports **subreapers**, which can adopt descendants before PID 1 does.

---

## PID 1 and `systemd`

On many modern Linux distributions, including typical Ubuntu installations, **PID 1 is `systemd`**.

PID 1 has special responsibilities, including:

- Starting and managing system services
- Reaping orphaned processes when appropriate
- Participating in system startup and shutdown

Check PID 1 with:

```bash
ps -p 1 -o pid,comm,args
```

The exact command line can differ between distributions and configurations.

---

## Process and Thread IDs

A process may contain multiple threads.

| Identifier | Meaning |
|---|---|
| **PID** | Process/thread-group identifier as commonly shown for a process |
| **PPID** | Parent process identifier |
| **TID** | Identifier for an individual thread |

In a single-threaded process, the main thread's TID and the process PID commonly have the same numeric value.

In a multithreaded process, individual threads have distinct TIDs while belonging to the same thread group and sharing the process's address space and other resources.

To inspect threads:

```bash
ps -eLf
```

---

## Terminating Processes

Linux provides the `kill` command to send a **signal** to a process.

Despite its name, `kill` does not necessarily mean "terminate immediately." It can send many different signals.

The default signal is `SIGTERM`:

```bash
kill <pid>
```

For example:

```bash
kill 4820
```

`SIGTERM` asks the process to terminate and gives it an opportunity to perform cleanup.

### `SIGKILL`

If a process does not terminate normally, you can use `SIGKILL`:

```bash
kill -9 4820
```

or:

```bash
kill -SIGKILL 4820
```

`SIGKILL` cannot be caught or ignored by the target process. The kernel terminates the process.

### Prefer `SIGTERM` First

A good general workflow is:

```bash
kill <pid>
```

If the process still does not terminate after a reasonable amount of time:

```bash
kill -9 <pid>
```

Avoid using `SIGKILL` by default because the process cannot perform normal signal-handling cleanup.

### Finding a Process

For example:

```bash
pgrep firefox
```

You can also inspect processes with:

```bash
ps aux
```

Normally, users can signal processes they own. Signaling processes owned by another user generally requires appropriate privileges such as `root` or `sudo`.

---

## User and Group IDs

Linux associates each process with user and group credentials. These credentials are important for determining what the process is allowed to access.

### User IDs

Two important user IDs are:

- **Real User ID (RUID)** — identifies the user associated with the process.
- **Effective User ID (EUID)** — used for many permission checks.

For ordinary processes, these are usually the same.

They can differ for programs using mechanisms such as **set-user-ID (setuid)**, which allow a carefully designed program to run with a different effective user identity.

> The classic `passwd` example is based on setuid behavior, but the exact implementation can vary by distribution and version. Do not assume every `passwd` installation permanently runs with EUID 0.

### Group IDs

Processes also have group credentials, including:

- **Real Group ID (RGID)**
- **Effective Group ID (EGID)**
- Supplementary group IDs

These group memberships participate in Linux permission checks.

Check your IDs with:

```bash
id
```

Example:

```text
uid=1000(student) gid=1000(student) groups=1000(student),27(sudo),4(adm)
```

---

## Process Priorities and Niceness

When several tasks are runnable, Linux's scheduler determines which tasks should receive CPU time.

For ordinary scheduling, Linux uses a process attribute called **niceness**.

The traditional nice-value range is:

```text
-20  → highest priority
  0  → default
+19  → lowest priority
```

The relationship is intentionally counterintuitive:

> **Lower nice value = higher ordinary scheduling priority.**

For example:

```text
nice -10   → higher priority
nice   0   → normal/default
nice +15   → lower priority
```

A process with a high nice value is being "nice" to other processes by yielding CPU preference.

### Starting a Process with a Nice Value

Use:

```bash
nice -n 10 ./my-program
```

### Changing an Existing Process

Use:

```bash
renice 10 -p <pid>
```

For example:

```bash
renice 10 -p 4820
```

Inspect nice values with:

```bash
ps -o pid,ni,comm
```

The `NI` column contains the nice value.

### Permissions

An unprivileged user can generally increase the nice value of their own processes, making them less CPU-favored.

Lowering the nice value, especially into the negative range, requires appropriate privileges such as `CAP_SYS_NICE` and is commonly performed with `sudo`.

Example:

```bash
sudo renice -5 -p 4820
```

---

## `nice` vs. `renice`

| Command | Purpose |
|---|---|
| `nice` | Start a new process with a specified nice value |
| `renice` | Change the nice value of an existing process |

Example:

```bash
nice -n 10 ./job
```

starts a new job with a higher nice value.

```bash
renice 10 -p 4820
```

changes the nice value of an existing process.

---

## Real-Time Scheduling

Linux also provides **real-time scheduling policies** for workloads that need stronger scheduling guarantees than ordinary processes.

Real-time scheduling can be useful for:

- Audio processing
- Robotics
- Industrial workloads
- Low-latency data processing

Real-time scheduling priorities are separate from ordinary nice values.

However, standard Linux real-time scheduling should **not** automatically be interpreted as a hard real-time guarantee. A general-purpose Linux system does not guarantee that every real-time task will always meet an absolute deadline.

Specialized real-time systems and appropriately configured kernels provide stronger guarantees for hard real-time requirements.

---

## Inspecting Processes with `ps`

`ps` is one of the most useful commands for inspecting running processes.

A simple:

```bash
ps
```

usually shows processes associated with the current terminal/session.

For more detail:

```bash
ps -f
```

For all processes:

```bash
ps -e
```

A useful custom format is:

```bash
ps -eo pid,ppid,stat,ni,comm
```

This displays:

- `pid` — process ID
- `ppid` — parent process ID
- `stat` — process state
- `ni` — nice value
- `comm` — command name

A commonly used broader view is:

```bash
ps aux
```

`ps` supports several option syntaxes inherited from different UNIX conventions, so its options can initially look unusual. For example:

```bash
ps -e
```

and:

```bash
ps e
```

are not equivalent.

When unsure, consult:

```bash
man ps
```

---

## Practical Process Investigation

When a program consumes too much CPU, uses excessive memory, or becomes unresponsive, a useful workflow is:

### 1. Find the process

```bash
pgrep <name>
```

### 2. Inspect it

```bash
ps -p <pid> -o pid,ppid,user,stat,ni,%cpu,%mem,comm
```

### 3. Try normal termination

```bash
kill <pid>
```

### 4. Force termination if necessary

```bash
kill -9 <pid>
```

### 5. Investigate its parent

```bash
ps -p <pid> -o pid,ppid,comm
```

This can help determine whether another process launched or supervises it.

---

## Key Takeaways

- A **program** is executable code; a **process** is a running instance of that code.
- A process can contain one or more **threads**.
- The Linux kernel manages process resources and scheduling.
- **PID** identifies a process; **PPID** identifies its parent.
- **PID 1** has special system-level responsibilities and is commonly `systemd` on modern Linux distributions.
- Processes can be **running, sleeping, stopped, or zombie**.
- `ps` is a fundamental process-inspection tool.
- `kill` sends signals; it does not inherently mean "force kill."
- Prefer **`SIGTERM`** before **`SIGKILL`**.
- User and group credentials affect process permissions.
- **Lower nice values mean higher ordinary scheduling priority.**
- `nice` sets the nice value when starting a process; `renice` changes it for an existing process.
- Real-time scheduling is separate from ordinary niceness and does not automatically provide hard real-time guarantees.