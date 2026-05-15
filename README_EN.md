# openclaw-agent-live

**Turn your AI Agent's work process into a watchable, interactive live stream.**

[中文](./README.md) | English

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

<p align="center">
  <img src="assets/icons/live_start.png" width="64" alt="OpenClaw Live">
</p>

## Introduction

openclaw-agent-live is a Skill plugin for the [OpenClaw](https://openclaw.ai) platform that streams an AI Agent's complete reasoning pipeline (thinking, tool calls, execution results) in real-time to a TRTC room, generating a publicly accessible viewer page.

Viewers can not only "see" what the Agent is thinking and doing, but also interact with it in real-time via chat.

## Key Features

| Feature | Description |
|---------|-------------|
| **Frame Rendering** | Pillow in-memory drawing → PyAV H.264+AAC → RTMP push (no Xvfb/tkinter/FFmpeg needed) |
| **Virtual Avatar** | Browser-side transparent video overlay (VP9 Alpha WebM + HEVC Alpha MOV for Safari) |
| **Two-way Interaction** | timbot IM C2C messaging → @RBT#001 → triggers Agent turn |
| **Voice Broadcast** | Tencent Cloud TTS API with adaptive backoff and auto text segmentation |
| **Music Playback** | TRTC StreamIngest with Audio Ducking (auto volume reduction during TTS) |
| **Process Management** | SIGTERM-resilient + supervisor 30s health checks + 999 auto-reconnects |
| **Viewer Page** | macOS-style UI with Dock bar, voice input, and barrage interaction |

## Architecture

```
User Browser                 TRTC Cloud
   ↑                            ↑
trtc-viewer.html  ←← RTMP ←← PyAV (H.264+AAC)
   ↑ (HTTP access)              ↑
   └─ Lighthouse Public IP      │
               ┌───────────────┴───────────────┐
               │  Pillow FrameRenderer          │
               │  In-memory draw → Zero-copy    │
               │  (No Xvfb/tkinter/FFmpeg)      │
               └───────────────┬───────────────┘
                               ↑
                      agent_bus.py  ← Agent key steps
                               ↑
                      timbot IM → Viewer interaction
```

## Quick Start

### Prerequisites

- Python 3.8+
- [OpenClaw](https://openclaw.ai) platform environment
- Tencent Cloud TRTC application (**Trial Edition** or above, [TRTC Console](https://console.cloud.tencent.com/trtc/app))
- A server with a public IP (Tencent Cloud Lighthouse recommended)

### Install Dependencies

```bash
pip install -r requirements.txt
```

> 💡 `setup.py --start` will also auto-detect and install missing dependencies.

### Additional Linux Dependencies (CJK Fonts)

```bash
# Debian/Ubuntu
sudo apt-get install -y fonts-noto-cjk

# CentOS/RHEL
sudo yum install -y google-noto-sans-cjk-sc-fonts
```

> macOS/Windows have built-in CJK fonts — no extra installation needed.

### Usage

**1. Initialize Configuration**

```bash
# Minimal (required params only)
python3 scripts/setup.py --sdkappid <YOUR_SDKAPPID> --secret <YOUR_SECRET_KEY>

# Full (with TTS voice broadcast + two-way interaction)
python3 scripts/setup.py \
  --sdkappid <SDKAPPID> --secret <SECRET_KEY> \
  --cam-secret-id <CAM_SECRET_ID> --cam-secret-key <CAM_SECRET_KEY> \
  --callback-token <IM_CALLBACK_TOKEN>
```

**2. Start Live System**

```bash
python3 scripts/setup.py --start
```

**3. Generate Viewer Page**

```bash
python3 scripts/setup.py --viewer --lighthouse-ip <YOUR_PUBLIC_IP>
```

**4. Watch the Stream**

Open `http://<YOUR_PUBLIC_IP>:19000` in your browser.

**5. Stop Live**

```bash
python3 scripts/setup.py --stop
```

### Check Status

```bash
python3 scripts/setup.py --status
```

## Configuration Parameters

| Parameter | Source | Required | Description |
|-----------|--------|:--------:|-------------|
| `SDKAppID` | [TRTC Console](https://console.cloud.tencent.com/trtc/app) | ✅ | TRTC App ID (Trial Edition or above) |
| `SecretKey` | TRTC Console → App Details | ✅ | TRTC App SecretKey |
| `CallbackToken` | [IM Console](https://console.cloud.tencent.com/im) | Recommended | IM callback auth token (enables two-way interaction) |
| `CAM SecretId` | [CAM Key Management](https://console.cloud.tencent.com/cam/capi) | Optional | Tencent Cloud API key ID (enables TTS voice broadcast) |
| `CAM SecretKey` | CAM Key Management | Optional | Tencent Cloud API key |

> ⚠️ See [config.json.example](./config.json.example) for a configuration file template.

## Project Structure

```
openclaw-agent-live/
├── SKILL.md                        # OpenClaw Skill spec (read by Agent)
├── _meta.json                      # Skill metadata
├── scripts/
│   ├── setup.py                    # One-click setup & control script
│   ├── stream_daemon.py            # Pillow rendering + PyAV RTMP streaming
│   ├── frame_renderer.py           # Dashboard frame renderer
│   ├── agent_bus.py                # Agent event bus
│   ├── supervisor.py               # Process health monitor
│   ├── tts_client.py               # Tencent Cloud TTS API wrapper
│   ├── tts_worker.py               # TTS voice broadcast daemon
│   ├── stream_ingest_client.py     # TRTC online media stream ingest
│   ├── platform_utils.py           # Cross-platform utilities
│   └── TLSSigAPIv2.py              # Tencent Cloud official UserSig algorithm
├── assets/
│   ├── avatar/                     # Virtual avatar animations (VP9/HEVC Alpha)
│   ├── icons/                      # UI icons
│   ├── fonts/                      # Bundled symbol fonts
│   └── trtc-viewer-template.html   # Viewer page HTML template
└── skills/                         # Bundled Skill plugins
    ├── email-skill/                # Email sending
    ├── music-search/               # Music search & playback
    └── weather/                    # Weather query
```

## External Service Dependencies

| Service | Purpose | Required |
|---------|---------|:--------:|
| [TRTC](https://www.tencentcloud.com/products/trtc) | RTMP push + viewer pull streaming | ✅ |
| [IM](https://www.tencentcloud.com/products/im) | Two-way interaction (timbot channel) | Recommended |
| [TTS](https://www.tencentcloud.com/products/tts) | Voice broadcast of Agent status | Optional |

## Version History

See [CHANGELOG.md](./CHANGELOG.md) for details.

| Version | Branch | Description |
|---------|--------|-------------|
| **v1.0** | `main` | Release: Lighthouse public network + Pillow rendering + VP9/HEVC video Avatar |
| v22 | `archive/dual-pillow-vrm3d` | Architecture pivot: Pillow rendering + Three.js VRM 3D Avatar |
| v21 | `archive/dual-xvfb-videoavatar` | Xvfb final: Dual-mode network + browser-side video Avatar |
| v9 | `archive/cloudide-xvfb-serveravatar` | Initial: CloudIDE internal + Xvfb + server-side Avatar |

## License

This project is licensed under the [Apache License 2.0](./LICENSE).

## Security

To report security vulnerabilities, see [SECURITY.md](./SECURITY.md).
