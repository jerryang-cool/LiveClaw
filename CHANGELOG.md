# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-04-11

### Architecture

- **Rendering**: Pillow in-memory frame rendering → PyAV H.264+AAC → RTMP push streaming (no Xvfb/tkinter/FFmpeg required)
- **Avatar**: Browser-side transparent video overlay (VP9 Alpha WebM + HEVC Alpha MOV for Safari)
- **Interaction**: Two-way interaction via timbot IM C2C messaging → @RBT#001 → triggers Agent turn
- **TTS**: Tencent Cloud TTS API with adaptive backoff polling and automatic long-text segmentation
- **Music**: TRTC StreamIngest push streaming with Audio Ducking (auto volume reduction during TTS)
- **Process Management**: SIGTERM-resilient + supervisor 30s health checks + 999 auto-reconnects
- **Viewer**: macOS-style UI with Dock bar, voice input, and barrage interaction

### Added

- Pillow-based `FrameRenderer` replacing Xvfb + tkinter + FFmpeg pipeline
- Browser-side transparent video Avatar (VP9 Alpha / HEVC Alpha)
- Public network access via Lighthouse IP (port 19000)
- Two-way interaction via timbot IM channel (replacing webhook)
- TTS voice broadcast daemon (`tts_worker.py`)
- Music playback via TRTC StreamIngest (`stream_ingest_client.py`)
- Auto-installation of Python dependencies (av/numpy/Pillow)
- Auto-installation of bundled skills (email-skill, music-search, weather)
- `_meta.json` standard Skill metadata
- Dynamic UserSig generation API endpoint (`/api/gen-usersig`)
- Pre-emit task API for instant Dashboard response (`/api/emit-task`)
- Cross-platform support (Linux/macOS/Windows)

### Changed

- Removed HTTPS code, unified to HTTP
- Fixed `WORK_DIR == SKILL_DIR` path conflict
- Fixed timbot `sdkAppId` type issue (number → string)
- Fixed `--viewer` blocking causing process termination
- Fixed Avatar animation out of sync with TTS (flag timestamp + 2s decay)
- Fixed Dashboard event stream not scrolling (changed to bottom-up rendering)
