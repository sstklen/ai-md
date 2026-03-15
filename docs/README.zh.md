[English](../README.md) · [日本語](README.ja.md)

# AI.MD

**你的 CLAUDE.md 每一輪都被 AI 讀，不是被你讀 — 所以要用 AI 的語言來寫。**

AI.MD 把人類寫的 `CLAUDE.md` 轉換成 AI 原生的結構化標籤格式。
不是「壓縮」— 而是**重新結構化**，讓 LLM 真的更能遵守你的規則。

## 問題所在

```
你打「hi」→ AI 讀你的 CLAUDE.md → 回應
50 輪後 → AI 已經讀了 50 次
```

Claude Code 在**每一輪對話**都會重讀 `CLAUDE.md`。大多數 CLAUDE.md 是用人類的文字寫的 — 段落、括號說明、把多條規則塞在同一句的長句子。

這既浪費 token，又降低遵守率。我們有數據證明。

## 實戰驗證結果

2026-03 用真實的 CLAUDE.md（washinmura.jp）測試，5 輪，4 個模型：

| 格式 | Codex (GPT-5.3) | Gemini 2.5 Pro | Claude Opus 4.6 |
|--------|-----------------|----------------|-----------------|
| 人類文字（多條規則）| 6/8 | 6.5/8 | 8/8 |
| AI 原生結構化 | **8/8** | **7/8** | **8/8** |

**相同規則。不同格式。所有模型的遵守率都提升了。**

令人不舒服的發現：用文字增加更多規則會**降低**遵守率（8/8 → 6/8）。
轉換成結構化格式則**恢復並超越**了原本（6/8 → 8/8）。

### 體積縮減

同一份真實 CLAUDE.md，轉換前後：

| 指標 | 轉換前（文字）| 轉換後（AI 原生）| 變化 |
|--------|---------------|-------------------|--------|
| 檔案大小 | 13,474 bytes | 6,332 bytes | **-53%** |
| 行數 | 224 行 | 142 行 | **-37%** |

Claude Code 官方建議 CLAUDE.md 保持在 **200 行以下**（每輪重讀）。
原始版本**超過限制**（224 行）。AI.MD 轉換後：**遠在限制內**（142 行）。

每輪燒更少 token × 每輪 × 每個 session = 複利節省。

## 為什麼有效

三個機制解釋了結構化標籤格式為何勝過文字：

### 1. 每行一個概念 = 專注注意力

LLM 不是「閱讀」— 它們是在**注意**。當規則共用一行，注意力就分散了。
每條規則有自己的行，每條都獲得完整的注意力權重。

```
# 不好：5 條規則，1 行 — AI 大約遵守 3 條
EVIDENCE: no-fabricate no-guess | banned:應該是 | Read/Grep→行號 | "好像"→self-test | guess=shame-wall

# 好：5 條規則，5 行 — AI 全部遵守
EVIDENCE:
  core: no-fabricate | no-guess | unsure=say-so
  banned: 應該是/可能是 → 先拿數據
  proof: all-claims-need(data/line#/source)
  hear-doubt: "好像"/"覺得" → self-test → 禁反問user
  violation: guess → shame-wall
```

### 2. 明確標籤 = 零推斷

標籤直接聲明意義。文字需要模型自己推斷。

```
# 不好：AI 必須推斷「防搞混」是什麼意思
GATE-1: 收到任務→先用一句話複述(防搞混)

# 好：標籤精確告訴 AI 每個部分的作用
GATE-1 複述:
  trigger: new-task
  action: first-sentence="你要我做的是___"
  exception: signal=處理一下 → skip
```

### 3. 語意錨定 = 直接匹配

標籤作為可匹配的標記。當使用者說「新增一個 API」，模型直接匹配到 `new-api:` — 像雜湊查找而不是全文搜尋。

```
MOAT:
  new-api: must check health-check.py coverage (GATE-5)
```

這個具體技術修復了一個在所有模型連續失敗 5 次的測試案例。

## 安裝

```bash
mkdir -p ~/.claude/skills/ai-md
curl -o ~/.claude/skills/ai-md/SKILL.md \
  https://raw.githubusercontent.com/sstklen/ai-md/main/SKILL.md
```

## 使用方式

在 Claude Code 中，說任何一句：
- `AI.MD` 或 `distill my CLAUDE.md`
- `rewrite my MD for AI`
- `蒸餾`

Skill 分兩個階段執行：
1. **預覽** — 測量你目前的 token 消耗，顯示前後對比範例
2. **蒸餾** — 備份後轉換，用多個模型測試，回報分數

## 轉換流程

完整方法論在 [`SKILL.md`](../SKILL.md) 裡，以下是摘要：

| 階段 | 做什麼 |
|-------|-------------|
| **1. 理解** | 像編譯器一樣讀 — 識別觸發條件、動作、限制、元資料，以及可刪除的人類說明 |
| **2. 拆解** | 把每個 `\|` 分隔符、`()` 括號說明、「and/but」連接詞拆解成原子規則 |
| **3. 標記** | 賦予功能標籤：`trigger:` `action:` `exception:` `banned:` `policy:` 等 |
| **4. 結構化** | 組織成層次：`<gates>` → `<rules>` → `<rhythm>` → `<conn>` → `<ref>` |
| **5. 解決衝突** | 偵測並解決規則之間的隱藏衝突（priority、yields-to、not-triggered）|
| **6. 測試** | 用 2+ 個 LLM 模型、8 個測試問題、相同通過/不通過標準驗證 |

## 特殊技巧

| 技巧 | 功能 |
|-----------|-------------|
| **雙語標籤** | 英文標籤（更短、通用）+ 本地語言輸出字串 |
| **狀態機關卡** | 每個關卡有 trigger → action → exception → priority，清楚的執行模型 |
| **XML 區塊標籤** | `<gates>` `<rules>` `<rhythm>` 建立硬性邊界，防止規則滲漏 |
| **交叉引用** | 單一事實來源 + `(GATE-5)` 引用，而非重複 |
| **「是什麼不是為什麼」** | 刪除所有解釋規則為何存在的說明。AI 需要「是什麼」，不需要「為什麼」。 |
| **不觸發清單** | 明確列出規則**不應該**觸發的情況範例，防止過度觸發。 |
| **衝突偵測** | 逐對檢查所有關卡的隱藏衝突，加上 priority/yields-to。 |

## 模板

```xml
# 專案名稱 | lang:xx | for-AI-parsing

<user>
identity, tone, signals, decision-style
</user>

<gates label="priority: gates>rules>rhythm">
GATE-1 name:
  trigger: ...
  action: ...
  exception: ...
</gates>

<rules>
RULE-NAME:
  core: ...
  banned: ...
  violation: ...
</rules>

<rhythm>
workflow patterns as key: value
</rhythm>

<conn>
connection strings (never compress facts)
</conn>
```

## 範例

見 [`examples/before.md`](../examples/before.md) 和 [`examples/after.md`](../examples/after.md) 的真實轉換案例。

## 核心洞察

> **文字規則越多 = 遵守率越差。**
> **相同規則換成結構 = 遵守率越好。**
>
> 你精心撰寫的 CLAUDE.md 可能正在傷害你的 AI 表現。
> 結構 > 文字。永遠如此。

## 授權條款

MIT
