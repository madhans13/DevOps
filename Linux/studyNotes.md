# 🐧 Linux Concepts - Personal Reference Guide

> **Personal study notes for Linux fundamentals, system administration, and DevOps concepts**

---

## 📋 Table of Contents

1. [Linux Boot Process](#-linux-boot-process)
2. [Kernel, Init & Services](#-kernel-init--services)
3. [Linux Filesystem Hierarchy](#-linux-filesystem-hierarchy)
4. [File Permissions & Ownership](#-file-permissions--ownership)
5. [Process Management](#-process-management)
6. [Package Management](#-package-management)
7. [Network Configuration](#-network-configuration)
8. [System Monitoring](#-system-monitoring)
9. [Log Management](#-log-management)
10. [Essential Commands Cheat Sheet](#-essential-commands-cheat-sheet)
11. [Troubleshooting Guide](#-troubleshooting-guide)

---

## 🔌 Linux Boot Process

### Boot Sequence Overview
```
BIOS/UEFI → MBR/GPT → Bootloader (GRUB) → Kernel → initrd/initramfs → init/systemd → Services → Login
```

### Key Components

| Component | Purpose | Location |
|-----------|---------|----------|
| **BIOS/UEFI** | Firmware that initializes hardware | Motherboard ROM |
| **MBR/GPT** | Partition table with bootloader info | First sector of disk |
| **GRUB** | Bootloader that loads kernel | `/boot/grub/` |
| **Kernel** | Core OS managing hardware/processes | `/boot/vmlinuz-*` |
| **initrd/initramfs** | Temporary root filesystem | `/boot/initrd.img-*` |
| **init/systemd** | First process (PID 1) | `/sbin/init` or `/lib/systemd/systemd` |

### Boot Mode Check
```bash
# Check if system is UEFI or BIOS
[ -d /sys/firmware/efi ] && echo "UEFI" || echo "BIOS"

# View boot messages
dmesg | less
journalctl -b
```

---

## ⚙️ Kernel, Init & Services

### Kernel Information
```bash
uname -r                    # Kernel version
uname -a                    # All system info
cat /proc/version           # Detailed kernel info
lsmod                       # List loaded modules
modinfo <module>            # Module information
```

### systemd Service Management
```bash
# Service Control
sudo systemctl start <service>
sudo systemctl stop <service>
sudo systemctl restart <service>
sudo systemctl reload <service>
sudo systemctl enable <service>      # Start at boot
sudo systemctl disable <service>     # Don't start at boot
sudo systemctl status <service>

# System State
systemctl list-units --type=service
systemctl list-unit-files
systemctl get-default               # Current target
sudo systemctl set-default <target> # Change default target
```

### systemd Targets (Runlevels)
| Target | Description | Old Runlevel |
|--------|-------------|--------------|
| `poweroff.target` | System shutdown | 0 |
| `rescue.target` | Single-user mode | 1 |
| `multi-user.target` | Multi-user, no GUI | 3 |
| `graphical.target` | Multi-user with GUI | 5 |
| `reboot.target` | System restart | 6 |

---

## 📁 Linux Filesystem Hierarchy

### Standard Directory Structure
```
/
├── bin/          # Essential binaries
├── boot/         # Boot files (kernel, initrd, grub)
├── dev/          # Device files
├── etc/          # System configuration
├── home/         # User home directories
├── lib/          # Essential libraries
├── media/        # Removable media mount points
├── mnt/          # Temporary mount points
├── opt/          # Optional software
├── proc/         # Process information (virtual)
├── root/         # Root user home
├── sbin/         # System binaries
├── srv/          # Service data
├── sys/          # System information (virtual)
├── tmp/          # Temporary files
├── usr/          # User programs and data
│   ├── bin/      # User binaries
│   ├── lib/      # User libraries
│   ├── local/    # Local programs
│   └── share/    # Shared data
└── var/          # Variable data (logs, mail, cache)
    ├── log/      # Log files
    ├── mail/     # Mail storage
    └── cache/    # Cache files
```

### Filesystem Types
| Type | Description | Use Case |
|------|-------------|----------|
| `ext4` | Fourth extended filesystem | Most Linux distributions |
| `xfs` | High-performance 64-bit filesystem | RHEL/CentOS default |
| `btrfs` | B-tree filesystem with snapshots | SUSE default |
| `zfs` | Advanced filesystem with RAID | Servers, storage systems |
| `tmpfs` | Memory-based filesystem | `/tmp`, `/run` |

### Filesystem Commands
```bash
# View filesystem info
df -h                       # Disk usage
df -i                       # Inode usage
du -sh <directory>          # Directory size
lsblk                       # Block devices
mount | column -t           # Mounted filesystems
findmnt                     # Mount points in tree format

# Mount/Unmount
sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb
sudo mount -a               # Mount all in /etc/fstab
```

---

## 🔒 File Permissions & Ownership

### Permission Types
| Permission | File | Directory |
|------------|------|-----------|
| **r (4)** | Read file content | List directory contents |
| **w (2)** | Write/modify file | Create/delete files in directory |
| **x (1)** | Execute file | Access directory |

### Permission Notation
```bash
# Symbolic notation
-rwxr-xr-x
# │└┬┘└┬┘└┬┘
# │ │  │  └── Other permissions
# │ │  └───── Group permissions  
# │ └──────── Owner permissions
# └─────────── File type (- = file, d = directory, l = link)

# Numeric notation
chmod 755 file.txt    # rwxr-xr-x
chmod 644 file.txt    # rw-r--r--
chmod 600 file.txt    # rw-------
```

### Ownership Commands
```bash
# Change ownership
sudo chown user:group file.txt
sudo chown -R user:group directory/
sudo chgrp group file.txt

# Change permissions
chmod u+x file.txt              # Add execute for user
chmod g-w file.txt              # Remove write for group
chmod o=r file.txt              # Set read-only for others
chmod a+r file.txt              # Add read for all

# Special permissions
chmod +t directory/             # Sticky bit
chmod g+s directory/            # Set group ID
chmod u+s file                  # Set user ID
```

---

## 🔄 Process Management

### Process Information
```bash
# View processes
ps aux                          # All processes
ps -ef                          # Full format
top                             # Real-time process viewer
htop                            # Enhanced top (install: apt install htop)
pstree                          # Process tree
jobs                            # Active jobs in current shell

# Process details
ps -p <PID>                     # Specific process
pgrep <process_name>            # Find PID by name
pidof <process_name>            # Find PID by name
```

### Process Control
```bash
# Start processes
command &                       # Run in background
nohup command &                 # Run detached from terminal
screen -S session_name          # Create screen session
tmux new -s session_name        # Create tmux session

# Control processes
kill <PID>                      # Terminate process (SIGTERM)
kill -9 <PID>                   # Force kill (SIGKILL)
killall <process_name>          # Kill all instances
pkill <process_name>            # Kill by name

# Job control
Ctrl+Z                          # Suspend current job
bg                              # Resume in background
fg                              # Resume in foreground
disown                          # Remove from job table
```

### System Information
```bash
# System resources
free -h                         # Memory usage
uptime                          # System uptime and load
lscpu                           # CPU information
lsblk                           # Block devices
lsusb                           # USB devices
lspci                           # PCI devices
```

---

## 📦 Package Management

### Ubuntu/Debian (APT)
```bash
# Package operations
sudo apt update                 # Update package list
sudo apt upgrade                # Upgrade packages
sudo apt install <package>      # Install package
sudo apt remove <package>       # Remove package
sudo apt purge <package>        # Remove package and config
sudo apt autoremove             # Remove unused packages

# Package information
apt list --installed            # List installed packages
apt search <package>            # Search packages
apt show <package>              # Package details
dpkg -l                         # List installed packages
dpkg -S <file>                  # Find package containing file
```

### CentOS/RHEL (YUM/DNF)
```bash
# Package operations
sudo yum update                 # Update packages
sudo yum install <package>      # Install package
sudo yum remove <package>       # Remove package
sudo yum search <package>       # Search packages
sudo yum info <package>         # Package information

# DNF (newer systems)
sudo dnf update
sudo dnf install <package>
sudo dnf remove <package>
```

---

## 🌐 Network Configuration

### Network Information
```bash
# Network interfaces
ip addr show                    # Show IP addresses
ip link show                    # Show network interfaces
ifconfig                        # Legacy command (may need net-tools)
hostname -I                     # Show IP addresses
hostname                        # Show hostname

# Network connectivity
ping <host>                     # Test connectivity
traceroute <host>              # Trace route to host
nslookup <domain>              # DNS lookup
dig <domain>                   # Advanced DNS lookup
```

### Network Configuration Files
```bash
# Ubuntu/Debian
/etc/network/interfaces         # Network interfaces (legacy)
/etc/netplan/                   # Netplan configuration

# CentOS/RHEL
/etc/sysconfig/network-scripts/ # Network scripts
/etc/hosts                      # Host mappings
/etc/resolv.conf               # DNS configuration
```

### Firewall Management
```bash
# UFW (Ubuntu)
sudo ufw enable                 # Enable firewall
sudo ufw status                 # Check status
sudo ufw allow 22               # Allow SSH
sudo ufw allow 80/tcp           # Allow HTTP
sudo ufw deny 23                # Deny telnet

# firewalld (CentOS/RHEL)
sudo firewall-cmd --state       # Check status
sudo firewall-cmd --list-all    # List rules
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload      # Reload rules
```

---

## 📊 System Monitoring

### Resource Monitoring
```bash
# CPU and Memory
top                             # Real-time system stats
htop                            # Enhanced top
vmstat 1                        # Virtual memory stats
iostat 1                        # I/O statistics
sar -u 1                        # CPU usage over time

# Disk Usage
df -h                           # Filesystem usage
du -sh *                        # Directory sizes
iotop                           # I/O usage by process
lsof                            # Open files
```

### Performance Tools
```bash
# System load
uptime                          # Load averages
w                               # Who is logged in and load
tload                           # Graphical load average

# Memory analysis
free -h                         # Memory usage
cat /proc/meminfo               # Detailed memory info
pmap <PID>                      # Process memory map
```

---

## 📝 Log Management

### Log Locations
```bash
/var/log/syslog                 # System log (Ubuntu/Debian)
/var/log/messages               # System log (CentOS/RHEL)
/var/log/auth.log               # Authentication log
/var/log/kern.log               # Kernel log
/var/log/apache2/               # Apache logs
/var/log/nginx/                 # Nginx logs
```

### journalctl (systemd logs)
```bash
# View logs
journalctl                      # All logs
journalctl -xe                  # Recent logs with explanation
journalctl -f                   # Follow logs (tail -f style)
journalctl -u <service>         # Service-specific logs
journalctl --since "2 hours ago"
journalctl --until "10 minutes ago"
journalctl -p err               # Error level logs only
journalctl --disk-usage         # Log disk usage
```

### Log Rotation
```bash
# logrotate configuration
/etc/logrotate.conf             # Main config
/etc/logrotate.d/               # Service-specific configs

# Manual rotation
sudo logrotate -f /etc/logrotate.conf
```

---

## 🚀 Essential Commands Cheat Sheet

### File Operations
```bash
# Navigation
cd /path/to/directory
cd ~                            # Home directory
cd -                            # Previous directory
pwd                             # Current directory

# File manipulation
ls -la                          # List files with details
cp source dest                  # Copy file
mv source dest                  # Move/rename file
rm file                         # Delete file
rm -rf directory                # Delete directory recursively
mkdir -p path/to/dir            # Create directory with parents
touch file.txt                  # Create empty file
```

### Text Processing
```bash
# View files
cat file.txt                    # Display file content
less file.txt                   # Page through file
head -n 10 file.txt             # First 10 lines
tail -f file.txt                # Follow file changes
grep "pattern" file.txt         # Search in file
grep -r "pattern" /path/        # Recursive search
```

### System Information
```bash
# Hardware info
lscpu                           # CPU information
lsmem                           # Memory information
lsblk                           # Block devices
lsusb                           # USB devices
lspci                           # PCI devices
dmidecode                       # Hardware details
```

### Archive and Compression
```bash
# tar archives
tar -czf archive.tar.gz dir/    # Create compressed archive
tar -xzf archive.tar.gz         # Extract compressed archive
tar -tzf archive.tar.gz         # List archive contents

# Other compression
zip -r archive.zip dir/         # Create zip archive
unzip archive.zip               # Extract zip archive
```

---

## 🔧 Troubleshooting Guide

### System Issues
```bash
# Boot problems
sudo journalctl -xb            # Boot logs
sudo dmesg                     # Kernel messages
sudo fsck /dev/sda1            # Check filesystem

# Performance issues
top                            # Check CPU usage
iotop                          # Check I/O usage
netstat -tulpn                 # Check network connections
ss -tulpn                      # Modern netstat alternative
```

### Service Problems
```bash
# Service won't start
sudo systemctl status <service>
sudo journalctl -u <service>
sudo systemctl reset-failed <service>

# Permission issues
ls -la /path/to/file
sudo chown user:group file
sudo chmod 644 file
```

### Network Issues
```bash
# Connectivity problems
ping 8.8.8.8                   # Test internet
ping gateway_ip                # Test local network
traceroute destination         # Trace network path
nslookup domain.com            # Test DNS resolution
```

### Disk Issues
```bash
# Disk full
df -h                          # Check disk usage
du -sh /var/log/*              # Check log sizes
sudo find / -type f -size +100M # Find large files
sudo logrotate -f /etc/logrotate.conf # Rotate logs
```

---

## 📚 Quick Reference

### Important Configuration Files
```bash
/etc/passwd                     # User accounts
/etc/shadow                     # User passwords
/etc/group                      # User groups
/etc/hosts                      # Host mappings
/etc/fstab                      # Filesystem table
/etc/crontab                    # Cron jobs
/etc/ssh/sshd_config           # SSH daemon config
/etc/sudoers                    # Sudo configuration
```

### Environment Variables
```bash
echo $PATH                      # Executable paths
echo $HOME                      # Home directory
echo $USER                      # Current user
export VAR=value               # Set variable
env                            # Show all variables
```

### Keyboard Shortcuts
```bash
Ctrl+C                         # Interrupt process
Ctrl+Z                         # Suspend process
Ctrl+D                         # End of input/logout
Ctrl+L                         # Clear screen
Ctrl+R                         # Search command history
Ctrl+A                         # Beginning of line
Ctrl+E                         # End of line
```

---

## 🎯 Study Goals Checklist

- [ ] Understand the complete boot process
- [ ] Master systemd service management
- [ ] Navigate filesystem hierarchy confidently
- [ ] Manage file permissions and ownership
- [ ] Monitor system performance effectively
- [ ] Troubleshoot common system issues
- [ ] Configure basic networking
- [ ] Manage packages efficiently
- [ ] Analyze logs using journalctl
- [ ] Automate tasks with cron jobs

---

## 📖 Additional Resources

- [Linux Documentation Project](https://tldp.org/)
- [Red Hat Enterprise Linux Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Arch Linux Wiki](https://wiki.archlinux.org/)

---

**Last Updated**: July 2025  
**Author**: Personal Study Notes  
**Purpose**: Linux system administration and DevOps reference