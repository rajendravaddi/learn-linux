# Scheduling Future Tasks

Linux provides several mechanisms for running commands automatically in the future. The right tool depends on whether the task:

- Runs **once** or **repeatedly**
- Needs an **exact clock time** or only a periodic interval
- Must **catch up after the machine was powered off**
- Simply needs to **pause before continuing**

The main tools are:

| Tool | Purpose |
|---|---|
| `at` | Run a one-time job at a future time |
| `cron` | Run recurring jobs at specific times |
| `anacron` | Run periodic jobs even when scheduled times were missed because the machine was off |
| `sleep` | Pause a command for a specified duration |
| systemd timers | Modern scheduling mechanism for systemd-based systems |

---

# One-Time Scheduling with `at`

## What is `at`?

`at` schedules a command or script to run **once at a specified future time**.

For example:

```bash
at 10:00 AM Jul 15
```

After running the command, `at` opens an interactive prompt:

```text
warning: commands will be executed using /bin/sh
at>
```

Enter the command:

```text
at> /usr/local/bin/backup.sh
```

Press:

```text
Ctrl-D
```

to finish and submit the job.

You will receive output similar to:

```text
job 3 at Tue Jul 15 10:00:00 2026
```

The job is now queued and will execute once at the specified time.

> `at` normally executes queued commands using `/bin/sh`, so scripts should explicitly specify their interpreter when appropriate.

---

## Useful `at` Commands

### List Pending Jobs

```bash
atq
```

Example:

```text
17  Tue Jul 15 08:55:00 2026 a student
18  Tue Jul 15 09:02:00 2026 a student
```

### Remove a Scheduled Job

```bash
atrm <job-id>
```

Example:

```bash
atrm 17
```

---

# Scheduling Relative to the Current Time

`at` also accepts relative times.

For example:

```bash
at now + 1 minute
```

or:

```bash
at now + 2 hours
```

This is useful for testing or when you care about a delay rather than a particular clock time.

You can also use the `-f` option to read commands from a script file:

```bash
at now + 1 minute -f testat.sh
```

---

# How `at` Works

`at` does not execute the job by itself. A background daemon called **`atd`** processes the queue and launches jobs when their scheduled time arrives.

Check the service:

```bash
systemctl status atd
```

Enable and start it:

```bash
sudo systemctl enable --now atd
```

On systems where the `at` package is not installed, install it first.

### Debian/Ubuntu

```bash
sudo apt install at
```

### Fedora/RHEL/CentOS Stream

```bash
sudo dnf install at
```

### openSUSE

```bash
sudo zypper install at
```

Then:

```bash
sudo systemctl enable --now atd
```

> A job can be successfully queued while `atd` is not running, but it will not actually execute until the daemon is available.

---

# `at` and the Working Directory

A scheduled job does not necessarily run with the same interactive environment you had when scheduling it.

For reliable scripts:

- Use absolute paths where practical.
- Do not assume the current working directory.
- Do not assume interactive shell configuration has been loaded.
- Set required environment variables explicitly.
- Redirect important output to a known location.

For example:

```bash
at now + 5 minutes
at> /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

---

# Output from `at` Jobs

If a scheduled command produces output and you do not redirect it, `at` attempts to mail that output to the user who scheduled the job.

On systems with local mail configured, the output may be available through the user's local mail spool.

For example:

```bash
cat /var/mail/$USER
```

However, many modern desktop and minimal installations do not have local mail delivery configured.

Therefore, for scripts where the output matters, explicitly redirect it:

```bash
at now + 1 minute
at> date > /tmp/datestamp
at> Ctrl-D
```

or:

```bash
at now + 1 minute -f testat.sh
```

where `testat.sh` contains:

```bash
#!/bin/bash

