# 螺壳搜索 luoke-search

- 仓库：`luoke-search/`（顶层的子模块）；术语：`luoke-search/CONTEXT.md`；窗口的搜索页、应用页用的字段：`luoke-search/docs/窗口接口.md`
- 本页由 luoke-search 的会话维护。

## 是什么

本机的搜索：给 Mac 上用户选的文件夹和照片图库建索引，按字词和按意思找回来。想不起东西放在哪、只记得内容的时候用。

- **第一版搜**：
  - 文档：txt、md、rtf、doc、docx、odt、html、xlsx、pptx、csv、tsv（含 GBK 编码的文字文件）；
  - PDF：一页一页读，扫描件按页 OCR；
  - 截图：认出截图，读里面的字；
  - 图片、视频（关键帧和音轨；只有声音的视频按声音读）、录音；
  - 照片图库：经 PhotoKit 读"照片"里的照片和视频，要用户允许。
  - 不搜代码。
- **第二步**：记忆里的记录和依据原文、打字记录（都经记忆读）、邮件（要邮件助手给读取接口）。都从已有组件读，螺壳搜索自己只存向量和关键词，不重复存原文。
- **没有自己的界面**：没有菜单栏图标、窗口、快捷键面板。
  - 用户在螺壳里问（对话由螺壳的后台组件 agent 负责），比如"上个月拍的那张白板照片""合同里写押金的那页"。
  - agent 调螺壳搜索，把结果画成卡片：标题、缩略图、命中的片段、出处（第几页、第几秒），点一下在 Mac 上打开或在访达中显示，也能"找相似的"。
  - 螺壳窗口的全局搜索直接调 `search_query`，侧栏里有螺壳搜索的应用页（状态、进度、搜不搜照片图库）。
  - 要用户定的事（搜哪些文件夹、要不要照片图库）也由 agent 的卡片来问。
- **对外叫法**（顶层 CONTEXT「对外怎么叫」，10-25）：名字「螺壳搜索」；用户入口写"在螺壳里"，按意思搜的推理由"螺壳"提供。下文的"主控"只用于讲内部分工。
- **形态**：螺壳.app 里 `Contents/Helpers` 下的一个程序（顶层 ADR 0022，11-05 起）。
  - 由螺壳本体在登录时拉起、常驻，起之前核签名和版本。
  - 装好的这份只由螺壳起：除了 `status` 和帮助，启动时先认本体，不是螺壳起的就退出（11-10）。
  - 签名标识 `ai.luoke.search`。同一个可执行文件也是命令行 `luoke-search`。
  - 开发时照旧能打成独立的 `螺壳搜索.app`。
- **只做 macOS**，最低 26.2。
- **首发**：随 beta 2（封闭测试）第一次发布，和 agent 一起。图片和声音的参数 11-03 已经进了主控的配方。
- **和 digup 的关系**：从头写。原来基于开源 DigUp（MIT）的 digup 不登记；顶层 ADR 0018「digup 的嵌入」一节由本项目接手，任务改名为 `luoke-search.embed`。
- **现状**：2026-10-20 立项，10-21 第一版代码进 main，11-03 起随内部版出。版本 0.1.17。0.1.4 以后：
  - 收进螺壳.app（11-05）；系统权限交给螺壳本体问（11-06）；
  - 侧栏的应用页（11-06）；
  - 收回照片许可以后不再返回照片（11-07）；
  - 「在访达中显示」修好（11-09）；
  - 结果里去掉弱的、`strong` 改成看这一条自己（11-09，数字在 `luoke-search/evals/README.md`）；
  - 装好的只由螺壳起（11-10）。

## 对外提供

**给 agent：MCP 工具**（agent 工具协议版本 1）。在 `~/.luoke-search/run/agent.sock` 上提供；`luoke-search mcp` 在标准输入输出上提供同一组工具。

