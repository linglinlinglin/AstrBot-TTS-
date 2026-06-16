# AstrBot 悟声 TTS 插件 (wusound_tts)

## 项目描述

这是一个为 [AstrBot](https://github.com/Soulter/AstrBot) 框架开发的文本转语音（TTS）插件。
当 AI 生成回复时，插件会自动调用当前会话的 LLM 将原文本翻译为自然口语的日语（可选），并请求 **悟声 AI** 接口实时生成日语音频，最后以文件或语音气泡的形式发送到群聊或私聊中。

---

## 具体功能

- **🔤 可选日文翻译**：内置严格英文翻译提示词，默认开启。**可在配置中关闭翻译**，直接将中文等原文发送给悟声 TTS。
- **🧠 翻译专用 LLM**：可指定独立的 LLM 提供商专门负责翻译，与聊天 LLM 分离，避免上下文污染。
- **⏱️ 短回复拦截**：支持上下限 Token 阈值，只有短回复触发语音，长篇大论不会刷屏。
- **🛡️ 双层权限控制**：独立的「群聊过滤」与「用户过滤」，均支持 `whitelist` / `blacklist` / `none` 三种模式。
- **🗣️ 多种发送模式**：支持 `file`（音频文件）与 `record`（原生语音）。`record` 模式自动下载远程音频到本地缓存后发送。
- **🛠️ 零消耗 Mock 模式**：不调用悟声 API，生成本地测试音频验证发送链路。
- **🔍 丰富诊断命令**：翻译预览、会话 ID 查询、发送链路测试等。

---

## 安装与操作

### 1. 安装步骤

1. 进入 AstrBot 的插件目录：
   ```bash
   cd data/plugins
   ```
2. 克隆本仓库：
   ```bash
   git clone https://github.com/linglinlinglin/astrbot-wusound-tts.git astrbot_plugin_wusound_tts
   ```
3. 在 AstrBot 管理面板重启框架或重载插件。

### 2. 基础配置

在 AstrBot 的 WebUI 配置页面找到 `wusound_tts` 插件，填写以下核心配置：

- **api_key**: 悟声开发者中心申请的 API Key（也可在运行环境设置 `WUSOUND_API_KEY` 变量）。
- **voice_id**: 悟声语音角色 ID。
- **send_as**: 选择发送格式（`file` 或 `record`）。推荐先用 `file` 测试链路。

#### 翻译设置

- **translate_to_japanese**: 翻译开关。**开启**时文本会先被 LLM 翻译为日语再送 TTS；**关闭**时直接发送原文，适用于中文等悟声直接支持的语种。
- **translation_provider**: 翻译专用 LLM 的 provider ID。留空则使用当前会话的 LLM 做翻译。推荐为翻译单独配一个轻量模型，避免与聊天上下文互相干扰。
- **translation_prompt**: 翻译提示词，`{text}` 代表原文。插件已内置默认英文提示词，一般无需修改。

### 3. 权限配置（白名单/黑名单）

插件支持灵活的**群聊过滤**和**用户过滤**，两者是**叠加关系**（必须同时通过两层校验，插件才会工作）。

- **群聊过滤 (`group_filter_mode`)**: 
  - 可选 `whitelist` / `blacklist` / `none`。
  - 在 `group_filter_list` 中填写群号或完整会话 ID（如 `123456` 或 `onebot:GroupMessage:123456`）。
- **用户过滤 (`user_filter_mode`)**: 
  - 可选 `whitelist` / `blacklist` / `none`。
  - 在 `user_filter_list` 中填写允许或屏蔽的用户 ID。

> **注意**：如果任一层配置为 `whitelist` 但其列表为空，该层过滤将不会放行任何请求！

### 4. 调试命令（仅管理员可用）

在聊天框中发送以下命令，帮助你快速调试和配置：

- `/wusound_where`：获取当前聊天的 `group_id`、`user_id` 和完整会话 `origin`，并显示权限是否放行，用于填写过滤名单。
- `/wusound_preview`：预览 LLM 翻译结果，**不调用悟声 API**。用于排查输出文本是否混入了中文杂质或提示词是否对当前大模型失效。
- `/wusound_test`：用当前 `send_as` 配置发送一条测试音频。
- `/wusound_file_test`：强制以**文件**形式发送测试音频，用于排查文件链路。
- `/wusound_record_test`：强制以**语音**形式发送测试音频，用于排查底层协议端是否支持原生语音。

---

## 常见问题 (FAQ)

- **Q: 为什么发送 `file`（文件）正常，但改成 `record`（语音）就不发了？**
  - **A**: 不同平台或消息适配器对 `record` 的支持度不同。如果发送失败，说明底层协议端不支持该语音格式，建议退回使用 `file` 模式。

- **Q: 我想让机器人直接说中文，不要翻译成日语？**
  - **A**: 在插件配置中将 `translate_to_japanese` 设为 `false`，插件会直接将 AI 原文发给悟声合成语音。

- **Q: 语音结果总是有非日语文本，怎么办？**
  - **A**: 使用 `/wusound_preview` 命令或在控制台中查看实际翻译结果。如果被当前 LLM 污染，建议在 `translation_provider` 中指定一个独立的、指令遵循能力强的 LLM 专门做翻译。

- **Q: 发现生成的音频质量很差、乱读，如何排查？**
  - **A**: 请在群里发一条触发回复的消息后，查看控制台输出。
    1. 用 `/wusound_preview` 或在控制台查看翻译结果是否纯净 —— 如有污染，先解决翻译端。
    2. 如果翻译结果纯净但音频仍差，则是悟声 API 或当前 `voice_id` 的问题。

- **Q: 悟声已经扣除积分，但群里没收到音频？**
  - **A**: 可能是网络超时或发送失败。请开启 `mock_mode` 测试 —— 如果 Mock 下也收不到，说明是 AstrBot 到聊天平台的发送链路问题。

- **Q: 为什么有些回复不会触发语音？**
  - **A**: 插件有自动拦截机制，以下情况不触发：
    1. 文本含代码块、HTTP 链接或以 `/` 开头的指令。
    2. 回复字数低于 `min_output_tokens` 或超过 `max_output_tokens` 阈值。
    3. 翻译开启但 LLM 返回了无效译文（不含假名）。

- **Q: 插件命令（如 `/wusound_preview`）的回复会不会意外触发 TTS？**
  - **A**: 不会。所有插件命令的回复均已自动标记为跳过 TTS。

---

## 致谢

- [AstrBot](https://github.com/Soulter/AstrBot) - 优秀的 LLM 机器人框架
- [悟声 AI](https://www.wusound.cn/) - 提供高质量的 TTS 接口支持

---

## 许可证

本项目采用 [GNU Affero General Public License v3.0 (AGPL-3.0)](LICENSE) 许可协议。