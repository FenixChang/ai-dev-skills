# 完整分析報告

## 背景：Claude Code 2026/03/31 原始碼洩漏

2026年3月31日，Anthropic 在 npm 上的 `@anthropic-ai/claude-code` v2.1.88 意外附帶了完整的 TypeScript source map，
導致 512,000 行原始碼公開數小時。本 repository 的所有 skills 皆基於對該次洩漏的分析提煉。

洩漏揭示的核心工程實踐：
- 完整 system prompt 與工具定義
- Bash tool 的設計哲學（優先 bash 執行多步驟操作）
- 字串替換式（string-replacement）的檔案編輯策略
- 條件式 system prompt 動態組裝架構
- 3+ 檔案修改強制觸發驗證的機制

---

## 各 Skill 的錯誤修正記錄

### architect-thinker

**原版問題**：
- 使用 `<thinking>` 標籤（Claude 特有語法，Gemini CLI 等工具不支援）
- `ls -R` 在大型專案產生過量輸出

**修正**：
- `<thinking>` → 改為 response 中的顯式「Analysis 區塊」
- `ls -R` → 改為 `find . -maxdepth 3 -not -path '*/node_modules/*'`

---

### atomic-patcher

**原版問題（事實性錯誤）**：
- `eslint --fix` 被描述為「syntax checker」，但它實際上是 auto-fixer
- 在 pre-flight 階段使用 auto-fix 會覆蓋開發者意圖

**修正**：
- 明確區分「syntax check」和「auto-format」為兩個獨立步驟
- Syntax check（驗證）：`eslint src/`、`ruff check .`、`go vet ./...`
- Auto-format（選用格式化）：`prettier --write`、`ruff format .`、`go fmt ./...`
- 新增「先讀後寫」前置條件（直接來自 leak 的 string-replacement 邏輯）

---

### self-healing-loop

**原版**：邏輯正確，是六份中最完整的。

**補強**：
- 新增「相同錯誤連續出現即停止」的退出條件（避免無效重試）
- 明確禁止修改測試檔案
- 新增結構化的 SELF-HEALING FAILED 輸出格式

依據：洩漏統計顯示單一 session 曾出現 3,272 次連續失敗，印證了設置上限的必要性。

---

### git-workflow-pro

**原版問題（邏輯錯誤）**：
- 「Check for un-staged changes before committing to avoid polluting commit history」— 此說法有誤。
  Un-staged changes 不會進入 commit（`git commit` 本來就只 commit staged 內容）。
  真正的問題是「不相關的檔案被錯誤 staged」。

**修正**：
- 移除錯誤的 un-staged 邏輯
- 核心改為：`git diff --staged` 驗證 staged 內容是否都與本次任務相關
- 新增平行執行 `git status` + `git diff --staged` 的正確順序（來自 leak 的 PR 建立邏輯）
- 新增 `--force-with-lease` 建議（比 `--force` 更安全的 force push）

---

### security-guardrail

**原版問題**：
- `chmod` 與 `rm -rf` 並列為同等危險指令，過於嚴格
  （`chmod +x deploy.sh` 是極其普通的操作）

**修正**：
- 採用「可逆性 × 影響範圍」雙維度評估框架（來自 leak 的實際設計）
- `chmod` 改為只針對「專案目錄外」才需要確認
- 新增常見 token pattern 的自動遮罩規則（`sk-`、`ghp_`、`AKIA`、`ya29.`）

**部署注意**：
- Gemini CLI：放入 `~/.gemini/GEMINI.md`（永久載入，不作為按需 skill）
- Antigravity：透過 UI Deny List 設定（permission layer，強於 prompt layer）

---

### multi-agent-orchestrator

**原版問題**：
- 假設工具支援真正的多 agent 並行，但 Gemini CLI 等工具是單一對話流
- Worker-A/B/C 的角色分工在單 agent 工具中無法並行執行

**修正**：
- 明確區分「單 agent 順序模擬」和「真正多 agent」兩種執行模式
- 新增直接來自 leak 的強制驗證觸發條件（3+ 檔案修改、後端/API 變更、基礎設施變更）
- 新增結構化的 VERIFICATION PASS 輸出格式

---

## 部署架構決策

### 為什麼 security-guardrail 不作為 SKILL.md？

Skills 是按需觸發的，有觸發條件。安全規則必須**永遠生效**，不能等到「符合描述」才啟動。

- Gemini CLI：放入 `GEMINI.md`（作為 system prompt 永久載入）
- Antigravity：透過 Deny List（在 permission layer 生效，比 prompt 更強）

### 為什麼 self-healing-loop 和 multi-agent-orchestrator 放 Workspace scope？

- `self-healing-loop` 需要專案有測試套件才有意義，沒有測試的專案觸發後無法執行
- `multi-agent-orchestrator` 在小型任務引入反而增加複雜度和 token 成本

這兩份放在 Workspace scope，由開發者根據專案特性決定是否啟用。

---

## 版本記錄

| 版本 | 日期 | 說明 |
|---|---|---|
| v1.0.0 | 2026/04 | 初始版本，基於 Claude Code leak 分析與驗證 |