date > /tmp/datestamp
```

---

# Recurring Scheduling with `cron`

## What is `cron`?

`cron` is designed for **repeating jobs**.

Typical examples include:

- Running backups every night
- Cleaning temporary files every week
- Running reports every Monday
- Performing maintenance at a fixed time
- Running a script every few minutes

The cron daemon continuously checks scheduled entries and launches commands when their schedule matches the current time.

---

# User Crontabs

To edit your personal crontab:

```bash
crontab -e
```

To display it:

```bash
crontab -l
```

A typical user crontab entry has five time fields followed by the command:

```text
MIN HOUR DOM MON DOW COMMAND
```

---

# Cron Field Syntax

| Field | Meaning | Typical values |
|---|---|---|
| `MIN` | Minute | `0-59` |
| `HOUR` | Hour | `0-23` |
| `DOM` | Day of month | `1-31` |
| `MON` | Month | `1-12` |
| `DOW` | Day of week | `0-7` |
| `COMMAND` | Command to execute | Any valid command |

For day of week:

```text
0 = Sunday
7 = Sunday
1 = Monday
...
6 = Saturday
```

---

# The `*` Wildcard

An asterisk means **every applicable value**.

For example:

```text
* * * * * command
```

means:

```text
Every minute
of every hour
of every day
of every month
regardless of weekday
```

Therefore:

```bash
* * * * * /usr/local/bin/script.sh
```

runs the script every minute.

---

# Cron Examples

## Every Minute

```cron
* * * * * /usr/local/bin/script.sh
```

## Every Day at 10:00 AM

```cron
0 10 * * * /usr/local/bin/script.sh
```

The fields mean:

```text
0    → minute 0
10   → hour 10
*    → every day of month
*    → every month
*    → every weekday
```

Therefore:

```text
10:00 AM every day
```

## Every Sunday at Midnight

```cron
0 0 * * 0 /usr/local/bin/backup.sh
```

## At 8:30 AM on June 10

```cron
30 8 10 6 * /home/sysadmin/full-backup
```

This runs at 08:30 on June 10.

---

# Understanding Cron's Schedule

A useful mental model is:

```text
┌──── minute
│ ┌─── hour
│ │ ┌── day of month
│ │ │ ┌─ month
│ │ │ │ ┌ day of week
│ │ │ │ │
* * * * * command
```

Because the syntax is compact, complex expressions can be difficult to read.

For example:

```cron
*/15 * * * * /path/to/script.sh
```

means:

```text
Every 15 minutes
```

Another example:

```cron
0 9-17 * * 1-5 /path/to/script.sh
```

means:

```text
At the beginning of every hour from 09:00 through 17:00,
Monday through Friday.
```

---

# Cron Services Across Distributions

The cron implementation and service name can differ between distributions.

### Ubuntu/Debian

The service is commonly called:

```text
cron
```

Check it with:

```bash
systemctl status cron
```

### RHEL/Fedora/CentOS Stream

The implementation is commonly **cronie**, with the service:

```text
crond
```

Check it with:

```bash
systemctl status crond
```

Install if necessary:

```bash
sudo dnf install cronie
```

Enable and start:

```bash
sudo systemctl enable --now crond
```

### openSUSE

The service is also commonly:

```text
cron
```

or may depend on the installed cron implementation and distribution version. Check the available service with:

```bash
systemctl list-unit-files | grep -E 'cron|crond'
```

If `cronie` is required:

```bash
sudo zypper install cronie
```

> Always check the service name on the specific distribution rather than assuming every Linux system uses the same name.

---

# System-Wide Cron Configuration

Cron jobs can come from multiple locations.

Common locations include:

```text
/etc/crontab
/etc/cron.d/
```

There are also directories commonly used for periodic system jobs:

```text
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```

Personal jobs are managed through:

```bash
crontab -e
```

and are stored by the cron system rather than being edited directly in `/etc`.

---

# Cron Environment

One of the most common sources of cron problems is assuming that a cron job has the same environment as an interactive terminal.

A cron job may have:

- A limited `PATH`
- Different environment variables
- No interactive terminal
- A different working directory
- No shell startup files such as `.bashrc`

For reliable jobs, use absolute paths.

Instead of:

```cron
0 10 * * * backup.sh
```

prefer something like:

```cron
0 10 * * * /usr/local/bin/backup.sh
```

Inside scripts, also use absolute paths for important commands when necessary.

---

# Capturing Cron Output

If a cron command produces output, cron traditionally attempts to send that output as mail to the job owner.

For reliable logging, redirect output explicitly:

```cron
0 10 * * * /tmp/myjob.sh >> /tmp/myjob.log 2>&1
```

Here:

```text
>>       append standard output
2>&1     redirect standard error to the same destination
```

This gives you a persistent log that can be inspected with:

```bash
cat /tmp/myjob.log
```

or:

```bash
tail -f /tmp/myjob.log
```

For production systems, prefer an appropriate log location rather than `/tmp`.

---

# `anacron`

## Why `cron` Is Not Enough for Some Machines

Traditional cron assumes the machine is running when the scheduled time arrives.

Suppose you configure:

```cron
0 10 * * * /usr/local/bin/backup.sh
```

but the laptop is powered off at 10:00 AM.

That particular cron execution is missed.

Cron does **not** automatically say:

```text
"The machine is back at 2:00 PM, so I should run yesterday's 10:00 AM job now."
```

It simply missed the scheduled occurrence.

---

# What `anacron` Does

`anacron` is designed for **periodic jobs** where exact clock time is less important than ensuring that the job eventually runs.

It is particularly useful for machines that are not powered on continuously, such as:

- Laptops
- Desktop workstations
- Intermittently powered systems

Typical periodic intervals are:

- Daily
- Weekly
- Monthly

If a periodic job was missed because the machine was off, `anacron` can run it after the machine becomes available.

Its configuration is commonly:

```text
/etc/anacrontab
```

---

# `cron` vs `anacron`

The key difference is the scheduling model.

### `cron`

Think:

```text
"Run this at exactly 10:00 AM."
```

If the machine is off at 10:00 AM, that occurrence is normally missed.

### `anacron`

Think:

```text
"Make sure this periodic task runs approximately once per day/week/month."
```

If the machine was unavailable, it can catch up when it becomes available.

Therefore:

```text
Exact time required → cron
Periodic execution with catch-up → anacron
```

> `anacron` is not a general replacement for cron. It is intended for periodic jobs where an exact clock time is not essential.

---

# systemd Timers

Modern Linux distributions that use systemd also provide **systemd timers**.

A systemd timer can schedule a corresponding service unit.

They provide features that can be useful for more complex scheduling, including:

- Integration with systemd services
- Logging through the journal
- Dependency management
- Resource and service controls
- Calendar-based scheduling
- Monotonic timers
- Optional handling of missed runs through timer configuration such as `Persistent=true`

Useful commands include:

```bash
systemctl list-timers
```

and:

```bash
systemctl status <timer-name>
```

View service logs with:

```bash
journalctl -u <service-name>
```

For simple portable jobs, cron remains easy to understand and widely supported. For system services and modern systemd-based deployments, systemd timers are often a better fit.

---

# Pausing Execution with `sleep`

## What is `sleep`?

`sleep` does not schedule an independent background job.

Instead, it **pauses the current shell or process for a specified duration**.

Syntax:

```bash
sleep NUMBER[SUFFIX]
```

Common suffixes:

| Suffix | Meaning |
|---|---|
| `s` | Seconds |
| `m` | Minutes |
| `h` | Hours |
| `d` | Days |

If no suffix is supplied, seconds are assumed.

Examples:

```bash
sleep 30
```

Wait 30 seconds.

```bash
sleep 5m
```

Wait 5 minutes.

```bash
sleep 2h
```

Wait 2 hours.

---

# Using `sleep` in Scripts

A common use is retry logic:

```bash
#!/bin/bash

