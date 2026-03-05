# 第一階段學習計畫：打造骨架

> 對應 `saga_repo_learning_plan_v2.md` 第 1 週：Skill_Seekers / memsearch / superpowers / claude-hooks
> 目標：理解「知識→技能」、「記憶持久化」、「技能編排」、「可程式化執行」四大核心能力

---

## 學習目標總覽

完成此階段後，你應該能回答：

1. **Skill_Seekers**：如何把一個 GitHub repo 或文件網站，自動轉換成 Claude 可用的技能資產？
2. **memsearch**：如何讓 AI Agent 跨 session 記住過去的決策和知識？
3. **superpowers**：如何用可組合的技能定義，讓 Agent 遵循有紀律的開發工作流？
4. **claude-hooks**：如何在 Claude Code 的工具呼叫前後，插入自訂邏輯？

---

## Day 1-2：Skill_Seekers — 知識→技能轉換引擎

> **核心問題**：原始資料（docs、repo、PDF）如何變成 Agent 可用的結構化技能？

### 閱讀路線

| 順序 | 檔案 / 路徑 | 重點觀察 | 預計時間 |
|------|-------------|---------|---------|
| 1 | `README.md` | 整體架構、5 種輸入來源、16 種輸出目標 | 30 min |
| 2 | `BULLETPROOF_QUICKSTART.md` | 實際跑一次完整流程，建立體感 | 45 min |
| 3 | `configs/claude-code.json` | 設定檔結構：`sources[]`、`selectors`、`merge_mode` | 20 min |
| 4 | `src/skill_seekers/cli/main.py` | CLI 23 個指令的分派邏輯，理解 `create` 如何自動偵測輸入類型 | 30 min |
| 5 | `src/skill_seekers/cli/doc_scraper.py` | 核心爬蟲（~790 行），重點看智慧分塊與衝突偵測邏輯 | 45 min |
| 6 | `src/skill_seekers/cli/adaptors/` | 挑 2-3 個適配器（如 claude、langchain、cursor），理解輸出格式差異 | 30 min |
| 7 | `CLAUDE.md` + `AGENTS.md` | 給 AI agent 的開發指引，學習如何描述自己的專案給 AI | 15 min |

### 實作練習

```bash
# 用一個小型 repo 或文件網站跑一次完整流程
pip install skill-seekers
skill-seekers create <your-target-url> --output claude

# 觀察輸出的 SKILL.md 結構
# 思考：這個格式能否被你的系統直接使用？缺少什麼？
```

### 重點筆記方向

- [ ] 畫出處理流水線：`Scrape → Build → Enhancement → Package → Upload`
- [ ] 記錄 configs JSON 的完整欄位規格
- [ ] 比較至少 2 種輸出適配器的格式差異
- [ ] 記錄「衝突偵測」機制：文件 API vs 實際程式碼的差異如何被發現

### 關鍵架構洞察

```
★ Insight ─────────────────────────────────────
1. Strategy Pattern 是核心：16 個平台各有一個 adaptor，共用同一條處理流水線
   → 新增平台只需寫一個新 adaptor，不動核心邏輯
2. GitHub 三流分析（Code / Docs / Insights）是差異化設計
   → 不只看程式碼，還看 issues 和社群知識，更接近「人類理解 repo」的方式
3. 設定檔驅動 vs 硬編碼：所有行為都由 JSON config 控制
   → 你可以借鏡這個模式，讓自己的系統也用設定檔驅動
─────────────────────────────────────────────────
```

---

## Day 3-4：memsearch — Markdown-first 長期記憶層

> **核心問題**：如何讓 AI Agent 像人一樣「記住」過去的經驗，並在需要時快速找回？

### 閱讀路線

| 順序 | 檔案 / 路徑 | 重點觀察 | 預計時間 |
|------|-------------|---------|---------|
| 1 | `README.md` | 整體概念：Markdown 是 source of truth，Milvus 是可重建快取 | 30 min |
| 2 | `CLAUDE.md` | 架構概覽，給 Agent 的說明 | 15 min |
| 3 | `src/memsearch/chunker.py` | 分塊邏輯：heading 切割、1500 字元上限、2 行重疊 | 30 min |
| 4 | `src/memsearch/store.py` | Milvus 向量資料庫封裝：schema 定義、混合搜尋（dense + BM25） | 45 min |
| 5 | `src/memsearch/core.py` | MemSearch 主類別：index / search / compact / watch 四大操作 | 30 min |
| 6 | `src/memsearch/config.py` | 分層設定系統（TOML）：全域 → 專案 → CLI flags | 15 min |
| 7 | `ccplugin/hooks/` | Claude Code 生命週期 hooks：SessionStart / Stop 如何自動摘要對話 | 30 min |
| 8 | `src/memsearch/compact.py` | LLM 驅動的記憶壓縮機制 | 20 min |

