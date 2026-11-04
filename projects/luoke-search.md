# 螺壳搜索 luoke-search

- 仓库：`luoke-search/`（顶层的子模块）；进度：`luoke-search/STATUS.md`；术语：`luoke-search/CONTEXT.md`
- 本页由 luoke-search 的会话维护。

## 是什么

本机的搜索：给 Mac 上用户选的文件夹和照片图库建索引，按字词和按意思找回来。想不起东西放在哪、只记得内容的时候用。

- **第一版搜**：
  - 文档：txt、md、rtf、doc、docx、odt、html、xlsx、pptx、csv、tsv（含 GBK 编码的文字文件）；
  - PDF：一页一页读，扫描件按页 OCR；
  - 截图：认出截图，读里面的字；
  - 图片、视频（关键帧和音轨）、录音；
  - 照片图库：经 PhotoKit 读"照片"里的照片和视频，要用户允许。
  - 不搜代码。
- **第二步**：记忆里的记录和依据原文、打字记录（都经记忆读）、邮件（要邮件助手给读取接口）。都从已有组件读，螺壳搜索自己只存向量和关键词，不重复存原文。
- **没有自己的界面**：没有菜单栏图标、窗口、快捷键面板。用户在螺壳里问（对话由螺壳的后台组件 agent 负责），比如"上个月拍的那张白板照片""合同里写押金的那页"，agent 调螺壳搜索，把结果画成卡片：标题、命中的片段、出处（第几页、第几秒），点一下在 Mac 上打开。要用户定的事（搜哪些文件夹、要不要照片图库）也由 agent 的卡片来问。
- **对外叫法**（顶层 CONTEXT「对外怎么叫」，10-25）：名字「螺壳搜索」；用户入口写"在螺壳里"，按意思搜的推理由"螺壳"提供。下文的"主控"只用于讲内部分工。
- **形态**：一个常驻的后台 .app `螺壳搜索.app`（不出现在程序坞），由螺壳（luoke-core）安装、在登录时拉起（11-03 起进内部版）；同一个可执行文件也是命令行 `luoke-search`。bundle id `ai.luoke.search`（已定）。
- **只做 macOS**，最低 26.2。
- **首发**：随 beta 2（封闭测试）第一次发布，和 agent 一起。图片和声音的向量要主控配方里打开对应的参数（还没合）；在那之前，装好的螺壳搜索里图片、视频、录音只能按文件名和 OCR 的字找。
- **和 digup 的关系**：从头写。原来基于开源 DigUp（MIT）的 digup 不登记；顶层 ADR 0018「digup 的嵌入」一节由本项目接手，任务改名为 `luoke-search.embed`。
- **现状**：2026-10-20 立项，第一版代码已在 main（2026-10-21）：读文档、PDF 和扫描件、截图、图片、录音、视频和照片图库，按字词和按意思搜，给 agent 的 MCP 和命令行，常驻服务和 .app。10-26 起装好后还没选要搜的地方时，经 outbox 给一张现成方案的卡片；10-28 接进螺壳的窗口（配件清单加了 `service`、`permissions`、`listing`，见下）。11-03 起随内部版出；11-04 对外名字从「螺壳深度搜索」改成「螺壳搜索」（包名一起改），换上新的应用图标。版本 0.1.4。进度见 `luoke-search/STATUS.md`。

## 对外提供

**给 agent：MCP 工具**（agent 工具协议版本 1）。在 `~/.luoke-search/run/agent.sock` 上提供；`luoke-search mcp` 在标准输入输出上提供同一组工具。

| 工具 | 只读 | 做什么 |
|---|---|---|
| `search_query` | 是 | 参数 `query`，可选 `kinds`（文档、PDF、截图、图片、视频、录音、照片）、`since` / `until`、`limit`。答复：结果列表（编号、类型、标题、出处、页码或时刻、命中片段、为什么对得上、分数）、`partial`（还在建索引或缺向量时为真，附一句说明），加一张 A2UI 结果卡片 |
| `search_open` | 否（列进 `actions`，要用户点按；`ai.luoke/where: mac`） | 按结果编号在 Mac 上打开，跳到那一页或那一刻；或在访达里显示 |
| `search_status` | 是 | 索引了多少、还在等什么、主控能不能算向量 |
| `search_settings` | 是 | 看现在的设置；给出要改的，就出一张确认卡片（方案） |
| `search_settings_apply` | 否（列进 `actions`） | 执行用户在卡片上点了的方案：搜哪些文件夹、跳过哪些、要不要照片图库、用电池时建不建索引；可选 `checks`（`needs-setup` 卡片上勾了哪几项） |
| `search_open_system_settings` | 是 | 打开 macOS 系统设置里对应的隐私页（给"要你允许"那张卡片上的按钮用） |
| `search_request_access` | 否（列进 `actions`，要用户点按；`ai.luoke/where: mac`） | 照片图库的申请（清单 `permissions` 里的 `request`）：用户在卡片上点了「继续」以后，由螺壳搜索自己的程序请 macOS 弹出询问，等用户答了（最多 4 分钟）再答复 |
| `outbox_ack` | 是 | agent 告诉螺壳搜索 outbox 里的一件事已经给用户看过、答过或放下了 |

