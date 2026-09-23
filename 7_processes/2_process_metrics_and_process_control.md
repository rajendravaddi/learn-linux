# Process Metrics and Process Control

## Load Averages

### Overview

A Linux system's **load average** indicates how many tasks are demanding CPU time or are waiting in certain kernel wait states.

Linux includes tasks that are:
- **Running/runnable (`R`)** — currently executing or ready to execute on a CPU.
- **Uninterruptible sleep (`D`)** — waiting for certain kernel resources, commonly I/O.

Load average is reported over three time windows:

```text
1 minute, 5 minutes, 15 minutes
```

View it with:

```bash
uptime
```

Example:

```text
15:59:35 up 8:13, 1 user, load average: 2.51, 2.55, 2.58
```

> **Important:** Load average is a measure of tasks competing for CPU or stuck in certain uninterruptible waits. It is **not a CPU-utilization percentage**.

### Understanding Load on a Single-Core System

Think of one CPU core as one checkout lane:

```text
Load ≈ 1.0  → roughly one runnable task per CPU
Load < 1.0  → generally spare CPU capacity
Load > 1.0  → runnable work can form a queue
```

For example:

```text
0.5 → relatively light load
1.0 → approximately one runnable task
2.0 → approximately two runnable tasks
```

This is an analogy, not a literal utilization percentage.

### Understanding Load on Multi-Core Systems

Compare load average with the number of logical CPUs.

For example, on a 4-CPU system:

```text
Load average = 4.0
Logical CPUs = 4
```

This means there is, on average, enough runnable work to keep four CPUs busy.

A rough normalized value is:

```text
normalized load = load average / number of logical CPUs
```

So:

```text
4.0 / 4 = 1.0
```

Treat this only as a rough capacity indicator.

| Load | Logical CPUs | Rough interpretation |
|---:|---:|---|
| `0.5` | 4 | Very light CPU demand |
| `2.0` | 4 | Generally spare CPU capacity |
| `4.0` | 4 | Around one runnable task per CPU |
| `8.0` | 4 | Significant runnable queue |

### High Load Does Not Always Mean High CPU Usage

Linux includes certain `D`-state tasks when calculating load average.

Therefore, a system can have:

```text
High load average
+
Low CPU utilization
```

This can happen when many tasks are blocked waiting for I/O or another uninterruptible kernel operation.

For example, severe storage latency can produce a high load average even when CPUs are not fully utilized.

> **Rule of thumb:** Use load average together with CPU, memory, disk-I/O, and other metrics. Never diagnose a performance problem from load average alone.

### Viewing Load Averages

```bash
uptime
```

```bash
w
```

```bash
top
```

`uptime` gives a concise summary, `w` adds logged-in user information, and `top` provides a continuously updating process/resource view.

---

# Background and Foreground Processes

A process started from a shell can run in the **foreground** or **background**.

## Foreground

A foreground job is associated with the terminal and normally occupies the shell until it finishes or is suspended:

```bash
sleep 500
```

The shell waits for it before displaying another prompt.

## Background

Append `&` to start a command as a background job:

```bash
sleep 500 &
```

Example:

```text
[1] 4820
```

Typically:
- `[1]` is the shell's job ID.
- `4820` is the process ID (PID).

The shell then displays a new prompt.

> **Important:** `&` puts the job into the shell's background job-control system. It does **not** automatically detach it from the terminal or guarantee that it survives logout.

For persistent work, use tools such as `nohup`, `disown`, `tmux`, or a service manager such as `systemd`, depending on the use case.

### Backgrounding Does Not Change Priority

This:

```bash
updatedb &
```

does not automatically lower CPU priority.

If you also want lower priority:

```bash
nice -n 10 updatedb &
```

---

# Controlling Foreground Jobs

## `Ctrl-C` — Interrupt

Press:

```text
Ctrl-C
```

to normally send `SIGINT` to the foreground job.

Unlike `SIGKILL`, `SIGINT` can be handled by the application.

## `Ctrl-Z` — Suspend

Press:

```text
Ctrl-Z
```

to suspend the foreground job.

Example:

```text
^Z
[1]+  Stopped    sleep 500
```

The process has not been terminated and can be resumed.

## `bg` — Resume in Background

```bash
bg
```

or:

```bash
bg %1
```

resumes a stopped job in the background.

## `fg` — Bring to Foreground

```bash
fg
```

or:

```bash
fg %1
```

brings a background or stopped job to the foreground.

---

# Shell Job IDs

Shell job IDs are different from PIDs.

```text
[1]+ 4820 Running  sleep 500 &
```

Here:

```text
[1]  → shell job ID
4820 → process ID
```

Use `%` when referring to a job ID:

```bash
fg %1
bg %1
kill %1
```

The job ID is meaningful to the current shell; the PID identifies the underlying process.

---

# The `jobs` Command

List jobs managed by the current shell:

```bash
jobs
```

Example:

```text
[1]-  Running  updatedb &
[2]+  Stopped  sleep 500
```

The symbols mean:

- `+` — current/default job
- `-` — previous job

