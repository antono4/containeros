# ContainerOS Architecture

## Table of Contents
1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Boot Process](#boot-process)
4. [Container Runtime](#container-runtime)
5. [GUI System](#gui-system)
6. [System Services](#system-services)
7. [Storage Layout](#storage-layout)
8. [Networking](#networking)
9. [Security Model](#security-model)

---

## Overview

ContainerOS adalah sistem operasi Linux ringan yang mengadopsi arsitektur RancherOS dengan containerization menyeluruh, dikombinasikan dengan antarmuka pengguna modern berbasis Android.

### Design Principles

1. **Minimalism**: Base system hanya berisi kernel Linux dan Docker runtime (~200MB)
2. **Containerization**: Semua services berjalan dalam containers
3. **Mobility**: Dirancang untuk touchscreen devices, dari Raspberry Pi hingga laptops
4. **Security**: Isolasi container dengan AppArmor profiles
5. **Performance**: Boot <15 detik, app launch <1 detik

---

## System Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                         USER SPACE                             │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                   GUI LAYER                              │  │
│   │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐   │  │
│   │  │Launcher │ │Settings │ │ Files   │ │ App Store   │   │  │
│   │  └────┬────┘ └────┬────┘ └────┬────┘ └──────┬──────┘   │  │
│   │       │           │           │              │          │  │
│   │  ┌────┴───────────┴───────────┴──────────────┴────┐     │  │
│   │  │              Surface Manager (wlroots)         │     │  │
│   │  └─────────────────────┬───────────────────────────┘     │  │
│   │                        │                                 │  │
│   └────────────────────────┼─────────────────────────────────┘  │
│                            │                                     │
│   ┌────────────────────────┼─────────────────────────────────┐  │
│   │                        │       USER DOCKER               │  │
│   │                        │    (Privileged Container)       │  │
│   │                        │                                   │  │
│   │  ┌──────────────────────┴────────────────────────────┐    │  │
│   │  │                 User Containers                   │    │  │
│   │  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐    │    │  │
│   │  │  │ Web App│ │Terminal│ │  IDE    │ │  Game  │    │    │  │
│   │  │  └────────┘ └────────┘ └────────┘ └────────┘    │    │  │
│   │  └─────────────────────────────────────────────────┘    │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                   SYSTEM DOCKER                          │   │
│   │                       (PID 1)                           │   │
│   │                                                          │   │
│   │  ┌────────────┐ ┌────────────┐ ┌────────────┐           │   │
│   │  │ NetworkMgr │ │   Audio    │ │   Bluetooth │           │   │
│   │  │  Container │ │  Container │ │  Container  │           │   │
│   │  └────────────┘ └────────────┘ └────────────┘           │   │
│   │  ┌────────────┐ ┌────────────┐ ┌────────────┐           │   │
│   │  │  Storage   │ │  Logging   │ │  Updates   │           │   │
│   │  │  Container │ │  Container │ │  Container │           │   │
│   │  └────────────┘ └────────────┘ └────────────┘           │   │
│   │                                                          │   │
│   │  ┌────────────────────────────────────────────────────┐  │   │
│   │  │              GUI Container (Weston + Apps)          │  │   │
│   │  │   ┌──────────┐ ┌──────────┐ ┌──────────┐          │  │   │
│   │  │   │ Launcher │ │StatusBar │ │ App Windows│         │  │   │
│   │  │   └──────────┘ └──────────┘ └──────────┘          │  │   │
│   │  └────────────────────────────────────────────────────┘  │   │
│   │                                                          │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                         KERNEL SPACE                             │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                      Linux Kernel                         │   │
│   │                                                          │   │
│   │   ├── cgroups (v2)    - Resource limiting                │   │
│   │   ├── namespaces      - Process isolation                 │   │
│   │   ├── overlayfs       - Filesystem layering               │   │
│   │   ├── netfilter       - Packet filtering                  │   │
│   │   ├── device mapper   - LVM/RAID support                  │   │
│   │   └── wireguard       - VPN support                       │   │
│   │                                                          │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Boot Process

### Stage 1: Hardware Init (0-2s)
```
Power On → UEFI/BIOS → Bootloader (GRUB) → Load Kernel + Initrd
```

### Stage 2: Early Userspace (2-5s)
```
Kernel Boot
    │
    ├── Mounting initrd (/dev, /proc, /sys)
    │
    ├── Loading required kernel modules
    │   ├── ext4, xfs, btrfs (filesystems)
    │   ├── overlay (for container rootfs)
    │   ├── tun/tap (networking)
    │   └── hardware-specific drivers
    │
    ├── Starting udev for device enumeration
    │
    └── Checking root filesystem (/dev/ram0)
```

### Stage 3: System Docker Start (5-8s)
```
Switchroot to persistent filesystem
    │
    ├── Mount /usr as read-only (overlay)
    │
    ├── Start System Docker daemon
    │   └── Runs as PID 1 (or via tini)
    │
    └── Pull/load essential system containers
        ├── network-manager
        ├── audio
        ├── bluetooth
        └── gui
```

### Stage 4: GUI Launch (8-15s)
```
System Docker starts GUI container
    │
    ├── Launch Weston compositor
    │
    ├── Initialize Wayland surfaces
    │
    ├── Load Launcher application
    │
    └── Display Home Screen
```

### Boot Time Breakdown

| Stage | Target | Cumulative |
|-------|--------|------------|
| Hardware Init | 2s | 2s |
| Kernel + Initrd | 3s | 5s |
| System Docker | 3s | 8s |
| GUI Launch | 7s | 15s |

---

## Container Runtime

### Dual Docker Architecture

RancherOS menggunakan dua instance Docker untuk memisahkan system services dari user workloads.

#### System Docker
- **Role**: System init dan service management
- **Startup**: Pertama kali Docker daemon berjalan
- **Management**: Menjalankan critical system services
- **Isolation**: Services terisolasi tapi berbagi kernel

#### User Docker
- **Role**: User application runtime
- **Startup**: Berjalan di dalam privileged container
- **Access**: Socket mounted dari System Docker
- **Isolation**: Full container isolation untuk user apps

### Container Images

#### Base Images
```dockerfile
# os-base: Base OS layer
FROM alpine:3.19
# Contains: kernel modules, firmware, base libraries
# Size: ~150MB
```

```dockerfile
# system-docker: System services container
FROM containeros/os-base:latest
# Contains: Docker daemon, networking stack
# Size: ~80MB
```

```dockerfile
# gui: Android-style GUI container
FROM containeros/os-base:latest
# Contains: Weston, GTK apps, JavaScript runtime
# Size: ~200MB
```

#### System Service Images
```dockerfile
# network-manager container
FROM containeros/os-base:latest
# Contains: NetworkManager, wpa_supplicant, iw
# Exposes: D-Bus socket for IPC
```

```dockerfile
# audio container  
FROM containeros/os-base:latest
# Contains: PulseAudio server
# Exposes: Unix socket for audio streaming
```

### Container Networking

```
┌─────────────────────────────────────────────────────────┐
│                    Container Network                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌──────────────┐     ┌──────────────┐                 │
│   │  User Docker │────▶│ System Docker │                │
│   │  (172.17.0.x)│     │  (bridge)    │                │
│   └──────────────┘     └──────┬───────┘                │
│                                │                         │
│                    ┌────────────┼────────────┐          │
│                    │            │            │          │
│               ┌────▼────┐ ┌─────▼────┐ ┌────▼────┐    │
│               │ Network │ │  Audio   │ │ Storage │    │
│               │ Manager │ │ Service  │ │ Service │    │
│               └─────────┘ └──────────┘ └─────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Volume Management

```
/var/lib/docker      → System Docker storage
/var/lib/containers → User container storage
/home                → Persistent user data
/opt                 → Additional software
```

---

## GUI System

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    GUI Container                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌──────────────────────────────────────────────────┐  │
│   │              JavaScript UI Layer                   │  │
│   │   ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐    │  │
│   │   │Launcher│ │Settings│ │ Files  │ │ Store  │    │  │
│   │   └────┬───┘ └────┬───┘ └────┬───┘ └────┬───┘    │  │
│   │        └──────────┼──────────┼──────────┘        │  │
│   │                     │          │                   │  │
│   │              ┌──────▼──────────▼──────┐            │  │
│   │              │    UI Framework (GIO)   │            │  │
│   │              │  (GTK4 / JavaScript)     │            │  │
│   │              └──────────┬──────────────┘            │  │
│   └─────────────────────────┼───────────────────────────┘  │
│                              │                              │
│   ┌─────────────────────────┼───────────────────────────┐  │
│   │                  GTK4 Toolkit                         │  │
│   │  ┌──────────┐ ┌──────────┐ ┌──────────┐              │  │
│   │  │Adwaita   │ │  GLib    │ │GObject   │              │  │
│   │  │Theme     │ │          │ │Introspect│              │  │
│   │  └──────────┘ └──────────┘ └──────────┘              │  │
│   └─────────────────────────┬───────────────────────────────┘  │
│                              │                                │
│   ┌─────────────────────────┼───────────────────────────────┐│
│   │                 Wayland Backend                          ││
│   │         (EGL + OpenGL ES 2.0)                            ││
│   └─────────────────────────┬───────────────────────────────┘│
│                              │                                │
│   ┌─────────────────────────┴───────────────────────────────┐│
│   │                   Weston Compositor                      ││
│   │   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ││
│   │   │ Shell    │ │  Input   │ │ Output   │ │  DRM     │ ││
│   │   │ Manager  │ │ Handler  │ │ Manager  │ │ Backend  │ ││
│   │   └──────────┘ └──────────┘ └──────────┘ └──────────┘ ││
│   └─────────────────────────┬───────────────────────────────┘│
│                              │                                │
│   ┌─────────────────────────┴───────────────────────────────┐│
│   │                 Linux Kernel                            ││
│   │            (DRM/KMS, Input, GEM/GBM)                    ││
│   └─────────────────────────────────────────────────────────┘│
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Component Details

#### 1. Weston Compositor
- Lightweight Wayland compositor
- Provides shell extension for app windows
- Manages input/output devices
- Uses DRM backend untuk hardware acceleration

#### 2. UI Framework (GTK4 + JavaScript)
- GTK4 untuk native look
- JavaScript bindings untuk rapid development
- GObject Introspection untuk bindings
- Adwaita theme untuk Material Design look

#### 3. Launcher Application
- Main entry point untuk user
- Displays app icons dalam grid layout
- Manages home screens (5 screens default)
- Handles app drawer overlay

#### 4. Window Manager
- XDG Shell untuk window management
- Title bar dengan window controls
- Drag-and-drop window positioning
- Minimize/maximize/close buttons

### Touch Input Handling

```
Touch Event
    │
    ├── libinput (kernel input handler)
    │   ├── Touch down/up/motion events
    │   ├── Multi-touch gesture detection
    │   └── Pointer emulation for stylus/mouse
    │
    ├── Wayland Protocol
    │   ├── wl_touch
    │   ├── wl_pointer  
    │   └── wp_gesture (for pinch/zoom)
    │
    └── JavaScript UI Framework
        ├── Touch event listeners
        ├── Gesture handlers
        └── Animation controller
```

### Android-Style Components

| Component | Description | Technologies |
|-----------|-------------|--------------|
| StatusBar | Top bar with clock, icons | GTK HeaderBar |
| NavigationBar | Back/Home/Recents buttons | Custom GTK widget |
| AppDrawer | Full app list overlay | GTK Stack + Clutter |
| NotificationShade | Notifications panel | GTK Revealer |
| RecentApps | App switcher view | GTK Grid |
| LockScreen | Security/clock screen | GTK Stack |

---

## System Services

### Containerized Services

```
┌─────────────────────────────────────────────────────────┐
│               System Docker Containers                   │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │   Network    │  │    Audio    │  │   Bluetooth │    │
│  │   Manager    │  │   Server    │  │    Service  │    │
│  │              │  │             │  │             │    │
│  │  - nmcli     │  │  - Pulse    │  │  - bluez    │    │
│  │  - iw        │  │  - ALSA     │  │  - bluetooth │   │
│  │  - wpa_sup   │  │             │  │             │    │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘    │
│         │                │                │            │
│         └────────────────┼────────────────┘            │
│                          │                             │
│                    D-Bus Socket                         │
│                   (IPC Channel)                         │
│                          │                             │
│                          ▼                             │
│               ┌─────────────────────┐                   │
│               │    GUI Container    │                   │
│               │   (Access Services) │                   │
│               └─────────────────────┘                   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Service Descriptions

#### NetworkManager Container
- Manages all network connections
- WiFi scanning dan authentication
- Ethernet configuration
- Mobile data (PPP)
- VPN connections
- **Exposed**: D-Bus, /etc/NetworkManager

#### Audio Container
- PulseAudio sound server
- ALSA compatibility layer
- Bluetooth audio (A2DP)
- HDMI audio output
- **Exposed**: Unix socket /run/user/1000/pulse

#### Bluetooth Container
- BlueZ Bluetooth stack
- Pairing management
- Device discovery
- A2DP/HFP profiles
- **Exposed**: D-Bus, /var/lib/bluetooth

#### Storage Container
- UDisks2 for disk management
- Mount/unmount handling
- Partition operations
- USB device detection
- **Exposed**: D-Bus, /media

#### Logging Container
- Journald logging service
- Log rotation
- Remote syslog (optional)
- Crash dump collection
- **Exposed**: /var/log, journal socket

---

## Storage Layout

### Partition Layout

```
┌─────────────────────────────────────────────────────────┐
│                    Storage Layout                        │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  /dev/sda (or eMMC)                                      │
│  ┌────────────┬─────────────────────┬────────────────┐ │
│  │   BIOS     │      ROOTFS         │    DATA        │ │
│  │   BOOT     │     ( squashfs )    │   ( ext4 )     │ │
│  │  (FAT32)   │                     │                │ │
│  │            │                     │                │ │
│  │  - Kernel  │   Base OS (ro)      │  - /home       │ │
│  │  - Initrd  │   ~150MB            │  - /var/lib    │ │
│  │  - GRUB    │                     │  - /etc        │ │
│  │            │                     │                │ │
│  │  256MB     │                     │   Remaining    │ │
│  └────────────┴─────────────────────┴────────────────┘ │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Directory Structure

```
/
├── boot/                    # Boot partition (FAT32)
│   ├── grub/
│   │   └── grub.cfg
│   ├── vmlinuz             # Kernel image
│   └── initrd.img          # Initial ramdisk
│
├── system/                 # Base OS (squashfs, read-only)
│   ├── bin/
│   ├── sbin/
│   ├── usr/
│   └── lib/
│
├── data/                   # Data partition (ext4)
│   ├── home/               # User home directories
│   │   └── rancher/
│   ├── var/
│   │   ├── lib/            # Container storage
│   │   │   ├── docker/
│   │   │   └── containers/
│   │   └── log/            # System logs
│   ├── etc/
│   │   ├── system-docker/  # System Docker config
│   │   └── user-docker/   # User Docker config
│   └── opt/                # Additional software
│
├── overlay/                # Union mount point
│   ├── usr/                # Overlay for /usr
│   └── etc/                # Overlay for /etc
│
└── tmp/                    # Temporary files
    └── system-docker/      # System Docker runtime
```

### Overlay Filesystem

```
Union Mount (overlay)
    │
    ├── upper: /overlay/usr (rw)
    │   └── User modifications
    │
    └── lower: /system/usr (ro)
        └── Base system files
```

---

## Networking

### Network Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Network Stack                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   User Containers (172.17.0.0/16)                        │
│   ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│   │  App 1   │ │  App 2   │ │  App 3   │              │
│   │  eth0    │ │  eth0    │ │  eth0    │              │
│   └────┬─────┘ └────┬─────┘ └────┬─────┘              │
│        │            │            │                     │
│        └────────────┼────────────┘                     │
│                     │                                   │
│               docker0 bridge                            │
│               (172.17.0.1)                              │
│                     │                                   │
├─────────────────────┼───────────────────────────────────┤
│                     │                                   │
│   System Docker     │                                   │
│   ┌─────────────────┴─────────────────────────┐       │
│   │         bridge (172.18.0.1)               │        │
│   │                                            │        │
│   │  ┌──────────┐ ┌──────────┐ ┌──────────┐ │       │
│   │  │ Network  │ │  Audio   │ │Bluetooth  │ │       │
│   │  │ Manager  │ │ Service  │ │ Service   │ │       │
│   │  └────┬─────┘ └──────────┘ └──────────┘ │       │
│   │       │                                   │       │
│   │       └──▶ Physical Interface (eth0/wlan0)       │
│   │                                            │       │
│   └────────────────────────────────────────────┘       │
│                                                          │
│   Host Network                                           │
│   ┌──────────┐ ┌──────────┐                             │
│   │   eth0   │ │  wlan0   │                             │
│   │ (DHCP)   │ │ (WiFi)   │                             │
│   └──────────┘ └──────────┘                             │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Network Configuration

```yaml
# /etc/networking.yaml
network:
  version: 1
  config:
    - type: physical
      name: eth0
      subnets:
        - type: dhcp
    - type: wifi
      name: wlan0
      enabled: true
      subnets:
        - type: dhcp
      wifi:
        ssid: "MyNetwork"
        auth:
          key-management: eap
          password: "secret"
```

---

## Security Model

### Container Isolation

```
┌─────────────────────────────────────────────────────────┐
│                 Container Isolation                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   App Container                                          │
│   ┌─────────────────────────────────────────────────┐  │
│   │  User: nobody (65534)                            │  │
│   │  Groups: nogroup                                │  │
│   │                                                  │  │
│   │  Capabilities (dropped):                         │  │
│   │  ✗ CAP_SYS_ADMIN                                │  │
│   │  ✗ CAP_NET_ADMIN                                │  │
│   │  ✗ CAP_SYS_MODULE                               │  │
│   │                                                  │  │
│   │  Syscalls (seccomp):                            │  │
│   │  ✗ mount                                        │  │
│   │  ✗ umount2                                      │  │
│   │  ✗ init_module                                  │  │
│   │                                                  │  │
│   │  AppArmor Profile: containeros-app               │  │
│   │  - Deny mount, sysctl, reboot                    │  │
│   │  - Allow read /home, /tmp                        │  │
│   │  - Allow network (client)                        │  │
│   └─────────────────────────────────────────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### AppArmor Profiles

#### System Services
```apparmor
# /etc/apparmor.d/containeros-network-manager
profile containeros-network-manager {
  # Allow NetworkManager to manage interfaces
  network netlink,
  capability net_admin,
  
  # Allow D-Bus communication
  dbus,
  
  # Read system configuration
  /etc/NetworkManager/** r,
  /var/lib/NetworkManager/** rw,
  
  # Execute binaries
  /usr/sbin/NetworkManager ix,
  /usr/bin/nmcli ix,
}
```

#### User Applications
```apparmor
# /etc/apparmor.d/containeros-app
profile containeros-app {
  # No network admin
  deny capability net_admin,
  
  # Read-only home
  /home/* r,
  
  # Read-only system
  /usr r,
  
  # Temporary files
  /tmp/** rw,
  
  # Network client only
  network inet stream,
  network inet6 stream,
}
```

### Seccomp Profile

```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "syscalls": [
    {"names": ["read", "write", "open", "close"], "action": "SCMP_ACT_ALLOW"},
    {"names": ["brk", "mmap", "mprotect", "madvise"], "action": "SCMP_ACT_ALLOW"},
    {"names": ["socket", "connect", "accept"], "action": "SCMP_ACT_ALLOW"},
    {"names": ["mount", "umount2", "init_module"], "action": "SCMP_ACT_ERRNO"}
  ]
}
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-07-06 | Initial architecture document |

---

## See Also

- [Build Guide](BUILD.md)
- [Project README](../README.md)
- [RancherOS Architecture](https://github.com/rancher/os)
