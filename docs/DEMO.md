# Demo: what a successful watch looks like

Captured from a real run on AstrBot 4.29.3 + Volcengine Ark `doubao-seed-2-1-lite`, with
`video_send_mode = frames` and 24 sampled frames. Personal identifiers removed; media not redistributed.

## 1. Frames sent to the vision model

For a 6-minute clip the fork samples 24 frames, each downscaled to a 1024 px long edge (≈15–40 KB per
frame as JPEG). The message layout is:

```
画面帧 1/24（第 14.4 秒）  +  <image>
画面帧 2/24（第 28.8 秒）  +  <image>
...
画面帧 24/24（第 345.6 秒） +  <image>
<instruction: describe only what is visible, return JSON>
```

## 2. Tool result returned to the model

```json
{
  "verified": true,
  "post_id": "7709066805xxxxxxxx",
  "url": "https://www.douyin.com/video/7709066805xxxxxxxx",
  "kind": "video",
  "transcript": {
    "summary": "动画风格混剪。开篇为面具人物近景质问镜头，随后切到都市天台夜景，串联多组不同画风下小王子与玫瑰互动的镜头，借助字幕推进叙事，落点在“投入时间让羁绊对象无可替代”。",
    "key_facts": [
      "2.2 秒：暖黄柔光下，头戴黄色宽边帽、黄白条纹外套的褶皱质感面具人物呈现面部近景",
      "6.5 秒：镜头切至天台夜景，人物背影处于城市灯光前",
      "12.1 秒：小王子与玫瑰同框，字幕强调“驯服”与“责任”"
    ],
    "audio": "",
    "onscreen_text": "投入的时间让羁绊对象变得无可替代",
    "meaning": "以动画混剪复述原作主题：关系由投入的时间赋予意义。",
    "uncertain": ["人物是否为同一角色，因画风切换较多无法确认"]
  },
  "watched_at": 1795577793.84
}
```

## 3. Cost of that run

| Metric | Value |
|---|---|
| frames | 24 |
| prompt tokens | ≈ 9.6k |
| completion tokens | ≈ 2.3k |
| approximate cost (lite) | ¥0.01–0.02 |

## 4. Notes

- `verified: true` means the plugin confirmed it actually retrieved media, not just the post title.
- `audio` is empty because this fork ships with `video_separate_audio = false` (the audio block caused
  HTTP 400 on this stack).
- `uncertain` is populated by the model when frames are insufficient — a useful signal that a longer
  sample or a faster model is needed.
