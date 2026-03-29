# GhostDesk — Remote Desktop Solution
## Project Synopsis

---

### Overview

**GhostDesk** is a lightweight, cross-platform remote desktop application built on top of the RustDesk open-source framework. It enables real-time screen sharing, remote mouse and keyboard control, and secure peer-to-peer connections — all through a minimal, efficient interface that can run silently in the background.

---

### Objectives

- Provide fast, low-latency screen sharing and remote control
- Run silently in the system tray with minimal resource usage
- Support Windows (primary), macOS, and Linux
- Enable self-hosted relay/signaling server for full data privacy
- Deliver a clean, modern UI built with Tauri + React
- Ensure end-to-end encrypted connections (TLS 1.3 + AES-256)

---

### Key Features

| Feature | Description |
|---|---|
| Screen Sharing | Real-time capture using DXGI (Windows) / X11 (Linux) / CGDisplay (macOS) |
| Remote Control | Full mouse and keyboard simulation on the host machine |
| Background Mode | Runs in system tray, starts on boot, no visible window unless triggered |
| P2P Connection | Direct WebRTC/QUIC-based peer connection, relay fallback if needed |
| Cross-Platform | Windows 10+, macOS 11+, Ubuntu 20.04+ |
| Self-Hosted Server | Deploy your own relay + signaling server (Rust + Tokio) |
| Encryption | TLS 1.3 handshake + AES-256-GCM session encryption |
| Minimal UI | Tauri + React frontend, sub-10MB binary size target |
| Auto-Start | Registers as a background service on system startup |
| Session Logging | Optional local logs of session activity |

---

### Architecture Overview

```
┌──────────────────────────────────────────────┐
│                  HOST MACHINE                │
│  ┌────────────┐   ┌──────────┐  ┌─────────┐ │
│  │Screen Cap  │──▶│ Encoder  │─▶│  Send   │ │
│  │DXGI/X11   │   │H264/VP9  │  │UDP/QUIC │ │
│  └────────────┘   └──────────┘  └────┬────┘ │
│                                       │      │
└───────────────────────────────────────┼──────┘
                                        │
                              ┌─────────▼──────────┐
                              │   SIGNALING SERVER  │
                              │  (Rust + Tokio/Axum)│
                              └─────────┬──────────┘
                                        │
┌───────────────────────────────────────┼──────┐
│                CLIENT MACHINE         │      │
│  ┌──────────┐   ┌──────────┐  ┌──────▼────┐ │
│  │  UI      │◀──│ Decoder  │◀─│  Receive  │ │
│  │Tauri+React│  │H264/VP9  │  │ UDP/QUIC  │ │
│  └────┬─────┘   └──────────┘  └───────────┘ │
│       │ Input Events (mouse/keyboard)        │
└───────┼──────────────────────────────────────┘
        │
        ▼ Sent back to HOST → Executed via SendInput API
```

---

### Technology Stack

| Layer | Technology |
|---|---|
| Core Language | Rust |
| Screen Capture | DXGI (Win) / X11 (Linux) / CGDisplay (macOS) |
| Video Encoding | VP9 via libvpx / H.264 via FFmpeg |
| Network Transport | QUIC (Quinn) + WebRTC fallback |
| Signaling Server | Rust + Tokio + Axum + WebSocket |
| Input Simulation | Windows SendInput / Linux uinput / macOS CGEvent |
| Frontend UI | Tauri v2 + React + TailwindCSS |
| Encryption | TLS 1.3 + AES-256-GCM |
| Build Tools | Cargo, npm, Tauri CLI |
| Deployment | Docker (server), standalone binary (client) |

---

### Target Platforms

| Platform | Support Level |
|---|---|
| Windows 10/11 | ✅ Primary |
| macOS 11+ | ✅ Full support |
| Ubuntu / Debian Linux | ✅ Full support |
| Android / iOS | 🔄 Planned (Phase 2) |

---

### Project Phases

| Phase | Scope |
|---|---|
| Phase 1 | Core screen capture + encoding (Windows) |
| Phase 2 | Network transport + P2P connection |
| Phase 3 | Input simulation (mouse + keyboard) |
| Phase 4 | Tauri UI + system tray + background mode |
| Phase 5 | macOS + Linux support |
| Phase 6 | Self-hosted server setup + Docker |
| Phase 7 | Installer + auto-start + polish |

---

*Built on top of RustDesk open-source framework (AGPL-3.0)*
