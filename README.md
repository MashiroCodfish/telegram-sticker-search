# telegram-stickers-brain

`telegram-stickers-brain` is a fresh OpenClaw plugin line focused on one thing only:

**Gemini Embedding 2 + local vector search for Telegram stickers.**

That is the whole idea.

## Design rules

This plugin intentionally does **not** include:

- LLM caption generation
- VLM / image description models
- sqlite-vec
- local embedding models
- llama / embeddinggemma / Ollama fallback paths
- `.qmd` metadata sidecars
- automatic sticker-set collection
- extra background maintenance jobs

It keeps a single narrow path:

```text
manual sticker-set sync -> Embedding 2 -> local SQLite storage -> in-memory cosine search -> sticker_id
```

## What it does

- Sync a Telegram sticker set by name or `t.me/addstickers/...` link
- Build embeddings with **Gemini Embedding 2**
- Store vectors locally in **SQLite**
- Load vectors into memory for fast cosine similarity search
- Return Telegram `sticker_id` values for agent-side sticker sending

## Exposed tools

- `sync_sticker_set_by_name`
- `get_sticker_stats`
- `search_sticker_by_emotion`

## Requirements

- OpenClaw with Telegram configured
- A working Telegram bot token
- A Gemini API key for embeddings
- Node.js **18+**
- `ffmpeg` recommended for animated / video stickers (`.tgs`, `.webm`) so the plugin can extract a preview frame

## Install

### From GitHub source

```bash
git clone https://github.com/MashiroCodfish/telegram-stickers-brain.git
cd telegram-stickers-brain
npm install
openclaw plugins install .
```

Then restart the Gateway.

### From npm (after publish)

```bash
openclaw plugins install @roitium/telegram-stickers-brain
```

Then restart the Gateway.

## Config

This plugin only needs embedding-related config.

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

### Config fields

| Field | Required | Default | Notes |
| --- | --- | --- | --- |
| `embeddingApiKey` | usually yes | none | Gemini API key for embedding calls |
| `embeddingModel` | no | `gemini-embedding-2-preview` | Embedding model used for sticker and query vectors |
| `embeddingDimensions` | no | `768` | Integer between `128` and `3072` |

If `embeddingApiKey` is omitted, the plugin also checks:

- `GEMINI_API_KEY`
- `GOOGLE_API_KEY`

## How indexing works

When you sync a sticker set:

1. The plugin fetches the sticker set from Telegram
2. Each sticker file is downloaded
3. If needed, a preview image is extracted with `ffmpeg`
4. The sticker is embedded with **Gemini Embedding 2**
5. The normalized vector is stored in local SQLite
6. Search loads vectors into memory and uses cosine similarity

There is no caption-generation step.
There is no text-generation model in the indexing path.

If a sticker preview cannot be built, the plugin still uses Embedding 2 on minimal metadata (`emoji`, `set name`, `file id`) so the sticker can remain indexable without adding a separate generation stack.

## Data layout

This plugin keeps local state very small:

- `STATE_DIR/telegram-stickers-brain.sqlite` - local vector index
- `STATE_DIR/telegram-stickers-brain-tmp/` - temporary preview files during conversion

There are no `.qmd` metadata files in this version.

## Search behavior

`search_sticker_by_emotion` works best with concrete Chinese emotion / action / trait phrases.

Examples:

- `开心 笑着 跑`
- `无奈 叹气 摆烂`
- `委屈 哭哭`
- `得意 比耶`

The tool returns JSON text like:

```json
{"sticker_id":"CAACAgUAAxkBA..."}
```

Your agent can then send that sticker through the message tool.

## Minimal operations model

This plugin is intentionally manual-first:

- sync a set
- index it
- search it
- send the result

No extra automation is required for the core workflow.

## Packaging

Create a package tarball with:

```bash
npm pack
```

OpenClaw can install from a local tarball too:

```bash
openclaw plugins install ./roitium-telegram-stickers-brain-1.0.0.tgz
```

## Agent install guide

See: [docs/AGENT_INSTALL.md](docs/AGENT_INSTALL.md)

## License

MIT
