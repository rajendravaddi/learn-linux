# Working with Files and Directories

File management is one of the most common tasks you will perform on Linux. From the command line, you can:

- View file contents
- Create files and directories
- Move and rename files
- Delete files and directories
- Explore directory structures
- Customize your command prompt

The following commands are among the most useful tools for everyday Linux administration.

---

# Viewing Files

Linux provides several commands for reading text files. The command you choose depends mainly on the size of the file and what you want to inspect.

| Command | Purpose |
|---|---|
| `cat` | Display the contents of a file |
| `tac` | Display a file from the last line to the first |
| `less` | View large files one screen at a time |
| `head` | Display the beginning of a file |
| `tail` | Display the end of a file |

---

## `cat`

`cat` displays the contents of a file directly in the terminal.

```bash
cat notes.txt
```

For a short file, this is usually the simplest option.

You can also display line numbers:

```bash
cat -n notes.txt
```

For example:

```text
     1  First line
     2  Second line
     3  Third line
```

### When Should You Use `cat`?

Use `cat` when the file is small enough to fit comfortably in the terminal.

For a very large file, `cat` can quickly fill the terminal with thousands of lines, making it difficult to find what you need.

---

## `tac`

`tac` is essentially `cat` in reverse.

```bash
tac notes.txt
```

If the file contains:

```text
Line 1
Line 2
Line 3
```

`tac` displays:

```text
Line 3
Line 2
Line 1
```

This can occasionally be useful when the newest or last entries in a text file are more important than the first entries.

---

# `less`

`less` is a **pager**. It allows you to view large files one screen at a time instead of dumping the entire file into the terminal.

```bash
less large-file.txt
```

For example, if a file contains thousands of lines, `less` lets you move through it interactively.

### Useful `less` Controls

Inside `less`:

| Key | Action |
|---|---|
| `Space` | Move forward one screen |
| `b` | Move backward one screen |
| `↑` / `↓` | Move one line |
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Go to the next search result |
| `N` | Go to the previous search result |
| `g` | Go to the beginning |
| `G` | Go to the end |
| `q` | Quit |

For example:

```text
/pattern
```

searches forward for `pattern`.

To search backward:

```text
?pattern
```

### Showing Line Numbers

You can use:

```bash
less -N large-file.txt
```

The `-N` option displays line numbers.

> **Memory trick:** The old `more` pager has fewer features. The name `less` is often remembered with the joke: **"less is more."**

---

# `head`

`head` displays the beginning of a file.

By default, it displays the first **10 lines**:

```bash
head notes.txt
```

To display a different number of lines, use `-n`:

```bash
head -n 20 notes.txt
```

This displays the first 20 lines.

You may also encounter the shorter form:

```bash
head -20 notes.txt
```

The `-n` form is generally clearer and easier to remember.

### Common Use

`head` is useful when you only need to inspect the beginning of a configuration file, script, or data file.

---

# `tail`

`tail` is the opposite of `head`: it displays the end of a file.

By default:

```bash
tail notes.txt
```

shows the last 10 lines.

To display the last 20 lines:

```bash
tail -n 20 notes.txt
```

### Following a File

One of the most useful features of `tail` is `-f`:

```bash
tail -f application.log
```

This keeps the command running and displays new lines as they are added to the file.

This is extremely useful for monitoring logs while an application or server is running.

Press:

```text
Ctrl + C
```

to stop following the file.

---

# Choosing the Right File-Viewing Command

A simple way to decide which command to use:

```text
Small file?
    ↓
   cat

Large file that you want to browse?
    ↓
   less

Need the first few lines?
    ↓
   head

Need the last few lines?
    ↓
   tail

Need to watch new log entries?
    ↓
   tail -f
```

For example, when troubleshooting an application:

```bash
tail -f /var/log/myapp.log
```

is often more useful than:

```bash
cat /var/log/myapp.log
```

because log files can be very large and continuously changing.

---

# Creating Files with `touch`

The `touch` command is commonly used to create an empty file:

```bash
touch notes.txt
```

If `notes.txt` does not exist, Linux creates an empty file.

You can verify it with:

```bash
ls -l notes.txt
```

### `touch` Does More Than Create Files

If the file already exists, `touch` normally updates its **access time (atime)** and **modification time (mtime)** to the current time.

For example:

```bash
touch notes.txt
```

does **not** erase the existing contents of `notes.txt`.

It updates its timestamps.

This makes `touch` useful both for creating empty files and for manipulating file timestamps.

---

# Setting a Specific Timestamp

The `-t` option lets you specify a timestamp:

```bash
touch -t 12091600 myfile
```

The format is:

```text
[[CC]YY]MMDDhhmm[.ss]
```

In:

```text
12091600
```

the components are:

```text
12 → December
09 → day 9
16 → 4:00 PM
00 → 00 minutes
```

Since the year is omitted, the current year is used.

You can then inspect the timestamp with:

```bash
ls -l myfile
```

> **Important:** `touch` does not let you arbitrarily set a file's `ctime` (change time). The filesystem updates `ctime` automatically whenever certain metadata changes. For example, changing timestamps with `touch` can itself cause `ctime` to reflect the time of that metadata change.

---

# Moving and Renaming Files with `mv`

The `mv` command means **move**.

It is used for both:

1. Moving files or directories
2. Renaming files or directories

The basic syntax is:

```bash
mv source destination
```

---

## Renaming a File

Suppose you have:

```text
notes.txt
```

You can rename it:

```bash
mv notes.txt notes_old.txt
```

The file is now called:

```text
notes_old.txt
```

No new copy is created; the directory entry is renamed.

---

## Moving a File

Suppose:

```text
notes.txt
```

is in your current directory and you want to move it into:

```text
/home/student/documents/
```

Use:

```bash
mv notes.txt /home/student/documents/
```

The file keeps its original name:

```text
/home/student/documents/notes.txt
```

---

## Move and Rename at the Same Time

You can specify a new filename at the destination:

```bash
mv notes.txt /home/student/documents/old-notes.txt
```

The file is both moved and renamed.

The result is:

```text
/home/student/documents/old-notes.txt
```

---

# Be Careful with `mv` Destinations

When moving files, make sure the destination is what you intended.

Suppose:

```text
/home/student/documents/
```

does not exist.

This command:

```bash
mv notes.txt /home/student/documents/
```

will normally fail because the destination directory does not exist.

However, without the trailing slash:

```bash
mv notes.txt /home/student/documents
```

`mv` can interpret `documents` as the **destination filename** if the parent directory exists.

You could unintentionally rename `notes.txt` to:

```text
documents
```

> **Tip:** When you intend to move something into a directory, verify that the directory exists first:
>
> ```bash
> ls -ld /home/student/documents
> ```

---

# Removing Files with `rm`

The `rm` command removes files.

For example:

```bash
rm notes.txt
```

After this command, `notes.txt` is removed from the directory.

Unlike a desktop file manager, `rm` normally does **not** move the file to a Trash or Recycle Bin.

> **Important:** Files removed with `rm` are generally not recoverable through a normal undo operation. Treat `rm` as a destructive command.

---

# Interactive Removal with `rm -i`

When you want extra protection, use:

```bash
rm -i notes.txt
```

Linux asks for confirmation:

```text
rm: remove regular file 'notes.txt'? y
```

Enter:

```text
y
```

to remove it or:

```text
n
```

to keep it.

For multiple files:

```bash
rm -i file1 file2 file3
```

you may be asked about each file.

> **Tip:** `rm -i` is particularly useful while learning shell commands or when deleting files using wildcards.

---

# Force Removal with `rm -f`

The `-f` option means **force**.

```bash
rm -f notes.txt
```

It suppresses many prompts and ignores nonexistent files.

Because it reduces protection against accidental deletion, use it carefully.

A particularly dangerous pattern is:

```bash
rm -rf ...
```

because it combines recursive deletion with force.

---

# Wildcards and `rm`

Linux shells support wildcards such as `*`.

For example:

```bash
rm *.tmp
```

can remove all `.tmp` files in the current directory.

Before using `rm` with a wildcard, it is safer to see what the pattern matches:

```bash
ls *.tmp
```

Then, if the results are correct:

```bash
rm *.tmp
```

This simple habit can prevent accidental deletion.

---

# Creating Directories with `mkdir`

The `mkdir` command creates directories.

```bash
mkdir sampdir
```

This creates:

```text
sampdir/
```

inside the current directory.

You can create multiple directories at once:

```bash
mkdir dir1 dir2 dir3
```

