# md-for-ai

**Your CLAUDE.md is read by AI, not by you. Write it that way.**

This skill converts your CLAUDE.md from Markdown format to AI-native XML format. No content is deleted or "optimized" — just reformatted.

| What you get | Details |
|-------------|---------|
| 🔄 **Format conversion only** | Same content, different format. Nothing is deleted or merged |
| 🔍 **Preview first** | See the size difference before changing anything |
| ⬆️ **Cleaner AI signal** | XML tags + key-value pairs = less noise for the model |
| 💰 **Smaller files** | Format alone saves 40-60% in bytes |
| ↩️ **One-click restore** | Don't like it? One command reverts everything |

## What it does

Converts this:
```markdown
## Rules

### Rule 1: No Guessing
Don't guess. Don't fabricate. If you're unsure, say so.
Every claim needs proof — show me the data, line numbers, or source.
Only change one thing at a time, verify immediately.
```

Into this:
```xml
<rules>
EVIDENCE: no-fabricate no-guess unsure=say-so | all-claims-need-proof(data/line#/source) | one-change-then-verify
</rules>
```

**Same rules. Different format. AI reads both equally — but the second one is smaller.**

## What it does NOT do

| ❌ Does NOT | Why |
|-------------|-----|
| Delete rules | Dangerous — you might lose something important |
| Merge files | Your rules/ files stay intact unless you choose to move them |
| "Optimize" content | Content decisions are yours, not the tool's |
| Compress meaning | Every rule in the original appears in the output |

## Why XML?

| Human format | AI-native format | Why |
|-------------|-----------------|-----|
| `## Section Header` | `<section>` XML tag | Unambiguous boundaries for AI parsing |
| "Please remember to always..." | `backup→edit` | Zero filler words = less noise |
| Full sentences | `key: value1 value2` | Same meaning, fewer bytes |
| "leads to", "or", "at most" | `→` `\|` `≤` | Single-token symbols |

Anthropic's [official docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags) recommend XML tags for structuring prompts. Claude was trained on XML-heavy data.

## Two-Stage Process

### Stage 1: Preview (read-only)

Measures your current file sizes and shows projected savings:

```
=== Current ===
CLAUDE.md:     13,474 bytes
rules/*.md:    10,004 bytes
Total:         23,478 bytes

=== After format conversion ===
Projected:     ~9,000 bytes (format change only, no content removal)
Savings:       ~60%
```

You decide whether to proceed.

### Stage 2: Convert (with backup)

1. **Backs up** all original files
2. **Converts** Markdown headers → XML tags
3. **Converts** prose sentences → key-value pairs
4. **Converts** connecting words → symbols (→ | ≤)
5. **You review** the result and confirm nothing was lost

## Real-world example

See [`examples/before.md`](examples/before.md) (Markdown, 4,665 bytes) vs [`examples/after.md`](examples/after.md) (XML, 2,547 bytes).

Same rules, same behavior — 45% smaller from format change alone.

## Install

```bash
mkdir -p ~/.claude/skills/md-for-ai
curl -o ~/.claude/skills/md-for-ai/SKILL.md \
  https://raw.githubusercontent.com/sstklen/md-for-ai/main/SKILL.md
```

## Usage

In Claude Code, say:
- `convert my CLAUDE.md to XML`
- `rewrite my MD for AI`
- `蒸餾`

Stage 1 (preview) runs first. You approve before anything changes.

## License

MIT