echo "Trying operation..."
some_command

sleep 30

echo "Trying again..."
some_command
```

More realistically:

```bash
while ! curl -fsS http://localhost:8080/health; do
    echo "Service not ready; retrying in 5 seconds..."
    sleep 5
done

echo "Service is ready."
```

Here, `sleep` pauses the loop between attempts.

---

# `sleep` vs `at`

These commands solve different problems.

### `sleep`

Delays the **current execution** for a duration:

```bash
sleep 10m
```

Meaning:

```text
Wait 10 minutes, then continue.
```

### `at`

Schedules a **separate one-time job** for a future time:

```bash
at 3:00 PM
```

Meaning:

```text
Run this job at 3:00 PM.
```

A simple distinction:

```text
sleep → duration-based delay
at    → clock-time-based one-off scheduling
```

---

# Scheduling Tools at a Glance

| Utility | Best used for | Example |
|---|---|---|
| `at` | One-time job at a future time | Run a script at 3 PM |
| `cron` | Repeating jobs at specific clock times | Backup every Sunday at midnight |
| `anacron` | Periodic jobs that should catch up after downtime | Weekly maintenance on a laptop |
| `systemd timer` | Integrated scheduling for systemd services | Run a maintenance service periodically |
| `sleep` | Pause current execution | Wait 30 seconds before retrying |

---

# Practical Lab: Schedule a One-Time Task with `at`

## 1. Install `at`

Ubuntu/Debian:

```bash
sudo apt install at
```

Fedora/RHEL/CentOS Stream:

```bash
sudo dnf install at
```

openSUSE:

```bash
sudo zypper install at
```

## 2. Start `atd`

```bash
sudo systemctl enable --now atd
```

Verify:

```bash
systemctl status atd
```

---

## Method 1: Schedule a Script

Create:

```bash
testat.sh
```

with:

```bash
#!/bin/bash

