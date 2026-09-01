# Linux Boot Process

The Linux boot process starts when the computer is powered on and ends when the OS is fully loaded.

**Main steps:**
1. Power ON
2. BIOS/UEFI
3. MBR or EFI System Partition (ESP)
4. Boot Loader (e.g., GRUB)
5. Kernel
6. Initial RAM Disk (`initramfs`)
7. `/sbin/init` (parent process)
8. Login shell via `getty`
9. GUI (X Window/Wayland)

## BIOS/UEFI

- **BIOS (Basic Input Output System)** is firmware stored on a ROM chip on the motherboard that starts when the computer powers on.
- Performs **POST (Power-On Self-Test)** to initialize and check hardware such as memory, keyboard, and display.
- After POST, it locates and transfers control to the boot loader.
- **UEFI (Unified Extensible Firmware Interface)** is the modern replacement for BIOS and uses the **EFI System Partition (ESP)**.

## MBR, EFI Partition & Boot Loader

- **BIOS/MBR:** Boot loader information starts in the **MBR (Master Boot Record)**, the first 512-byte sector of the disk.
- **UEFI:** Boot loaders are stored as `.efi` applications in the **EFI System Partition (ESP)**.
- Common Linux boot loaders:
  - **GRUB (GRand Unified Boot Loader)** – most common
  - **ISOLINUX** – removable media
  - **U-Boot** – embedded systems

### BIOS vs UEFI

**BIOS:**
`BIOS → MBR → Boot Loader → Kernel → OS`

**UEFI:**
`UEFI → UEFI Boot Loader → Kernel → OS`

## Boot Loader in Action

### BIOS/MBR

1. BIOS loads the code from the MBR.
2. MBR code locates the bootable partition.
3. Loads the second-stage boot loader (e.g., GRUB) into RAM.

### UEFI

1. UEFI reads its boot entries.
2. Locates the EFI System Partition.
3. Launches the selected `.efi` boot loader.

### Second-Stage Boot Loader

Usually located under `/boot`.

1. Displays a boot menu.
2. Loads the selected **kernel** and `initramfs` into RAM.
3. Passes control to the kernel.

The kernel then:

- Decompresses itself.
- Detects and initializes hardware and built-in drivers.

## Initial RAM Disk (`initramfs`)

The kernel needs drivers to access the root filesystem, but those drivers may be stored on the root filesystem itself. `initramfs` solves this problem by providing a temporary filesystem and required tools in RAM.

Main tasks:

- **Driver loading:** Uses `udev` to detect hardware and load required drivers.
- **Filesystem support:** Provides support for filesystems such as Ext4 and XFS.
- **Root filesystem:** Locates, checks, and prepares the root filesystem for mounting.

## Text-Mode Login

After booting, the `init` process starts login prompts using `getty`.

Linux supports multiple **virtual terminals (TTYs)**:

- **Text mode:** `Alt + F1` to `Alt + F6`
- **From GUI:** `Ctrl + Alt + F1` to `Ctrl + Alt + F6`
- **GUI:** Commonly available on `F1` or `F7`, depending on the distribution.

After login, you can access the command shell. If a graphical environment is enabled, the display manager starts the GUI.