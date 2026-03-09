# md-for-ai

**Your CLAUDE.md is read by AI, not by you. Write it that way.**

Most people write CLAUDE.md like a handbook for a new employee — full sentences, markdown tables, repeated rules. But the only reader is an AI model. It doesn't need prose. It needs structure.

Write your MD in AI's language. Get the same behavior with 87% fewer tokens.

| What you get | Details |
|-------------|---------|
| 🔍 **Preview first** | See exactly how many tokens you're wasting — before changing anything |
| ✅ **Same functionality** | Every rule is preserved. Nothing is deleted, only reformatted |
| ⬆️ **Better AI compliance** | AI parses XML tags + key-value pairs more accurately than prose |
| 💰 **87% fewer tokens** | ~250K tokens saved per 50-turn session |
| ↩️ **One-click restore** | Don't like it? One command reverts everything |

## Why This Matters

```
Turn 1:  AI reads 20KB of your instructions → responds
Turn 2:  AI reads 20KB of your instructions AGAIN → responds
Turn 50: AI has re-read 1MB of instructions just to remember who you are
```

The problem isn't the content — it's the format. Human prose costs 4-8x more tokens than AI-native format, with no benefit to the model.

## Human prose vs AI-native format

**Before** — written for humans:
```markdown
**No Guessing Rule:** Don't guess. Before editing any file, use Read/Grep to get
line numbers. Before saying "the problem is X", paste evidence first.
Only change one thing at a time, verify immediately.
```

**After** — written for AI:
```
EVIDENCE: no-fabricate no-guess unsure=say-so | all-claims-need-proof(data/line#/source) | one-change-then-verify
```

43% fewer bytes. Same AI compliance. The model understands both equally — you're just paying less.

## Why AI-native format is better (not just cheaper)

| Human prose | AI-native format | Why AI-native wins |
|-------------|-----------------|-------------------|
| `## Rules` markdown header | `<rules>` XML tag | AI parses XML boundaries with near-zero ambiguity |
| "Please remember to always back up before editing" | `backup→edit` | Zero filler words = less noise for attention mechanism |
| Same rule written 4 different ways across 3 files | Stated once, compressed | Deduplication = no conflicting interpretations |
| 20 specific rules | 5 compressed principles | Smart models derive specifics — over-specifying actually *constrains* them |
| Rule described in text + enforced by hook | Hook only | Text is 100% redundant when hooks auto-enforce |

The AI doesn't just tolerate compressed format — it **parses it more reliably**. XML tags create unambiguous boundaries. Key-value pairs eliminate filler words that dilute attention. And 5 principles give the model room to reason, instead of 20 rigid rules that conflict at the edges.

**You're not just saving money. You're giving the AI a cleaner signal.**

## Real-world example

See [`examples/before.md`](examples/before.md) (human prose, 4,665 bytes) vs [`examples/after.md`](examples/after.md) (AI-native, 2,547 bytes). Same rules, same behavior — 45% smaller, cleaner signal.

## Format Cheatsheet

| Human format | AI-native format | Savings |
|-------------|-----------------|---------|
| Full sentences | `key: value1 value2` | ~70% |
| `## Section Header` | `<section>` XML tag | Better parsing |
| "leads to", "or", "at most" | `→` `\|` `≤` | Single-token |
| 20 specific rules | 5 compressed principles | Smart models derive specifics |
| Rule text + hook enforcement | Hook only | 100% (text is redundant) |

## Two-Stage Process

### Stage 1: Preview (read-only, nothing changes)

The skill measures your current token burn and shows projected savings:

```
=== Current Token Burn ===
CLAUDE.md:     13,474 bytes
rules/*.md:    10,004 bytes
Total:         23,478 bytes ≈ 5,869 tokens/turn

50-turn session: 293,450 tokens on system instructions alone

=== After Distillation ===
Projected:     2,944 bytes ≈ 736 tokens/turn
Savings:       87%
50-turn session: saves 256,650 tokens
```

You decide whether to proceed. Nothing is modified.

### Stage 2: Distill (with full backup)

If you approve, the skill:

1. **Backs up** all original files (one command to restore)
2. **Rewrites** CLAUDE.md in AI-native format (XML tags + telegraphic notation)
3. **Moves** detailed content to `~/.claude/ref/` (loaded on-demand, not every turn)
4. **Empties** rules/ directory (content merged or moved)
5. **Verifies** all rules are preserved

## Architecture

```
Tier 1: Always loaded (target: <1000 tokens)
  └── CLAUDE.md — XML tags, compressed
       <user>    identity, tone
       <rules>   5-7 principles
       <rhythm>  workflow
       <conn>    credentials (exact)
       <ref>     pointers to Tier 2
       <learn>   self-improvement

Tier 2: On-demand (zero cost until needed)
  └── ~/.claude/ref/
       ├── projects.md
       ├── debug.md
       ├── lessons.md
       └── profile.md
```

## Results

| Metric | Before | After |
|--------|--------|-------|
| Auto-loaded size | 15-30 KB | 2-3 KB |
| Tokens per turn | 4,000-8,000 | 500-800 |
| 50-turn session waste | 200-400K tokens | 25-40K tokens |
| Behavioral compliance | baseline | same or better |
| Effective session capacity | 1x | **~2x** |

## Install

```bash
mkdir -p ~/.claude/skills/md-for-ai
curl -o ~/.claude/skills/md-for-ai/SKILL.md \
  https://raw.githubusercontent.com/sstklen/md-for-ai/main/SKILL.md
```

## Usage

In Claude Code, say any of:
- `distill my CLAUDE.md`
- `rewrite my MD for AI`
- `my tokens are burning too fast`
- `蒸餾`

The skill runs Stage 1 (preview) first. You approve before anything changes.

## License

MIT
