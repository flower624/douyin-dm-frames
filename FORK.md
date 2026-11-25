# Douyin DM Plugin — Frame-Extraction Fork (unofficial)

A patched fork of [`lxy005tt-collab/astrbot_plugin_douyin_dm`](https://github.com/lxy005tt-collab/astrbot_plugin_douyin_dm),
based on upstream **v0.5.4**. MIT licensed; original copyright belongs to the upstream author.

## Why this fork exists

Tested on AstrBot 4.29.3 with Volcengine Ark (Doubao) vision models, the upstream release failed in two
ways and was flaky in a third:

| Symptom | Root cause | Fix in this fork |
|---|---|---|
| `BadRequestError` from `douyin_watch_recommended` | AstrBot's provider layer has **no `video_url` support** (`core/provider` contains no video handling), so the request is assembled into an invalid payload | New `video_send_mode = frames`: sample frames with ffmpeg and send them as images |
| Still HTTP 400, body: `Invalid PDF input: May not be a PDF file` | Upstream defaults `video_separate_audio = true` and sends an audio file block, which the endpoint parses as a PDF | Disable separate audio transcription (`video_separate_audio = false`) |
| Intermittent `locator.evaluateAll: Execution context was destroyed` | The Douyin SPA navigates while Playwright is evaluating, invalidating the context | Navigation retry around the feed-link extraction in `mimi_douyin/browser/explore.mjs` (3 attempts, growing backoff) |

## Changes

1. **`mimi_douyin/perception.py`**
   - Added `_probe_duration()` and `_video_frames()`
   - Frame count scales with duration: `clamp(round(duration / 20) + 6, 8, 24)`
   - Frames are downscaled to a 1024 px long edge and encoded as JPEG
   - `transcribe_video()` honours `video_send_mode`: `frames` (default) or `video` (upstream behaviour)
   - Extraction failures raise an explicit error instead of silently falling back to the post title
2. **`mimi_douyin/browser/explore.mjs`**
   - Retry wrapper for `evaluateAll` when the page navigates mid-evaluation
3. Runtime defaults documented below

## Recommended configuration

```json
{
  "video_input_mode": "转述为文本",
  "video_send_mode": "frames",
  "video_separate_audio": false,
  "video_provider_id": "ark/doubao-seed-2-1-lite-260915",
  "max_video_duration_seconds": 900,
  "max_download_mb": 300,
  "video_transcription_timeout_seconds": 180,
  "cache_cleanup_enabled": true,
  "cache_cleanup_interval_hours": 1,
  "cache_retention_hours": 6
}
```

`video_input_mode` stays on the upstream `"转述为文本"` (transcribe-to-text) value; the fork only changes
*how* the media reaches the vision model.

## Requirements

Same as upstream: Node.js 20+, Playwright (`npm install`), FFmpeg/FFprobe, Edge or Chrome.
In addition, the host needs a **vision-capable model** — this fork was validated with
Volcengine Ark `doubao-seed-2-1-lite`.

## Cost reference (Doubao lite)

| Video length | Frames sent | Tokens | Approx. cost |
|---|---|---|---|
| 1 min | 8 | ~6k | under ¥0.01 |
| 6 min | 24 | ~10k | ¥0.01–0.02 |
| 15 min | 24 (cap) | ~10k | ¥0.02–0.03 |

## Known limitations

- Sampling is discrete: very fast cuts may fall between frames. Longer videos get more frames, but this is
  not frame-by-frame understanding.
- Audio transcription is off by default. Hosts that support audio/file content blocks can re-enable
  `video_separate_audio`.
- Host upgrades may change the plugin bridge or provider layer; re-validate after upgrading AstrBot.

## Merging upstream updates

All changes live in two files: `mimi_douyin/perception.py` and `mimi_douyin/browser/explore.mjs`.
When merging a new upstream release, keep the upstream versions first, then re-apply the two patches.

## License

MIT — see `LICENSE`. Third-party sources and licenses are listed in `ATTRIBUTIONS.md`.
This is **not** an official upstream release; please report issues in this repository rather than
bothering the upstream author.
