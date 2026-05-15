# openclaw-agent-live

**让 AI Agent 的工作过程变成一场可观看、可互动的实时直播。**

[English](./README_EN.md) | 中文

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

<p align="center">
  <img src="assets/icons/live_start.png" width="64" alt="OpenClaw Live">
</p>

## 简介

openclaw-agent-live 是一个 [OpenClaw](https://openclaw.ai) 平台的 Skill 插件，能够将 AI Agent 的完整推理链路（思考、工具调用、执行结果）实时推流到 TRTC 房间，并生成可公网访问的观看页面。

观众不仅可以"看到" Agent 在想什么、做什么，还可以通过弹幕与 Agent 实时对话。

## 核心特性

| 特性 | 说明 |
|------|------|
| **帧渲染** | Pillow 内存绘制 → PyAV H.264+AAC → RTMP 推流（无需 Xvfb/tkinter/FFmpeg） |
| **虚拟形象** | 浏览器端透明视频 Avatar（VP9 Alpha WebM + HEVC Alpha MOV Safari 兼容） |
| **双向交互** | timbot IM C2C 单聊 → @RBT#001 → 触发 Agent turn |
| **语音播报** | 腾讯云 TTS API，自适应退避轮询，长文本自动分段 |
| **音乐播放** | TRTC StreamIngest 推流，Audio Ducking（TTS 播报时自动降低音量） |
| **进程管理** | 忽略 SIGTERM（抗容器清理）+ supervisor 30s 巡检自愈 + 999 次自动重连 |
| **观看页** | macOS 风格 UI，Dock 栏预置应用，语音输入，弹幕互动 |

## 系统架构

```
用户侧浏览器                TRTC 云端
   ↑                          ↑
trtc-viewer.html  ←← RTMP ←← PyAV (H.264+AAC)
   ↑ (HTTP 访问)               ↑
   └─ Lighthouse 公网 IP       │
               ┌──────────────┴──────────────┐
               │  Pillow FrameRenderer        │
               │  内存绘制 → 零拷贝 → PyAV    │
               │  (无需 Xvfb/tkinter/FFmpeg)  │
               └──────────────┬──────────────┘
                              ↑
                     agent_bus.py  ← Agent 每个关键步骤
                              ↑
                     timbot IM → 观众双向交互
```

## 快速开始

### 前置条件

- Python 3.8+
- [OpenClaw](https://openclaw.ai) 平台环境
- 腾讯云 TRTC 应用（**体验版**及以上，[TRTC 控制台](https://console.cloud.tencent.com/trtc/app)）
- 一台带公网 IP 的服务器（推荐腾讯云 Lighthouse）

### 安装依赖

```bash
pip install -r requirements.txt
```

> 💡 `setup.py --start` 也会自动检测并安装缺失的依赖。

### Linux 额外依赖（CJK 字体）

```bash
# Debian/Ubuntu
sudo apt-get install -y fonts-noto-cjk

# CentOS/RHEL
sudo yum install -y google-noto-sans-cjk-sc-fonts
```

> macOS/Windows 系统自带 CJK 字体，无需额外安装。

### 使用步骤

**1. 初始化配置**

```bash
# 最小配置（仅必填参数）
python3 scripts/setup.py --sdkappid <YOUR_SDKAPPID> --secret <YOUR_SECRET_KEY>

# 完整配置（含 TTS 语音播报 + 双向交互）
python3 scripts/setup.py \
  --sdkappid <SDKAPPID> --secret <SECRET_KEY> \
  --cam-secret-id <CAM_SECRET_ID> --cam-secret-key <CAM_SECRET_KEY> \
  --callback-token <IM_CALLBACK_TOKEN>
```

**2. 启动直播系统**

```bash
python3 scripts/setup.py --start
```

**3. 生成观看页面**

```bash
python3 scripts/setup.py --viewer --lighthouse-ip <YOUR_PUBLIC_IP>
```

**4. 访问直播**

在浏览器中打开 `http://<YOUR_PUBLIC_IP>:19000`

**5. 停止直播**

```bash
python3 scripts/setup.py --stop
```

### 查看状态

```bash
python3 scripts/setup.py --status
```

## 配置参数

| 参数 | 来源 | 必填 | 说明 |
|------|------|:---:|------|
| `SDKAppID` | [TRTC 控制台](https://console.cloud.tencent.com/trtc/app) | ✅ | TRTC 应用 ID（需体验版或以上） |
| `SecretKey` | TRTC 控制台 → 应用详情 | ✅ | TRTC 应用 SecretKey |
| `CallbackToken` | [IM 控制台](https://console.cloud.tencent.com/im) | 推荐 | IM 消息回调鉴权 Token（启用双向交互） |
| `CAM SecretId` | [CAM 密钥管理](https://console.cloud.tencent.com/cam/capi) | 可选 | 腾讯云 API 密钥 ID（启用 TTS 语音播报） |
| `CAM SecretKey` | CAM 密钥管理 | 可选 | 腾讯云 API 密钥 Key |

> ⚠️ 配置文件示例见 [config.json.example](./config.json.example)

## 项目结构

```
openclaw-agent-live/
├── SKILL.md                        # OpenClaw Skill 执行规范（Agent 读取）
├── _meta.json                      # Skill 元数据
├── scripts/
│   ├── setup.py                    # 一键配置/启停脚本
│   ├── stream_daemon.py            # Pillow 帧渲染 + PyAV RTMP 推流
│   ├── frame_renderer.py           # Dashboard 帧渲染器
│   ├── agent_bus.py                # Agent 事件总线
│   ├── supervisor.py               # 进程健康看护
│   ├── tts_client.py               # 腾讯云 TTS API 封装
│   ├── tts_worker.py               # TTS 语音播报守护进程
│   ├── stream_ingest_client.py     # TRTC 在线媒体流推流
│   ├── platform_utils.py           # 跨平台工具模块
│   └── TLSSigAPIv2.py              # 腾讯云官方 UserSig 算法
├── assets/
│   ├── avatar/                     # 虚拟形象动画（VP9/HEVC Alpha）
│   ├── icons/                      # UI 图标
│   ├── fonts/                      # 预置符号字体
│   └── trtc-viewer-template.html   # 观看页 HTML 模板
└── skills/                         # 预置 Skill 插件
    ├── email-skill/                # 邮件发送
    ├── music-search/               # 音乐搜索与播放
    └── weather/                    # 天气查询
```

## 外部服务依赖

| 服务 | 用途 | 是否必须 |
|------|------|:---:|
| [TRTC](https://cloud.tencent.com/product/trtc) | RTMP 推流 + 观众拉流 | ✅ |
| [IM](https://cloud.tencent.com/product/im) | 双向交互（timbot 渠道） | 推荐 |
| [TTS](https://cloud.tencent.com/product/tts) | 语音播报 Agent 状态 | 可选 |

## 版本历史

详见 [CHANGELOG.md](./CHANGELOG.md)

| 版本 | 分支 | 说明 |
|------|------|------|
| **v1.0** | `main` | 发布版：Lighthouse 公网 + Pillow 帧渲染 + VP9/HEVC 视频 Avatar |
| v22 | `archive/dual-pillow-vrm3d` | 架构转折版：Pillow 帧渲染 + Three.js VRM 3D Avatar |
| v21 | `archive/dual-xvfb-videoavatar` | Xvfb 最终版：双模网络 + 浏览器端视频 Avatar |
| v9 | `archive/cloudide-xvfb-serveravatar` | 初始版：CloudIDE 内网 + Xvfb + 服务端 Avatar |

## 许可证

本项目基于 [Apache License 2.0](./LICENSE) 开源。

## 安全

发现安全漏洞请参阅 [SECURITY.md](./SECURITY.md)。