Include PIDs with:

```bash
jobs -l
```

Example:

```text
[1]- 4820 Running  updatedb &
[2]+ 4855 Stopped  sleep 500
```

> **Important:** `jobs` only shows jobs known to the **current shell**. Another terminal normally has a different job list.

---

# Practical Job-Control Example

Start a long-running command:

```bash
sleep 500
```

Suspend it:

```text
Ctrl-Z
```

Check it:

```bash
jobs -l
```

Resume it in the background:

```bash
bg %1
```

Bring it back to the foreground:

```bash
fg %1
```

Suspend it again:

```text
Ctrl-Z
```

Terminate it:

```bash
kill %1
```

The basic lifecycle is:

```text
Foreground
    │
    │ Ctrl-Z
    ↓
Stopped
    │
    │ bg
    ↓
Background
    │
    │ fg
    ↓
Foreground
    │
    │ Ctrl-Z / kill
    ↓
Stopped / Terminated
```

---

# Viewing Load and Process Information

## `uptime`

```bash
uptime
```

Example:

```text
15:59:35 up 8:13, 1 user, load average: 0.51, 0.55, 0.58
```

It reports:
- Current time
- System uptime
- Number of logged-in users
- 1-minute load average
- 5-minute load average
- 15-minute load average

## `top`

```bash
top
```

`top` provides a continuously updating process and system view. Press `q` to quit.

For a single non-interactive snapshot:

```bash
top -b -n 1 | head -3
```

Here:
- `-b` — batch mode
- `-n 1` — one iteration

Example:

```text
top - 15:59:37 up 8:13, 1 user, load average: 0.51, 0.55, 0.58
Tasks: 403 total, 1 running, 402 sleeping, 0 stopped, 0 zombie
%Cpu(s): 1.2 us, 0.5 sy, 0.0 ni, 98.1 id, 0.2 wa, 0.0 hi, 0.0 si, 0.0 st
```

Important CPU fields include:

| Field | Meaning |
|---|---|
| `us` | User-space CPU time |
| `sy` | Kernel/system CPU time |
| `ni` | CPU time used by niced processes |
| `id` | Idle CPU time |
| `wa` | I/O wait time |
| `hi` | Hardware interrupt handling |
| `si` | Software interrupt handling |
| `st` | Time stolen by a hypervisor |

## `w`

```bash
w
```

`w` combines load information with logged-in user activity.

---

# Graphical Applications and Job Control

Graphical applications can also be launched from a terminal.

For example, on systems with GNOME Calculator:

```bash
gnome-calculator
```

The terminal remains occupied while the application is in the foreground.

Suspend it:

```text
Ctrl-Z
```

Resume it in the background:

```bash
bg
```

Bring it back:

```bash
fg
```

The exact graphical application available depends on the distribution and desktop environment.

---

# Process Control vs. Job Control

These concepts are related but different.

### Process control

Deals with operating-system processes and signals:

```bash
ps
kill
pgrep
renice
```

### Job control

Deals with jobs managed by an interactive shell:

```bash
jobs
bg
fg
Ctrl-Z
```

For example:

```bash
sleep 500 &
```

creates a process and registers it as a background job in the current shell.

Job control is therefore a shell-level abstraction over the underlying processes.

---

# Useful Commands at a Glance

| Command / Shortcut | Purpose |
|---|---|
| `uptime` | Show uptime and load averages |
| `w` | Show uptime, load, and logged-in users |
| `top` | Monitor processes and system activity |
| `ps` | Inspect processes |
| `jobs` | List jobs managed by the current shell |
| `jobs -l` | List shell jobs with PIDs |
| `command &` | Start a command as a background job |
| `Ctrl-C` | Send `SIGINT` to the foreground job |
| `Ctrl-Z` | Suspend the foreground job |
| `bg` | Resume a stopped job in the background |
| `fg` | Bring a job to the foreground |
| `kill <pid>` | Send a signal to a process |
| `nice` | Start a process with a specified nice value |
| `renice` | Change the nice value of an existing process |

---

# Key Takeaways

- **Load average** measures runnable work and certain uninterruptible waits; it is not directly a CPU-utilization percentage.
- Linux reports load averages for **1, 5, and 15 minutes**.
- Compare load with the number of **logical CPUs**, rather than assuming `1.0` always means 100% utilization.
- A high load average can result from **I/O bottlenecks**, not only CPU saturation.
- `uptime`, `w`, and `top` can show load averages.
- Adding `&` starts a command as a **background job** in the current shell.
- Backgrounding does not automatically lower priority or make a process survive terminal closure.
- `Ctrl-C` normally sends `SIGINT` to the foreground job.
- `Ctrl-Z` suspends the foreground job.
- `bg` resumes a stopped job in the background.
- `fg` brings a background/stopped job to the foreground.
- `jobs` shows jobs managed by the current shell.
- **Job IDs and PIDs are different identifiers.**
- Use process tools such as `ps`, `pgrep`, and `kill` for process-level operations, and `jobs`, `bg`, and `fg` for shell job control.