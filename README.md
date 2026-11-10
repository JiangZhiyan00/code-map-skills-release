# Code Map · Interactive Architecture Blueprint Generator

> **Turn complex codebases and system workflows into interactive, self-contained single-file HTML code maps.**

<p align="center">
  <img src="assets/images/code-map.png" alt="Code Map Cover" width="480">
</p>

Code Map is an engineering-grade visualization skill designed for AI Coding Agents (such as Codex, Claude Code, Cursor, OpenCodeInterpreter, Gemini CLI, etc.) and software engineers.

Given codebase context—whether a recent Git diff, a feature PRD, or an unfamiliar repository—Code Map produces an **interactive, self-contained, single-file HTML panorama**. Readers can follow a guided narrative walkthrough or click any symbol to see the project structure tree, Mermaid diagrams, and deep-dive documentation cards synchronize with dynamic animated connectors. No server setup or complex dependencies required: simply double-click to open in any modern browser.

---

## Interactive Architecture

```
┌─────────────────────────┬───────────────────────────────────────────┬─────────────────────────┐
│       LEFT PANEL        │               CENTER PANEL                │       RIGHT PANEL       │
│      Project Tree       │          Tri-Diagram Dashboard            │     Deep-Dive Cards     │
│                         │                                           │                         │
│  IDE-style tree view    │  🗺 Overview (flowchart LR)                │  Architectural intent   │
│  • Instant search       │  ⏱ Sequence (sequenceDiagram)             │  • "Why it exists"      │
│  • Fold/expand toggles  │  🔀 Workflow (flowchart TB)               │  • Core algorithms      │
│  • Status badges        │  🧭 Guided Tour Rail (6~10 stations)      │  • Concurrency/pitfalls │
│    [NEW] [MOD] [BASE]   │  🎨 Standard 5-color semantic badges      │  • Category filter chips│
└────────────┬────────────┴─────────────────────┬─────────────────────┴────────────┬────────────┘
             │                                  │                                  │
             └─────────────────────── Tri-Directional Sync ────────────────────────┘
                              (Animated SVG dynamic connectors)
```

- **Left Panel · Project Tree**: IDE-style package/class/method hierarchy supporting instant keyword search, expand/collapse toggles, and status badges (`NEW` / `MOD` / `BASE` or localized).
- **Center Panel · Tri-Diagram Dashboard**:
  - **Architecture Overview (`flowchart LR`)**: Layered representation of modules, port contracts, and dependencies.
  - **Execution Sequence (`sequenceDiagram`)**: Step-by-step trace of critical end-to-end request flows.
  - **Method Workflow (`flowchart TB`)**: Internal algorithm branches, error boundaries, and circuit-breaker fallbacks.
  - **Standard 5-Color Semantics**: Green (`🆕`) for newly introduced elements, Grey dashed for reused baselines, Blue for external gateways, Amber for fallbacks, and Red for errors.
- **Right Panel · Node Deep-Dive Cards**: Focuses on "Why, not just what"—architectural rationale, state transitions, and edge-case pitfalls with category filtering (`New`, `Modified`, `Test`, `Reused`).
- **Tri-Directional Linkage**: Clicking any node across the tree, diagram, or documentation immediately focuses and highlights the corresponding elements across all three panels, connected by animated dynamic SVG lines.
- **Guided Narrative Tour (TOUR)**: The top navigation rail features 6 to 10 curated stations walking readers through the entire pipeline without needing to read thousands of lines of raw code.
- **Zero-Friction Multi-Language Adaptation (i18n)**: Automatically detects conversation language and localizes all UI chrome controls (buttons, tabs, legends, search bars) into **English, Simplified Chinese, or Japanese**, with seamless extensibility for additional languages.
- **Strict Single-File Delivery (Zero Trailing Folders)**: Outputs as a **single `.html` file** with zero companion `vendor/` folders. Supports fast CDN delivery by default, and in-place engine inlining for air-gapped offline environments.

---

## Dual-Mode Delivery (Never Create Companion Folders)

Code Map eliminates messy companion directories, ensuring generated maps never fail due to broken relative paths:

| Delivery Mode | Typical Size | Engine Source | Trailing Folders | Best For |
|---|---|---|---|---|
| **Default Mode (Online CDN)** | ~35 KB | Pinned public CDN (jsdelivr v11.17.2) | **None** | Daily development, PR reviews, lightweight git storage |
| **Offline Mode (Inlined Engine)** | ~3.5 MB | Embedded directly in HTML via `<script>` | **None** | Air-gapped networks, enterprise intranets, standalone sharing |

To convert a lightweight map into a self-contained offline map, run:
```bash
node scripts/inline-engine.js <path-to-map.html>
```

---

## Zero-Friction Multi-Language Adaptation (i18n)

The engine features a built-in i18n subsystem. The AI Agent automatically detects your prompt language and sets `META.lang`, completely localizing the output without any user friction:

| Language | `META.lang` | Content Generation | UI Chrome (Buttons, Tabs, Legend, Controls) |
|---|---|---|---|
| **English** | `'en'` | 100% English | `Walk Through Flow`, `Overview`, `Sequence`, `New`, etc. |
| **Simplified Chinese** | `'zh-CN'` | 100% Simplified Chinese | `跟着流程走一遍`, `全景图`, `时序图`, `新增`, etc. |
| **Japanese** | `'ja'` | 100% Japanese | `プロセスに沿って進む`, `全体図`, `シーケンス`, `新規`, etc. |
| **Extensible** | ISO Code | Target language | Business text in target language; UI gracefully falls back to English |

---

## Supported Scenarios (S1 ~ S5)

| Scenario Code | Use Case | Primary Input | Value & Output |
|---|---|---|---|
| **S1** | **Tech Teaching / Evolution** | Chapter commits / Tutorial docs | Step-by-step evolution map: Scaffolding → Core abstractions → Primary pipeline → Boundary handling. |
| **S2** | **Iteration Review / Version Diff** | Git diff across two baselines | Change review map: Motivation, blast radius analysis, and rollback plans. |
| **S3** | **Feature Implementation Blueprint** | PRD / Technical specification | Pre-implementation roadmap: Target-state classes; tour station order serves as the recommended coding and testing sequence. |
| **S4** | **PR Review Walkthrough** | Git diff of a single PR | Focused review map: Scoped diff closure highlighting concurrency safety, edge handling, and review checklists. |
| **S5** | **Architecture Onboarding** | Full repository codebase | Developer onboarding map: Layered responsibilities, end-to-end request lifecycle, and local setup guide. |

---

## Generation Pipeline

Core design principle: **Replace data payload; keep the engine immutable.**

```
Scenario & Lang Detection ───> Context Gathering ───> Data Construction ───> Static Validation ───> Delivery
   (S1 ~ S5 & zh/en/ja)          (Git/Source/Spec)      (Replace MAP-DATA)     (validate-map.js)    (present_files)
```

1. **Immutable Engine**: `assets/code-map-template.html` serves as the fixed rendering engine. Generation consists solely of replacing the 7 constants in the `MAP-DATA` region:
   - `META`: Metadata object driving title, subtitle, language (`lang`), counters, package namespace, and welcome modal.
   - `OVER_SRC`: High-level architecture overview (Mermaid `flowchart LR`).
   - `SEQ_SRC`: Execution sequence (Mermaid `sequenceDiagram`).
   - `FLOW_SRC`: Method logic & branching (Mermaid `flowchart TB`).
   - `TREE`: Hierarchical project tree structure (JSON).
   - `DOCS`: Detailed documentation cards (JSON).
   - `TOUR`: Guided narrative tour stations (JSON).
2. **Zero Broken Links**: Tree nodes anchor to diagrams via `kw` keywords. The validator statically verifies that every keyword matches a substring in diagram sources.
3. **Adaptive Tabs**: If a scenario only requires 1 or 2 diagrams, simply remove the unused tab DOM nodes and variables; the engine dynamically adapts its layout.

---

## Quality Gates & Static Validation

Before delivering any map, run the static consistency validator to ensure **0 errors**:

```bash
node scripts/validate-map.js <path-to-map.html>
```