date > /tmp/datestamp
```

Then:

```bash
chmod +x testat.sh
```

Schedule it:

```bash
at now + 1 minute -f testat.sh
```

Example output:

```text
warning: commands will be executed using /bin/sh

job 17 at Tue Jul 15 08:55:00 2026
```

Check the queue:

```bash
atq
```

After the scheduled time:

```bash
cat /tmp/datestamp
```

You should see the date and time at which the script executed.

---

# Method 2: Schedule Interactively

Run:

```bash
at now + 1 minute
```

Enter:

```text
warning: commands will be executed using /bin/sh
at> date > /tmp/datestamp
```

Press:

```text
Ctrl-D
```

You should receive output similar to:

```text
job 18 at Tue Jul 15 09:02:00 2026
```

Check:

```bash
atq
```

After the job executes:

```bash
cat /tmp/datestamp
```

---

# Practical Lab: Schedule a Daily Cron Job

Suppose you want a script to run every day at 10:00 AM.

## 1. Create the Script

Create:

```text
/tmp/myjob.sh
```

with:

```bash
#!/bin/bash

echo "Hello, I am running $0 at $(date)"
```

Make it executable:

```bash
chmod +x /tmp/myjob.sh
```

---

## 2. Create a Crontab File

Create:

```text
mycrontab
```

containing:

```cron
0 10 * * * /tmp/myjob.sh
```

The schedule:

```text
0 10 * * *
```

means:

```text
10:00 AM
every day
every month
regardless of weekday
```

---

## 3. Install the Crontab

Load the file:

```bash
crontab mycrontab
```

Verify:

```bash
crontab -l
```

Expected:

```text
0 10 * * * /tmp/myjob.sh
```

---

# Removing Cron Jobs

To edit individual entries:

```bash
crontab -e
```

Remove the unwanted line and save.

To remove the **entire personal crontab**:

```bash
crontab -r
```

Be careful:

```bash
crontab -r
```

removes all jobs in that user's crontab, not just one entry.

A safer approach when you have multiple jobs is usually:

```bash
crontab -e
```

and delete only the entry you no longer need.

---

# Cron and Missed Runs

Consider:

```cron
0 10 * * * /tmp/myjob.sh
```

If the machine is powered off at 10:00 AM:

```text
10:00 AM → machine is off
          ↓
       run missed
          ↓
