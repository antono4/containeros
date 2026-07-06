# ContainerOS - Android-Style Container OS

<p align="center">
  <img src="https://img.shields.io/badge/Status-Planning-yellow?style=for-the-badge" alt="Status">
  <img src="https://img.shields.io/badge/License-Apache%202.0-blue?style=for-the-badge" alt="License">
  <img src="https://img.shields.io/badge/Platform-Linux%20%7C%20ARM%20%7C%20x86-orange?style=for-the-badge" alt="Platform">
</p>

## Overview

**ContainerOS** adalah sistem operasi Linux ringan yang mengadopsi arsitektur [RancherOS](https://github.com/rancher/os) dengan containerization menyeluruh, dikombinasikan dengan antarmuka pengguna grafis (GUI) berbasis Android yang modern dan intuitif.

### Key Features

- 🐳 **Dual Docker Architecture** - System Docker + User Docker seperti RancherOS
- 📱 **Android-Style GUI** - UI modern dengan gesture controls
- ⚡ **Fast Boot** - Boot time <15 detik
- 🔒 **Secure** - Container isolation dengan AppArmor
- 📦 **Minimal Footprint** - Base system ~200MB
- 🛠️ **Customizable** - App Store untuk install aplikasi

## Quick Preview

![ContainerOS GUI Preview](gui/prototype/preview.gif)

Coba prototipe interaktif: [Buka Demo](gui/prototype/index.html)

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    USER SPACE                            │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────┐    │
│  │           ANDROID-STYLE GUI LAYER               │    │
│  │   (Launcher, StatusBar, NotificationShade)     │    │
│  └─────────────────────────────────────────────────┘    │
│                            │                             │
│  ┌─────────────────────────┴───────────────────────┐    │
│  │              USER DOCKER DAEMON                   │    │
│  │           (Runs in privileged container)          │    │
│  └─────────────────────────┬───────────────────────┘    │
├────────────────────────────┼─────────────────────────────┤
│  ┌─────────────────────────┴───────────────────────┐    │
│  │              SYSTEM DOCKER DAEMON               │    │
│  │        (Manages system services)                 │    │
│  └─────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────┤
│                      LINUX KERNEL                        │
└─────────────────────────────────────────────────────────┘
```

## Project Structure

```
containeros/
├── README.md
├── prd-containeros-android-gui.md      # Product Requirements
├── docs/
│   └── ARCHITECTURE.md                 # System Architecture
├── gui/
│   └── prototype/
│       └── index.html                  # Interactive GUI prototype
└── (more coming soon)
```

## Design Principles

1. **Minimalism** - Base system hanya berisi kernel dan Docker runtime
2. **Containerization** - Semua services berjalan dalam containers
3. **Mobility** - Dirancang untuk touchscreen devices
4. **Security** - Isolasi container dengan AppArmor profiles
5. **Performance** - Boot <15 detik, app launch <1 detik

## Tech Stack

| Component | Technology |
|-----------|------------|
| Base OS | Alpine Linux / Buildroot |
| Kernel | Custom Linux kernel |
| Container Runtime | Docker + containerd |
| GUI Framework | GTK4 + JavaScript |
| Display Server | Wayland (wlroots) |
| Compositor | Weston |
| Build System | Docker + Makefile |

## Comparison with RancherOS

| Feature | RancherOS | ContainerOS |
|---------|-----------|-------------|
| CLI Management | ros CLI | ros CLI + GUI |
| User Interface | Terminal only | Android-style GUI |
| Touch Support | No | Yes |
| App Distribution | Docker Hub only | App Store |
| Boot Time | <10s | <15s |
| ISO Size | ~128MB | ~300MB (with GUI) |

## Documentation

- [Product Requirements Document](prd-containeros-android-gui.md)
- [System Architecture](docs/ARCHITECTURE.md)
- [Live Demo](gui/prototype/index.html)

## Try the Prototype

Buka file `gui/prototype/index.html` di browser untuk melihat prototipe interaktif:

```bash
# Open in browser
open gui/prototype/index.html
# Or use Python's HTTP server
python3 -m http.server 8000
```

### Prototype Features

- ✅ Boot animation
- ✅ Lock screen dengan swipe to unlock
- ✅ Home screen dengan app icons
- ✅ Weather widget
- ✅ App drawer dengan swipe up
- ✅ Notification shade dengan quick settings
- ✅ Navigation bar
- ✅ Settings app dengan Docker info
- ✅ Terminal app dengan docker ps
- ✅ App Store dengan install/update buttons
- ✅ Touch gesture support

## Roadmap

### Phase 1: Foundation
- [ ] Base OS build system
- [ ] Kernel configuration
- [ ] System Docker integration
- [ ] Basic CLI interface

### Phase 2: GUI Core
- [ ] Wayland compositor setup
- [ ] Basic window management
- [ ] Home screen dengan app icons
- [ ] App drawer
- [ ] Status bar dan navigation bar

### Phase 3: System Apps
- [ ] Settings app
- [ ] Files app
- [ ] Terminal app
- [ ] Container Manager app
- [ ] Web browser

### Phase 4: App Store
- [ ] App Store backend
- [ ] Package indexing
- [ ] Install/Uninstall flow
- [ ] App permissions

## Contributing

Kontribusi sangat diterima! Silakan baca [CONTRIBUTING.md](CONTRIBUTING.md) untuk detail.

## License

Apache License 2.0 - lihat [LICENSE](LICENSE) untuk detail.

## Acknowledgments

- [RancherOS](https://github.com/rancher/os) - Inspiration utama
- [Alpine Linux](https://alpinelinux.org/) - Base distribution
- [Weston/wlroots](https://wayland.freedesktop.org/) - Wayland compositor
- [GTK](https://gtk.org/) - GUI toolkit
- [Material Design](https://material.io/design) - Design inspiration

---

<p align="center">
  Made with ❤️ for the container community
</p>
