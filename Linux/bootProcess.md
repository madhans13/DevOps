# ⚙️ Linux Boot Process

## 🎯 Quick Summary

When you power on a Linux system, the **UEFI firmware** performs hardware checks and then loads the bootloader (**GRUB**) from the EFI System Partition.

**GRUB** then loads the Linux kernel and the **initramfs** (temporary root filesystem) into memory.

The **kernel** initializes hardware, drivers, and system memory, mounts the real root filesystem (`/`), and sets up the environment for the OS.

Once the kernel is ready, it starts the first user-space process (**PID 1**) — **systemd**, which then launches all other system services and brings the system to the login or graphical interface.

---

## 📋 Boot Process Stages

### 1️⃣ Power On / BIOS–UEFI

- Performs **POST** (Power-On Self-Test)
- Finds and loads the bootloader from a bootable disk

### 2️⃣ Bootloader (GRUB)

- Loads the Linux kernel image (`vmlinuz`) into memory
- Loads the **initramfs** (temporary root filesystem)
- Passes control and boot parameters to the kernel

### 3️⃣ Kernel

- Initializes CPU, memory, and hardware drivers
- Mounts the real root filesystem (`/`)
- Starts the first user-space process → **PID 1** (usually `systemd`)

### 4️⃣ Init System (systemd / PID 1)

- Reads configuration files from `/etc/systemd/`
- Starts and manages all system services (daemons)
- Switches to appropriate targets (e.g., `multi-user.target`, `graphical.target`)

### 5️⃣ User Space

- Login prompt or display manager (GUI) appears
- User logs in and runs applications

---

## 🧩 Key Components

| Component | Description |
|-----------|-------------|
| **BIOS/UEFI** | Firmware that initializes hardware and locates the bootloader |
| **EFI System Partition** | Special partition containing bootloader files (FAT32 format, ~100-500MB) |
| **GRUB** | Grand Unified Bootloader - loads kernel and initramfs |
| **vmlinuz** | Compressed Linux kernel image stored in `/boot/` |
| **initramfs** | Initial RAM Filesystem - temporary root filesystem during early boot |
| **Kernel** | Core of the OS - manages hardware, memory, and processes |
| **PID 1** | Process ID 1 - the first user-space process (systemd) |
| **systemd** | Modern init system that manages services and system state |
| **Targets** | Systemd's system states (replaces old runlevels) |
| **Daemons** | Background services (sshd, httpd, etc.) |
| **User Space** | Environment where user applications run |

---

## 🔄 Visual Flow

```
┌─────────────────────────────────────────────┐
│          1. Power Button Pressed            │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│     2. BIOS/UEFI Firmware Initialization    │
│        • POST (Hardware Check)              │
│        • Locate Bootloader                  │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         3. GRUB Bootloader Stage            │
│        • Read /boot/grub/grub.cfg           │
│        • Load vmlinuz into RAM              │
│        • Load initramfs into RAM            │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         4. Kernel Initialization            │
│        • Decompress kernel                  │
│        • Initialize hardware drivers        │
│        • Mount initramfs (temporary root)   │
│        • Mount real root filesystem (/)     │
│        • Execute /sbin/init (systemd)       │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         5. systemd (PID 1) Takes Over       │
│        • Read /etc/systemd/ configs         │
│        • Start system services in parallel  │
│        • Mount filesystems from /etc/fstab  │
│        • Reach target (multi-user/graphical)│
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│            6. User Space Ready              │
│        • Login prompt (getty) or GUI        │
│        • User authentication                │
│        • Shell and applications start       │
└─────────────────────────────────────────────┘
```

---

## 💬 Interview Answer

### ⏱️ Short Version (30 seconds)

> *"The Linux boot process has 5 main stages: First, UEFI performs POST and loads GRUB from the EFI partition. GRUB then loads the kernel (vmlinuz) and initramfs into memory. The kernel initializes hardware, mounts the root filesystem, and starts systemd as PID 1. Systemd starts all services and reaches the appropriate target. Finally, the login prompt appears in user space."*

### 📝 Detailed Version (1-2 minutes)

> *"When you power on a Linux system, the UEFI firmware first performs a Power-On Self-Test to verify hardware components like RAM and CPU. It then locates and loads the GRUB bootloader from the EFI System Partition."*
>
> *"GRUB reads its configuration file, displays the boot menu if configured, and loads two critical components into RAM: the compressed kernel image called vmlinuz, and the initramfs which is a temporary root filesystem."*
>
> *"The kernel decompresses itself and initializes the CPU, memory management, and essential hardware drivers. It mounts initramfs as a temporary root, then switches to the actual root filesystem on disk and executes /sbin/init."*
>
> *"On modern systems, /sbin/init is systemd with PID 1, which becomes the parent of all processes. Systemd reads configuration from /etc/systemd/, starts services in parallel for faster boot, mounts remaining filesystems from /etc/fstab, and activates the default target—typically multi-user.target for servers or graphical.target for desktops."*
>
> *"Finally, the system reaches user space where getty provides login prompts or a display manager presents the graphical login screen."*

---

## 🔍 Common Interview Questions

### Q1: What is the difference between BIOS and UEFI?

**Answer:**
- **BIOS** (Basic Input/Output System): Legacy firmware, uses MBR partitioning, supports disks up to 2TB, slower boot process
- **UEFI** (Unified Extensible Firmware Interface): Modern firmware, uses GPT partitioning, supports larger disks (>2TB), faster boot, includes Secure Boot, has a graphical interface

### Q2: What is initramfs and why is it needed?