| 工具 | 只读 | 做什么 |
|---|---|---|
| `search_query` | 是 | 参数 `query`（只按时间、类型、来源列时可以不带），可选 `kinds`（文档、PDF、截图、图片、视频、录音、照片）、`sources`（`files`、`photos`）、`since` / `until`、`limit`。答复：结果列表（编号、类型、标题、出处、页码或时刻、命中片段、为什么对得上、分数、`strong`）、`totals`（各来源一共对上几条）、`partial`（还在建索引或缺向量时为真，附一句说明），加一张 A2UI 结果卡片 |
| `search_similar` | 是 | 和一条结果相似的（按向量），参数 `id`，可选 `sources`、`limit`；答复和 `search_query` 一样 |
| `search_open` | 否（列进 `actions`，要用户点按；`ai.luoke/where: mac`） | 按结果编号在 Mac 上打开，跳到那一页或那一刻；`reveal` 为真时在访达中显示 |
| `search_status` | 是 | 索引了多少、还在等什么（停着的说为什么）、主控能不能算向量、在搜哪些文件夹 |
| `search_settings` | 是 | 看现在的设置；给出要改的，就出一张确认卡片（方案） |
| `search_settings_apply` | 否（列进 `actions`） | 执行用户在卡片上点了的方案：搜哪些文件夹、跳过哪些、要不要照片图库、用电池时建不建索引；可选 `checks`（`needs-setup` 卡片上勾了哪几项）；应用页上的照片开关不带方案、只带 `photos` |
| `search_open_system_settings` | 是 | 打开 macOS 系统设置里对应的隐私页（文件和文件夹、完全磁盘访问、照片） |
| `outbox_ack` | 是 | agent 告诉螺壳搜索 outbox 里的一件事已经给用户看过、答过或放下了 |

`strong`：这一条本身就很对得上，不看它排第几。要找的字都在它自己的名字里（文件名，不算扩展名和上面的文件夹；照片的标题），或者意思上很近。一条都没有也可能；只按时间、类型列出的都不算。全局搜索顶上的「最相关」用它。

**资源**（工具协议 §1a 的相对 id）：
- `summary`：概况；
- `permissions`：每项权限现在的状态；
- `page/overview`：侧栏应用页，一张列表：精确搜索、语义搜索的状态和进度，文件夹，搜照片图库的开关，最近读入的时间；
- `thumb/<编号>`：结果的缩略图，不列出，编号从结果里来；
- `outbox`：同下。

**outbox** `luoke://luoke-search/outbox`（相对 id `outbox`），只报三类事，不推搜索结果：
- `needs-setup`（normal）：装好后还没选任何文件夹、照片也没开时报一件。卡片上是一个现成的方案：桌面、文稿、下载（勾上），照片图库（不勾）；用户点了 [好] 才改，[算了] 以后不再报；
- `needs-access`（low）：要用户允许读某个文件夹或照片图库，带去系统设置的说明。照片的权限在螺壳名下，由螺壳本体在自己的窗口里问，螺壳搜索不向 macOS 要权限。每次又要用户处理时换新编号（`needs-access/<文件夹>/<编号>`、`needs-access/photos/<编号>`），agent 当成新的一件；
- `index-ready`（low）：第一遍索引建好了。

**命令行** `luoke-search`：
- 命令：`serve`、`index`（`index --job` 是能停能续的第一遍）、`search`、`open`、`status`、`settings`、`eval`、`mcp`，开发用的 `photos-probe`、`probe-audio`。
- `search --json` 的输出和 MCP `search_query` 的答复同一个格式。
- 只给 agent 用的 `search_similar`、`search_open_system_settings`、`outbox_ack` 没有命令。
- 开发时 `--index` 指定索引目录、`LUOKE_HUB_SOCKET` 指定主控，开发和测试不碰正式的索引和正在用的主控。
- 装好的那份（螺壳.app 里的）：
  - 除了 `status` 和帮助，只由螺壳起；
  - 开发开关里 `--index`、`--socket`、`LUOKE_SEARCH_HOME`、`CFFIXED_USER_HOME` 会让它拒绝启动，别的开关去掉以后照常跑。
- 照片图库只有装好的这份读，终端里跑的命令行不读。

**配件清单**（`Resources/luoke-accessory.json`，0.1.17）：

