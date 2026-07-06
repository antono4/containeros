# PRD: ContainerOS - Android-Style Container OS

## Introduction

**ContainerOS** adalah sistem operasi Linux minimal yang menjalankan seluruh OS sebagai container Docker, dengan antarmuka pengguna grafis (GUI) berbasis Android. Proyek ini terinspirasi dari RancherOS yang terkenal, tetapi dengan sentuhan modern: pengalaman pengguna yang familiar seperti Android di smartphone/tablet, namun dengan fleksibilitas dan keamanan containerization penuh.

Tujuan utama adalah menciptakan OS yang:
- **Minimal**: Hanya komponen esensial yang diperlukan (~200MB base image)
- **Secure**: Semua services berjalan dalam container yang terisolasi
- **Portable**: Bootable di berbagai hardware (x86, ARM, termasuk Raspberry Pi)
- **User-Friendly**: GUI Android-style yang intuitif untuk pengguna akhir
- **Fast**: Boot time cepat, resource efficient

## Goals

1. Membangun base OS minimal berbasis Alpine/Buildroot yang bootable dalam <10 detik
2. Mengimplementasikan dual-Docker architecture (System Docker + User Docker) seperti RancherOS
3. Menciptakan Android-style GUI menggunakan Wayland compositor + custom launcher
4. Mendukung touchscreen input dan gesture controls
5. Menyediakan App Store berbasis Docker untuk instalasi aplikasi
6. Mendukung cloud-init untuk provisioning
7. Mencapai kompatibilitas dengan Docker Hub images

---

## User Stories

### US-001: Boot ke Desktop GUI
**Description:** Sebagai user, saya ingin menyalakan device dan langsung melihat GUI Android-style sehingga saya bisa langsung menggunakan komputer tanpa konfigurasi awal.

**Acceptance Criteria:**
- [ ] Device boot langsung ke lockscreen/home screen dalam waktu <15 detik
- [ ] UI menampilkan widget waktu, tanggal, dan status system
- [ ] User bisa unlock device dengan swipe gesture

### US-002: Launch dan Manage Apps
**Description:** Sebagai user, saya ingin membuka aplikasi dari app drawer dengan tap icon sehingga saya bisa menggunakan aplikasi dengan familiar gesture Android.

**Acceptance Criteria:**
- [ ] App drawer accessible via swipe up dari home screen
- [ ] Apps launch dengan animation fade-in
- [ ] Multi-window support untuk split-screen mode
- [ ] Apps bisa di-close dengan swipe up dari recent apps

### US-003: Install Apps dari App Store
**Description:** Sebagai user, saya ingin menginstall aplikasi baru dari App Store dengan satu tap sehingga saya bisa memperluas fungsionalitas sistem dengan mudah.

**Acceptance Criteria:**
- [ ] App Store menampilkan daftar aplikasi dengan screenshots dan rating
- [ ] Install progress terlihat dengan progress bar
- [ ] App icon muncul di app drawer setelah install
- [ ] Uninstall bisa dilakukan dengan long-press + drag ke uninstall zone

### US-004: System Settings
**Description:** Sebagai user, saya ingin mengakses settings untuk mengkonfigurasi WiFi, display, accounts, dan preferences.

**Acceptance Criteria:**
- [ ] Settings app dengan category-based navigation
- [ ] WiFi configuration dengan available networks list
- [ ] Display settings: brightness, wallpaper, theme
- [ ] System info menampilkan storage, memory, CPU usage

### US-005: Container Management
**Description:** Sebagai user, saya ingin melihat dan mengelola Docker containers melalui GUI sehingga saya bisa monitor services tanpa CLI.

**Acceptance Criteria:**
- [ ] Containers app menampilkan list semua containers (system + user)
- [ ] Status indicators: running (green), stopped (gray), error (red)
- [ ] Actions: start, stop, restart, delete container
- [ ] Logs viewer untuk troubleshooting

### US-006: Notification System
**Description:** Sebagai user, saya ingin menerima notifications dari apps di notification shade sehingga saya tetap informed tanpa mengganggu workflow.

**Acceptance Criteria:**
- [ ] Notification shade accessible via swipe down dari status bar
- [ ] Notifications tampil dengan app icon, title, dan preview text
- [ ] Tap notification untuk open related app
- [ ] Clear all button untuk dismiss notifications

### US-007: Quick Settings Panel
**Description:** Sebagai user, saya ingin akses quick settings (WiFi, Bluetooth, Torch) dari status bar sehingga saya bisa toggle fitur umum dengan cepat.

**Acceptance Criteria:**
- [ ] Expanded status bar menampilkan grid toggle buttons
- [ ] Long-press tile untuk open full settings
- [ ] Brightness slider di quick settings

### US-008: Files dan Storage Management
**Description:** Sebagai user, saya ingin manage files dan melihat storage usage melalui Files app.

**Acceptance Criteria:**
- [ ] File browser dengan folder navigation
- [ ] Storage breakdown visualization (pie chart)
- [ ] Mount external storage (USB, SD card)
- [ ] Support USB mass storage access

---

## Functional Requirements