### 實作練習

```bash
pip install memsearch

# 初始化設定
memsearch config init --project

# 建立幾個測試用的 .md 記憶檔案
# 手動索引
memsearch index ./memory/

# 測試語意搜尋
memsearch search "如何處理 API 錯誤"

# 觀察混合搜尋結果（語意 + 關鍵字）的排序差異
memsearch search "error handling" --json-output
```

### 重點筆記方向

- [ ] 畫出資料流：`.md 檔案 → Scanner → Chunker → Embedding → MilvusStore → 混合搜尋`
- [ ] 記錄 Chunk ID 的生成公式：`sha256(markdown:{source}:{startLine}:{endLine}:{contentHash}:{model})[:16]`
- [ ] 理解三層漸進揭露：`search → expand → transcript`
- [ ] 記錄 Claude Code Plugin 的 4 個 Hook 生命週期與各自職責

### 關鍵架構洞察

```
★ Insight ─────────────────────────────────────
1. 「Markdown 是 Source of Truth」是最重要的設計決策
   → 向量資料庫隨時可重建，人類可直接閱讀和編輯記憶
   → 這比純向量儲存更有韌性，也更容易除錯
2. 混合搜尋（Dense + BM25 + RRF）比純語意搜尋更可靠
   → 語意搜尋擅長「同義詞」，BM25 擅長「精確匹配」
   → RRF 重排序不需要調權重，簡單且穩定
3. SHA-256 複合 ID 天然去重，不需額外查重查詢
   → 相同內容 + 相同位置 + 相同模型 = 相同 ID
─────────────────────────────────────────────────
```

---

## Day 5-6：superpowers — 可組合技能與工作流框架

> **核心問題**：如何把零散的 AI 能力組合成有紀律的開發工作流？

### 閱讀路線

| 順序 | 檔案 / 路徑 | 重點觀察 | 預計時間 |
|------|-------------|---------|---------|
| 1 | `README.md` | 系統哲學、技能清單、安裝方式 | 20 min |
| 2 | `skills/using-superpowers/SKILL.md` | 元技能：如何判斷「該不該使用某個技能」 | 20 min |
| 3 | `skills/brainstorming/SKILL.md` | 工作流起點：如何從模糊需求到具體方案 | 20 min |
| 4 | `skills/writing-plans/SKILL.md` | 計畫撰寫：如何把方案拆成 2-5 分鐘的原子任務 | 25 min |
| 5 | `skills/test-driven-development/SKILL.md` | TDD 核心流程：RED → GREEN → REFACTOR | 25 min |
| 6 | `skills/subagent-driven-development/SKILL.md` | 子代理開發：每個任務派發獨立 subagent | 30 min |
| 7 | `skills/systematic-debugging/SKILL.md` | 系統性偵錯四階段 | 20 min |
| 8 | `skills/verification-before-completion/SKILL.md` | 完成前驗證：禁止不確定詞、要求具體證據 | 15 min |
| 9 | `skills/writing-skills/SKILL.md` | 元元技能：如何寫出好的技能定義 | 25 min |
| 10 | `skills/writing-skills/persuasion-principles.md` | 說服力工程：如何提高 LLM 遵從率 | 20 min |

### 重點筆記方向

- [ ] 畫出完整工作流鏈：`brainstorming → writing-plans → subagent-driven-dev → review → finish`
- [ ] 記錄 SKILL.md 的標準格式：Name / Description / Body 各區塊的規範
- [ ] 理解「Rigid skills vs Flexible skills」的差異與適用場景
- [ ] 記錄說服力設計的 4 種策略：Authority / Commitment / Scarcity / Social Proof
- [ ] 理解 Description 的鐵律：「只描述觸發條件，絕不摘要工作流程」

### 關鍵架構洞察

```
★ Insight ─────────────────────────────────────
1. 技能是目錄，不是單一檔案
   → SKILL.md 是入口，但複雜技能需要附屬參考文件
   → 這讓技能可以按需載入，不一次佔滿 context window
2. 「對技能文件本身做 TDD」是最有價值的 meta-pattern
   → RED: 觀察 agent 沒有技能時如何失敗
   → GREEN: 寫剛好夠用的文件修正那些失敗
   → REFACTOR: 觀察新漏洞，持續補強
3. 說服力設計不是花招，是工程方法
   → 測試 28,000 次對話，遵從率從 33% → 72%
   → 禁用 Liking 和 Reciprocity，保護誠實回饋文化
─────────────────────────────────────────────────
```

---

## Day 7：claude-hooks — 可程式化執行層

> **核心問題**：如何讓 Claude Code 變成可觀測、可攔截、可擴充的自動化平台？

### 閱讀路線

