# Linux Kernel Initialization

After the bootloader loads the **kernel** and **initramfs** into RAM, control is handed to the kernel.

The kernel initializes and manages the system hardware:

- **Memory:** Detects and manages physical RAM.
- **CPU:** Detects and initializes available processors.
- **I/O & Storage:** Configures devices such as keyboards, disks, and other peripherals.
- **Early Userspace:** Uses `initramfs` drivers and tools to prepare the system and mount the root filesystem.

## `/sbin/init` and Services

Once hardware initialization is complete and the root filesystem is mounted, the kernel starts **`/sbin/init`**.

- `init` is the **first userspace process (PID 1)**.
- It is the parent/manager of most other userspace processes.
- Starts and manages system services.
- Cleans up processes after they exit.
- Handles services during system shutdown.

On modern **systemd** distributions, `/sbin/init` is typically a symlink to the `systemd` binary.

# From SysVinit to systemd

### SysVinit

Traditional Linux systems used **SysVinit**:

- Started services **sequentially**.
- Used **runlevels** to define system states.
- Relied heavily on shell scripts.
- Booting was slower and configuration was less flexible.

### Why It Changed

SysVinit's sequential approach couldn't efficiently use modern multi-core CPUs. Modern systems also require faster startup, especially for **servers, containers, and embedded systems**.

Two major alternatives emerged:

- **Upstart:** Developed by Ubuntu in 2006; later phased out.
- **systemd:** Adopted by Fedora in 2011 and became the dominant init/service manager.

# systemd

**systemd** is the modern system and service manager used by most Linux distributions.

### Key Features

- **Parallelization:** Starts independent services simultaneously.
- **Declarative configuration:** Uses `.service` unit files instead of complex startup scripts.
- **PID 1:** Runs as the first userspace process and manages other processes.
- **Service management:** Primarily controlled using `systemctl`.

## Common `systemctl` Commands

### Start, Stop & Restart

```bash
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
```

### Enable/Disable at Boot
```bash
sudo systemctl enable apache2
sudo systemctl disable apache2
```

### Check Service Status
```bash
sudo systemctl status apache2
```

.service can usually be omitted; systemctl assumes the unit is a service.