### FR-1: Boot System
- FR-1.1: Initramfs-based boot process
- FR-1.2: Kernel + initrd untuk early userspace
- FR-1.3: Support untuk UEFI dan legacy BIOS boot
- FR-1.4: Boot to GUI dalam <15 detik pada hardware modern

### FR-2: Container Runtime
- FR-2.1: System Docker daemon berjalan sebagai PID 1 atau process init
- FR-2.2: User Docker berjalan dalam privileged container
- FR-2.3: Docker socket untuk CLI access
- FR-2.4: Container networking dengan bridge mode
- FR-2.5: Volume management untuk persistent storage

### FR-3: GUI System
- FR-3.1: Wayland compositor (wlroots-based untuk lightweight)
- FR-3.2: Weston sebagai reference compositor
- FR-3.3: OpenGL ES rendering untuk smooth animations
- FR-3.4: Touch input handling via libinput
- FR-3.5: Multi-touch gesture recognition

### FR-4: Android-Style UI Components
- FR-4.1: Status bar dengan clock, battery, WiFi icons
- FR-4.2: Navigation bar dengan back, home, recents buttons
- FR-4.3: App launcher dengan grid of icons
- FR-4.4: Notification shade dengan expandable panel
- FR-4.5: Recent apps overview dengan screenshots
- FR-4.6: Lock screen dengan clock widget

### FR-5: Application Framework
- FR-5.1: WebKitGTK-based web app container
- FR-5.2: XDG App runtime untuk Linux desktop apps
- FR-5.3: Flatpak/Flathub integration untuk app distribution
- FR-5.4: App permissions system

### FR-6: System Services (System Containers)
- FR-6.1: NetworkManager container untuk networking
- FR-6.2: PulseAudio container untuk audio
- FR-6.3: bluetoothd container untuk Bluetooth
- FR-6.4: UDisks2 container untuk storage management
- FR-6.5: systemd-logind container untuk session management

### FR-7: System Configuration
- FR-7.1: cloud-init support untuk automated provisioning
- FR-7.2: ROS CLI tool untuk command-line management
- FR-7.3: Web-based admin panel
- FR-7.4: Config persistence di /usr/local partition

---

## Non-Goals

- **No Google Play Services** - Fokus pada open-source ecosystem
- **No Android runtime (ART)** - Bukan emulator Android, tapi OS baru dengan UI Android-style
- **No full desktop environment** - Didesain untuk touchscreen, bukan mouse-heavy workflow
- **No multi-user support v1** - Single user focus untuk simplicity
- **No full SELinux enforcement** - AppArmor untuk simplicity

---

## Design Considerations

### UI/UX Guidelines

#### Visual Design
- **Design Language**: Material Design 3-inspired dengan rounded corners, subtle shadows
- **Color Palette**: 
  - Primary: #1A73E8 (Google Blue)
  - Secondary: #34A853 (Green)
  - Background: #FFFFFF (light) / #121212 (dark)
  - Surface: #F5F5F5 (light) / #1E1E1E (dark)
- **Typography**: Roboto font family
- **Icons**: Material Design icons

#### Interaction Patterns
- **Gestures**:
  - Swipe up: Open app drawer
  - Swipe down: Notification shade
  - Swipe left/right: Navigate between home screens
  - Long press: App options / move icon
  - Drag to trash: Uninstall app
- **Animations**:
  - Page transitions: Shared element transitions
  - App launch: Scale + fade animation
  - Notification: Slide down with bounce

