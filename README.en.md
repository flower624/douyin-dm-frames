# douyin-dm-frames

Unofficial fork of `astrbot_plugin_douyin_dm` v0.5.4 that makes Douyin video understanding work on hosts
without `video_url` support.

- **Frame sampling** instead of video upload (`video_send_mode = frames`)
- **No audio block** by default (avoids `Invalid PDF input` 400)
- **Navigation retry** for Playwright DOM reads
- Long videos supported up to 15 minutes (8–24 frames, adaptive)

## Why

AstrBot's provider layer has no video input support, so upstream's `video_url` payload is rejected with
HTTP 400. On this stack the audio sidecar block is also parsed as a PDF and rejected. This fork converts
videos into sampled frames and sends them as images, which every vision model accepts.

## Links

- [FORK.md](./FORK.md) — full rationale and diff summary
- [docs/INSTALL.md](./docs/INSTALL.md) — install and configure
- [docs/TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md) — the three known failure modes
- [docs/DEMO.md](./docs/DEMO.md) — a real transcript sample
- [CONTRIBUTING.md](./CONTRIBUTING.md) — how to patch and test

## License

MIT. Upstream author: vivi. This fork is maintained separately and is **not** an official release;
please report fork-specific issues here.
