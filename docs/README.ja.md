[English](../README.md) · [中文版](README.zh.md)

# AI.MD

**あなたのCLAUDE.mdは毎ターンAIに読まれています、あなたではなく — だからAIの言語で書きましょう。**

AI.MDは人間が書いた`CLAUDE.md`をAIネイティブな構造化ラベル形式に変換します。
「圧縮」ではありません — LLMがルールをより確実に従えるよう**再構造化**します。

## 問題

```
「hi」と入力 → AIがCLAUDE.mdを読む → 応答
50ターン後 → AIは50回読み返している
```

Claude Codeは**会話の毎ターン**に`CLAUDE.md`を読み直します。ほとんどのCLAUDE.mdは人間の文章で書かれています — 段落、括弧による説明、複数のルールが詰め込まれた長い文章。

これはトークンを無駄にするだけでなく、遵守率も下げます。データで証明しました。

## 実戦検証結果

2026年3月に実際のCLAUDE.md（washinmura.jp）で5ラウンド、4モデルでテスト：

| フォーマット | Codex (GPT-5.3) | Gemini 2.5 Pro | Claude Opus 4.6 |
|--------|-----------------|----------------|-----------------|
| 人間の文章（多数のルール）| 6/8 | 6.5/8 | 8/8 |
| AIネイティブ構造化 | **8/8** | **7/8** | **8/8** |

**同じルール。異なるフォーマット。全モデルで遵守率が向上。**

不快な発見：文章でルールを追加すると遵守率が**低下**しました（8/8 → 6/8）。
構造化フォーマットに変換すると遵守率が**回復し、超えました**（6/8 → 8/8）。

### サイズ削減

同じ実際のCLAUDE.md、変換前後：

| 指標 | 変換前（文章）| 変換後（AIネイティブ）| 変化 |
|--------|---------------|-------------------|--------|
| ファイルサイズ | 13,474 bytes | 6,332 bytes | **-53%** |
| 行数 | 224行 | 142行 | **-37%** |

Claude Codeは公式にCLAUDE.mdを**200行以内**に保つことを推奨しています（毎ターン読み直し）。
元のファイルは**制限を超えていました**（224行）。AI.MD変換後：**余裕で制限内**（142行）。

1ターンあたりのトークン消費削減 × 毎ターン × 毎セッション = 複利的な節約。

## なぜ機能するのか

構造化ラベル形式が文章より優れている理由を3つのメカニズムで説明します：

### 1. 1行1概念 = 集中した注意

LLMは「読む」のではなく**注意を向けます**。ルールが1行を共有すると、注意が分散します。
各ルールに独自の行があると、それぞれが完全な注意の重みを得ます。

```
# 悪い例：5つのルール、1行 — AIは約3つだけ従う
EVIDENCE: no-fabricate no-guess | banned:應該是 | Read/Grep→行號 | "好像"→self-test | guess=shame-wall

# 良い例：5つのルール、5行 — AIは全て従う
EVIDENCE:
  core: no-fabricate | no-guess | unsure=say-so
  banned: 應該是/可能是 → 先拿數據
  proof: all-claims-need(data/line#/source)
  hear-doubt: "好像"/"覺得" → self-test → 禁反問user
  violation: guess → shame-wall
```

### 2. 明示的なラベル = 推論ゼロ

ラベルは意味を宣言します。文章はモデルに推論を強います。

```
# 悪い例：AIは（防搞混）が何を意味するか推論しなければならない
GATE-1: 收到任務→先用一句話複述(防搞混)

# 良い例：ラベルがAIに各部分の機能を正確に伝える
GATE-1 複述:
  trigger: new-task
  action: first-sentence="你要我做的是___"
  exception: signal=處理一下 → skip
```

### 3. セマンティックアンカリング = 直接マッチング

ラベルはマッチング可能なタグとして機能します。ユーザーが「APIを追加して」と言うと、モデルは直接`new-api:`にマッチします — フルテキスト検索ではなくハッシュルックアップのように。

```
MOAT:
  new-api: must check health-check.py coverage (GATE-5)
```

この特定の技術が、全モデルで5回連続失敗したテストケースを修正しました。

## インストール

```bash
mkdir -p ~/.claude/skills/ai-md
curl -o ~/.claude/skills/ai-md/SKILL.md \
  https://raw.githubusercontent.com/sstklen/ai-md/main/SKILL.md
```

## 使い方

Claude Codeで以下のいずれかを言います：
- `AI.MD` または `distill my CLAUDE.md`
- `rewrite my MD for AI`
- `蒸餾`

スキルは2段階で実行されます：
1. **プレビュー** — 現在のトークン消費量を測定し、変換前後の例を表示
2. **蒸留** — バックアップ後に変換、複数のモデルでテスト、スコアを報告

## 変換プロセス

完全な方法論は[`SKILL.md`](../SKILL.md)にありますが、概要はこちら：

| フェーズ | 内容 |
|-------|-------------|
| **1. 理解** | コンパイラのように読む — トリガー、アクション、制約、メタデータ、削除可能な人間向け説明を識別 |
| **2. 分解** | すべての`\|`セパレーター、`()`括弧説明、「and/but」接続詞を原子ルールに分解 |
| **3. ラベル付け** | 機能ラベルを割り当て：`trigger:` `action:` `exception:` `banned:` `policy:` など |
| **4. 構造化** | 階層に整理：`<gates>` → `<rules>` → `<rhythm>` → `<conn>` → `<ref>` |
| **5. 解決** | ルール間の隠れた競合を検出・解決（priority、yields-to、not-triggered）|
| **6. テスト** | 2以上のLLMモデル、8つのテスト質問、同じ合格/不合格基準で検証 |

## 特別なテクニック

| テクニック | 機能 |
|-----------|-------------|
| **バイリンガルラベル** | 英語ラベル（短く、汎用的）+ ネイティブ言語の出力文字列 |
| **ステートマシンゲート** | 各ゲートにtrigger → action → exception → priority。明確な実行モデル。 |
| **XMLセクションタグ** | `<gates>` `<rules>` `<rhythm>`がハードバウンダリを作り、ルールの滲み出しを防ぐ |
| **相互参照** | 単一の情報源 + 重複の代わりに`(GATE-5)`参照 |
| **「なぜではなく何を」** | ルールが存在する理由の説明を全て削除。AIが必要なのは「何を」、「なぜ」ではない。 |
| **非トリガーリスト** | ルールが発火すべきでない場合の明示的な例。過剰トリガーを防ぐ。 |
| **競合検出** | 全ゲートのペアを隠れた競合についてチェック。priority/yields-toを追加。 |

## テンプレート

```xml
# プロジェクト名 | lang:xx | for-AI-parsing

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

## サンプル

実際の変換例は[`examples/before.md`](../examples/before.md)と[`examples/after.md`](../examples/after.md)をご覧ください。

## 核心的な洞察

> **文章でルールが多い = 遵守率が悪い。**
> **同じルールを構造化すると = 遵守率が良くなる。**
>
> 丁寧に書いたCLAUDE.mdが、AIのパフォーマンスを損なっているかもしれません。
> 構造 > 文章。常に。

## ライセンス

MIT
