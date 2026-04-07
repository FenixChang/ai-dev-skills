# Antigravity 部署指南

## 架構說明

| 機制 | 路徑 | 說明 |
|---|---|---|
| Global scope skills | `~/.gemini/antigravity/skills/` | 跨所有專案，按需觸發 |
| Workspace scope skills | `{project}/.agents/skills/` | 僅限特定專案，版控共享 |
| 安全控制 | Antigravity UI → Settings → Deny List | 系統層級，優於 prompt 層 |

> **重要**：`security-guardrail` 在 Antigravity 上**不應**作為 SKILL.md 部署。
> Antigravity 有原生的 Allow/Deny List 機制，安全控制在 permission layer 生效，
> 比 prompt 層的 SKILL.md 指令更強且更可靠。

## 部署步驟

### Step 1：設定 Deny List（取代 security-guardrail）

在 Antigravity UI → Settings → Security → Deny List 加入以下規則：

**檔案 Deny List（File Patterns）**：
```
.env
.env.*
*.key
*.pem
*.pfx
*.p12
id_rsa
id_ed25519
*.secret
*credentials*
*_credentials*
```

**指令確認規則（Command Confirmation）**：
在 Terminal Command Auto Execution 設定中，將以下指令設為「需要確認」：
- `rm -rf`
- `git push --force`
- `chmod` （目錄外）

### Step 2：安裝 Global scope skills（跨所有專案）

```bash
mkdir -p ~/.gemini/antigravity/skills

cp -r skills/architect-thinker ~/.gemini/antigravity/skills/
cp -r skills/atomic-patcher ~/.gemini/antigravity/skills/
cp -r skills/git-workflow-pro ~/.gemini/antigravity/skills/
```

這三份 skill 適用於所有專案，不論技術棧。

### Step 3：安裝 Workspace scope skills（按專案）

對於**有完整測試套件**的專案：
```bash
cd {your-project}
mkdir -p .agents/skills
cp -r path/to/ai-dev-skills/skills/self-healing-loop .agents/skills/
```

對於**大型、多模組**的專案：
```bash
cd {your-project}
mkdir -p .agents/skills
cp -r path/to/ai-dev-skills/skills/multi-agent-orchestrator .agents/skills/
```

建議將 `.agents/skills/` 加入 git，供團隊共享。

### Step 4：確認安裝

在 Antigravity 的 Agent Manager 中啟動一個新 agent，
側邊欄應顯示已識別的 skills 數量。

## Skills 與 Antigravity Manager 的搭配

Antigravity 支援真正的多 agent 並行，`multi-agent-orchestrator` 的
「True Multi-Agent Mode」部分在這裡可以完整發揮：

1. 在 Manager View 開啟新任務
2. 主 agent 執行 `multi-agent-orchestrator` 的分解邏輯
3. 將 Coder / Tester / Reviewer 階段分派給不同 agent
4. 主 agent 執行最終整合測試

## 更新 skills

```bash
git pull origin main
cp -r skills/architect-thinker ~/.gemini/antigravity/skills/
# 重啟 Antigravity 以重新載入 skills
```
