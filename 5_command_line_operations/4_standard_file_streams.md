# Standard File Streams, I/O Redirection, Pipes, and File Searching

Linux commands communicate with the shell and with other commands
through **standard file streams**. Understanding these streams makes
redirection, pipelines, logging, and automation much easier.

------------------------------------------------------------------------

## 1. Standard File Streams

When a command starts, Linux normally provides three standard streams:

  Stream            Abbreviation     File Descriptor Default
  ----------------- -------------- ----------------- ----------
  Standard input    `stdin`                      `0` Keyboard
  Standard output   `stdout`                     `1` Terminal
  Standard error    `stderr`                     `2` Terminal

A **file descriptor (FD)** is simply a number that a process uses to
refer to an open file or I/O stream.

Think of the three standard streams as three communication channels:

``` text
                 Command
                /   |   \
               /    |    \
          stdin   stdout  stderr
            0       1       2
            ↑       ↓       ↓
         keyboard terminal terminal
```

### Standard input --- `stdin` --- FD 0

`stdin` is where a command normally receives input.

For an interactive command, this usually means the keyboard:

``` bash
$ cat
hello
hello
```

The text typed on the keyboard becomes the command's standard input.

However, `stdin` does not have to come from the keyboard. It can come
from a file or another command.

### Standard output --- `stdout` --- FD 1

`stdout` is where a command normally sends its normal results.

For example:

``` bash
$ ls
file1.txt
file2.txt
```

The listing is written to `stdout`, which normally appears in the
terminal.

### Standard error --- `stderr` --- FD 2

`stderr` is used for diagnostic messages such as errors and warnings.

For example:

``` bash
$ ls does-not-exist
ls: cannot access 'does-not-exist': No such file or directory
```

The error message is written to `stderr`, not `stdout`.

A successful command may produce no `stderr` output, although some
programs can write warnings or other diagnostic messages even when they
succeed.

------------------------------------------------------------------------

## 2. File Descriptors

Linux represents open files and I/O streams using **file descriptors**.

The first three descriptors have conventional meanings:

``` text
0 → stdin
1 → stdout
2 → stderr
```

When a program opens additional files, the operating system normally
assigns descriptors starting at `3`:

``` text
0 → stdin
1 → stdout
2 → stderr
3 → another open file
4 → another open file
...
```

You do not normally need to manage these numbers yourself, but knowing
them is essential for I/O redirection.

------------------------------------------------------------------------

# I/O Redirection

**I/O redirection** allows you to change where a command gets its input
or where it sends its output and errors.

The most important operators are:

  Operator     Meaning
  ------------ -----------------------------------------------------------
  `< file`     Read `stdin` from a file
  `> file`     Write `stdout` to a file, replacing its previous contents
  `>> file`    Append `stdout` to a file
  `2> file`    Write `stderr` to a file
  `2>> file`   Append `stderr` to a file
  `2>&1`       Send `stderr` to the current destination of `stdout`

------------------------------------------------------------------------

## 3. Redirecting Standard Input

Use `<` to make a file become the command's standard input.

``` bash
$ do_something < input-file
```

Instead of reading from the keyboard, `do_something` reads from
`input-file`.

For example, `wc` can count the lines in a file through `stdin`:

``` bash
$ wc -l < names.txt
5
```

The file is not passed as a normal argument to `wc`; its contents are
supplied through standard input.

------------------------------------------------------------------------

## 4. Redirecting Standard Output

Use `>` to send `stdout` to a file:

``` bash
$ do_something > output-file
```

The command's normal output goes into `output-file` instead of appearing
in the terminal.

### Important: `>` overwrites

If the file already exists, its contents are normally replaced:

``` bash
$ ls > files.txt
```

If you want to **append** instead, use `>>`:

``` bash
$ ls >> files.txt
```

A useful way to remember this:

``` text
>   → replace
>>  → append
```

By default, `>` means `1>` because `stdout` is file descriptor `1`:

``` bash
$ command > output.txt
```

is equivalent to:

``` bash
$ command 1> output.txt
```

------------------------------------------------------------------------

## 5. Redirecting Standard Error

