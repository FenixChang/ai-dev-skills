# AI Dev Skills

六份基於 Claude Code 2026/3/31 原始碼洩漏內容提煉的開發行為規範，適用於 Gemini CLI、Antigravity 等 agentic 開發工具。

## 背景

2026年3月31日，Anthropic 意外在 npm 上發佈了 Claude Code v2.1.88 的完整 TypeScript 原始碼（512,000 行）。本 repository 的 skills 是從該次洩漏中提取的核心工程實踐，並針對跨工具部署進行了驗證與修正。

## Repository 結構

```
ai-dev-skills/
├── skills/                        # 六份核心 skill（跨工具通用格式）
│   ├── architect-thinker/
│   ├── atomic-patcher/
│   ├── self-healing-loop/
│   ├── git-workflow-pro/
│   ├── security-guardrail/
│   └── multi-agent-orchestrator/
├── deploy/
│   ├── gemini-cli/                # Gemini CLI 專用部署指南與 GEMINI.md
│   └── antigravity/               # Antigravity 專用部署指南
└── docs/
    └── ANALYSIS.md                # 完整分析報告與錯誤修正記錄
```

## Skills 總覽

| Skill | 用途 | 部署位置建議 |
|---|---|---|
| `architect-thinker` | 修改前的環境感知與上下文理解 | User scope（跨專案） |
| `atomic-patcher` | 精準最小化修改，避免大規模重寫 | User scope（跨專案） |
| `self-healing-loop` | 自動測試與錯誤自癒迴圈 | Workspace scope（有測試套件的專案） |
| `git-workflow-pro` | Git 提交規範與 PR 流程 | User scope（跨專案） |
| `security-guardrail` | 安全底線，防止敏感資訊洩漏 | Global（永久載入） |
| `multi-agent-orchestrator` | 大型任務分解與角色模擬 | Workspace scope（大型專案） |

## 快速部署

### Gemini CLI
```bash
# 1. 建立 skills 目錄
mkdir -p ~/.gemini/skills

# 2. 複製 User scope skills（跨專案通用）
cp -r skills/architect-thinker ~/.gemini/skills/
cp -r skills/atomic-patcher ~/.gemini/skills/
cp -r skills/git-workflow-pro ~/.gemini/skills/
cp -r skills/multi-agent-orchestrator ~/.gemini/skills/
cp -r skills/self-healing-loop ~/.gemini/skills/

# 3. 複製全域安全規則
cat deploy/gemini-cli/GEMINI.global.md >> ~/.gemini/GEMINI.md
```

### Antigravity
```bash
# 1. 建立 Global skills 目錄
mkdir -p ~/.gemini/antigravity/skills

# 2. 複製 Global scope skills
cp -r skills/architect-thinker ~/.gemini/antigravity/skills/
cp -r skills/atomic-patcher ~/.gemini/antigravity/skills/
cp -r skills/git-workflow-pro ~/.gemini/antigravity/skills/

# 3. 在有測試套件的專案中加入 self-healing-loop
mkdir -p .agents/skills
cp -r skills/self-healing-loop .agents/skills/

# 4. security-guardrail 請透過 Antigravity UI 的 Deny List 設定（見 deploy/antigravity/README.md）
```

## 版本記錄

- `v1.0.0`：初始版本，基於 Claude Code leak 分析（2026/04）
