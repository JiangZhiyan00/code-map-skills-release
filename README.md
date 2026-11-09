# Code Map · 代码地图生成技能

> **把复杂工程与业务逻辑，讲成一张可交互、可下钻的代码全景地图。**

Code Map 是一个面向 AI Coding Agent（如 Codex、Claude Code、Cursor、WorkBuddy 等）与研发工程师的**代码地图生成引擎**。

输入工程上下文（一次迭代 Commit、一份需求文档、一个陌生仓库），输出一张**交付级、自包含、单文件交互式 HTML 地图**。读者沿主线动线浏览，点击任意节点实现工程树、拓扑图与说明卡的三方联动高亮，全程无需配置复杂运行环境，双击即可离线运行。

---

## 核心交互形态

一张标准的 Code Map 包含以下核心要素：

- **左栏 · 工程结构树**：IDE 风格的包/类/方法树，支持按名搜索、折叠收起以及变更状态徽标（新增/调整/复用）。
- **中栏 · 三图联动看板**：
  - **全景模块图（Flowchart LR）**：按架构分层呈现模块依赖与拓扑。
  - **核心时序图（SequenceDiagram）**：完整拆解一次典型请求的调用链路。
  - **方法流程图（Flowchart TB）**：展现关键方法内部逻辑分支、降级与异常出口。
  - **标准五色语义标记**：核心绿（🆕） / 基准复用灰 / 外部依赖蓝 / 分支兜底橙 / 错误拦截红。
- **右栏 · 节点深度卡片**：聚焦“设计意图、实现细节、避坑指南”，支持按类型（新增/调整/测试/复用）过滤。
- **三方联动体系**：在任意一栏点击节点，另外两栏同步聚焦并高亮，辅以动态 SVG 连线直达目标。
- **主线故事导览（TOUR）**：顶部站点导航条预设 6~10 个关键站点，串联完整业务故事，无需阅读全量代码即可理清主脉络。
- **零外部依赖交付**：内嵌离线 Mermaid 引擎，单文件 HTML 双击即开，离线可用，方便评审演示与沉淀入库。

> **产物语言规则**：地图中的展示文本（节点名称、解析卡片、导览文案）**遵循用户 Agent 的当前语言环境与对话偏好**，支持中文、英文等任意语言输出，不作硬性限制。

---

## 适用场景（S1 ~ S5）

| 场景代码 | 适用场景 | 输入素材 | 产出成果与价值 |
|---|---|---|---|
| **S1** | **技术教学 / 演进章节** | 章节 commit / 教程文档 | 串联演进图：骨架装配 → 核心抽象 → 主链路 → 边界兜底，末尾带思考题 |
| **S2** | **迭代复盘 / 版本对比** | 两个 Git 基线（`extract-diff.sh`） | 变更复盘图：改动原因 / 影响面扩散 / 回滚预案，重点标注风险点 |
| **S3** | **需求实现蓝图（施工图）** | 需求描述 + 现有代码架构 | 开发施工图：目标态类清单 + 导览站点即推荐实现与提测顺序 |
| **S4** | **PR 讲解 / 审查导览** | 一个 PR 的 Git diff | 审查导览图：局部变更子图 + 意图/实现/关注点，末站附评审结论 |
| **S5** | **系统认知 / 架构 Onboarding**| 陌生仓库全量结构 | 新人认知地图：分层职责 + 一次请求的生命周期 + 本地快速启动卡 |

---

## 生成管线（生成原理）

核心设计哲学：**生成 = 换数据，不改引擎。**

```
场景识别 ───> 素材采集 ───> 数据构建 ───> 一致性校验 ───> 单文件交付
  (S1~S5)    (Git/源码/需求)  (置换 MAP-DATA)  (validate-map)   (present_files)
```

1. **引擎固定**：`assets/code-map-template.html` 为固定渲染引擎 + 示例骨架。生成时仅替换 `MAP-DATA` 区间的 7 个核心数据常量：
   - `META`（元信息配置对象，驱动标题/副标题/语言/统计/欢迎弹层）
   - `OVER_SRC`（模块全景 Mermaid 源码）
   - `SEQ_SRC`（调用时序 Mermaid 源码）
   - `FLOW_SRC`（方法流程 Mermaid 源码）
   - `TREE`（工程树结构 JSON）
   - `DOCS`（说明卡片列表 JSON）
   - `TOUR`（主线导览站点 JSON）
