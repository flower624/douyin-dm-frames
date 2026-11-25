# Troubleshooting

All three failures below were reproduced on AstrBot 4.29.3 + Volcengine Ark before this fork fixed them.

## 1. `视频转述接口调用失败：BadRequestError`

**What it means:** the vision call itself returned HTTP 400.

**Root causes seen in practice**

| Log detail | Cause | Fix |
|---|---|---|
| provider request contains `video_url` | AstrBot's provider layer does not support video input; the payload is malformed | set `video_send_mode = "frames"` |
| `Invalid PDF input: May not be a PDF file` | an audio/file block was sent and the endpoint parsed it as a document | set `video_separate_audio = false` |
| `InvalidParameter` about image size / count | too many or too large frames | lower `max_model_video_mb` or the frame cap |

**How to confirm the model works at all (bypassing AstrBot):**

```bash
curl https://ark.cn-beijing.volces.com/api/v3/chat/completions \
  -H "Authorization: Bearer $ARK_API_KEY" -H "Content-Type: application/json" \
  -d '{"model":"doubao-seed-2-1-lite-260915","messages":[{"role":"user","content":[
       {"type":"image_url","image_url":{"url":"data:image/jpeg;base64,<...>"}},
       {"type":"text","text":"describe this frame"}]}]}'
```

If this works but the plugin fails, the problem is the host layer, not the model.

## 2. `locator.evaluateAll: Execution context was destroyed`

**What it means:** the Douyin page navigated or reloaded while Playwright was reading the DOM.

**Mitigations**

- This fork retries the feed-link extraction up to 3 times with growing backoff.
- Keep the dedicated browser window on the Douyin page; avoid clicking around in it.
- If the login session expired, reopen the login window and scan again.
- If Douyin shows a captcha or verification page, complete it manually once.

## 3. Frames extracted but transcript looks thin

| Symptom | Likely cause |
|---|---|
| only 8 frames for a long video | duration probe failed (`ffprobe` missing or path wrong) |
| transcript repeats the post title | the model returned title text instead of describing frames; check the prompt / provider |
| `视频抽帧失败，没有取得任何画面` | ffmpeg failed on the file, or the file is not a video |

Check the media directory (`media_dir`) and the plugin log; the fork logs the frame count it sent.

## 4. Nothing happens when the bot is asked to browse

- `enabled` or `autonomy_enabled` is false
- the browser session is not logged in (`state: login_required`)
- no vision provider configured (`video_provider_id` empty or wrong)
- the daily tool-call quota of your host is exhausted

Inspect the plugin page status block; it reports dependency readiness, login state and binding separately.

## 5. Chat replies are delayed or the page freezes

The plugin runs a real browser. On low-memory machines, close other Chromium instances, or set
`browser_headless = true` **only after** the login profile exists (headless mode may trigger captcha).
