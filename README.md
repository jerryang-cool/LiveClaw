# openclaw-agent-live

**让 AI Agent 的工作过程变成一场可观看、可互动的实时直播。**

## 版本信息

- **版本**: v1.0（发布版）
- **分支**: `main`

## 架构特征

| 特性 | 说明 |
|------|------|
| **网络访问** | Lighthouse 公网 HTTP（端口 19000） |
| **帧渲染** | Pillow 内存绘制 → PyAV H.264+AAC → RTMP 推流（无需 Xvfb/tkinter/FFmpeg） |
| **虚拟形象** | 浏览器端透明视频 Avatar（VP9 Alpha WebM + HEVC Alpha MOV Safari 兼容） |
| **双向交互** | timbot IM C2C 单聊 → @RBT#001 → 触发 Agent turn |
| **语音播报** | 腾讯云 TTS API，自适应退避轮询，长文本自动分段 |
| **音乐播放** | TRTC StreamIngest 推流，Audio Ducking（TTS 播报时自动降低音量） |
| **进程管理** | 忽略 SIGTERM（抗容器清理）+ supervisor 30s 巡检自愈 + 999 次自动重连 |
| **观看页** | macOS 风格 UI，Dock 栏预置应用，语音输入，弹幕互动 |

## 相较于历史版本的变更

- 基于 `archive/dual-xvfb-videoavatar`（v21）的视频 Avatar + `archive/dual-pillow-vrm3d`（v22）的 Pillow 渲染架构合并
- 删除 CloudIDE 内网访问，仅保留 Lighthouse 公网
- 删除 HTTPS 相关代码，统一使用 HTTP
- 修复 WORK_DIR == SKILL_DIR 路径冲突
- 修复 timbot sdkAppId 类型问题（number → string）
- 修复 `--viewer` 阻塞导致进程被杀
- 修复 Avatar 动画与 TTS 不同步（flag 时间戳 + 2s 消退）
- 修复 Dashboard 事件流不滚动（改为从底部向上渲染）
- 新增 Python 依赖自动安装（av/numpy/Pillow）
- 新增 SKILL 全局 skills 目录自动安装
- 新增 `_meta.json` 标准 Skill 元数据

## 其他分支

| 分支 | 说明 |
|------|------|
| `archive/cloudide-xvfb-serveravatar` | 初始版：CloudIDE 内网 + Xvfb + 服务端 Python Avatar 叠加 |
| `archive/dual-xvfb-videoavatar` | Xvfb 架构最终版：双模网络 + 浏览器端透明视频 Avatar |
| `archive/dual-pillow-vrm3d` | 架构转折版：Pillow 帧渲染 + Three.js VRM 3D 模型 Avatar |