**Answer:**
Initramfs is a temporary root filesystem loaded into RAM during boot. It contains essential drivers and tools needed to mount the real root filesystem, especially when it's on RAID, LVM, encrypted volumes, or requires specific drivers not built into the kernel.

### Q3: What happens if GRUB fails?

**Answer:**
If GRUB is corrupted or misconfigured, the system won't boot. You can:
1. Boot from a live USB/CD
2. Mount the root filesystem
3. Chroot into the system
4. Reinstall GRUB using `grub-install /dev/sdX`
5. Update GRUB config with `update-grub`

### Q4: What is PID 1 and why is it important?

**Answer:**
PID 1 is the first user-space process started by the kernel. It's typically systemd (or init on older systems). It's critical because:
- It's the parent of all other processes
- It manages system initialization and service startup
- It handles orphaned processes
- If PID 1 crashes, the entire system crashes

### Q5: What are systemd targets?

**Answer:**
Targets are systemd's way of grouping services to define system states (replacing old SysV runlevels):
- `poweroff.target` - Shutdown (runlevel 0)
- `rescue.target` - Single-user mode (runlevel 1)
- `multi-user.target` - Multi-user CLI (runlevel 3)
- `graphical.target` - GUI mode (runlevel 5)
- `reboot.target` - Reboot (runlevel 6)

---

## 📁 Important Boot Files

```
/boot/
├── vmlinuz-5.15.0-56-generic    # Kernel image
├── initramfs-5.15.0-56-generic  # Initial RAM filesystem
├── grub/
│   ├── grub.cfg                 # GRUB configuration (auto-generated)
│   └── grubenv                  # GRUB environment variables
└── System.map-5.15.0-56-generic # Kernel symbol table

/etc/
├── fstab                        # Filesystem mount table
├── default/
│   └── grub                     # GRUB default settings (edit this)
└── systemd/
    ├── system/                  # System unit files
    └── system.conf              # Systemd configuration

/sys/firmware/efi/
└── efivars/                     # UEFI variables (on UEFI systems)
```

---

## 🛠️ Useful Commands

### Check Boot Process

```bash
# View kernel messages
dmesg
dmesg | less
dmesg | grep -i error

# View boot logs with systemd
journalctl -b              # Current boot
journalctl -b -1           # Previous boot
journalctl -k              # Kernel messages only

# Check boot time and analyze
systemd-analyze
systemd-analyze blame      # Show service startup times
systemd-analyze critical-chain
```

### Manage Systemd

```bash
# Check current target
systemctl get-default

# Change default target
systemctl set-default multi-user.target   # Text mode
systemctl set-default graphical.target    # GUI mode

# List all services
systemctl list-units --type=service
systemctl list-unit-files

# Check service status
systemctl status sshd
systemctl is-enabled sshd
systemctl is-active sshd
```

### GRUB Management

```bash
# View GRUB configuration
cat /boot/grub/grub.cfg

# Edit GRUB defaults (then update)
sudo nano /etc/default/grub
sudo update-grub           # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/CentOS

# Reinstall GRUB (from live USB)
sudo mount /dev/sda1 /mnt
sudo grub-install --root-directory=/mnt /dev/sda
```

### Boot Troubleshooting

```bash
# Boot into rescue mode
# Add 'systemd.unit=rescue.target' to kernel parameters in GRUB

# Boot into emergency mode
# Add 'systemd.unit=emergency.target' to kernel parameters

# Boot with different init (bypass systemd)
# Add 'init=/bin/bash' to kernel parameters

# Check for failed services
systemctl --failed

# View logs for specific service
journalctl -u sshd.service
```

---

## 🚨 Common Boot Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **GRUB not found** | "No bootable device" error | Boot from live USB, reinstall GRUB |
| **Kernel panic** | System halts with kernel messages | Check kernel parameters, boot older kernel |
| **Root filesystem not mounted** | "Unable to mount root fs" | Check `/etc/fstab`, verify UUID/device names |
| **Systemd failure** | Drops to emergency shell | Check logs with `journalctl -xb` |
| **Service won't start** | System boots but service fails | Check `systemctl status <service>` and logs |
| **Slow boot** | Takes long to reach login | Use `systemd-analyze blame` to identify slow services |

---

## 📚 Additional Resources

- [Systemd Documentation](https://www.freedesktop.org/wiki/Software/systemd/)
- [GRUB Manual](https://www.gnu.org/software/grub/manual/)
- [Linux Kernel Documentation](https://www.kernel.org/doc/html/latest/)
- [UEFI Specification](https://uefi.org/specifications)
- [Red Hat Boot Process Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/)

---

## ✅ Key Takeaways

1. **5 Main Stages**: UEFI → GRUB → Kernel → systemd → User Space
2. **UEFI** is modern and replaces legacy BIOS
3. **GRUB** is the most common bootloader in Linux
4. **initramfs** is temporary and essential for mounting the real root filesystem
5. **systemd** (PID 1) is the first user-space process and parent of all processes
6. **Targets** replace old runlevels for defining system states
7. Boot process is logged in `/var/log/` and accessible via `journalctl`

---

## 💡 Pro Tips for Interviews

✅ **Always mention the 5 stages** in order  
✅ **Know the difference** between initramfs and root filesystem  
✅ **Understand systemd targets** vs old SysV runlevels  
✅ **Be familiar with** `/boot/`, `/etc/fstab`, and `/etc/systemd/`  
✅ **Know basic troubleshooting** commands like `journalctl` and `systemctl`  
✅ **Explain PID 1's importance** and what happens if it fails  

---

**⭐ Star this repo if it helped you ace your Linux interview!**

---

*Last Updated: November 2025*