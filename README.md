# better-prompt — 面向开发者的提示词需求澄清 skill

一个面向中文小白开发者用户，能把模糊的初版提示词，澄清成 AI 能准确执行、可复用的终版提示词的一个skill。

目前该skill暂时只适配了deepseek harness。

覆盖三类生成域：**生图 / 生文 / 生成代码**。

**只交付提示词，不执行任务。** 提问是有限的预算，不是默认动作；原文够好就不改。

## 四步流程

```text
用户给初版提示词
   │
   ├─ 判定优先级  先判域 → 原话够清楚就放行 → 只筛「阻塞题」→ 0 题即交付
   │
   ├─ 第 1 步  澄清问卷   简单 0-1 / 复杂 2-3 题、上限 3 题
   │           判定口诀：一个条件句能接住它，就不问
   │
   ├─ 第 2 步  检索同类开源项目（GitHub/Gitee 及 Hugging Face、npm/PyPI 等）
   │           ≤4 次搜索、≤3 个仓库、≤1 次额外抓取；采纳 ≤2 条机制
   │           检索失败不阻塞 → 用内置机制库兜底，并如实标注
   │
   ├─ 第 3 步  第二轮优化   冻结目标（默认不阻塞）→ 只做四类改动 → 输出终版
   │
   └─ 第 4 步  交用户审核   "用这条？要改哪句直接说。" 改哪句只改哪句
```

## 设计要点（差异化）

> 只列结论；具体规则、话术与数值一律以 [SKILL.md](skill/better-prompt/SKILL.md) 为准，本表不重复其正文。

| 要点 | 说明 |
|---|---|
| 三域覆盖 | 生图 / 生文 / 生成代码，各有独立提问树与终版骨架；跨域取主交付物那一域 |
| 预算化提问 | 按阻塞维度数分档：简单任务 0-1 题、复杂任务 2-3 题；区间不是配额，0 题交付是正常结果 |
| 判定优先级置顶 | 先判域、再判够不够清楚、只筛阻塞题——避免规则互相推翻 |
| 阻塞题判定口诀 | 「一个条件句能不能接住它？」能接住就不问 |
| 原文够好即不改 | 明确允许"无需修改"的结论，不为改而改 |
| 改动白名单 | 只允许：消歧义 / 补输出契约 / 加删约束 / 删冗余。禁止加角色装饰与语气词 |
| 内容完整性优先 | 字数上限是软约束，允许超 ≤20% 并说明；不许为压字数删条件句/接口/验收项 |
| 参考机制不抄文案 | 采纳 ≤2 条，必须落到加/改/删动作；终版不出现项目名 |
| 三档来源标注 | 【已确认】/【暂定】/【待确认】；具体数值、数量、参考物同属断言 |
| 渐进披露 | 主文件只放流程，模板/平台表/机制库放 `references/`，一轮通常 2-3 份 |
| 执行模式 | 用户直接要交付物时仍走流程，交付后问"要我接着生成吗"，不代执行 |

## 兼容性

当前适配：DeepSeek Harness（DSH）。其他 harness 暂未验证。

Roadmap: 后续计划适配更多 harness，SKILL.md + references/ 的通用结构保持不变。

## 安装

DSH 的本地 skill 来源（按优先级）：

| 优先级 | 来源 | 路径 |
|---|---|---|
| 100 | 项目级 | `<项目根>/.dsh/skills` |
| 200 | 项目级（兼容） | `<项目根>/.agents/skills` |
| 300 | 自定义 | 配置项 `customSkillDirs` |
| 400 | 用户级 | `~/.dsh/skills` |

安装到用户级（所有项目可用）：

```powershell
Copy-Item -Recurse -Force '.\skill\better-prompt' "$env:USERPROFILE\.dsh\skills\better-prompt"
```

安装到项目级（只在该项目可用）：

```powershell
New-Item -ItemType Directory -Force '.\.dsh\skills' | Out-Null
Copy-Item -Recurse -Force '.\skill\better-prompt' '.\.dsh\skills\better-prompt'
```

要求：

- bundle 目录**必须直接包含 `SKILL.md`**，不要多套一层。
- frontmatter 必填 `name`（kebab-case，须与目录名一致）与 `description`。
- 不支持嵌套 `**/SKILL.md` 发现。
- `references/` 属于 bundle 资源，编辑它不会触发目录刷新，但每次加载都会重读正文。

