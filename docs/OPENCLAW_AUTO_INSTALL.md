# OpenClaw Auto-Install Guide: telegram-sticker-search 1.0.0

Use this document when another OpenClaw deployment, or an agent managing that deployment, needs to install this plugin automatically.

## Plugin identity

- npm package: `telegram-sticker-search`
- plugin id: `telegram-sticker-search`
- config path: `plugins.entries.telegram-sticker-search`

## Prerequisites

The target OpenClaw instance should already have:

- Telegram configured
- a valid Telegram bot token
- a Gemini API key
- Node.js 18+
- `ffmpeg` recommended for animated or video sticker previews

## Install options

### Option A: install from npm

```bash
openclaw plugins install telegram-sticker-search
```

### Option B: install from a release tarball

Download the release asset first, then run:

```bash
openclaw plugins install ./telegram-sticker-search-1.0.0.tgz
```

### Option C: install from source

```bash
git clone https://github.com/MashiroCodfish/telegram-sticker-search.git
cd telegram-sticker-search
npm install
openclaw plugins install .
```

## Config template

Merge this into the target OpenClaw config:

```json5
{
  "plugins": {
    "entries": {
      "telegram-sticker-search": {
        "enabled": true,
        "config": {
          "embeddingApiKey": "YOUR_GEMINI_API_KEY",
          "embeddingModel": "gemini-embedding-2-preview",
          "embeddingDimensions": 768,
          "autoCollect": true
        }
      }
    }
  }
}
```

Set `autoCollect` to `false` if the target deployment should remain fully manual.

## Restart

After install and config changes:

```bash
openclaw gateway restart
```

## Verification

### 1. Confirm the plugin is installed

```bash
openclaw plugins list
```

### 2. Confirm the plugin info looks correct

```bash
openclaw plugins info telegram-sticker-search
```

### 3. Confirm the tools exist

The target deployment should expose:

- `sync_sticker_set_by_name`
- `get_sticker_stats`
- `search_sticker_by_emotion`

## Recommended smoke test

1. Sync one known sticker set
2. Wait for indexing to finish
3. Run `get_sticker_stats`
4. Run `search_sticker_by_emotion`
5. Confirm a `sticker_id` is returned

### Example: sync a sticker set

```text
sync_sticker_set_by_name({"setNameOrUrl":"https://t.me/addstickers/<SET_NAME>"})
```

### Example: stats

```text
get_sticker_stats({})
```

### Example: search

```text
search_sticker_by_emotion({"query":"开心 笑着 跑"})
```

## Summary

This plugin provides:

- Gemini Embedding 2 for sticker vectors
- local SQLite storage
- local in-memory search
- manual sticker-set sync
- optional automatic collection with `autoCollect`