Because `stderr` has file descriptor `2`, place `2` before the
redirection operator:

``` bash
$ do_something 2> error-file
```

Now:

-   normal output (`stdout`) still goes to the terminal
-   error output (`stderr`) goes into `error-file`

For example:

``` bash
$ find / -name "*.conf" 2> errors.txt
```

The search results appear on the terminal while permission errors are
saved in `errors.txt`.

You can also append errors:

``` bash
$ command 2>> errors.txt
```

------------------------------------------------------------------------

## 6. Redirecting stdout and stderr Separately

You can send normal output and errors to different files:

``` bash
$ command > output.txt 2> errors.txt
```

This is useful for scripts and automation where you want successful
results and diagnostic information separated.

``` text
command
  │
  ├── stdout (1) ──→ output.txt
  │
  └── stderr (2) ──→ errors.txt
```

------------------------------------------------------------------------

## 7. Redirecting stdout and stderr to the Same File

Sometimes you want to capture **everything** in one file.

Use:

``` bash
$ command > all-output.txt 2>&1
```

This means:

1.  `> all-output.txt` sends `stdout` (FD 1) to the file.
2.  `2>&1` sends `stderr` (FD 2) to the same destination currently used
    by `stdout`.

So both streams go to `all-output.txt`.

``` text
                  command
                 /       \
        stdout (1)       stderr (2)
             │                │
             └──────┬─────────┘
                    ↓
             all-output.txt
```

### Redirection order matters

These two commands are **not equivalent**:

``` bash
command > output.txt 2>&1
```

and:

``` bash
command 2>&1 > output.txt
```

The shell processes redirections from **left to right**.

#### Correct: both go to the file

``` bash
command > output.txt 2>&1
```

At the point `2>&1` is processed, `stdout` is already pointing to
`output.txt`, so `stderr` is sent there too.

#### Different result

``` bash
command 2>&1 > output.txt
```

Here, `stderr` is first connected to the terminal's current `stdout`.
Then `stdout` is redirected to `output.txt`.

Therefore:

-   `stdout` → `output.txt`
-   `stderr` → terminal

### Bash shorthand

Bash also supports:

``` bash
command &> output.txt
```

This is a Bash-specific shorthand for redirecting both standard output
and standard error to the same destination.

------------------------------------------------------------------------

# Pipes

## 8. What Is a Pipe?

A **pipe** connects the `stdout` of one command directly to the `stdin`
of another command.

The pipe operator is:

``` text
|
```

Example:

``` bash
$ command1 | command2 | command3
```

The data flows like this:

``` text
command1
   │
 stdout
   ↓
 stdin → command2
           │
         stdout
           ↓
 stdin → command3
```

This is called a **pipeline**.

------------------------------------------------------------------------

## 9. Example: Counting Files

``` bash
$ ls | wc -l
```

Here:

1.  `ls` produces a directory listing.
2.  Its `stdout` is connected to `wc`'s `stdin`.
3.  `wc -l` counts lines.

No temporary file is needed.

A more robust way to count directory entries for many situations is to
use tools/options designed specifically for that purpose; `ls | wc -l`
is mainly a simple demonstration of how pipes work.

------------------------------------------------------------------------

## 10. Why Pipelines Are Useful

Pipelines follow an important Unix/Linux design idea:

> Build complex tasks by combining small programs that each do one job
> well.

For example:

``` bash
$ command1 | command2 | command3
```

Each command performs one step.

### Pipelines avoid intermediate files

Without a pipe, you might do:

``` bash
$ command1 > temporary.txt
$ command2 < temporary.txt
```

With a pipe:

``` bash
$ command1 | command2
```

The data can flow directly between the commands.

### Commands can process data as it arrives

Pipeline stages can operate concurrently. A later command can begin
processing data while an earlier command is still producing it.

This can reduce waiting and avoid unnecessary disk I/O.

**Note:** A pipeline is not automatically faster in every situation.
Performance depends on the commands, data size, CPU, and I/O involved.

------------------------------------------------------------------------

# Searching for Files

Linux provides several ways to locate files. Three important concepts
are:

1.  `locate` --- searches a prebuilt database quickly.
2.  `find` --- searches the filesystem directly using flexible
    conditions.
3.  Bash wildcards --- match filenames using patterns.

These tools solve related but different problems.

------------------------------------------------------------------------

# The `locate` Utility

## 11. How `locate` Works

`locate` searches a database containing file and directory paths rather
than scanning the filesystem every time.

For example:

``` bash
$ locate zip
```

This can be extremely fast.

However, the database may not contain files that were created or moved
very recently.

The database is maintained by `updatedb`.

On systems that use the traditional `locate` interface, you can update
it with:

``` bash
$ sudo updatedb
```

Modern Linux distributions may use **plocate**, which provides a
compatible `locate` command while using a more efficient database/search
implementation.

### `locate` vs `find`

  -----------------------------------------------------------------------
  Feature                 `locate`                `find`
  ----------------------- ----------------------- -----------------------
  Searches                Database                Filesystem

  Speed                   Usually very fast       Depends on search area

  Newly created files     May not appear until    Found immediately
                          database update         

  Conditions              Relatively limited      Very flexible

  Good for                Quickly locating known  Precise searches
                          names                   
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 12. Filtering `locate` Results with `grep`

You can pipe `locate` into `grep`:

``` bash
$ locate zip | grep bin
```

Here:

-   `locate zip` finds paths containing `zip`
-   `grep bin` keeps only lines containing `bin`

For example, a result might include:

``` text
/usr/bin/gzip
```

Remember that both searches are performed against the **full path**, not
just the filename.

------------------------------------------------------------------------

# Bash Wildcards

## 13. What Are Wildcards?

Wildcards, also called **globs**, allow you to describe filename
patterns.

The Bash shell expands the pattern **before** the command runs.

For example:

``` bash
$ ls *.txt
```

Bash may expand it to something like:

``` bash
$ ls file1.txt file2.txt notes.txt
```

`ls` receives the expanded filenames rather than the original `*.txt`
pattern.

This is why wildcards work with many different commands.

------------------------------------------------------------------------

## 14. Common Wildcards

  Pattern    Meaning                        Example
  ---------- ------------------------------ -------------------
  `?`        Exactly one character          `file?.txt`
  `*`        Zero or more characters        `*.txt`
  `[abc]`    One character from the set     `file_[abc].txt`
  `[a-z]`    One character in a range       `file_[a-z].txt`
  `[!abc]`   One character not in the set   `file_[!abc].txt`

------------------------------------------------------------------------

## 15. Using `?`

`?` matches exactly one character.

If you have filenames such as:

``` text
bar.out
bat.out
bag.out
```

you can use:

``` bash
$ ls ba?.out
```

The `?` represents the single unknown character.

It does **not** match zero characters or multiple characters.

------------------------------------------------------------------------

## 16. Using `*`

`*` matches zero or more characters.

For example:

``` bash
$ ls *.out
```

matches:

``` text
a.out
test.out
backup.out
.out
```

because the `*` can represent any string, including an empty string.

Another example:

``` bash
$ ls a*log*
```

This matches names that:

-   start with `a`
-   contain `log` somewhere later
-   may have other characters before or after `log`

------------------------------------------------------------------------

## 17. Using `[set]`

A character set matches **one character** from the specified set.

``` bash
$ ls file_[abc].out
```

This can match:

``` text
file_a.out
file_b.out
file_c.out
```

but not:

``` text
file_d.out
```

You can also specify ranges:

``` bash
$ ls file_[a-z].out
```

This matches a single lowercase letter.

------------------------------------------------------------------------

## 18. Using `[!set]`

`[!abc]` matches one character that is **not** `a`, `b`, or `c`.

For example:

``` bash
$ ls file_[!abc].txt
```

can match:

``` text
file_d.txt
file_1.txt
file_x.txt
```

but not:

``` text
file_a.txt
file_b.txt
file_c.txt
```

------------------------------------------------------------------------

## 19. Important: Wildcards Are Expanded by the Shell

Consider:

``` bash
$ command *.txt
```

Bash expands `*.txt` first.

Conceptually:

``` text
You type:
command *.txt

Shell expands it:
command file1.txt file2.txt notes.txt

Then command runs.
```

This matters because **the command itself may never see `*.txt`**.

### Preventing wildcard expansion

If you want a command to receive the literal characters `*` or `?`,
quote the pattern:

``` bash
$ command 'vmware*'
```

or:

``` bash
$ command "vmware*"
```

The quotes prevent Bash from performing filename expansion.

------------------------------------------------------------------------

## 20. Why Quoting Matters

Suppose your current directory contains files such as:

``` text
vmware-network.log
vmware-service.log
vmware-manager.log
```

If you run:

``` bash
$ apt install vmware*
```

the shell may expand `vmware*` into those filenames before `apt` runs:

``` text
apt install vmware-network.log vmware-service.log vmware-manager.log
```

That is probably not what you intended.

If you want `apt` to receive the literal pattern:

``` bash
$ apt install 'vmware*'
```

then `apt` gets the pattern itself and can interpret it according to its
own package-matching rules, if supported.

### General lesson

Before using `*`, `?`, or other glob patterns, ask:

> **Do I want the shell to expand this pattern, or do I want the program
> itself to receive the pattern?**

------------------------------------------------------------------------

## 21. What Happens When a Wildcard Matches Nothing?

In Bash's default behavior, if a wildcard matches no filenames, the
pattern is usually passed to the command unchanged.

For example, if there are no `.out` files:

``` bash
$ ls *.out
```

Bash may effectively pass:

``` text
*.out
```

to `ls`, resulting in an error such as:

``` text
ls: cannot access '*.out': No such file or directory
```

Bash has options such as `nullglob` that can change this behavior, but
the default behavior is important to understand.

------------------------------------------------------------------------

# The `find` Utility

## 22. What Is `find`?

`find` is one of the most powerful Linux utilities for locating files
and directories.

Unlike `locate`, `find` normally searches the filesystem starting from a
directory that you specify.

Basic syntax:

``` bash
find [starting-directory] [conditions]
```

For example:

``` bash
$ find /usr -name gcc
```

This searches under `/usr` for entries named `gcc`.

If you omit the starting directory:

``` bash
$ find -name gcc
```

`find` searches from the current directory.

------------------------------------------------------------------------

## 23. Searching by Name

### Search for a specific name

``` bash
$ find /usr -name gcc
```

### Case-insensitive name search

Use `-iname`:

``` bash
$ find /usr -iname gcc
```

This can match names such as:

``` text
gcc
GCC
Gcc
```

depending on the exact filename.

------------------------------------------------------------------------

## 24. Searching by File Type

The `-type` option restricts the result to a particular kind of
filesystem object.

Common types:

  Type   Meaning
  ------ ---------------
  `f`    Regular file
  `d`    Directory
  `l`    Symbolic link

Examples:

``` bash
$ find /usr -type d -name gcc
```

Find directories named `gcc`.

``` bash
$ find /usr -type f -name gcc
```

Find regular files named `gcc`.

``` bash
$ find /usr -type l -name gcc
```

Find symbolic links named `gcc`.

------------------------------------------------------------------------

## 25. Limit Search Depth with `-maxdepth`

By default, `find` recursively searches subdirectories.

You can limit how deep it searches.

For example:

``` bash
$ find . -maxdepth 1 -type d
```

This searches only the current directory level.

``` bash
$ find . -maxdepth 2 -type d
```

This searches the current directory and one level below it.

This is useful when a recursive search produces too many results.

------------------------------------------------------------------------

# Running Commands with `find`

## 26. The `-exec` Option

`find` can run another command for every matching entry.

For example:

``` bash
$ find . -type f -exec ls -l {} \;
```

Here:

-   `-exec` tells `find` to run another command.
-   `ls -l` is the command to execute.
-   `{}` is replaced with the current matching pathname.
-   `\;` tells `find` where the `-exec` command ends.

Conceptually:

``` text
find result
     ↓
{} is replaced with the pathname
     ↓
ls -l pathname
```

If `find` discovers:

``` text
./file1.txt
./file2.txt
```

