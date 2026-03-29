# 👻 GhostDesk

> A lightweight, cross-platform remote desktop application with screen sharing and remote control — built on RustDesk, powered by Rust.

![Version](https://img.shields.io/badge/version-0.1.0-blue)
![License](https://img.shields.io/badge/license-AGPL--3.0-green)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)
![Built with Rust](https://img.shields.io/badge/built%20with-Rust-orange)

---

## ✨ Features

- 🖥️ **Real-time Screen Sharing** — Low latency capture via DXGI / X11 / CGDisplay
- 🖱️ **Remote Control** — Full mouse and keyboard input simulation
- 🔒 **End-to-End Encrypted** — TLS 1.3 + AES-256-GCM
- 🌐 **Cross-Platform** — Windows 10+, macOS 11+, Linux
- 🔧 **Self-Hosted** — Run your own relay + signaling server
- 🕶️ **Background Mode** — Runs silently in the system tray
- ⚡ **Minimal & Fast** — Sub-10MB binary, <256MB RAM idle
- 🚀 **Auto-Start** — Registers on system startup automatically
- 📡 **P2P First** — Direct connection via QUIC, relay fallback only when needed

---

## 📁 Project Structure

```
ghostdesk/
├── src/                        # Rust core
│   ├── main.rs                 # Entry point
│   ├── capture/                # Screen capture module
│   │   ├── mod.rs
│   │   ├── dxgi.rs             # Windows DXGI capture
│   │   ├── x11.rs              # Linux X11 capture
│   │   └── cg.rs               # macOS CGDisplay capture
│   ├── codec/                  # Video encoding/decoding
│   │   ├── mod.rs
│   │   ├── encoder.rs          # VP9 / H264 encoder
│   │   └── decoder.rs          # VP9 / H264 decoder
│   ├── network/                # Transport layer
│   │   ├── mod.rs
│   │   ├── quic.rs             # QUIC transport (Quinn)
│   │   ├── relay.rs            # Relay fallback
│   │   └── signaling.rs        # WebSocket signaling client
│   ├── input/                  # Input simulation
│   │   ├── mod.rs
│   │   └── handler.rs          # Mouse + keyboard events
│   ├── server/                 # Signaling + relay server
│   │   ├── mod.rs
│   │   ├── signal.rs           # WebSocket signaling server
│   │   └── relay.rs            # Relay server
│   ├── crypto/                 # Encryption utilities
│   │   └── mod.rs
│   └── config.rs               # App configuration
├── ui/                         # Tauri + React frontend
│   ├── src/
│   │   ├── App.tsx
│   │   ├── components/
│   │   │   ├── TrayMenu.tsx    # System tray menu
│   │   │   ├── ConnectPanel.tsx
│   │   │   ├── SessionView.tsx # Live stream view
│   │   │   └── Settings.tsx
│   │   ├── store/              # Zustand state
│   │   └── styles/
│   ├── index.html
│   └── vite.config.ts
├── tauri.conf.json             # Tauri configuration
├── Cargo.toml                  # Rust dependencies
├── package.json                # Frontend dependencies
├── docker-compose.yml          # Self-hosted server
├── .env.example
├── SYNOPSIS.md
├── REQUIREMENTS.md
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

```bash
# 1. Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 2. Install Node.js (v18+)
# Download from https://nodejs.org

# 3. Install Tauri CLI
cargo install tauri-cli

# 4. Linux only — install system dependencies
sudo apt install libwebkit2gtk-4.1-dev libssl-dev \
  libayatana-appindicator3-dev librsvg2-dev \
  libx11-dev libxdo-dev libxfixes-dev
```

### Clone & Setup

```bash
# Clone the repo
git clone https://github.com/yourname/ghostdesk.git
cd ghostdesk

# Install frontend dependencies
cd ui && npm install && cd ..

# Copy environment config
cp .env.example .env
```

### Development

```bash
# Run in development mode (with hot reload)
cargo tauri dev

# Run only the Rust backend
cargo run

# Run only the frontend
cd ui && npm run dev
```

### Build for Production

```bash
# Build release binary + installer
cargo tauri build

# Output:
# Windows: target/release/bundle/msi/GhostDesk_0.1.0_x64.msi
# macOS:   target/release/bundle/dmg/GhostDesk_0.1.0_x64.dmg
# Linux:   target/release/bundle/deb/ghostdesk_0.1.0_amd64.deb
```

---

## 🌐 Self-Hosted Server Setup

```bash
# Using Docker (recommended)
docker-compose up -d

# Or run directly
cargo run --bin ghostdesk-server
```

Then update your `.env`:
```env
GHOSTDESK_RELAY_HOST=your-server-ip
GHOSTDESK_RELAY_PORT=21116
GHOSTDESK_SIGNAL_PORT=21115
```

---

## 🖥️ How It Works

```
1. Host runs GhostDesk → starts capturing screen
2. Host connects to signaling server → gets a session ID
3. Client enters session ID → signaling server brokers the connection
4. P2P QUIC connection established (or relay if NAT blocks P2P)
5. Encoded video frames streamed from Host → Client
6. Client sends mouse/keyboard events back → Host executes them
```

---

## ⚙️ Configuration

Edit `.env` or use the Settings panel in the UI:

| Key | Default | Description |
|---|---|---|
| `GHOSTDESK_RELAY_HOST` | `relay.ghostdesk.io` | Relay server address |
| `GHOSTDESK_RELAY_PORT` | `21116` | Relay port |
| `GHOSTDESK_AUTO_START` | `true` | Start on system boot |
| `GHOSTDESK_TRAY_ONLY` | `true` | Run in tray (no main window on start) |
| `GHOSTDESK_ENABLE_FFMPEG` | `false` | Use FFmpeg instead of libvpx |
| `RUST_LOG` | `ghostdesk=info` | Log level |

---

## 🔐 Security

- All connections use **TLS 1.3** with certificate pinning
- Session data encrypted with **AES-256-GCM**
- Session IDs are ephemeral and randomly generated
- No data stored on relay server — pure passthrough
- Optionally self-host for full control of your data

---

## 📸 UI Preview

```
┌─────────────────────────────┐
│  👻 GhostDesk          [×]  │
├─────────────────────────────┤
│  Your ID:  [  847-293  ]    │
│  Password: [  ••••••   ]    │
├─────────────────────────────┤
│  Connect to:                │
│  [ Enter remote ID...  ]    │
│  [      Connect        ]    │
├─────────────────────────────┤
│  ⚙ Settings   📋 Log       │
└─────────────────────────────┘
```

Minimizes to system tray. Double-click tray icon to restore.

---

## 🗺️ Roadmap

- [x] Project structure & planning
- [ ] Phase 1 — Screen capture engine (Windows)
- [ ] Phase 2 — Video encoding (VP9)
- [ ] Phase 3 — QUIC network transport
- [ ] Phase 4 — Input simulation
- [ ] Phase 5 — Tauri UI + system tray
- [ ] Phase 6 — macOS + Linux support
- [ ] Phase 7 — Self-hosted server + Docker
- [ ] Phase 8 — Installer + auto-start
- [ ] Phase 9 — Android/iOS client (planned)

---

## 🤝 Contributing

1. Fork the repo
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **AGPL-3.0 License** — see [LICENSE](LICENSE) for details.

Built on top of [RustDesk](https://github.com/rustdesk/rustdesk) open-source framework.

---

## 🙏 Acknowledgements

- [RustDesk](https://github.com/rustdesk/rustdesk) — Core remote desktop framework
- [Tauri](https://tauri.app) — Lightweight UI framework
- [Quinn](https://github.com/quinn-rs/quinn) — QUIC transport implementation
- [scrap](https://github.com/quadrupleslap/scrap) — Cross-platform screen capture
- [enigo](https://github.com/enigo-rs/enigo) — Cross-platform input simulation

---

<p align="center">Made with ❤️ and 🦀 Rust</p>
