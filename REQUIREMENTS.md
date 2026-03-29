# GhostDesk — Requirements

---

## System Requirements

### Development Machine
| Requirement | Minimum |
|---|---|
| OS | Windows 10+, macOS 11+, Ubuntu 20.04+ |
| RAM | 8 GB |
| Disk | 5 GB free (for Rust toolchain + deps) |
| CPU | x86_64 or ARM64 |
| Internet | Required for dependency downloads |

### Runtime (End User)
| Requirement | Minimum |
|---|---|
| OS | Windows 10 v1903+, macOS 11+, Ubuntu 20.04+ |
| RAM | 256 MB (idle background mode) |
| CPU | Any dual-core |
| Network | 1 Mbps minimum (5+ Mbps recommended) |
| GPU | Optional (DXGI hardware acceleration) |

---

## Toolchain Requirements

```bash
# Rust (latest stable)
rustup install stable
rustup default stable
rustup target add x86_64-pc-windows-msvc   # Windows
rustup target add x86_64-apple-darwin      # macOS
rustup target add x86_64-unknown-linux-gnu # Linux

# Node.js (for Tauri frontend)
node >= 18.x
npm >= 9.x

# Tauri CLI
cargo install tauri-cli

# Additional build tools (Windows)
# Install Visual Studio Build Tools 2022
# Install Windows SDK 10.0.19041+

# Additional build tools (Linux)
sudo apt install libwebkit2gtk-4.1-dev libssl-dev \
  libayatana-appindicator3-dev librsvg2-dev \
  libx11-dev libxdo-dev libxfixes-dev

# Additional build tools (macOS)
xcode-select --install
```

---

## Cargo.toml (Rust Dependencies)

```toml
[package]
name = "ghostdesk"
version = "0.1.0"
edition = "2021"
authors = ["Your Name"]
description = "Lightweight cross-platform remote desktop"
license = "AGPL-3.0"

[lib]
name = "ghostdesk_lib"
crate-type = ["staticlib", "cdylib", "rlib"]

[[bin]]
name = "ghostdesk"
path = "src/main.rs"

[dependencies]

# === Tauri (UI framework) ===
tauri = { version = "2.0", features = ["tray-icon", "notification"] }
tauri-build = { version = "2.0", features = [] }

# === Async runtime ===
tokio = { version = "1", features = ["full"] }

# === Networking ===
quinn = "0.11"                          # QUIC transport
rustls = { version = "0.23", features = ["ring"] }
webrtc = "0.11"                         # WebRTC P2P fallback
tokio-tungstenite = "0.21"             # WebSocket (signaling)
axum = { version = "0.7", features = ["ws"] }  # Signaling server

# === Screen Capture ===
scrap = "0.5"                           # Cross-platform screen capture
# Windows-specific
[target.'cfg(target_os = "windows")'.dependencies]
windows = { version = "0.58", features = [
  "Win32_Graphics_Dxgi",
  "Win32_Graphics_Direct3D11",
  "Win32_UI_Input_KeyboardAndMouse",
  "Win32_UI_WindowsAndMessaging",
  "Win32_System_Threading",
]}

# === Video Encoding ===
vpx-sys = "0.3"                         # libvpx bindings (VP9)
ffmpeg-next = { version = "7.0", optional = true }  # FFmpeg (optional)

# === Input Simulation ===
enigo = "0.2"                           # Cross-platform mouse/keyboard

# === Serialization ===
serde = { version = "1", features = ["derive"] }
serde_json = "1"

# === Encryption ===
aes-gcm = "0.10"
rand = "0.8"
sha2 = "0.10"

# === Compression ===
lz4 = "1.24"
zstd = "0.13"

# === Logging ===
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }

# === Error handling ===
anyhow = "1"
thiserror = "1"

# === Config ===
config = "0.14"
dirs = "5"

# === Image processing ===
image = "0.25"
bytemuck = "1"

[features]
default = []
ffmpeg = ["ffmpeg-next"]

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true

[profile.dev]
opt-level = 1
```

---

## package.json (Frontend Dependencies)

```json
{
  "name": "ghostdesk-ui",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "tauri": "tauri"
  },
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "@tauri-apps/api": "^2.0.0",
    "@tauri-apps/plugin-notification": "^2.0.0",
    "@tauri-apps/plugin-autostart": "^2.0.0",
    "@tauri-apps/plugin-single-instance": "^2.0.0",
    "zustand": "^4.5.0",
    "clsx": "^2.1.0"
  },
  "devDependencies": {
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "@vitejs/plugin-react": "^4.3.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0",
    "tailwindcss": "^3.4.0",
    "typescript": "^5.5.0",
    "vite": "^5.4.0"
  }
}
```

---

## Environment Variables (.env)

```env
# Server config
GHOSTDESK_RELAY_HOST=your-server.com
GHOSTDESK_RELAY_PORT=21116
GHOSTDESK_SIGNAL_PORT=21115

# Encryption
GHOSTDESK_SECRET_KEY=change-this-to-a-random-secret

# Logging
RUST_LOG=ghostdesk=info

# Feature flags
GHOSTDESK_ENABLE_FFMPEG=false
GHOSTDESK_AUTO_START=true
GHOSTDESK_TRAY_ONLY=true
```

---

## Optional: Docker (Self-Hosted Server)

```yaml
# docker-compose.yml
version: '3.8'
services:
  ghostdesk-server:
    image: rustdesk/rustdesk-server:latest
    ports:
      - "21115:21115"   # Signal
      - "21116:21116"   # Relay (TCP)
      - "21116:21116/udp"
      - "21117:21117"   # Relay (WS)
    volumes:
      - ./data:/root
    restart: unless-stopped
```
