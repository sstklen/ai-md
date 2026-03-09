# 🧪 Prompt Distillery

**Your CLAUDE.md is burning your tokens. This skill fixes it.**

| What you get | Details |
|-------------|---------|
| 🔍 **Preview first** | See exactly how many tokens you're wasting — before changing anything |
| ✅ **Same functionality** | Every rule is preserved. Nothing is deleted, only compressed |
| ⬆️ **Better AI compliance** | AI parses structured format more accurately than prose |
| 💰 **87% fewer tokens** | ~250K tokens saved per 50-turn session |
| ↩️ **One-click restore** | Don't like it? One command reverts everything |

Every conversation turn, Claude Code re-reads your entire `~/.claude/CLAUDE.md` and `~/.claude/rules/*.md`. If you wrote them in human-readable prose (like most people do), you're paying thousands of tokens per turn for formatting the AI doesn't need.

## The Problem

```
Turn 1:  AI reads 20KB of instructions → responds
Turn 2:  AI reads 20KB of instructions AGAIN → responds
Turn 50: AI has read 1MB of instructions just to remember who you are
```

Most CLAUDE.md files are written like employee handbooks — full sentences, markdown tables, repeated rules. The AI doesn't need any of that.

## The Solution

Prompt Distillery compresses human prose into AI-native format:

**Before** (human prose):
```markdown
**No Guessing Rule:** Don't guess. Before editing any file, use Read/Grep to get
line numbers. Before saying "the problem is X", paste evidence first.
Only change one thing at a time, verify immediately.
```

**After** (AI-native):
```
EVIDENCE: no-fabricate no-guess unsure=say-so | all-claims-need-proof(data/line#/source) | one-change-then-verify
```

43% fewer bytes. Same AI compliance.

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
2. **Compresses** CLAUDE.md into XML tags + telegraphic notation
3. **Moves** detailed content to `~/.claude/ref/` (loaded on-demand, not every turn)
4. **Empties** rules/ directory (content merged or moved)
5. **Verifies** all rules are preserved

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
# Copy the skill to your Claude Code skills directory
mkdir -p ~/.claude/skills/prompt-distillery
curl -o ~/.claude/skills/prompt-distillery/SKILL.md \
  https://raw.githubusercontent.com/washinmura/prompt-distillery/main/SKILL.md
```

## Usage

In Claude Code, say any of:
- `distill my CLAUDE.md`
- `my tokens are burning too fast`
- `蒸餾`
- `compress my config`

The skill will run Stage 1 (preview) first. You approve before anything changes.

## Key Insight

> **Write for the model, not for the human.**
>
> CLAUDE.md is read by AI, not by you. Optimize for AI parsing.
> Results matter, not readability.

## Format Cheatsheet

| Human format | AI-native format | Savings |
|-------------|-----------------|---------|
| Full sentences | `key: value1 value2` | ~70% |
| `## Section Header` | `<section>` XML tag | Better parsing |
| "leads to", "or", "at most" | `→` `\|` `≤` | Single-token |
| 20 specific rules | 5 compressed principles | Smart models derive specifics |
| Rule text + hook enforcement | Hook only | 100% (text is redundant) |

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

## License

MIT
