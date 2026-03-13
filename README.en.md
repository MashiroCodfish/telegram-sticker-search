[中文](./README.md) | [English](./README.en.md)

# tg-stickers-chat

A Telegram sticker chat enhancement plugin for **OpenClaw**.

OpenClaw already has native Telegram sticker send and cached sticker search support. This project is not about making stickers possible in the first place, but about making sticker usage in chat more natural, proactive, and context-aware.

Its goal is simple: help the agent use stickers more naturally and proactively in chat, instead of replying with plain text only.

## Screenshot Example

![tg-stickers-chat screenshot](https://raw.githubusercontent.com/MashiroCodfish/tg-stickers-chat/main/IMG_9061.jpeg)

---

## Quick Install

### Option 1: Install directly from npm

```bash
openclaw plugins install tg-stickers-chat
openclaw gateway restart
```

### Option 2: Install from the release package

Download `tg-stickers-chat-1.0.0.tgz` from the release page, then run:

```bash
openclaw plugins install ./tg-stickers-chat-1.0.0.tgz
openclaw gateway restart
```

### Option 3: Install from source

```bash
git clone https://github.com/MashiroCodfish/tg-stickers-chat.git
cd tg-stickers-chat
npm install
openclaw plugins install .
openclaw gateway restart
```

---

## Tech Stack

This project is intentionally small and uses only a few pieces:

- **OpenClaw Plugin API** for integration
- **Telegram Bot API** for sticker set and file access
- **Gemini Embedding 2** for sticker and query vectors
- **SQLite** for local vector storage
- **In-memory similarity search** for local retrieval
- **ffmpeg** (recommended) for preview extraction from `.tgs` and `.webm` stickers

---

## How It Works

This refactor changes the main flow from “search stickers with mood keywords” into:

1. Sync a Telegram sticker set
2. Download sticker files
3. Use the original image directly for static stickers, or extract a PNG preview frame for animated/video stickers
4. Generate multimodal vectors with **Gemini Embedding 2**
5. Store vectors in local **SQLite** and keep retrieval local/in-memory during chat
6. During chat, the agent first decides the actual final text it wants to send
7. Then it passes structured expression intent like `replyText / emotion / act / intensity / context / forbid`
8. The plugin runs **top-k recall + lightweight reranking**, optimized for emotional fit, action fit, intensity, and context fit
9. If confidence is low or the moment is not sticker-friendly, it explicitly skips and lets the agent send text only
10. Only when the sticker really improves expression does it return a `sticker_id`

If `autoCollect` is enabled, newly seen sticker sets in chats can still be queued automatically. The original must-keep path remains intact: auto sync, persistent same-set dedupe, WEBP/GIF/TGS/WEBM -> PNG preview conversion, Gemini Embedding 2, SQLite indexing, and local search.

### Why not add per-sticker emotion/action tags during sync?

This version intentionally does **not** add a second external tagging pipeline for every sticker.

Why:

- the existing multimodal embeddings already preserve most visual semantics
- extra per-sticker labeling would make sync slower and more failure-prone
- it would add more schema/migration complexity for limited practical gain right now
- structured intent + facet-aware reranking already improves emotional fit while keeping chat-time latency low

So this refactor chooses the simpler path on purpose: **keep sync light enough, keep chat-time retrieval very light, and make intent the first-class input.**

The plugin now exposes these four tools to OpenClaw:

- `sync_sticker_set_by_name`
- `get_sticker_stats`
- `select_sticker_for_reply` (new primary entry point)
- `search_sticker_by_emotion` (legacy-compatible, but now also accepts structured JSON)

---

## Configuration

Write config under:

```text
plugins.entries.tg-stickers-chat
```

Minimal example:

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

### Config fields

- `embeddingApiKey`
  - Gemini API key used for embeddings
  - Required for building the vector index

- `embeddingModel`
  - Default: `gemini-embedding-2-preview`
  - Usually does not need to change

- `embeddingDimensions`
  - Default: `768`
  - Usually does not need to change

- `autoCollect`
  - Whether newly seen sticker sets should be queued automatically
  - `true` = enabled
  - `false` = fully manual mode

If `embeddingApiKey` is not set in plugin config, the plugin also checks:

- `GEMINI_API_KEY`
- `GOOGLE_API_KEY`

If you want the agent to send stickers more often or less often, you can tell it directly in chat, or store that preference in memory so it can adjust its sticker frequency on its own.

If you want another OpenClaw deployment to install this plugin automatically, see:

- [OpenClaw auto-install guide](./docs/OPENCLAW_AUTO_INSTALL.md)

---

## License

MIT