**资源**（工具协议 §1a 的相对 id）：`summary`（概况，螺壳窗口里螺壳搜索那一页用）、`permissions`（每项权限现在的状态）、`outbox`（同下）。

**outbox** `luoke://luoke-search/outbox`（相对 id `outbox`），只报三类事，不推搜索结果：
- `needs-setup`（normal）：装好后还没选任何文件夹、照片也没开时报一件。卡片上是一个现成的方案：桌面、文稿、下载（勾上），照片图库（不勾）；用户点了 [好] 才改，[算了] 以后不再报；
- `needs-access`（low）：要用户允许读某个文件夹或照片图库，带去系统设置的说明；照片的卡片上先点「继续」，再请 macOS 问。每次又要用户处理时换新编号（`needs-access/<文件夹>/<编号>`、`needs-access/photos/<编号>`），agent 当成新的一件；
- `index-ready`（low）：第一遍索引建好了。

**命令行** `luoke-search`：`serve`、`index`、`search`、`open`、`status`、`settings`、`eval`、`mcp`，开发用的 `photos-probe`、`probe-audio`；`search --json` 的输出和 MCP `search_query` 的答复同一个格式。只给 agent 用的 `search_open_system_settings`、`search_request_access`、`outbox_ack` 没有命令。照片图库只有 app 本身读（app 里的 `open`、`mcp`、`index --job` 都读），终端里跑的命令行不读。`--index` 指定索引目录，`LUOKE_HUB_SOCKET` 指定主控，开发和测试不碰正式的索引和正在用的主控。发布版（.app 里的）只接 `serve`、`index`、`status`、`search`、`open`、`settings`、`mcp`、`help`；开发开关里 `--index`、`--socket`、`LUOKE_SEARCH_HOME`、`CFFIXED_USER_HOME` 会让它拒绝启动，别的去掉以后照常跑。

**配件清单**（`Resources/luoke-accessory.json`，11-04 的 main，0.1.4）：

```json
{
 "contract": 1,
 "accessory": "luoke-search",
 "bundle_id": "ai.luoke.search",
 "name": "螺壳搜索",
 "version": "0.1.4",
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
   {"name": "search_open", "title": "在 Mac 上打开搜到的文件，或在访达中显示", "writes": true},
   {"name": "search_request_access", "title": "请 macOS 问你要不要让螺壳搜索读照片图库", "writes": true}
  ],
  "service": {"summary": true, "ask": {"search": "search_query"}},
  "permissions": [
   {"id": "photos", "kind": "system", "tcc": "Photos", "title": "照片图库",
    "facts": ["只在你让它搜照片时才读", "只读，不改图库里的东西", "只用这台 Mac 上已有的，不从 iCloud 下载", "结果不出这台 Mac"],
    "without": "degrade", "request": "search_request_access"}
  ],
  "listing": {"tagline": "在这台 Mac 上找文件和照片，答案带出处", "category": "效率"}
 }
}
```

**评测集**（交给 luoke-core）：
- 参考向量集：9 条参考输入和标准向量（文字 5、截图 / 照片 / 视频帧各 1、声音 1），从 digup 原样拷来，文件不改。主控算的向量对它余弦 ≥ 0.999 是及格线（ADR 0018）。
- 检索评测：全部是合成语料和题目。起点是 digup 的四组（中文 51 题、文档格式 11 题、英文 37 题、孟加拉文和阿拉伯文 48 题，后两组是现在的题数），加小票 7 题；新加了英文有答案的 36 题、"语料里没有答案"的题（中文 37、英文 39）和文档多、图少的几组。照片图库的题、带时间范围的口语问题还没有。

## 依赖

| 依赖 | 用来做什么 | 现状 |
|---|---|---|
| luoke-core | 嵌入任务 `luoke-search.embed`（`POST /tasks/luoke-search.embed/v1/embed`）、`GET /tasks`；安装、在登录时拉起常驻 | 接口已有，任务名 `luoke-search.embed`（11-02 改名），三档都分到 embeddinggemma-2；图片和声音的参数还没进正式配方（见下） |
| luoke-agent | 唯一的入口：调 MCP 工具、画结果卡片、转达 outbox | 照工具协议版本 1（含 §1a 的 `service`、`permissions`、`listing`）接好了；全局搜索的 `service.ask.search`、统一的结果形状、缩略图等 luoke-agent#3 定 |
| luoke-mem | 第二步：读记录和依据原文 | 第一版不用 |
| luoke-mail | 第二步：搜全部邮件 | 待商定 |
| macOS | Vision（OCR）、PDFKit、AVFoundation、PhotoKit、FSEvents | 系统自带 |

