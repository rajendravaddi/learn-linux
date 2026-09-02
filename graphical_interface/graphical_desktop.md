# Graphical Desktop

Linux provides two primary ways to interact with the system:

- **CLI (Command Line Interface)** — interact with the system by entering commands in a terminal.
- **GUI (Graphical User Interface)** — interact with the system using graphical elements such as windows, menus, icons, and buttons.

## How the Graphical Desktop Loads

When a Linux system starts, a **display manager** is responsible for managing graphical logins and starting the graphical desktop session.

Common display managers include:

- **GDM3 (GNOME Display Manager)** — commonly used by GNOME-based systems such as Ubuntu, Fedora, and RHEL (Red Hat Enterprise Linux).
- **SDDM (Simple Desktop Display Manager)** — commonly used by KDE Plasma installations.

> **Note:** Older documentation may reference **KDM (KDE Display Manager)**. KDM has been discontinued and replaced by SDDM on modern KDE Plasma installations.

### Display Manager Responsibilities

A display manager primarily:

- Manages graphical logins.
- Starts the graphical session.
- Works with the display server to provide the graphical environment.

The display manager relies on a **display server**, which handles communication between the hardware and graphical applications.

### X11 and Wayland

Historically, Linux graphical environments have primarily used the **X Window System**, commonly called **X or X11**. X has been used for decades and remains available on many systems, but it was designed for an earlier generation of computing.

**Wayland** is the modern replacement for X11. It uses a simpler architecture and provides improved security, modern display support, and more responsive input handling.

Wayland is now the default on many current Linux distributions, including modern versions of Ubuntu, Fedora, and RHEL. Older distributions, such as Ubuntu 18.04 and CentOS 7, commonly use X11.

For everyday desktop use, the transition from X11 to Wayland is mostly transparent. The desktop looks and behaves similarly, while the underlying architecture differs.

Wayland also provides better support for:

- High-resolution displays.
- Fractional display scaling.
- Modern input handling.
- Improved graphical security.

### XWayland

Some older applications were designed specifically for X11 and do not support Wayland natively.

**XWayland** provides a compatibility layer that allows X11 applications to run inside a Wayland session. It is included with modern Wayland installations, so most older applications can continue to work without additional configuration.

### The Transition from X11 to Wayland

The Linux ecosystem is gradually moving away from X11 toward Wayland. Some distributions still provide an option to choose between Wayland and X11 at the login screen, but the availability of X11 sessions is decreasing in newer releases.

For most users, this transition does not require any special action.

### Starting the Graphical Desktop Manually

If the graphical desktop does not start automatically, it can sometimes be started manually from a text console.

The command depends on the distribution and display manager:

**Ubuntu and some Debian-based systems:**

```bash
sudo systemctl start gdm3
```

**Fedora, RHEL (Red Hat Enterprise Linux), and some other distributions:**

```bash
sudo systemctl start gdm
```

These commands start the **GNOME Display Manager**, which then presents the graphical login screen.

The older `startx` command is also available on some systems. It can start an X11 graphical session directly, bypassing the graphical login screen. It is less common on modern installations but can still be useful in certain troubleshooting or minimal-system scenarios.

---

## Desktop Environment

Unlike Windows or macOS, Linux does not have one fixed desktop interface. The graphical desktop is provided by a separate component called a **desktop environment**.

A desktop environment can be replaced, customized, or even installed alongside other desktop environments on the same Linux system. This flexibility is one of the defining characteristics of Linux.

A desktop environment typically combines:

- A **session manager** — starts and maintains the components of the graphical session.
- A **window manager** — controls how application windows are displayed, moved, resized, positioned, and decorated.
- **Utilities and applications** — provide common functionality such as settings, file management, system menus, and other desktop features.

A simplified view is:

> **Desktop Environment = Session Manager + Window Manager + Utilities and Applications**

Although components from different desktop environments can be mixed, they are generally designed to work together as a coordinated environment.

## GNOME

