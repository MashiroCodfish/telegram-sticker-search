# telegram-stickers-brain

一个给 **OpenClaw** 用的 Telegram 表情包语义搜索插件。

它做的事情很简单：

1. 同步 Telegram 表情包合集
2. 用 **Gemini Embedding 2** 给表情包建立向量
3. 把向量存到本地 **SQLite**
4. 搜索时在本地内存里做相似度匹配
5. 返回最合适的 `sticker_id`

如果你想要的是：
- 让 OpenClaw 更会发表情包
- 能按“开心 / 委屈 / 无语 / 摆烂”这种感觉去找图
- 不想搞一堆重型依赖和复杂服务

那这个插件就是干这个的。

## 能做什么

- 手动同步一个表情包合集
- 可选：自动收集聊天里新出现的表情包合集
- 用自然语言搜索表情包
- 返回 Telegram 可直接发送的 `sticker_id`

## 适合什么场景

比如你想让 agent：
- 看到别人发了一个新表情包合集后，自动记住它
- 聊天时根据“开心、得意、委屈、无奈”自动找贴纸
- 不依赖外部向量库，数据都留在本机

## 依赖要求

需要这些东西：

- 已经配置好的 **OpenClaw**
- 已经能正常工作的 **Telegram bot token**
- 一个 **Gemini API key**（用于 Embedding）
- **Node.js 18+**
- 建议有 `ffmpeg`（这样 `.tgs`、`.webm` 这类动图贴纸也能抽预览帧）

## 安装

### 方式一：从源码安装

```bash
git clone https://github.com/MashiroCodfish/telegram-stickers-brain.git
cd telegram-stickers-brain
npm install
openclaw plugins install .
```

装完后重启 Gateway。

### 方式二：从 release 包安装

先下载 release 里的 `roitium-telegram-stickers-brain-1.0.0.tgz`，然后执行：

```bash
openclaw plugins install ./roitium-telegram-stickers-brain-1.0.0.tgz
```

装完后重启 Gateway。

### 方式三：从 npm 安装

```bash
openclaw plugins install @roitium/telegram-stickers-brain
```

> 这个方式要等 npm 真正发布之后才能直接用。

## 配置

把配置写到：

```text
plugins.entries.telegram-stickers-brain
```

最小示例：

```json5
{
  "plugins": {
    "entries": {
      "telegram-stickers-brain": {
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

### 这些配置分别是什么意思？

- `embeddingApiKey`
  - Gemini 的 API key
  - 这是最重要的配置，没有它就没法建向量

- `embeddingModel`
  - 默认是 `gemini-embedding-2-preview`
  - 一般不用改

- `embeddingDimensions`
  - 向量维度
  - 默认 `768`
  - 一般也不用改

- `autoCollect`
  - 是否自动收集聊天里新出现的表情包合集
  - `true` = 开启
  - `false` = 完全手动

如果你没在插件配置里写 `embeddingApiKey`，插件也会尝试读取这些环境变量：

- `GEMINI_API_KEY`
- `GOOGLE_API_KEY`

## 怎么用

这个插件会给 OpenClaw 提供 3 个工具：

- `sync_sticker_set_by_name`
- `get_sticker_stats`
- `search_sticker_by_emotion`

### 1）同步一个表情包合集

可以传合集名，也可以直接传 Telegram 链接：

```json
{"setNameOrUrl":"https://t.me/addstickers/YourStickerSet"}
```

### 2）查看当前索引状态

```json
{}
```

它会告诉你：
- 现在已经索引了多少张表情包
- 队列里还有几个合集
- 自动收集现在是不是开着

### 3）按语义搜索表情包

比如：

```json
{"query":"开心 笑着 跑"}
```

或者：

```json
{"query":"无奈 叹气 摆烂"}
```

返回结果长这样：

```json
{"sticker_id":"CAACAgUAAxkBA..."}
```

拿到这个 `sticker_id` 以后，agent 就可以直接发贴纸。

## 推荐使用方式

### 纯手动模式

适合你想完全自己控制：

1. 手动同步几个常用表情包合集
2. 等索引完成
3. 聊天时按语义搜索
4. 发出匹配到的贴纸

### 自动收集模式

适合你想让它越用越聪明：

1. 打开 `autoCollect`
2. 聊天里出现新的表情包合集时，插件会自动入队
3. 后台慢慢建索引
4. 之后搜索时就能搜到这些新图

## 搜索建议

搜索词尽量写得像人在描述表情：

好例子：
- `开心 笑着 跑`
- `得意 比耶`
- `委屈 哭哭`
- `无奈 叹气 摆烂`
- `生气 拍桌子`

一般来说，**情绪 + 动作 + 特征** 这种组合最好用。

## 数据存在哪里

插件会在本地保存这些数据：

- `STATE_DIR/telegram-stickers-brain.sqlite`
  - 表情包向量索引

- `STATE_DIR/telegram-stickers-brain-tmp/`
  - 临时文件目录

- `STATE_DIR/telegram/sticker-cache.json`
  - 自动收集模式下，用来识别新合集

## 其他文档

如果你是人类运维或者别的 agent，可以看：

- 安装/验证文档：`docs/AGENT_INSTALL.md`
- 给其他 OpenClaw 实例自动安装的文档：`docs/OPENCLAW_AUTO_INSTALL.md`

## License

MIT
