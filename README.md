[中文](./README.md) | [English](./README.en.md)

# tg-stickers-chat

一个给 **OpenClaw** 用的 Telegram 贴纸聊天增强插件。

OpenClaw 本身已经支持 Telegram 贴纸的基础发送与缓存搜索；这个项目要解决的不是“能不能发”，而是“能不能在聊天里更自然、更主动、更贴切地发”。

它的目标很简单：让 agent 在聊天里更自然地主动使用贴纸，丰富对话，而不是只会发纯文字。

## 截图示例

![tg-stickers-chat screenshot](https://raw.githubusercontent.com/MashiroCodfish/tg-stickers-chat/main/IMG_9061.jpeg)

---

## 快速安装

### 方式一：直接从 npm 安装

```bash
openclaw plugins install tg-stickers-chat
openclaw gateway restart
```

### 方式二：从 Release 包安装

先下载 Release 页面里的 `tg-stickers-chat-1.0.0.tgz`，然后执行：

```bash
openclaw plugins install ./tg-stickers-chat-1.0.0.tgz
openclaw gateway restart
```

### 方式三：从源码安装

```bash
git clone https://github.com/MashiroCodfish/tg-stickers-chat.git
cd tg-stickers-chat
npm install
openclaw plugins install .
openclaw gateway restart
```

---

## 技术栈

这个项目刻意保持简单，只用到这些东西：

- **OpenClaw Plugin API**：把插件接入 OpenClaw
- **Telegram Bot API**：拉取 sticker set 和 sticker 文件
- **Gemini Embedding 2**：给贴纸和查询词生成向量
- **SQLite**：本地存储向量索引
- **内存相似度搜索**：查询时直接在本地内存里匹配
- **ffmpeg（推荐）**：处理 `.tgs` / `.webm` 这类动图或视频贴纸的预览帧

---

## 实现原理

这一版把主流程从“给几个情绪关键词搜图”重构成了：

1. 同步一个 Telegram 表情包合集
2. 下载每张贴纸文件
3. 如果是静态图就直接用原图；如果是动图或视频，就抽一帧 PNG 预览图
4. 用 **Gemini Embedding 2** 给贴纸生成多模态向量
5. 把向量存到本地 **SQLite**，聊天检索时直接加载到内存
6. 聊天时，agent 先决定自己真正要发送的最终文字
7. 再把 `replyText / emotion / act / intensity / context / forbid` 这样的表达意图喂给检索器
8. 本地做 **top-k 召回 + 轻量重排**，重排目标是“情绪、动作、强度、语境是否贴合”
9. 如果置信度不足或场景不适合，就直接 skip，只发文字
10. 只有在判断“这张贴纸真的更会说话”时才返回 `sticker_id`

如果打开了 `autoCollect`，聊天里出现新的表情包合集时，插件会自动把它加入处理队列；同步期仍然保留原来的自动收集、同 set 去重、WEBP/GIF/TGS/WEBM -> PNG 预览、Gemini Embedding 2、SQLite 索引这条链路。

### 为什么没有额外给每张贴纸预先打 emotion/act 标签？

这次重构刻意**没有**再加一套同步期的外部标签生成流程。原因很简单：

- 现在的多模态 embedding 已经把图像语义保留下来了
- 再为每张贴纸单独跑一次标签生成，会明显增加同步成本和复杂度
- 这些标签还会带来额外的 schema、回填、失败重试和兼容成本
- 当前阶段，用“结构化表达意图 + embedding facet 重排”就能把效果拉上来，而且聊天期仍然保持很轻

所以这版优先选择：**不加重依赖、不加重同步复杂度，先把表达意图这条主路径做干净。**

插件会向 OpenClaw 提供这 4 个工具：

- `sync_sticker_set_by_name`
- `get_sticker_stats`
- `select_sticker_for_reply`（新的主入口）
- `search_sticker_by_emotion`（兼容旧调用，但内部也支持结构化 JSON）

---

## 配置

配置写在这里：

```text
plugins.entries.tg-stickers-chat
```

最小配置示例：

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

### 配置项说明

- `embeddingApiKey`
  - Gemini 的 API key
  - 没有它就没法建立向量索引

- `embeddingModel`
  - 默认值：`gemini-embedding-2-preview`
  - 一般不用改

- `embeddingDimensions`
  - 默认值：`768`
  - 一般不用改

- `autoCollect`
  - 是否自动收集聊天里新出现的表情包合集
  - `true` = 开启
  - `false` = 完全手动

如果你没有在插件配置里写 `embeddingApiKey`，插件也会尝试读取这些环境变量：

- `GEMINI_API_KEY`
- `GOOGLE_API_KEY`

如果你希望 agent 发贴纸更频繁一点、或者克制一点，也可以直接在聊天里告诉它，或者把偏好写进 memory，让它自己调整发贴纸的频率。

如果你要让别的 OpenClaw 实例全自动安装这个插件，可以看：

- [OpenClaw auto-install guide](./docs/OPENCLAW_AUTO_INSTALL.md)

---

## License

MIT