2. **联动防断链**：树节点通过 `kw` 关键词锚定图内节点，校验器静态比对每个 kw 是否均为某张图源码文本的子串，杜绝“点击无响应”，并针对过短关键词预警。
3. **页签自适应**：按需裁剪页签（例如仅需全景图时），删除多余页签 DOM 及对应变量，引擎自适应渲染。

---

## 质量红线与静态校验

在地图交付前，必须运行一致性校验脚本并确保 **0 错误**：

```bash
node scripts/validate-map.js <输出地图.html>
```

校验器严格检查以下维度：
- **数据结构完整性**：MAP-DATA 区间必须声明必需数据结构（TREE / DOCS / TOUR + 至少 1 个图源码；可选声明 META 并验证格式）。
- **说明卡完整性与引用**：DOCS 基础字段齐全，TREE 节点的 `doc` 引用必须在 DOCS 中真实存在，预警孤儿卡片。
- **联动精准度（核心）**：TREE 的全部 `kw` 关键词必须在 Mermaid 图源码中有效命中，对不足 3 字符的模糊关键词做防歧义警告。
- **TOUR 动线合理性**：站点引用的页签合法，引用的树节点必须能匹配，站点数量建议 6~10 站。
- **Mermaid 语法硬红线拦截**：
  - Flowchart **严禁嵌套 subgraph**（打包版 Mermaid 会解析报 Syntax error）。
  - 时序图 **禁止使用独立保留词 `END`/`end` 作为 Actor 标识**（避免与块闭合关键字冲突）。

---

## 目录结构

```
code-map-skills/
├── .github/workflows/package.yml # GitHub Actions 自动校验与打包发布流水线
├── SKILL.md                      # AI 技能定义入口（工作流、触发词、质量红线）
├── README.md                     # 项目使用指南与设计规范
├── assets/
│   ├── code-map-template.html    # 基础渲染模板（生成时置换 MAP-DATA 区间）
│   └── vendor/mermaid.min.js     # 离线 Mermaid 渲染库（交付时随图分发）
├── references/
│   ├── data-spec.md              # 数据契约：TREE / DOCS / TOUR / 图源码规范与限制
│   └── scenarios.md              # 5大场景策略表、素材来源与动线编排指南
└── scripts/
    ├── extract-diff.sh           # 采集两个 Git 基线间的变更（S2/S4 场景素材）
    └── validate-map.js           # 生成后一致性静态校验脚本（0 错误红线保障）
```

---

## 快速上手

### 1. 安装技能

方式一：从公开发布仓库 `code-map-skills-release` 的 `latest` Release 下载 `code-map-skills.zip`，解压到 Agent 的 skills 目录。

方式二：将本目录拷贝或软链接至 Agent 的 skills 目录，例如：

```bash
# 复制到对应 Agent 的技能目录
cp -r code-map-skills ~/.agents/skills/
```

### 2. 对话式调用指令示例

在支持该技能的 AI Agent 中直接下发任务：

- *“生成这次迭代的代码地图，对比基线 v1.0 与 HEAD”*（触发 S2 迭代对比）
- *“根据这个业务需求和现有仓储，出具一份开发实现蓝图导览”*（触发 S3 需求蓝图）
- *“帮我串讲当前 PR 的改动点与潜在影响”*（触发 S4 PR 评审）
- *“为当前项目生成一份新人架构认知与请求生命周期地图”*（触发 S5 系统 Onboarding）
- *“用英文为当前模块生成一份代码地图”*（产物自动以英文输出，设置 META.lang = 'en'）

### 3. 人工调试与快速上手（非 Agent 模式）

无需 Agent 亦可手动生成或调试地图：

1. 拷贝模板与资源库：
   ```bash
   mkdir -p docs/code-map/my-feature/vendor
   cp assets/code-map-template.html docs/code-map/my-feature/index.html
   cp assets/vendor/mermaid.min.js docs/code-map/my-feature/vendor/
   ```
2. 参考 `references/data-spec.md` 替换 `index.html` 中的 `MAP-DATA-START` 至 `MAP-DATA-END` 区间。
3. 运行静态一致性校验：
   ```bash
   node scripts/validate-map.js docs/code-map/my-feature/index.html
   ```
4. 双击 `index.html` 即可在任意现代浏览器中离线交互浏览。
