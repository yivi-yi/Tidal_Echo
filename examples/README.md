# examples/ — 接入示例

给**不用 Claude Code**、或需要无人值守跑的人。完整部署 SOP 见仓库根的 [`AGENTS.md`](../AGENTS.md)。

| 文件 | 干什么 | 平台 |
|---|---|---|
| [`bridge_any_llm.py`](bridge_any_llm.py) | 把任意 OpenAI 兼容模型(GPT/DeepSeek/Gemini/GLM/Kimi/通义/本地…)接成 AI 侧 | 任意 |
| [`.env.example`](.env.example) | `bridge_any_llm.py` 的配置模板 | — |
| [`confirm_dev_channel_win.py`](confirm_dev_channel_win.py) | Windows 上自动确认 Claude Code 的 DevChannelsDialog 弹框 | Windows |

---

## 用任意 LLM 当大脑(bridge_any_llm.py)

它替代 `channel/` 插件,不依赖 Claude Code。原理是个三步薄循环:SSE 收
`/channel/in` → 拉历史拼 messages + 调你的模型 → POST `/channel/out`。零第三方依赖。

```bash
cd examples
cp .env.example .env
#  编辑 .env:填 RELAY_URL、RELAY_SECRET(和后端一致),以及你的模型三件套
#  LLM_API_BASE / LLM_API_KEY / LLM_MODEL(各家取值见 .env.example 里的注释表)
python3 bridge_any_llm.py
```

跑起来后,在手机 PWA 发一条 → 终端打印 `[in] #.. ` → 模型生成 → 手机收到回复。

- **换模型**只改 `.env` 的三件套,代码不动。Gemini 用它的 OpenAI 兼容端点即可。
- **兜底链**:填 `LLM_*_2` / `LLM_*_3`,主模型 401/403/429/5xx 时自动顺次切。
- **失忆?** 调大 `HISTORY_N`(默认喂最近 12 条)。
- **看图**:默认把附件降级成文字提示;要真看图,在 `handle_human_message` 里下载
  `/uploads/{name}?token=` 再按多模态格式喂(代码里有注释标位置)。

---

## Claude Code 的确认框自动过(无人值守)

只有走 Claude Code 这条路才有这个框。**Linux/macOS 用 tmux 最干净:**

```bash
tmux new-session -d -s cc 'claude --dangerously-load-development-channels server:companion'
sleep 3 && tmux send-keys -t cc Enter        # 替你确认 DevChannelsDialog
```

**Windows(无 tmux)** 用 [`confirm_dev_channel_win.py`](confirm_dev_channel_win.py):

```bash
python confirm_dev_channel_win.py -- claude --dangerously-load-development-channels server:companion
```

细节(为什么躲不掉、覆盖范围)见 [`AGENTS.md` §4](../AGENTS.md)。

---

## ⚠️ 单身体原则

relay 是单用户单通道。**同一时刻只跑一个 AI 侧** —— 别同时开着 Claude Code channel
和这个 bridge,否则两个都回复,用户看到双重消息。换大脑时先停旧的再起新的。
