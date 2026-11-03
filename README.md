# AIollama —— QQ 本地机器人（AstrBot + Ollama 版）

## 系统架构

```
QQ 消息 ⇄ NapCat(QQ协议端, 反向WS :6190) ⇄ AstrBot(4.25.6, WebUI :6185)
    AstrBot ──► Ollama 本地多模型（deepseek-r1 / qwen2.5 / smegmma / rwkv7）
            ├─ 插件 simple_search  ：免 Key Bing 联网搜索（/search + web_search_simple）
            ├─ 插件 ai_friend      ：用户记忆 / 表情包收藏发送 / 主动分享
            ├─ 知识库「网络知识库」：Bing 定时抓取 → nomic-embed-text → faiss 向量库
            └─ 每日定时任务        ：AIollama_daily_fetch（每天 02:30 抓取入库）
```

## 各组件与数据位置

| 组件 | 位置 | 说明 |
|---|---|---|
| AstrBot | `E:\AIollama\astrbot` | v4.25.6，工作目录含 `.astrbot`；WebUI `http://localhost:6185` |
| AstrBot 数据 | `E:\AIollama\astrbot\data` | cmd_config.json（UTF-8 BOM）、plugins、知识库 |
| 插件 | `E:\AIollama\astrbot\data\plugins` | simple_search（搜索）、ai_friend（记忆/表情/分享） |
| Ollama 模型数据 | `E:\AIollama\models` | deepseek-r1:7b、qwen2.5:7b、Yinr/Smegmma:9b、rwkv7-g1l:7.2b、nomic-embed-text |
| 知识库 | `E:\AIollama\astrbot\data\knowledge_base` | faiss 向量库 + sqlite 文档库（kb_id `e86ea809-…`） |
| 定时抓取 | `E:\AIollama\bot\daily_fetch.py` | 5 主题 Bing 抓取 → 写 `data\knowledge\` → 上传知识库 |
| 表情包库 | `E:\AIollama\data\emojis` | 白名单 QQ 发的表情自动收藏（上限 500 张） |
| QQ 交流数据 | `E:\AIollama\data\YYYY-MM-DD.jsonl` | 历史消息记录（旧桥接保留） |
| 日志 | `E:\AIollama\logs` | astrbot.log（含最新初始密码）、daily_fetch.log 等 |
| NapCat | `E:\AIollama\napcat` | v4.18.42，账号级配置 `onebot11_3792752848.json` |
| Git 仓库 | `H:\AOllama Git` | data 为指向 `E:\AIollama\data` 的 junction，每日快照 |

## 已安装软件清单

| 软件 | 版本 | 位置 |
|---|---|---|
| Ollama | 0.40.13 | `C:\Users\Rice-MO\AppData\Local\Programs\Ollama`（未入 PATH，须全路径调用） |
| AstrBot | 4.25.6 | `E:\AIollama\tools\python313\Scripts\astrbot.exe` |
| Python（嵌入式） | 3.13.16 | `E:\AIollama\tools\python313` |
| NapCat | v4.18.42 | `E:\AIollama\napcat` |
| Git | 2.56.0 | `C:\Program Files\Git` |
| QQ 客户端（QQNT） | — | `D:\Program Files\Tencent\QQNT\QQ.exe`（登录账号 3792752848） |

## 使用方式（QQ）

- **私聊**：直接给机器人发消息即回复
- **群聊**：@机器人 或唤醒词（默认 `/` 前缀）触发；`/reset` 重置当前会话
- **白名单**：QQ `3211853507`（唯一白名单；非白名单消息不触发主动回复/表情收藏）
- **每用户独立会话**：`unique_session=true`，每个 QQ 用户保留独立聊天窗口
- **联网搜索**：`/search <关键词>`（别名 `/搜索`）；模型也可自主调用 `web_search_simple`
- **表情包**：白名单 QQ 发的图片自动收藏；`/表情` 随机发一张；`/表情列表` 查看
- **用户记忆**：模型自主调用 `remember_user` 记录用户信息（姓名/喜好等），每次回复自动带出记忆
- **主动分享**：模型可自主调用 `send_emoji`（发表情）、`share_content`（分享内容）
- **主动回复**：群聊中 15% 概率主动搭话（`active_reply`，仅白名单）

## WebUI 调试

- 地址：`http://localhost:6185`，用户名 `astrbot`
- **初始密码每次重启随机**，从 `E:\AIollama\logs\astrbot.log` 末行 `Initial password:` 读取；
  登录后请立即修改密码（password_change_required=true）

