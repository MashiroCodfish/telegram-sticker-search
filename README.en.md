[中文](./README.md) | [English](./README.en.md)

# tg-stickers-chat

A Telegram sticker chat plugin for OpenClaw.

Its goal is simple: help the agent choose stickers based on the actual reply text and emotion it wants to express.

## Screenshot

![tg-stickers-chat screenshot](https://raw.githubusercontent.com/MashiroCodfish/tg-stickers-chat/main/IMG_9061.jpeg)

## Install

### npm

```bash
openclaw plugins install tg-stickers-chat
openclaw gateway restart
```

### Release package

Download `tg-stickers-chat-1.0.1.tgz` from Releases, then run:

```bash
openclaw plugins install ./tg-stickers-chat-1.0.1.tgz
openclaw gateway restart
```

### Source

```bash
git clone https://github.com/MashiroCodfish/tg-stickers-chat.git
cd tg-stickers-chat
npm install
openclaw plugins install .
openclaw gateway restart
```

## How It Works

1. Sync Telegram sticker sets
2. Download stickers and build preview images
3. Generate embeddings with Gemini Embedding 2
4. Store the index in local SQLite
5. During chat, select stickers from final reply text and expression intent
6. Use local recall and lightweight reranking to return the best `sticker_id`
7. Skip stickers when the moment is not suitable

## Tools

- `sync_sticker_set_by_name`
- `get_sticker_stats`
- `select_sticker_for_reply`
- `search_sticker_by_emotion`

Recommended chat flow: decide the final reply text first → decide whether a sticker is needed → if needed, call `select_sticker_for_reply` → send the sticker first → send the text after that.

## Configuration

Config path:

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

Fields:

- `embeddingApiKey`: Gemini API key
- `embeddingModel`: default `gemini-embedding-2-preview`
- `embeddingDimensions`: default `768`
- `autoCollect`: automatically collect newly seen sticker sets

Fallback environment variables:

- `GEMINI_API_KEY`
- `GOOGLE_API_KEY`

Auto-install guide:

- [OpenClaw auto-install guide](./docs/OPENCLAW_AUTO_INSTALL.md)

## License

MIT