machine starts later
          ↓
cron does not normally run the missed occurrence
```

This is different from periodic catch-up mechanisms such as anacron or appropriately configured systemd timers.

If your requirement is:

```text
"Run every day, and make sure it eventually runs even if the machine was off"
```

then a fixed-time personal cron entry may not be the appropriate mechanism.

---

# A Better Mental Model

Think about scheduling in terms of **what you actually need**:

```text
                  Need to schedule work?
                           │
              ┌────────────┴────────────┐
              │                         │
           One time                  Repeatedly
              │                         │
             at                ┌────────┴────────┐
              │                │                 │
        Exact future       Exact clock time   Periodic/catch-up
          clock time            │                 │
              │                cron            anacron
              │
             at
```

If the requirement is only:

```text
"Pause before continuing"
```

use:

```bash
sleep
```

For modern systemd-based services:

```text
systemd timer
```

is another important option.

---

# Common Scheduling Mistakes

## Mistake 1: Using `sleep` as a permanent scheduler

This:

```bash
sleep 24h
```

only keeps the current process waiting. It is not a robust replacement for a scheduler.

For recurring jobs, use cron, anacron, or systemd timers.

---

## Mistake 2: Assuming Cron Has Your Interactive Environment

This may work in a terminal:

```bash
backup.sh
```

but fail under cron because `PATH` or other environment variables differ.

Prefer:

```cron
0 2 * * * /usr/local/bin/backup.sh
```

and make the script self-contained.

---

## Mistake 3: Forgetting to Redirect Logs

A scheduled job may run successfully but provide no visible output in your terminal.

For troubleshooting:

```cron
0 10 * * * /tmp/myjob.sh >> /tmp/myjob.log 2>&1
```

Then:

```bash
cat /tmp/myjob.log
```

---

## Mistake 4: Assuming Cron Catches Up

A normal fixed-time cron entry does not automatically catch up after a shutdown.

If catch-up behavior matters, consider:

- `anacron`
- A systemd timer with appropriate persistence configuration

---

## Mistake 5: Removing the Entire Crontab Accidentally

This:

```bash
crontab -r
```

removes the user's entire crontab.

If you only need to remove one job:

```bash
crontab -e
```

and delete that entry.

---

# Key Takeaways

- **`at`** schedules a command to run once in the future.
- **`atq`** lists queued `at` jobs.
- **`atrm`** removes a queued `at` job.
- `at` relies on the **`atd` daemon** to execute queued jobs.
- **`cron`** schedules recurring jobs using five time fields followed by a command.
- A cron entry has the form:

```text
MIN HOUR DOM MON DOW COMMAND
```

- `*` means every applicable value.
- Use `crontab -e` to edit your personal cron jobs.
- Use `crontab -l` to inspect them.
- `crontab -r` removes the entire personal crontab.
- Cron jobs may have a different environment from an interactive shell, so use absolute paths and explicitly configure required environment variables.
- Redirect scheduled-job output to logs when reliable diagnostics are important.
- **`anacron`** is designed for periodic daily/weekly/monthly jobs that may be missed because a machine is powered off.
- **systemd timers** provide an integrated scheduling mechanism for systemd-based systems.
- **`sleep`** pauses the current execution for a duration; it is not a general-purpose job scheduler.
- The simplest rule is:

```text
One-time future job       → at
Recurring exact-time job  → cron
Periodic catch-up job     → anacron
systemd-integrated job    → systemd timer
Delay current execution   → sleep
```