---

# Creating Directories with Absolute Paths

You can specify an absolute path:

```bash
mkdir /usr/sampdir
```

However, creating a directory under `/usr` normally requires appropriate permissions.

A regular user will usually receive:

```text
Permission denied
```

unless they have the necessary permissions or use an administrative command such as:

```bash
sudo mkdir /usr/sampdir
```

> **Important:** Do not use `sudo` automatically. Use it only when the operation actually requires administrative privileges.

---

# Creating Parent Directories with `mkdir -p`

Suppose you want to create:

```text
projects/2026/reports
```

but none of the parent directories exist.

This command:

```bash
mkdir projects/2026/reports
```

will fail if `projects` or `2026` does not already exist.

Use:

```bash
mkdir -p projects/2026/reports
```

The `-p` option creates all missing parent directories.

The result is:

```text
projects/
└── 2026/
    └── reports/
```

This is extremely useful in scripts and deployment commands.

---

# Renaming Directories

Directories are renamed with the same `mv` command used for files.

For example:

```bash
mv projects archive
```

renames:

```text
projects/
```

to:

```text
archive/
```

You can also move a directory:

```bash
mv projects /home/student/documents/
```

---

# Removing Directories

There are two common approaches, depending on whether the directory is empty.

## `rmdir`

`rmdir` removes an **empty directory**:

```bash
rmdir sampdir
```

If the directory contains files or subdirectories, the command fails.

For example:

```text
rmdir: failed to remove 'sampdir': Directory not empty
```

Hidden files count too, so a directory that looks empty in a normal `ls` listing may still contain files.

You can check with:

```bash
ls -la sampdir
```

---

# Removing Non-Empty Directories with `rm -r`

To remove a directory and its contents recursively:

```bash
rm -r sampdir
```

The `-r` option means **recursive**.

It allows `rm` to enter the directory, remove its contents, and then remove the directory itself.

For example:

```text
sampdir/
├── file1
├── file2
└── subdir/
    └── file3
```

Running:

```bash
rm -r sampdir
```

removes the entire tree.

---

# `rm -rf`: Recursive + Force

You may encounter:

```bash
rm -rf sampdir
```

This combines:

- `-r` → recursive
- `-f` → force

It can remove an entire directory tree without interactive confirmation.

**This is one of the commands that deserves extreme caution.**

A mistake in the path can delete a large amount of data.

### Safer While Learning

Use:

```bash
rm -ri sampdir
```

This combines recursive deletion with interactive confirmation.

You can also inspect the directory first:

```bash
ls -la sampdir
```

> **Important:** There is no `rmdir -r` equivalent you should rely on. `rmdir` is specifically for empty directories; recursive directory deletion is performed with `rm -r`.

---

# File and Directory Command Summary

| Command | Purpose |
|---|---|
| `cat file` | Display a file |
| `tac file` | Display a file in reverse line order |
| `less file` | Browse a file interactively |
| `head file` | Show first 10 lines |
| `tail file` | Show last 10 lines |
| `tail -f file` | Follow new content added to a file |
| `touch file` | Create an empty file or update timestamps |
| `mv source destination` | Move or rename |
| `rm file` | Remove a file |
| `rm -i file` | Remove interactively |
| `rm -f file` | Force removal |
| `mkdir dir` | Create a directory |
| `mkdir -p path` | Create directories and missing parents |
| `rmdir dir` | Remove an empty directory |
| `rm -r dir` | Recursively remove a directory and its contents |
| `rm -rf dir` | Force recursive removal |

---

# Customizing the Command Prompt with `PS1`

When you open a shell, you normally see a prompt similar to:

```text
student@server:~$
```

The prompt is controlled by the shell's **`PS1` variable**.

`PS1` stands for the shell's primary prompt string.

You can inspect it with:

```bash
echo "$PS1"
```

The exact value varies between distributions and shell configurations.

---

# Why Customize the Prompt?

A customized prompt can display useful information such as:

- Your username
- The hostname
- Your current directory
- Whether you are operating as root

This is especially helpful when working with multiple machines.

For example:

```text
student@server1:~$
student@server2:~$
```

The hostname helps you immediately recognize which machine you are working on.

This can prevent mistakes such as accidentally running a destructive command on a production server instead of a development machine.

---

