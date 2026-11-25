# Install (this fork)

This fork is a drop-in replacement for upstream `astrbot_plugin_douyin_dm`. It keeps the same plugin id,
folder name and configuration keys, so you can install it over an existing copy.

## 1. Requirements

| Component | Notes |
|---|---|
| AstrBot | 4.29.0 or newer (validated on 4.29.3) |
| Node.js | 20 or newer, must be reachable by AstrBot (PATH or `node_binary`) |
| Playwright | installed **inside the plugin folder** (`npm install`) |
| FFmpeg / FFprobe | both required; put them on PATH or set `ffmpeg_binary` / `ffprobe_binary` |
| Browser | Edge or Chrome installed; the plugin opens its own profile |
| Vision model | any OpenAI-compatible model that accepts `image_url` (validated with Volcengine Ark `doubao-seed-2-1-lite`) |

## 2. Install the plugin

Copy the folder so that `main.py` sits directly at:

```
AstrBot/data/plugins/astrbot_plugin_douyin_dm/main.py
```

Do not nest an extra directory. Then:

```bash
cd AstrBot/data/plugins/astrbot_plugin_douyin_dm
npm install --omit=dev
```

Restart AstrBot and confirm the log contains:

```
Plugin astrbot_plugin_douyin_dm (0.5.4) ...
[与bot刷抖音] 接入已加载，自动回复=True，已绑定=False。
[provider.func_tool_manager:390]: Added llm tool: douyin_watch_recommended
```

## 3. Configure

Minimum viable config (`AstrBot/data/config/astrbot_plugin_douyin_dm_config.json`):

```json
{
  "enabled": true,
  "auto_reply_enabled": false,
  "video_input_mode": "转述为文本",
  "video_send_mode": "frames",
  "video_separate_audio": false,
  "video_provider_id": "<your-vision-provider-id>",
  "max_video_duration_seconds": 900,
  "max_download_mb": 300,
  "ffmpeg_binary": "C:/ffmpeg/bin/ffmpeg.exe",
  "ffprobe_binary": "C:/ffmpeg/bin/ffprobe.exe",
  "node_binary": "C:/node/node.exe",
  "cache_cleanup_enabled": true,
  "cache_cleanup_interval_hours": 1,
  "cache_retention_hours": 6
}
```

`video_provider_id` must point at a provider **inside AstrBot** whose model accepts images.
If you use Volcengine Ark, add an `openai_chat_completion` source with
`api_base = https://ark.cn-beijing.volces.com/api/v3` and a provider such as
`ark/doubao-seed-2-1-lite-260915`.

## 4. Log in to Douyin

1. Open the plugin page: `#/plugin-view/astrbot_plugin_douyin_dm/<view name>`
2. Click **open login window** — a dedicated browser window appears
3. Scan the QR code **in that window** (not in your normal browser)
4. Open Douyin messages and a target conversation, then bind the session

The login profile is stored under `plugin_data/astrbot_plugin_douyin_dm/browser-profile`; it survives restarts.

## 5. Verify

Ask the bot to browse, or trigger `douyin_watch_recommended`. A healthy result looks like:

```json
{
  "verified": true,
  "kind": "video",
  "transcript": {
    "summary": "...",
    "key_facts": ["2.2s: ...", "5.8s: ..."]
  }
}
```

If you get an error, see [TROUBLESHOOTING.md](./TROUBLESHOOTING.md).