## 定时任务

| 任务 | 时间 | 内容 |
|---|---|---|
| AIollama_daily_fetch | 每天 02:30 | Bing 抓取 5 主题 → 写入 `E:\AIollama\data\knowledge\` → 上传知识库（multipart） |

知识库嵌入：`nomic-embed-text`（768 维，provider `ollama_emb`）。文档入库约 6 块/篇。

## 维护操作

- 启动 AstrBot：`E:\AIollama\tools\python313\Scripts\astrbot.exe run`（工作目录 `E:\AIollama\astrbot`）
- 停止 AstrBot：`E:\AIollama\logs\astrbot_kill.py`（杀进程后重启，否则旧进程占用端口/文件）
- 启动 NapCat：`E:\AIollama\bot\start_napcat_hidden.bat`（含 `-q 3792752848` 免扫码）；
  NapCat WebUI `http://127.0.0.1:6099/webui`
- 立即抓取入库：`E:\AIollama\tools\python313\python.exe E:\AIollama\bot\daily_fetch.py`
- 查看知识库：WebUI → 知识库面板（接口 `/api/kb/list`、`/api/kb/stats?kb_id=e86ea809-c4ab-46f0-8e48-496dde126bb9`）

## 关键配置（cmd_config.json，UTF-8 BOM 读写）

- provider：ollama-deepseek（默认，deepseek-r1:7b）/ ollama-qwen / ollama-smegmma / ollama-rwkv / ollama_emb（embedding，768 维）
- platform：aiocqhttp-qq（`0.0.0.0:6190`，token `AIollama2026`）
- `id_whitelist: ["3211853507"]`、`unique_session: true`
- `active_reply: {enable: true, method: possibility_reply, possibility_reply: 0.15, whitelist: ["3211853507"]}`

## 注意事项与排障经验

1. **cmd_config.json 是 UTF-8 BOM**：改配置须用 utf-8-sig 读写，否则中文乱码/解析失败。
2. **NapCat 只认账号级配置**：改 `config\onebot11_3792752848.json` 才生效（通用 onebot11.json 也要改）。
3. **NapCat 重启须 `-q 3792752848`**：否则落到二维码登录导致连不上 6190。
4. **deepseek-r1 不调用 llm_tool**（工具调用能力弱）：联网搜索可让用户用 `/search`，或换 qwen2.5 模型。
5. **知识库删除**：AstrBot 运行中删除报 WinError 32（文件占用）→ 先重启 AstrBot 再删。
6. **知识库入库失败「存储失败」**：曾因 AstrBot 4.25.6 用 naive datetime 与新版 sqlmodel 冲突，
   已补丁 `document_storage.py`（`datetime.now()` → `datetime.now(timezone.utc)`，5 处）。
7. **WebUI 初始密码每次重启随机**：脚本（daily_fetch.py 等）自动从 astrbot.log 末行提取，勿写死。
8. **Ollama 未入 PATH**：脚本一律用全路径 `C:\Users\Rice-MO\AppData\Local\Programs\Ollama\ollama.exe`。
9. **schtasks 不在 PATH**：用全路径 `C:\Windows\System32\schtasks.exe`。
10. **旧桥接 main.py（:6199）**：已被 AstrBot 接管 QQ 对话，去留未定（其 23:50 记忆提炼/快照与
    AstrBot 侧能力有重叠，暂无冲突）。

## 已验证（2026-11-04）

- AstrBot 接管 QQ：aiocqhttp(OneBot v11) 适配器已连接，netstat 6190 双向 ESTABLISHED ✓
- 多模型调度：WebUI 全链路（登录 → new_session → send）deepseek-r1 正常回复 ✓
- 联网搜索（simple_search）：`web_search_simple` 工具注册；qwen2.5:7b 自主调用并基于结果总结 ✓
- 定时抓取入库：daily_fetch.py 5 主题 → 知识库 **5 文档 / 30 块** ✓；计划任务 AIollama_daily_fetch（02:30）✓
- 知识库补丁：document_storage.py 时区修复后入库成功（faiss_repro.py 回归通过）✓
- 新插件 ai_friend：remember_user / recall_user / send_emoji / share_content 工具注册成功 ✓
- 白名单 / unique_session / active_reply 配置已写入并在重启后生效 ✓
