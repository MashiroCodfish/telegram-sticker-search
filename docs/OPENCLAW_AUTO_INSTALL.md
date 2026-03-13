# OpenClaw Auto-Install Guide: tg-stickers-chat 1.0.2

Use this document when another OpenClaw deployment, or an agent managing that deployment, needs to install this plugin automatically.

## Plugin identity

- npm package: `tg-stickers-chat`
- plugin id: `tg-stickers-chat`
- config path: `plugins.entries.tg-stickers-chat`

## What this plugin does

This plugin helps an OpenClaw agent use Telegram stickers more naturally in chat.

Under the hood it provides:

- sticker set syncing
- local indexing with Gemini Embedding 2
- local SQLite storage
- local in-memory similarity search
- optional automatic collection through `autoCollect`

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
openclaw plugins install tg-stickers-chat
```

### Option B: install from a release tarball

Download the release asset first, then run:

```bash
openclaw plugins install ./tg-stickers-chat-1.0.2.tgz
```

### Option C: install from source

```bash
git clone https://github.com/MashiroCodfish/tg-stickers-chat.git
cd tg-stickers-chat
npm install
openclaw plugins install .
```

## Config template

Merge this into the target OpenClaw config:

```json5
{
  "plugins": {
    "entries": {
      "tg-stickers-chat": {
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
openclaw plugins info tg-stickers-chat
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
