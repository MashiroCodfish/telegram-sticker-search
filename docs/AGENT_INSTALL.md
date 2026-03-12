# Agent Install Guide: telegram-stickers-brain 1.0.0

Use this document when another agent or operator needs to install the minimal sticker-search plugin.

## Goal

Install a clean OpenClaw plugin that does only this:

- sync Telegram sticker sets manually
- embed stickers with Gemini Embedding 2
- store vectors locally in SQLite
- search locally with cosine similarity

This version does **not** use caption generation or other LLM-style generation steps.

## Preconditions

Before installing, verify all of these are true:

- OpenClaw is already installed
- Telegram channel is configured and working
- You have a Gemini API key for embeddings
- You can restart the OpenClaw Gateway on this machine
- Node.js 18+ is available
- `ffmpeg` is available if animated/video stickers need preview extraction

## Install paths

Choose **one** path.

### Path 1: source checkout

```bash
git clone https://github.com/MashiroCodfish/telegram-stickers-brain.git
cd telegram-stickers-brain
npm install
openclaw plugins install .
```

### Path 2: local tarball

```bash
openclaw plugins install ./roitium-telegram-stickers-brain-1.0.0.tgz
```

### Path 3: npm package

Use only if the package has already been published to npm.

```bash
openclaw plugins install @roitium/telegram-stickers-brain
```

## Enable and configure

The plugin id is:

```text
telegram-stickers-brain
```

Write config under:

```text
plugins.entries.telegram-stickers-brain
```

### Minimal config example

```json5
{
  "plugins": {
    "entries": {
      "telegram-stickers-brain": {
        "enabled": true,
        "config": {
          "embeddingApiKey": "YOUR_GEMINI_API_KEY",
          "embeddingModel": "gemini-embedding-2-preview",
          "embeddingDimensions": 768
        }
      }
    }
  }
}
```

## Restart

Plugin config changes require a Gateway restart.

```bash
openclaw gateway restart
```

## Verification checklist

After restart, verify these items.

### 1. Plugin is installed

```bash
openclaw plugins list
```

Look for `telegram-stickers-brain`.

### 2. Plugin is enabled

```bash
openclaw plugins info telegram-stickers-brain
```

### 3. Tools are available

Expected tools:

- `sync_sticker_set_by_name`
- `get_sticker_stats`
- `search_sticker_by_emotion`

### 4. Manual sync test

Ask the agent to call:

```text
sync_sticker_set_by_name({"setNameOrUrl":"https://t.me/addstickers/<SET_NAME>"})
```

or provide a bare set name:

```text
sync_sticker_set_by_name({"setNameOrUrl":"<SET_NAME>"})
```

### 5. Stats check

Ask the agent to call:

```text
get_sticker_stats({})
```

Expected result shape:

```text
当前语义索引中共有 X 张表情包，当前同步队列中有 Y 个合集。
```

### 6. Search test

Ask the agent to call:

```text
search_sticker_by_emotion({"query":"开心 笑着 跑"})
```

Expected result shape:

```json
{"sticker_id":"..."}
```

## Operational notes

- The plugin stores vectors in local SQLite
- Search happens locally in memory using cosine similarity
- There is no caption-generation model in the indexing path
- The sync flow is manual-first and queue-based
- Animated/video sticker preview extraction benefits from `ffmpeg`

## First failure checks

If installation looks correct but usage fails, inspect these first:

1. Telegram bot token missing or invalid
2. Gemini embedding API key missing or invalid
3. Plugin not restarted after config change
4. No sticker sets have been synced yet
5. `ffmpeg` missing for animated/video preview extraction

## Recommended smoke test flow

1. Install plugin
2. Enable config with embedding API key
3. Restart Gateway
4. Sync one known sticker set
5. Run `get_sticker_stats`
6. Run `search_sticker_by_emotion`
7. Send the returned sticker id in chat

## Exact tool intent summary

- `sync_sticker_set_by_name`: queue a Telegram sticker set for indexing
- `get_sticker_stats`: report indexed sticker count and queue size
- `search_sticker_by_emotion`: semantic lookup that returns a Telegram `sticker_id`
