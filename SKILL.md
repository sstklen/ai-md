---
name: md-for-ai
version: 3.0.0
description: >
  Convert your CLAUDE.md from Markdown to AI-native XML format.
  No content is deleted — just reformatted. Same rules, smaller file, cleaner signal.
  Run with: "convert my CLAUDE.md to XML" or "rewrite my MD for AI" or "蒸餾"
triggers:
  - user says "CLAUDE.md 太長" "太肥" "額度掉很快" "token 太多"
  - user says "蒸餾" "distill" "轉XML" "rewrite MD for AI" "convert to XML" "md-for-ai"
  - user notices quota draining fast
  - CLAUDE.md + rules total > 5KB
tags: [format-conversion, token-saving, system-prompt, ai-native, xml]
---

# md-for-ai — Convert Your MD to AI-Native XML

## What This Does

Converts CLAUDE.md from Markdown format to XML format. **Format conversion only — no content is deleted.**

| ✅ Does | ❌ Does NOT |
|---------|------------|
| `## Header` → `<section>` XML tag | Delete any rules |
| Full sentences → key-value pairs | Merge or "compress" meaning |
| "leads to", "or" → `→` `\|` symbols | Decide what's important for you |
| Markdown tables → inline notation | Move files without asking |

## Why Convert?

Claude Code re-reads `~/.claude/CLAUDE.md` + `~/.claude/rules/*.md` on **every turn**.
XML format is smaller than Markdown for the same content → fewer tokens per turn.
Anthropic recommends XML tags for structuring prompts — Claude parses them with near-zero ambiguity.

---

## Two-Stage Workflow

### Stage 1: PREVIEW — Measure, don't touch

No files are modified. Read-only.

#### Step 1.1: Measure current size

```bash
echo "=== Current File Sizes ==="
claude_md=$(wc -c < ~/.claude/CLAUDE.md 2>/dev/null || echo 0)
rules=$(cat ~/.claude/rules/*.md 2>/dev/null | wc -c || echo 0)
total=$((claude_md + rules))
echo "CLAUDE.md:     $claude_md bytes"
echo "rules/*.md:    $rules bytes"
echo "Total:         $total bytes"
echo ""
echo "=== Projected After Format Conversion ==="
projected=$((total * 45 / 100))
echo "Projected:     ~$projected bytes (format change only)"
echo "Savings:       ~55%"
```

#### Step 1.2: Show format comparison

Read the user's CLAUDE.md. For each section, show a before/after of the format change:

```
Section: "## Communication Style"
Before (Markdown, 342 bytes):
  - **Talk in plain language.** No jargon. Use analogies (one is enough).
  - **Keep options to 3 or fewer.**
  ...

After (XML, 156 bytes):
  <user>
  tone: plain-language no-jargon one-analogy-enough | options≤3
  ...

Savings: 54% (format only, same content)
```

Do this for 2-3 sections so the user can see the pattern.

#### Step 1.3: Ask permission

**Stop. Ask the user: "Want to proceed with format conversion?"**

Do NOT modify any files until the user confirms.

---

### Stage 2: CONVERT — Reformat with backup

**Only run after user confirms.**

#### Step 2.1: Backup

```bash
cp ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.bak-pre-convert
[ -d ~/.claude/rules ] && cp -r ~/.claude/rules ~/.claude/rules.bak-pre-convert
```

Tell the user: "Backed up. Restore anytime with one command."

#### Step 2.2: Convert format

**Conversion rules (format only, no content changes):**

| Markdown | XML | Rule |
|----------|-----|------|
| `## Section Name` | `<section-name>` ... `</section-name>` | Headers become XML tags |
| `- **Bold label:** explanation text` | `label: compressed-text` | List items become key-value |
| "leads to", "results in" | `→` | Connecting words become symbols |
| "or", "alternatively" | `\|` | |
| "at most", "maximum" | `≤` | |
| "equals", "is the same as" | `=` | |
| Markdown table | Inline `key: val1 val2` | Tables become flat notation |

**Critical: Never change facts, credentials, URLs, or numbers. Only change format.**

#### Step 2.3: Side-by-side verification

After conversion, show the user a side-by-side comparison:

```
Original rule: "Don't guess. Before editing any file, use Read/Grep to get line numbers."
Converted:     "no-guess | Read/Grep→行號"

✅ Same meaning    ❌ Meaning changed (flag this)
```

Walk through EVERY rule. The user confirms each one.

#### Step 2.4: Show results

```
=== Conversion Complete ===

Before: {old_total} bytes
After:  {new_total} bytes
Saved:  {percent}% (format conversion only)

All rules preserved: {count} rules verified
Backup: ~/.claude/CLAUDE.md.bak-pre-convert
Restore: cp ~/.claude/CLAUDE.md.bak-pre-convert ~/.claude/CLAUDE.md
```

---

## Conversion Examples

Before (Markdown):
```markdown
## Rules

### Rule 1: No Guessing
Don't guess. Don't fabricate. If you're unsure, say so.
Every claim needs proof — show me the data, line numbers, or source.
Only change one thing at a time, verify immediately.
```

After (XML):
```xml
<rules>
1. EVIDENCE: no-fabricate no-guess unsure=say-so | all-claims-need-proof(data/line#/source) | one-change-then-verify
</rules>
```

Same rules. Different format. 43% fewer bytes.

---

## XML Tag Reference

| Tag | Contains |
|-----|----------|
| `<user>` | Identity, communication preferences, decision style |
| `<rules>` | Behavioral rules as numbered key-value principles |
| `<rhythm>` | Workflow patterns, triggers, reminders |
| `<conn>` | Connection strings, credentials, tools (never compress facts) |
| `<ref>` | Pointers to other files (read on-demand) |
| `<learn>` | How the system improves over time |

---

## Important Limits

- **This is format conversion, not content optimization.** Don't delete, merge, or "improve" rules.
- **Credentials and URLs must stay exact.** Never shorten a password or connection string.
- **The user reviews every converted rule.** Don't skip verification.
- **If unsure whether a conversion changes meaning, keep the original wording.**