# Useful `PS1` Escape Sequences

Bash provides special escape sequences that are expanded whenever the prompt is displayed.

| Escape | Meaning |
|---|---|
| `\u` | Current username |
| `\h` | Hostname up to the first `.` |
| `\w` | Current working directory |
| `\$` | `#` for root, `$` for other users |

For example:

```bash
PS1='\u@\h \w \$ '
```

could produce:

```text
student@server /home/student $
```

If you change to `/var/log`:

```text
student@server /var/log $
```

If the shell is running as root, the final character becomes:

```text
#
```

For example:

```text
root@server /var/log #
```

---

# Why Use `\$` Instead of `$`?

The sequence:

```text
\$
```

has special meaning in Bash prompts:

- Normal user → `$`
- Root user → `#`

This provides a useful visual warning when you have elevated privileges.

For example:

```text
student@server $
```

versus:

```text
root@server #
```

The difference can help prevent accidental execution of dangerous commands as root.

---

# Why Use Single Quotes When Setting `PS1`?

Use single quotes when assigning a prompt containing Bash escape sequences:

```bash
PS1='\u@\h \w \$ '
```

Single quotes prevent the shell from interpreting the backslashes during the assignment.

The prompt can then interpret them dynamically each time it is displayed.

For example, `\w` changes automatically as you move between directories.

You can verify the stored value with:

```bash
echo "$PS1"
```

---

# Making the Prompt Permanent

A command such as:

```bash
PS1='\u@\h \w \$ '
```

normally affects only the **current shell session**.

When you close the terminal and open a new one, the setting may disappear.

For Bash, you can place the setting in:

```text
~/.bashrc
```

For example:

```bash
echo "PS1='\u@\h \w \$ '" >> ~/.bashrc
```

Then start a new shell or reload the configuration:

```bash
source ~/.bashrc
```

> **Tip:** Be careful when modifying shell startup files. A syntax mistake can affect every new shell you open.

---

# Practical File-Management Example

Imagine you are starting with an empty project directory.

First create a directory:

```bash
mkdir project
```

Enter it:

```bash
cd project
```

Create some files:

```bash
touch app.log config.txt
```

Check them:

```bash
ls -l
```

View the configuration file:

```bash
cat config.txt
```

Rename it:

```bash
mv config.txt application.conf
```

Create a directory for logs:

```bash
mkdir logs
```

Move the log file into it:

```bash
mv app.log logs/
```

Check the result:

```bash
tree
```

You should have something similar to:

```text
.
├── application.conf
└── logs
    └── app.log
```

When you are finished and want to remove the project:

```bash
cd ..
rm -ri project
```

The interactive option lets you review deletion requests rather than immediately deleting everything.

---

# A Simple Mental Model

You can think of the basic file commands in groups:

### Look

```bash
cat
less
head
tail
```

**"Show me what's inside."**

### Create

```bash
touch
mkdir
```

**"Create a file or directory."**

### Move / Rename

```bash
mv
```

**"Change where it is or what it is called."**

### Delete

```bash
rm
rmdir
```

**"Remove it."**

### Explore

```bash
ls
tree
pwd
```

**"Show me where I am and what is here."**

---

# Key Takeaways

1. Use **`cat`** for quickly displaying small files.
2. Use **`less`** for interactively browsing large files.
3. Use **`head`** and **`tail`** when you only need the beginning or end of a file.
4. Use **`tail -f`** to monitor a file such as a continuously changing log.
5. **`touch`** creates an empty file if it does not exist and updates timestamps if it does.
6. **`mv`** is used for both moving and renaming files and directories.
7. **`rm`** removes files and does not normally send them to a Trash/Recycle Bin.
8. Use **`rm -i`** when you want confirmation before deletion.
9. **`mkdir`** creates directories, while **`mkdir -p`** also creates missing parent directories.
10. **`rmdir`** only removes empty directories.
11. **`rm -r`** recursively removes a directory and its contents.
12. **`rm -rf`** is powerful and potentially dangerous because it recursively removes data without interactive confirmation.
13. **`PS1`** controls Bash's primary command prompt.
14. Prompt escape sequences such as `\u`, `\h`, `\w`, and `\$` let you display useful information dynamically.
15. A well-designed prompt can help you recognize **who you are, which machine you are using, where you are, and whether you are root**.