```json
{
 "contract": 1,
 "accessory": "luoke-search",
 "bundle_id": "ai.luoke.search",
 "name": "螺壳搜索",
 "version": "0.1.17",
 "type": "app",
 "entry": [],
 "resident": true,
 "memory": false,
 "ui_element": true,
 "tasks": [{"name": "luoke-search.embed", "kind": "embed", "main": true}],
 "data": ["~/.luoke-search"],
 "min_macos": "26.2",
 "capabilities": {
  "protocol": 1,
  "socket": "~/.luoke-search/run/agent.sock",
  "reports": [
   {"kind": "needs-setup", "title": "螺壳搜索还没选要搜的地方", "urgency": "normal"},
   {"kind": "needs-access", "title": "螺壳搜索需要你允许读取", "urgency": "low"},
   {"kind": "index-ready", "title": "第一遍索引建好了", "urgency": "low"}
  ],
  "actions": [
   {"name": "search_settings_apply", "title": "改螺壳搜索的设置（搜哪些文件夹、要不要照片图库）", "writes": true},
   {"name": "search_open", "title": "在 Mac 上打开搜到的文件，或在访达中显示", "writes": true}
  ],
  "service": {"summary": true, "pages": [{"id": "overview", "title": "概览"}], "ask": {"search": "search_query"}},
  "permissions": [
   {"id": "photos", "kind": "system", "owner": "luoke", "tcc": "Photos", "title": "照片图库",
    "facts": ["仅在开启照片搜索时读取", "只读，不修改图库", "不从 iCloud 下载", "数据不离开本机"],
    "without": "degrade"},
      {"id": "folders", "kind": "each-time", "title": "读你选的文件夹",
       "facts": ["仅读取你选择的文件夹", "首次读取时由 macOS 询问", "数据不离开本机"]},
      {"id": "quicktime", "kind": "each-time", "title": "让 QuickTime Player 跳到视频的那一秒",
       "facts": ["仅在播放视频片段时使用", "首次使用时由 macOS 询问"]}
  ],
  "listing": {"tagline": "在这台 Mac 上找文件和照片，答案带出处", "category": "效率"}
 }
}
```

**评测集**（交给 luoke-core）：
- 参考向量集：9 条参考输入和标准向量（文字 5、截图 / 照片 / 视频帧各 1、声音 1），从 digup 原样拷来，文件不改。主控算的向量对它余弦 ≥ 0.999 是及格线（ADR 0018）。
- 检索评测：全部是合成语料和题目。
  - 起点是 digup 的四组：中文 51 题、文档格式 11 题、英文 37 题、孟加拉文和阿拉伯文 48 题（后两组是现在的题数）。
  - 加了：小票 7 题；英文有答案的 36 题；"语料里没有答案"的题（中文 37、英文 39）；文档多、图少的几组；同主题的文档和图（护照、钢琴课、发票这类）；录音的题。
  - 照片图库的题、带时间范围的口语问题还没有。
  - 去掉弱结果、`strong` 的门槛是在这些题上量的，数字在 `luoke-search/evals/README.md`。

## 依赖

| 依赖 | 用来做什么 | 现状 |
|---|---|---|
| luoke-core | 嵌入任务 `luoke-search.embed`（`POST /tasks/luoke-search.embed/v1/embed`）、`GET /tasks`；本体拉起常驻程序、核签名和版本、替它问系统权限、播放视频片段 | 任务名 `luoke-search.embed`（11-02 改名），三档都分到 embeddinggemma-2；图片和声音的参数 11-03 进了配方 |
| luoke-agent | 唯一的入口：调 MCP 工具、画结果卡片、转达 outbox；窗口的全局搜索和侧栏应用页 | 照工具协议版本 1（含 §1a 的 `service`、`pages`、`permissions`、`listing`）接好了 |
| luoke-mem | 第二步：读记录和依据原文 | 第一版不用 |
| luoke-mail | 第二步：搜全部邮件 | 待商定 |
| macOS | Vision（OCR）、PDFKit、AVFoundation、PhotoKit、FSEvents | 系统自带 |

## 对别的项目的需求

