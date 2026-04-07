# Gemini CLI 部署指南

## 架構說明

| 機制 | 路徑 | 說明 |
|---|---|---|
| 全域永久指令 | `~/.gemini/GEMINI.md` | 每次對話都載入 |
| User scope skills | `~/.gemini/skills/` | 跨所有專案，按需觸發 |
| Workspace scope skills | `{project}/.gemini/skills/` | 僅限特定專案，版控共享 |

## 部署步驟

### Step 1：安裝全域安全規則

```bash
# 如果 GEMINI.md 不存在則建立，存在則附加
cat deploy/gemini-cli/GEMINI.global.md >> ~/.gemini/GEMINI.md
```

確認內容：
```bash
cat ~/.gemini/GEMINI.md
```

### Step 2：安裝 User scope skills（跨專案通用）

```bash
mkdir -p ~/.gemini/skills

cp -r skills/architect-thinker ~/.gemini/skills/
cp -r skills/atomic-patcher ~/.gemini/skills/
cp -r skills/git-workflow-pro ~/.gemini/skills/
cp -r skills/multi-agent-orchestrator ~/.gemini/skills/
cp -r skills/self-healing-loop ~/.gemini/skills/
```

> **注意**：`self-healing-loop` 放在 User scope 是安全的。
> 當專案沒有測試套件時，Gemini 識別不到匹配的任務描述，不會觸發。
> 若你希望更嚴格控制，可改放在個別專案的 Workspace scope。

### Step 3：確認 skills 已被識別

啟動 Gemini CLI 後執行：
```
/skills
```

應看到五個 skill 出現在列表中。

### Step 4（選用）：Workspace scope 部署

對於特定專案（例如大型後端服務），可在專案根目錄加入：
```bash
mkdir -p .gemini/skills
cp -r path/to/ai-dev-skills/skills/self-healing-loop .gemini/skills/
cp -r path/to/ai-dev-skills/skills/multi-agent-orchestrator .gemini/skills/
```

如需版控共享給團隊，將 `.gemini/skills/` 加入 git（而非 `.gitignore`）。

## Skills 觸發方式

Gemini CLI 會自動識別任務描述並請求用戶確認：

| 你說了什麼 | 自動觸發的 skill |
|---|---|
| 「幫我看看這個專案的結構」 | `architect-thinker` |
| 「修改這個函式」 | `atomic-patcher` |
| 「跑測試看看有沒有問題」 | `self-healing-loop` |
| 「幫我 commit 這些變更」 | `git-workflow-pro` |
| 「這個功能需要改很多檔案」 | `multi-agent-orchestrator` |

安全規則（`GEMINI.global.md`）永遠在背景生效，不需要觸發。

## 更新 skills

```bash
# 從 GitHub 拉取最新版本
git pull origin main

# 重新複製更新的 skill
cp -r skills/architect-thinker ~/.gemini/skills/

# 在 Gemini CLI 中重新載入
/memory refresh
```