the command can effectively execute:

``` bash
ls -l ./file1.txt
ls -l ./file2.txt
```

------------------------------------------------------------------------

## 27. `\;` and `';'`

The end of `-exec` can be written as:

``` bash
\;
```

or:

``` bash
';'
```

For example:

``` bash
find . -type f -exec ls -l {} \;
```

The backslash prevents the shell from treating `;` as the end of the
shell command.

If copying commands from formatted documents, use normal straight
quotes, not typographic "smart quotes".

------------------------------------------------------------------------

## 28. Safer `find` Actions

Be careful when combining `find` with destructive commands.

For example:

``` bash
$ find . -name "*.swp" -exec rm {} \;
```

This finds matching files and removes them.

A safer approach is to inspect the results first:

``` bash
$ find . -name "*.swp"
```

Then, if appropriate, perform the deletion.

You can also use `-ok`:

``` bash
$ find . -name "*.swp" -ok rm {} \;
```

`-ok` asks for confirmation before executing the command on each match.

### Another useful approach

For many tasks, `-exec` can process multiple matches at once:

``` bash
find . -type f -name "*.log" -exec ls -l {} +
```

Using `+` can be more efficient than running the command separately for
every file.

------------------------------------------------------------------------

# Finding Files by Time

## 29. File Timestamps

Linux files have several important timestamps. `find` can search based
on them.

Common options:

  Option     Searches based on
  ---------- ----------------------------------------
  `-mtime`   Modification time
  `-atime`   Access time
  `-ctime`   Metadata/status change time
  `-mmin`    Modification time in minutes
  `-amin`    Access time in minutes
  `-cmin`    Metadata/status change time in minutes

### Important: `ctime` is not creation time

For example:

``` bash
$ find / -ctime 3
```

`-ctime` refers to the time when the file's **metadata/status changed**,
such as permissions, ownership, or other inode-related information.

It does **not** mean "file creation time".

Standard `find` time predicates are based primarily on access,
modification, and status-change times.

------------------------------------------------------------------------

## 30. Understanding `n`, `+n`, and `-n`

Many numeric `find` tests support these forms:

  -----------------------------------------------------------------------
  Form                                Meaning
  ----------------------------------- -----------------------------------
  `n`                                 Approximately/exactly the matching
                                      unit according to `find`'s age
                                      calculation

  `+n`                                More than `n` units

  `-n`                                Less than `n` units
  -----------------------------------------------------------------------

For example:

``` bash
find . -mtime +7
```

finds files whose modification time is more than seven 24-hour periods
ago.

For exact boundary behavior, consult:

``` bash
man find
```

because `find` rounds/truncates age calculations in ways that can
surprise beginners.

------------------------------------------------------------------------

# Finding Files by Size

## 31. The `-size` Option

Use `-size` to search by file size:

``` bash
$ find . -size 0
```

This finds files whose size matches the specified `find` size unit.

You can specify units:

  Suffix   Unit
  -------- -------
  `c`      Bytes
  `k`      KiB
  `M`      MiB
  `G`      GiB

For example:

``` bash
$ find . -size +10M
```

finds files larger than 10 MiB according to `find`'s size semantics.

A particularly useful example is:

``` bash
$ find . -type f -size +10M -ls
```

This finds regular files larger than 10 MiB and displays detailed
information.

------------------------------------------------------------------------

## 32. Searching from `/`

You can search the entire filesystem:

``` bash
$ find / -name "*.conf"
```

But this can:

-   take a long time
-   produce many results
-   produce permission errors as a regular user

You can discard `stderr`:

``` bash
$ find / -name "*.conf" 2> /dev/null
```

Here, `/dev/null` is a special device that discards anything written to
it.

Or, when appropriate, use:

``` bash
$ sudo find / -name "*.conf"
```

### Tip

When learning `find`, start with a smaller location:

``` bash
$ find ~ -name "*.txt"
```

or:

``` bash
$ find /tmp -type f
```

This is faster and easier to understand.

------------------------------------------------------------------------

# Useful `find` Examples

### Find all regular files under the current directory

``` bash
$ find . -type f
```