## 对别的项目的需求

**对 luoke-core**（[luokelabs/luoke-core#16](https://github.com/luokelabs/luoke-core/issues/16)）：
1. 任务 `digup.embed` 改名为 `luoke-search.embed`，接口不变 —— **已做**（11-02）。
2. 分档 —— **已做**：16、32、128 三档都分到 embeddinggemma-2。分档前要补的几项（起嵌入进程之前查 `fits_memory`、`wired_gb` 按实测写进配方、闲时停、内存压力下停、参数变了重启）请 luoke-core 确认。
3. 配方的 `verified` 加 luoke-search（不加的话全部 403）—— **已做**（`dev.json`）。
4. 图片和声音：配方里打开图片和声音的参数（`image_batch`、`audio_batch` 这类）—— **还没合**；参考向量集里 4 条媒体向量也要余弦 ≥ 0.999。
5. 进内部版的流水线（`tools/internal_build.py`）—— **已做**（11-03）。
6. `tools/embed_parity.py` 改成 luoke-search 的说法 —— **已做**。
7. 清单登记 `jobs`（第一遍建索引交给主控的空闲任务，主控已支持 `exec`）—— 主控那边已做，这边还没登记，常驻进程里的兜底在用。
8. 量和延迟：
   - 查询：每问一句一条，`user-waiting`，≤ 64 字；端到端 p90 在 100 ms 以内（M1）。实验室主控上实测端到端（要查询向量 + 混合排序）：20 个查询 p50 15 ms、p90 17 ms、最大 35 ms（10-20）。
   - 建索引：`background`。第一次几千到几万个请求，每个最多 32 段；之后只有改动的文件，每分钟几个。
   - 嵌入进程的内存：开了图片和声音 3.1–4.3 GB（phys_footprint，实验室主控多次实测）。

**对 luoke-agent**（[luokelabs/luoke-agent#1](https://github.com/luokelabs/luoke-agent/issues/1)）：照工具协议接上面的 MCP 工具、结果卡片和 outbox。

**待谈**：
- **拒答门槛**：评测数字已经有了（起步建议的门槛拦下 78% 没答案的题，有答案的题被拒 16%，见 `luoke-search/evals/README.md`）。门槛还没开：答复里的 `confident` 除了一个结果都没有时，都是 `null`；agent 不把弱结果当答案说。
- **邮件读取接口**：搜全部邮件要邮件助手给一个读取接口（例如按时间范围列出邮件、按编号取正文）。第二步时在 luoke-mail 开 issue 商定；在那之前不碰邮件。
- **和 agent**：谁判断"这一问要搜"、手机上要什么，agent 那边已经定了；A2UI 组件目录里没有图片组件，缩略图怎么办还没定。

## 在记忆里读写什么

- **第一版**：不读也不写。搜索词不写进记忆。
- **第二步**：读各工具的记录和依据原文（经 `luoke_mem`，ADR 0013 允许），在自己的索引里存向量和关键词，引用记录编号；不写记录和理解，不复制原文。记忆下结论，螺壳搜索找回原话和原件，两边不重叠。

## 要守的底线

- **不带模型**（ADR 0018）：向量只找主控要；主控不在或不提供时只按字词搜，不退回自带的模型。OCR 用系统的 Vision。
- **数据不出本机**（ADR 0013）：索引在 `~/.luoke-search/`（目录只有本账户能读），排除出 Time Machine；向主控只送文字、图片和声音的字节，不送路径。
- **凭证出去时遮**（ADR 0013，10-22 修订）：名字像密码、密钥、证书的文件不读；读到的文件里的完整证件号、卡号等原样进本机索引、原样送主控算向量，拿完整号码能搜到；结果、卡片、片段、标题交给 agent 时再遮：卡号、证件号只留后四位，密码、验证码、密钥整串遮。outbox 里不放凭证。
- **日志只记元数据**：不记搜索词、片段和文件内容。
- **只读用户的文件**：不改、不移动、不上传。
- **不主动找用户**（ADR 0014）：只经 agent 的 outbox 报三类事；改设置、打开文件、请 macOS 问照片权限，都只在用户在卡片上点了以后（工具协议 §5）。
- 开发时把 MCP 接到用云端模型的客户端上，搜到的片段会离开本机，所以这样用时只对着合成的测试语料。