The validator strictly verifies:
- **Data Structure Completeness**: `MAP-DATA` region exists and contains all required structures (`TREE`, `DOCS`, `TOUR`, and at least 1 Mermaid source).
- **Doc Card Reference Integrity**: Every `doc` ID in `TREE` resolves to a card in `DOCS`; warns on orphan documentation cards.
- **Linkage Accuracy (Core)**: Every keyword in `TREE.kw` matches at least one Mermaid diagram source; warns on ambiguous keywords under 3 characters.
- **Tour Navigation Validity**: Every tour station specifies a declared tab, and every `tree` reference resolves to an existing tree node.
- **Mermaid Guardrails**:
  - Flowcharts **must not contain nested subgraphs** (breaks bundled Mermaid parser).
  - Sequence diagrams **must not use standalone reserved words `END`/`end` as actor identifiers**.

---

## Skill Package Structure

```
code-map-skills/
├── SKILL.md                      # AI Coding Agent skill entrypoint (prompts, workflow, rules)
├── README.md                     # Project documentation and user guide
├── LICENSE                       # MIT License (v1.0.0)
├── assets/
│   ├── images/
│   │   └── code-map.png          # Skill cover illustration
│   ├── code-map-template.html    # Base HTML template with built-in i18n engine
│   └── vendor/mermaid.min.js     # Standalone Mermaid engine source (used by inline-engine.js)
├── references/
│   ├── data-spec.md              # Data contracts: TREE / DOCS / TOUR / Mermaid syntax specifications
│   └── scenarios.md              # Decision matrix, inputs, and narrative guide for S1~S5 scenarios
└── scripts/
    ├── extract-diff.sh           # Git diff change extractor for S2 / S4 scenarios
    ├── inline-engine.js          # Inlines Mermaid engine for 100% offline single-file delivery
    └── validate-map.js           # Static consistency validator (0-error quality gate)
```

---

## Quick Start & Usage

### 1. Install Skill

Copy or symlink this directory into your AI Agent's skill folder:

```bash
# Example for Claude Code / Codex / Custom Agents
cp -r code-map-skills ~/.agents/skills/
```

### 2. Conversational Prompts for AI Agents

Invoke the skill directly within your AI Coding Agent in your preferred language:

- **Simplified Chinese**:
  - *"生成这次迭代的代码地图，对比基线 v1.0 与 HEAD"* *(自动识别中文，生成 100% 简体中文 HTML)*
  - *"生成离线单文件版的代码地图"* *(触发内嵌引擎，生成无外部依赖的离线单文件)*
  - *"帮我串讲当前 PR 的改动点与潜在影响"* *(触发 S4 PR 评审)*
  - *"为当前项目生成一份新人架构认知与请求生命周期地图"* *(触发 S5 系统 Onboarding)*
- **English**:
  - *"Generate an interactive code map for this sprint comparing v1.0 to HEAD"* *(Automatic English detection & full English HTML)*
  - *"Generate an offline inlined code map"* *(Inlines engine for zero-dependency offline use)*
  - *"Based on this PRD and our existing repository, create a feature implementation blueprint"* *(Triggers S3 Blueprint)*
  - *"Walk me through the changes and architectural risks in this pull request"* *(Triggers S4 PR Review)*
- **Japanese**:
  - *"このリポジトリの全体的なアーキテクチャマップを作成してください"* *(日本語を自動検出し、100% 日本語の HTML を生成)*
  - *"オフライン単一ファイル版のコードマップを生成してください"* *(オフラインエンジン埋め込み)*
  - *"PR の変更点とリスクをウォークスルーするコードマップを作成してください"* *(S4 PR レビュー)*

### 3. Manual Workflow (Without an Agent)

You can also manually build or debug code maps:

1. Copy the template:
   ```bash
   mkdir -p docs/code-map/my-feature
   cp assets/code-map-template.html docs/code-map/my-feature/index.html
   ```
2. Open `index.html` and replace the `MAP-DATA-START` to `MAP-DATA-END` block following `references/data-spec.md`. Set `META.lang` to `'en'`, `'zh-CN'`, or `'ja'`.
3. (Optional) Convert to offline single-file:
   ```bash
   node scripts/inline-engine.js docs/code-map/my-feature/index.html
   ```
4. Run the static consistency validator:
   ```bash
   node scripts/validate-map.js docs/code-map/my-feature/index.html
   ```
5. Double-click `index.html` to view the interactive map in any modern browser.

---

## License

[MIT](LICENSE) © jiangzhiyan (v1.0.0)