### Find all directories

``` bash
$ find . -type d
```

### Find `.log` files

``` bash
$ find . -type f -name "*.log"
```

### Case-insensitive search for `.LOG`/`.log`-style names

``` bash
$ find . -type f -iname "*.log"
```

### Find empty files

``` bash
$ find . -type f -size 0
```

### Find files larger than 100 MiB

``` bash
$ find . -type f -size +100M
```

### Find recently modified files

``` bash
$ find . -type f -mtime -1
```

### Find files modified more than 30 days ago

``` bash
$ find . -type f -mtime +30
```

### Find directories named `init.d`

``` bash
$ find / -type d -name init.d 2> /dev/null
```

------------------------------------------------------------------------

# Lab 7.4: Finding a Directory and Creating a Symbolic Link

This exercise combines two skills:

1.  Use `find` to locate a directory.
2.  Use `ln -s` to create a symbolic link.

## Task

1.  Find the `init.d` directory, starting from `/`.
2.  Create a symbolic link to it in your home directory.

### Step 1: Find the directory

``` bash
$ find / -type d -name init.d 2> /dev/null
```

A typical result is:

``` text
/etc/init.d
```

On a modern system, `/etc/init.d` is generally retained for
compatibility with older SysVinit-style service scripts. A system may
not have it, especially if it is minimal or configured differently.

### Step 2: Create the symbolic link

Move to your home directory:

``` bash
$ cd ~
```

Then:

``` bash
$ ln -s /etc/init.d .
```

The syntax is:

``` text
ln -s TARGET LINK_NAME
```

In this example:

``` text
TARGET     → /etc/init.d
LINK_NAME  → ./init.d
```

Because `.` means the current directory, `ln` creates the link inside
your home directory.

### Step 3: Verify the link

``` bash
$ ls -l init.d
```

You may see something similar to:

``` text
lrwxrwxrwx ... init.d -> /etc/init.d
```

Two things identify it as a symbolic link:

-   The first character is `l`.
-   The `->` arrow shows the target.

Conceptually:

``` text
~/init.d
   │
   └──── symbolic link ────→ /etc/init.d
```

The link does not contain a separate copy of the directory. It points to
the target path.

------------------------------------------------------------------------

# Quick Reference

## Standard Streams

``` text
0 → stdin  → input
1 → stdout → normal output
2 → stderr → errors/diagnostics
```

## Redirection

``` bash
command < input.txt
command > output.txt
command >> output.txt
command 2> errors.txt
command 2>> errors.txt
command > all.txt 2>&1
command &> all.txt        # Bash
```

## Pipes

``` bash
command1 | command2
command1 | command2 | command3
```

## Wildcards

``` text
?        one character
*        zero or more characters
[abc]    one character from a, b, or c
[a-z]    one character in the range a-z
[!abc]   one character not in a, b, or c
```

## `locate`

``` bash
locate filename
locate pattern | grep text
sudo updatedb
```

## `find`

``` bash
find . -name "filename"
find . -iname "filename"
find . -type f
find . -type d
find . -type l
find . -maxdepth 1 -type d
find . -size +10M
find . -mtime -1
find . -exec command {} \;
```

------------------------------------------------------------------------

# Key Takeaways

-   Linux commands normally start with three standard streams: `stdin`
    (`0`), `stdout` (`1`), and `stderr` (`2`).
-   `<` redirects standard input from a file.
-   `>` redirects standard output and replaces the destination file;
    `>>` appends to it.
-   `2>` redirects standard error.
-   `2>&1` connects `stderr` to the current destination of `stdout`;
    redirection order matters.
-   A pipe (`|`) connects one command's `stdout` directly to another
    command's `stdin`.
-   `locate` is fast because it searches a database, but its results can
    be out of date.
-   `find` searches the filesystem and supports powerful conditions such
    as name, type, depth, time, and size.
-   Bash wildcards are expanded by the shell before the command runs.
-   Quote a wildcard when you need the literal pattern to reach the
    command instead of being expanded by Bash.
-   Be especially careful when combining `find`, wildcards, `sudo`, and
    destructive commands such as `rm`.