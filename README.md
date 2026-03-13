[中文](./README.md) | [English](./README.en.md)

# tg-stickers-chat

用于 OpenClaw 的 Telegram 贴纸聊天插件。

它的目标很简单：让 agent 在聊天时根据自己真正要发送的文字和情绪，更自然地搭配贴纸。

## 截图

![tg-stickers-chat screenshot](https://raw.githubusercontent.com/MashiroCodfish/tg-stickers-chat/main/IMG_9061.jpeg)

## 安装

### npm

```bash
openclaw plugins install tg-stickers-chat
openclaw gateway restart
```

### Release 包

下载 Release 页面里的 `tg-stickers-chat-1.0.1.tgz`，然后执行：

```bash
openclaw plugins install ./tg-stickers-chat-1.0.1.tgz
openclaw gateway restart
```

### 源码

```bash
git clone https://github.com/MashiroCodfish/tg-stickers-chat.git
cd tg-stickers-chat
npm install
openclaw plugins install .
openclaw gateway restart
```

## 工作方式

1. 同步 Telegram 贴纸包
2. 下载贴纸并生成预览图
3. 使用 Gemini Embedding 2 建立向量索引
4. 将索引保存在本地 SQLite
5. 聊天时根据最终回复文字和表达意图选择贴纸
6. 通过本地召回与轻量重排返回最合适的 `sticker_id`
7. 如果当前场景不适合贴纸，则跳过，只发文字

## 工具

- `sync_sticker_set_by_name`
- `get_sticker_stats`
- `select_sticker_for_reply`
- `search_sticker_by_emotion`

## 配置

配置路径：

```text
plugins.entries.tg-stickers-chat
```

最小示例：

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

配置项：

- `embeddingApiKey`: Gemini API key
- `embeddingModel`: 默认 `gemini-embedding-2-preview`
- `embeddingDimensions`: 默认 `768`
- `autoCollect`: 是否自动收集新贴纸包

环境变量备用读取：

- `GEMINI_API_KEY`
- `GOOGLE_API_KEY`

自动安装说明：

- [OpenClaw auto-install guide](./docs/OPENCLAW_AUTO_INSTALL.md)

## License

MIT