| 順序 | 檔案 / 路徑 | 重點觀察 | 預計時間 |
|------|-------------|---------|---------|
| 1 | `README.md` | 整體概念、安裝方式、基本用法 | 20 min |
| 2 | `templates/hooks/lib.ts` | 型別定義：8 種 Hook 事件的 Payload 與 Response 型別 | 30 min |
| 3 | `templates/hooks/index.ts` | 參考實作：如何用 `runHook()` 統一派發事件 | 20 min |
| 4 | `templates/hooks/session.ts` | Session 追蹤：如何記錄每個 Hook 事件到 JSON 檔 | 15 min |
| 5 | `src/commands/init.ts` | CLI 初始化邏輯：如何生成 `.claude/hooks/` 目錄結構 | 15 min |

### 實作練習

```bash
# 在一個測試專案中初始化 hooks
npx claude-hooks --local

# 修改 .claude/hooks/index.ts，加入一個簡單的 PreToolUse hook：
# - 記錄所有工具呼叫到 console
# - 阻擋含有 "rm -rf" 的 Bash 指令

# 啟動 Claude Code，觀察 hook 是否正確觸發
```

### 重點筆記方向

- [ ] 畫出事件生命週期：`UserPromptSubmit → SessionStart → PreToolUse → 工具執行 → PostToolUse → Stop`
- [ ] 記錄 8 種 Hook 事件各自的 Payload 與可控制欄位
- [ ] 理解 `settings.json` 的 hooks 設定格式
- [ ] 記錄通訊協定：`stdin JSON → handler → stdout JSON`
- [ ] 理解 `stop_hook_active` 防無限迴圈機制

### 關鍵架構洞察

```
★ Insight ─────────────────────────────────────
1. stdin/stdout JSON 協定是極簡但強大的設計
   → 語言無關：任何能讀 stdin 寫 stdout 的語言都可實作 Hook
   → claude-hooks 只是讓 TypeScript 版本更方便，不是唯一選擇
2. 「空物件 = 不干預」的預設行為降低了誤操作風險
   → Handler 只在需要控制時才回傳特定欄位
3. 選擇 Bun 而非 Node.js 是刻意的
   → Hook 是高頻短暫執行，Bun 的啟動速度優勢在這裡最明顯
   → 直接執行 TypeScript 無需編譯步驟
─────────────────────────────────────────────────
```

---

## 跨 Repo 整合思考（Day 7 下午）

完成 4 個 repo 的學習後，花 2 小時做以下整合分析：

### 1. 畫出四層架構圖

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 4: Workflow Orchestration (superpowers)               │
│  ─ 技能定義、工作流組合、subagent 派發、驗證流程              │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Execution Hooks (claude-hooks)                     │
│  ─ 攔截工具呼叫、注入邏輯、權限控制、session 追蹤             │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Memory & Retrieval (memsearch)                     │
│  ─ 跨 session 記憶、語意搜尋、自動摘要、漸進揭露              │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: Knowledge Ingestion (Skill_Seekers)                │
│  ─ 原始資料爬取、智慧分塊、多平台輸出、衝突偵測               │
└─────────────────────────────────────────────────────────────┘
```

### 2. 回答整合問題

- [ ] Skill_Seekers 的輸出格式能否直接餵給 memsearch 索引？差距在哪？
- [ ] memsearch 的記憶能否被 superpowers 的技能自動引用？如何串接？
- [ ] claude-hooks 能否在 PreToolUse 時自動查詢 memsearch 注入上下文？
- [ ] superpowers 的 SKILL.md 格式和 Skill_Seekers 的技能輸出格式有何異同？

### 3. 輸出成果

- [ ] 在 `tasks/lessons.md` 記錄至少 5 條架構洞察
- [ ] 畫一張完整的資料流圖（從原始資料到 Agent 執行）
- [ ] 列出 3 個你想在自己系統中借鏡的設計模式

---

## 學習進度追蹤

| Day | 主題 | 狀態 | 完成日期 |
|-----|------|------|---------|
| 1-2 | Skill_Seekers：知識→技能 | ⬜ 未開始 | |
| 3-4 | memsearch：長期記憶層 | ⬜ 未開始 | |
| 5-6 | superpowers：技能與工作流 | ⬜ 未開始 | |
| 7 | claude-hooks：可程式化執行 | ⬜ 未開始 | |
| 7+ | 跨 Repo 整合思考 | ⬜ 未開始 | |

---

## 附錄：快速參考連結

| Repo | URL | 核心技術 |
|------|-----|---------|
| Skill_Seekers | https://github.com/yusufkaraaslan/Skill_Seekers | Python, Strategy Pattern, AST |
| memsearch | https://github.com/zilliztech/memsearch | Python, Milvus, Hybrid Search |
| superpowers | https://github.com/obra/superpowers | Markdown skills, Claude Code Plugin |
| claude-hooks | https://github.com/johnlindquist/claude-hooks | TypeScript, Bun, stdin/stdout |