生效方式：新增/改名/删除 skill 会在**下一个模型步骤**触发目录刷新，通常无需重启。

## 使用

**本 skill 不会自动触发**（当前安装副本带 `disable-model-invocation: true`）。在对话里点名，或从 `/` 菜单选：

```text
/better-prompt 帮我优化这条提示词：写个脚本处理日志，要健壮一点
```

开发者三域都适用，例如：

| 域 | 示例请求 |
|---|---|
| 生图 | 帮我画张产品主图，要高大上一点 |
| 生文 | 写个接口文档，专业一点 |
| 生成代码 | 加个分页接口，别动现有的返回结构 |

**值得点名调用**：请求含糊、缺交付物/栈/接口/输出契约/成功标准，且猜错会浪费一次生成。
**不必点名**：请求已具体可直接执行；你只要"现在就把事做了"；只是问概念、聊想法、报 bug。

想让模型自动判断是否调用，就删掉 `SKILL.md` frontmatter 里的 `disable-model-invocation: true`——代价是简单任务上可能误触发。

## 目录结构

```text
better-prompts\
├── README.md                                      13.6 KB   本文件：定位、安装、使用、改名记录、缺陷修正史
├── LICENSE                                         1.0 KB   MIT 协议
│
└── skill\
    └── better-prompt\                                       ← 可安装产物（整个目录即一个 skill bundle）
        ├── SKILL.md                               15.3 KB   判定优先级 + 四步流程 + 门槛与纪律
        └── references\
            ├── questionnaire.md                    10.8 KB   域判定、三域提问树、题面模板、反例
            ├── prompts-by-domain.md                5.2 KB   生图/生文/生成代码 的终版骨架与范例
            ├── prompt-templates.md                 3.8 KB   四层结构（指令/情境/参考/输出）与写法规则
            ├── research-brief.md                  10.0 KB   检索词、平台清单、源优先级与实测结论、降级、事实核查
            ├── patterns.md                         3.3 KB   检索不可用时的内置机制库
            └── vagueness-map.md                    5.1 KB   模糊词映射（代码域/视觉域/通用）
```

**skill 部分 7 个文件 / 53.4 KB**（仓库中的测试证据见 [examples/README.md](examples/README.md)）。

三条结构约定：

1. **`skill/better-prompt/` 是可安装产物**，目录名必须与 frontmatter 的 `name` 一致（DSH 会跳过不一致的 skill）。安装就是整个目录复制到 `~/.dsh/skills/`。
2. **`SKILL.md` 必须在 bundle 根部**，`references/` 属于 bundle 资源、不属于发现层级——DSH 不会把 `references/*.md` 当成独立 skill。
3. **`references/` 按需加载**：主文件指明每个文件"何时读"，一轮通常 2 份、最多 3 份。

## 工具链踩坑记录

**不要用 PowerShell 的内置 cmdlet 改写本仓库的中文 Markdown。** 实测：`Get-Content -Raw` + 正则替换 + `Set-Content -Encoding UTF8` 会把整个文件的中文写成乱码（单次操作即损坏全文，本次导致根 `README.md` 被重写重建）。

- 改中文文档一律用编辑工具（读取 → 精确替换），不要用 shell 字符串替换。
- 复制/移动文件用 `Copy-Item` / `Move-Item` **不做内容读写**，是安全的；复制后用 SHA256 校验（本次三份原始导出哈希与源文件一致）。
- 若必须用 PowerShell 读中文，显式指定 `[System.IO.File]::ReadAllText($p,[System.Text.Encoding]::UTF8)`。
- 体检方法：查乱码特征序列 `鈥|鍩|閽|锛|绗|銆`——注意导出文件里本来就含技术字符，需结合人工抽样确认，避免误报。

## 更新方向

当前只适配：DeepSeek Harness（DSH）。

已完成：
- 三域覆盖（生图 / 生文 / 生成代码）
- 按阻塞维度数分档的提问预算

计划中：
- 生视频：并入生图域（共享画面/构图/风格骨架，新增时长与运动两维）
- 生音频：暂不独立成域，用「就近映射」兜底（TTS→生文、音乐/音效→生图）
- 三域字数上限：待实测上限字数是否适合大部分场景，再进行后续字数上限调整

不做：
- 生音频独立成第四域（低频需求，会破坏单轮读取份数）
- 多镜头分镜 / 音画同步（超出"提示词澄清"边界）