#### Layout System
- **Grid**: 5x7 icon grid on home screen
- **Widget**: Resizable widget support
- **Status bar**: 24dp height, left-aligned clock, right-aligned status icons
- **Navigation bar**: 48dp height, centered buttons

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    USER SPACE                           │
├─────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │  Apps   │  │ Settings│  │ Files   │  │Container│   │
│  │  (GUI)  │  │  (GUI)   │  │  (GUI)  │  │Manager  │   │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘   │
│       │            │            │            │         │
│  ┌────┴────────────┴────────────┴────────────┴────┐   │
│  │            ANDROID-STYLE GUI LAYER              │   │
│  │  (Launcher, StatusBar, NotificationShade, etc) │   │
│  └─────────────────────────┬───────────────────────┘   │
│                            │                           │
│  ┌─────────────────────────┴───────────────────────┐   │
│  │              WAYLAND COMPOSITOR                  │   │
│  │            (wlroots-based surface)               │   │
│  └─────────────────────────┬───────────────────────┘   │
│                            │                           │
│  ┌─────────────────────────┴───────────────────────┐   │
│  │            USER DOCKER DAEMON                     │   │
│  │        (Runs in privileged container)             │   │
│  └─────────────────────────┬───────────────────────┘   │
│                            │                           │
├────────────────────────────┼───────────────────────────┤
│                            │                           │
│  ┌─────────────────────────┴───────────────────────┐   │
│  │           SYSTEM DOCKER DAEMON                  │   │
│  │        (PID 1 or managed by tini)                │   │
│  └─────────────────────────┬───────────────────────┘   │
│                            │                           │
│  ┌────────┬────────┬───────┴───────┬────────┬────────┐ │
│  │Network │  Audio │   Bluetooth   │ Storage│  etc   │ │
│  │Manager │Server  │    Service    │Manager │        │ │
│  │Container│Container│   Container  │Container│       │ │
│  └────────┴────────┴───────────────┴────────┴────────┘ │
│                                                          │
├──────────────────────────────────────────────────────────┤
│                    LINUX KERNEL                          │
│           (Custom kernel dengan container support)       │
└──────────────────────────────────────────────────────────┘
```

---

## Technical Considerations

### Build System
- **Base Image**: Alpine Linux atau Buildroot untuk minimal footprint
- **Kernel**: Custom kernel dengan AUFS/OverlayFS, cgroups, namespaces support
- **Build Tool**: Docker + Makefile untuk reproducible builds
- **ISO Generation**: Using xorriso untuk bootable ISO

### GUI Stack
| Component | Technology | Rationale |
|-----------|------------|-----------|
| Compositor | wlroots | Lightweight, customizable |
| UI Toolkit | GTK4 / Qt6 | Modern, Wayland-native |
| JavaScript | WPE/WebKit | Lightweight web runtime |
| Icons | Material Icons | Consistent with Android |
| Fonts | Roboto | Android default |

### Container Runtime
| Component | Technology | Version |
|-----------|------------|---------|
| Container Runtime | containerd | v1.7+ |
| Docker | moby | v24+ |
| Runtime Default | runc | Latest |
| Init System | tini | For proper signal handling |

### Hardware Support
- **x86_64**: Intel/AMD desktop dan laptop
- **ARM64**: Raspberry Pi 4/5, ARM servers
- **UEFI**: Modern hardware dengan secure boot capable
- **Legacy**: BIOS boot untuk older hardware

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Boot time | <15 seconds | From power-on to home screen |
| Memory usage (idle) | <512MB | System monitor reading |
| ISO size | <300MB | Download size |
| App launch time | <1 second | From tap to app visible |
| Container start time | <2 seconds | cold start measurement |
| Stability | 99% uptime | 24-hour stress test |

---

## Project Structure

```
containeros/
├── README.md
├── Makefile
├── Dockerfile.build
├── README.md
│
├── boot/
│   ├── initrd
│   └── grub.cfg
│
├── kernel/
│   ├── config
│   └── patches/
│
├── packages/
│   ├── base/
│   │   ├── Dockerfile
│   │   └── rootfs/
│   │
│   ├── system-docker/
│   │   ├── Dockerfile
│   │   └── container/
│   │
│   ├── user-docker/
│   │   ├── Dockerfile
│   │   └── container/
│   │
│   └── images/
│       ├── os-base/
│       ├── console/
│       ├── network-manager/
│       ├── audio/
│       └── cloud-init/
│
├── gui/
│   ├── compositor/
│   │   └── weston-config/
│   │
│   ├── launcher/
│   │   ├── src/
│   │   │   ├── main.js
│   │   │   ├── HomeScreen.js
│   │   │   ├── AppDrawer.js
│   │   │   ├── StatusBar.js
│   │   │   └── NotificationShade.js
│   │   └── styles/
│   │
│   └── apps/
│       ├── settings/
│       ├── files/
│       ├── store/
│       └── containers/
│
├── scripts/
│   ├── build.sh
│   ├── package.sh
│   └── iso.sh
│
└── docs/
    ├── ARCHITECTURE.md
    ├── BUILD.md
    └── CONTRIBUTING.md
```

---

## Open Questions

1. **Display Server**: Wayland vs X11 - Wayland lebih modern tapi X11 lebih compatible?
2. **App Distribution**: Flatpak vs Docker images vs custom package format?
3. **Update Mechanism**: OSTree-based update vs Docker pull-based update?
4. **Multi-monitor**: Should v1 support external display/output?
5. **Keyboard Input**: Mobile-first atau hybrid (dengan keyboard dock)?
6. **Persistence**: Overlayfs untuk /usr atau full partition approach?
7. **SELinux/AppArmor**: Which MAC framework untuk container isolation?

---

## Roadmap (Proposed)

### Phase 1: Foundation (Months 1-3)
- [ ] Base OS build system
- [ ] Kernel configuration dan compilation
- [ ] System Docker integration
- [ ] Basic CLI interface (ros console)

### Phase 2: GUI Core (Months 4-6)
- [ ] Wayland compositor setup
- [ ] Basic window management
- [ ] Home screen dengan app icons
- [ ] App drawer
- [ ] Status bar dan navigation bar

### Phase 3: System Apps (Months 7-9)
- [ ] Settings app
- [ ] Files app
- [ ] Terminal app
- [ ] Container Manager app
- [ ] Web browser

### Phase 4: App Store (Months 10-12)
- [ ] App Store backend
- [ ] Package indexing
- [ ] Install/Uninstall flow
- [ ] App permissions

### Phase 5: Polish (Ongoing)
- [ ] Animations dan transitions
- [ ] Theme engine
- [ ] Multi-language support
- [ ] Accessibility features
- [ ] Hardware enablement