**GNOME** is one of the most widely used desktop environments in the Linux ecosystem. It is the default desktop environment for several major distributions, including Fedora and RHEL (Red Hat Enterprise Linux).

Ubuntu also uses GNOME as its underlying desktop environment, with additional customizations to provide the Ubuntu desktop experience.

GNOME focuses on a clean, simple, and modern interface with an emphasis on reducing unnecessary configuration.

## KDE Plasma

**KDE Plasma** is another major Linux desktop environment. It is highly configurable and feature-rich.

Its traditional layout includes:

- A taskbar or panel.
- An application launcher.
- A system tray.
- Desktop and window-management features familiar to Windows users.

KDE Plasma has traditionally been associated with SUSE and openSUSE, although openSUSE also provides GNOME and other desktop options.

## Other Desktop Environments

The Linux ecosystem includes many other desktop environments:

### XFCE and LXQt

**XFCE** and **LXQt** are lightweight desktop environments designed to use fewer system resources. They are useful on older hardware, virtual machines, or systems where performance and resource usage are priorities.

### Cinnamon

**Cinnamon** is the default desktop environment on Linux Mint.

It uses a traditional desktop layout with:

- A taskbar.
- An application or start menu.
- A system tray.

This makes it familiar to users transitioning from Windows.

### MATE

**MATE** is a continuation of the traditional GNOME 2 desktop experience. It is designed for users who prefer a more classical desktop interface.

### Desktop Differences

If you use a desktop environment other than the one shown in a tutorial, your screen and menus may look different. This is normal.

The underlying Linux concepts remain largely the same, and most command-line workflows are independent of the desktop environment.

---

## GNOME Tweaks

GNOME's default **Settings** application intentionally provides a streamlined set of configuration options. Some advanced customization options are therefore not exposed there.

**GNOME Tweaks (`gnome-tweaks`)** is a separate utility that provides additional configuration options for the GNOME desktop.

It can be used to:

- Adjust interface, document, and monospace fonts.
- Configure window titlebar layouts and button placement.
- Change icon and cursor themes.
- Configure legacy **GTK (GIMP Toolkit)** application themes.
- Configure keyboard behavior, such as remapping the **Caps Lock** key to act as an additional **Ctrl (Control)** key.
- Manage startup applications.
- Configure certain GNOME Shell settings.

GNOME Tweaks is not installed by default on every distribution, but it is available through the software repositories of most GNOME-based systems.

### Installing GNOME Tweaks

**Ubuntu and other Debian-based distributions:**

```bash
sudo apt install gnome-tweaks
```

**Fedora and other RHEL (Red Hat Enterprise Linux)-based distributions:**

```bash
sudo dnf install gnome-tweaks
```

After installation, launch **Tweaks** from the application menu. On systems that support it, you can also launch it from the run dialog with:

```text
Alt+F2
```

and enter:

```text
gnome-tweaks
```

> **Note:** Older documentation may refer to GNOME Tweaks as `gnome-tweak-tool`. This was its original name; `gnome-tweaks` is the current command.

---

## GNOME Extensions

**GNOME Extensions (`gnome-extensions-app`)** is a dedicated application for managing **GNOME Shell extensions**.

GNOME Shell extensions are add-ons that modify or extend the behavior and appearance of the GNOME desktop.

Extensions can provide features such as:

- Additional system-tray functionality.
- Top-bar customization.
- Improved window tiling.
- Additional desktop controls.
- Features that are not included in the default GNOME interface.

GNOME Extensions can be browsed and installed through the official GNOME Extensions ecosystem.

### GNOME Tweaks vs GNOME Extensions

GNOME Tweaks and GNOME Extensions serve different but complementary purposes:

| Tool | Primary Purpose |
| --- | --- |
| **GNOME Tweaks** | Advanced appearance and desktop behavior settings |
| **GNOME Extensions** | Installing and managing GNOME Shell extensions |

You may use both tools when customizing a GNOME desktop.