**对 luoke-core**：
1. 任务 `digup.embed` 改名为 `luoke-search.embed`，接口不变 —— **已做**（11-02）。
2. 分档 —— **已做**：16、32、128 三档都分到 embeddinggemma-2。
3. 配方的 `verified` 加 luoke-search（不加的话全部 403）—— **已做**。
4. 图片和声音：配方里打开图片和声音的参数（`image_batch`、`audio_batch` 等）—— **已做**（11-03）。参考向量集里 4 条媒体向量也要余弦 ≥ 0.999。
5. 进内部版的流水线 —— **已做**（11-03）。
6. 嵌入的对拍工具改成 luoke-search 的说法 —— **已做**。
7. 清单登记 `jobs`（第一遍建索引交给主控的空闲任务）—— **还没登记**，现在由常驻程序里的兜底跑第一遍。
   - 原来等的「空闲任务的进程不带主控的系统权限」主控已经做了。
   - 搜索收进螺壳.app 以后，要像扫描一样登记成经螺壳起的任务，用螺壳的照片权限；登记前先和主控确认。
8. 量和延迟：
   - 查询：每问一句一条，`user-waiting`，≤ 64 字；端到端 p90 在 100 ms 以内（M1）。实验室主控上实测端到端（要查询向量 + 混合排序）：20 个查询 p50 15 ms、p90 17 ms、最大 35 ms（10-20）。
   - 建索引：`background`。第一次几千到几万个请求，每个最多 32 段；之后只有改动的文件，每分钟几个。
   - 嵌入进程的内存：开了图片和声音 3.1–4.3 GB（phys_footprint，实验室主控多次实测）。
9. 「在访达中显示」时把访达叫到前面：螺壳搜索是螺壳的子进程，macOS 不让它把别的应用叫到前面，要螺壳本体来做 —— **待主控定**。

**对 luoke-agent**：照工具协议接上面的 MCP 工具、结果卡片和 outbox；窗口的全局搜索照 `luoke-search/docs/窗口接口.md` 接。

**待谈**：
- **拒答门槛**：评测数字已经有了（起步建议的门槛拦下 78% 没答案的题，有答案的题被拒 16%，见 `luoke-search/evals/README.md`）。门槛还没开：答复里的 `confident` 除了一个结果都没有时，都是 `null`；agent 不把弱结果当答案说。
- **全局搜索里各应用的结果怎么排**：先用各应用的 `strong` 放顶上的「最相关」；要不要把各应用的结果按意思混排，在和 agent 商量。
- **邮件读取接口**：搜全部邮件要邮件助手给一个读取接口（例如按时间范围列出邮件、按编号取正文）。第二步时和邮件助手商定；在那之前不碰邮件。索引里多存一份邮件，要等索引加密（ADR 0024）。

## 在记忆里读写什么

- **第一版**：不读也不写。搜索词不写进记忆。
- **第二步**：读各工具的记录和依据原文（经 `luoke_mem`，ADR 0013 允许），在自己的索引里存向量和关键词，引用记录编号；不写记录和理解，不复制原文。记忆下结论，螺壳搜索找回原话和原件，两边不重叠。

## 要守的底线

- **不带模型**（ADR 0018）：向量只找主控要；主控不在或不提供时只按字词搜，不退回自带的模型。OCR 用系统的 Vision。
- **数据不出本机**（ADR 0013）：索引在 `~/.luoke-search/`（目录只有本账户能读），排除出 Time Machine；向主控只送文字、图片和声音的字节，不送路径。
- **凭证出去时遮**（ADR 0013，10-22 修订）：
  - 名字像密码、密钥、证书的文件不读。
  - 读到的文件里的完整证件号、卡号等原样进本机索引、原样送主控算向量，拿完整号码能搜到。
  - 结果、卡片、片段、标题交给 agent 时再遮：卡号、证件号只留后四位，密码、验证码、密钥整串遮。
  - outbox 里不放凭证。
- **日志只记元数据**：不记搜索词、片段和文件内容。
- **只读用户的文件**：不改、不移动、不上传。
- **不主动找用户**（ADR 0014）：只经 agent 的 outbox 报三类事；改设置、打开文件都只在用户在卡片上点了以后（工具协议 §5）。系统权限由螺壳本体问，螺壳搜索只读状态。
- 开发时把 MCP 接到用云端模型的客户端上，搜到的片段会离开本机，所以这样用时只对着合成的测试语料。
