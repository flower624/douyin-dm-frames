# Contributing

This is a small, focused fork. Please keep changes surgical.

## Where changes belong

| Area | File |
|---|---|
| Frame sampling, prompt assembly, provider call | `mimi_douyin/perception.py` |
| Feed scraping / DOM navigation | `mimi_douyin/browser/*.mjs` |
| Host adapter (AstrBot bridge) | `main.py`, `bridge.py`, `setup_api.py` |

Prefer configuration over code. If a behaviour can be expressed as a config key, add the key instead of
hard-coding a branch.

## Before opening a PR

1. `python -m py_compile` every changed `.py` file
2. `node --check` every changed `.mjs` file
3. Run one real watch end to end and paste the resulting `transcript` (redact ids)
4. Confirm no secrets: `git grep -nE "ark-|sk-[0-9a-f]{20,}|fwr_"`
5. Do not bump the upstream version; describe your patch in `CHANGELOG.md`

## Reporting bugs

Include:
- AstrBot version, host OS
- provider id and model name used for `video_provider_id`
- the exact error string from the tool result
- the relevant few lines of `astrbot.log`

Never paste API keys, cookies, or the contents of `browser-profile`.
