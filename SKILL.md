---
name: md-for-ai
version: 2.0.0
description: >
  Your CLAUDE.md is read by AI, not by you — so write it in AI's language.
  Most people write CLAUDE.md as human prose. The AI doesn't need prose, it needs structure.
  This skill rewrites your MD files into AI-native format (XML tags + telegraphic notation).
  Same behavior, 87% fewer tokens. One session saves 250K+ tokens.
  Run with: "distill my CLAUDE.md" or "rewrite my MD for AI" or "蒸餾"
triggers:
  - user says "CLAUDE.md 太長" "太肥" "額度掉很快" "token 太多"
  - user says "蒸餾" "distill" "壓縮 MD" "rewrite MD for AI" "md-for-ai"
  - user notices quota draining fast
  - CLAUDE.md + rules total > 10KB
tags: [optimization, token-saving, system-prompt, ai-native, meta]
---

# md-for-ai — Rewrite Your MD in AI's Language

## The Problem

Claude Code re-reads your `~/.claude/CLAUDE.md` + `~/.claude/rules/*.md` on **EVERY conversation turn**.

```
You type "hi"      → AI reads 20KB of instructions → responds
You type "ok"      → AI reads 20KB of instructions AGAIN → responds
...
50 turns later     → AI has read 1MB of instructions total
```

Most users write these files in human-readable prose.
The AI doesn't need prose. It needs structure.
You're paying tokens for readability that the AI doesn't benefit from.

## What Gets Audited

| File | Loaded when | Priority |
|------|------------|----------|
| `~/.claude/CLAUDE.md` | Every turn | 🔴 Highest |
| `~/.claude/rules/*.md` | Every turn | 🔴 High |
| Project-level `CLAUDE.md` | Every turn (in project) | 🟡 Medium |
| Skill trigger descriptions | Every turn | 🟢 Low |

NOT audited (zero token cost): hooks/, ref/ files, settings.json

---

## Two-Stage Workflow

### Stage 1: PREVIEW — See the damage, don't touch anything

**Goal: Show the user exactly how much they're burning, and what they'd save.**

No files are modified in this stage. Read-only.

#### Step 1.1: Measure current burn

```bash
echo "=== Current Token Burn Per Turn ==="
claude_md=$(wc -c < ~/.claude/CLAUDE.md 2>/dev/null || echo 0)
rules=$(cat ~/.claude/rules/*.md 2>/dev/null | wc -c || echo 0)
total=$((claude_md + rules))
tokens=$((total / 4))
echo ""
echo "CLAUDE.md:     $claude_md bytes"
echo "rules/*.md:    $rules bytes"
echo "Total:         $total bytes"
echo "≈ Tokens/turn: $tokens"
echo ""
echo "=== Projected Session Cost ==="
echo "30-turn session:  $((tokens * 30)) tokens on system instructions"
echo "50-turn session:  $((tokens * 50)) tokens on system instructions"
echo "100-turn session: $((tokens * 100)) tokens on system instructions"
```

#### Step 1.2: Find redundancy

Read all auto-loaded files. Identify:

1. **Duplicate concepts** — same rule stated in CLAUDE.md AND rules/ file
2. **Restated principles** — same idea with different wording (count occurrences)
3. **Hook-enforced rules** — rules that hooks already auto-enforce (100% waste as text)
4. **Prose overhead** — full sentences where key-value pairs would suffice
5. **Decorative formatting** — markdown tables, headers, dividers that add bytes

Present findings as a table:

```
| Concept | Times repeated | Files | Waste |
|---------|---------------|-------|-------|
| "Don't guess" | 4x | CLAUDE.md ×2, rules/debug.md, rules/socratic.md | 3x redundant |
| "Backup before edit" | 3x | CLAUDE.md ×3 | 2x redundant |
| ... | ... | ... | ... |
```

#### Step 1.3: Show projected savings

```
=== Distillation Preview ===

Current:    {total} bytes ≈ {tokens} tokens/turn
Projected:  {compressed} bytes ≈ {new_tokens} tokens/turn
Reduction:  {percent}%
Per 50-turn session: saves {saved} tokens

Functionality: UNCHANGED — all rules preserved, compressed format
```

**Stop here. Ask the user: "Want to proceed with distillation?"**

Do NOT modify any files until the user confirms.

---

### Stage 2: DISTILL — Compress with safety net

**Only run after user confirms Stage 1 results.**

#### Step 2.1: Backup everything

```bash
cp ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.bak-pre-distill
cp -r ~/.claude/rules ~/.claude/rules.bak-pre-distill
```

Tell the user: "Backed up. If you don't like the result, one command restores everything."

#### Step 2.2: Compress to AI-native format

**Format principles:**

| Use this | Not this | Savings |
|----------|----------|---------|
| `<rules>` XML tags | `## Rules` markdown headers | Clearer parsing |
| `key: value1 value2` | "Please remember to value1 and value2" | ~70% |
| `→` `|` `≤` `=` symbols | "leads to" "or" "at most" "equals" | Single-token |
| 5 compressed principles | 20 specific rules | Smart models derive specifics |
| State once | State three ways | ÷3 |

**Template structure:**

```xml
# TITLE | lang:xx | this-file=for-AI-parsing | optimize=results-over-format

<user>
identity, communication preferences, decision style
</user>

<rules>
1. PRINCIPLE-NAME: compressed-rule | compressed-rule | compressed-rule
2. PRINCIPLE-NAME: compressed-rule | compressed-rule
...max 5-7 principles
</rules>

<rhythm>
workflow patterns, what-triggers-what
</rhythm>

<conn>
connection strings, credentials, tool shortcuts (keep exact, never compress facts)
</conn>

<ref label="on-demand Read only">
file-path → when-to-read
</ref>

<learn>
how the system gets smarter over time
</learn>
```

#### Step 2.3: Create Tier 2 (on-demand ref/ files)

```bash
mkdir -p ~/.claude/ref
```

Move detailed content from CLAUDE.md and rules/ into ref/ files:
- Detailed procedures → `ref/` (read only when that procedure is needed)
- Past mistakes/lessons → `ref/lessons.md`
- User preferences → `ref/profile.md` (grows over time)
- Philosophy/principles → `ref/philosophy.md`

These files are NOT auto-loaded. The AI reads them only when relevant.

#### Step 2.4: Empty the rules/ directory

Move rules/*.md → either compressed into Tier 1 CLAUDE.md, or into ref/.

```bash
mkdir -p ~/.claude/rules-archived
mv ~/.claude/rules/*.md ~/.claude/rules-archived/
```

#### Step 2.5: Verify

- [ ] Every behavioral rule from original → exists in distilled version
- [ ] Credentials/connection strings → exact, not compressed
- [ ] Hooks → still reference correct paths
- [ ] ref/ files → contain the detailed content that was moved
- [ ] Originals → backed up

#### Step 2.6: Show results

```
=== Distillation Complete ===

Before: {old_total} bytes ≈ {old_tokens} tokens/turn
After:  {new_total} bytes ≈ {new_tokens} tokens/turn
Saved:  {percent}% reduction
Per 50-turn session: {saved_total} fewer tokens

Backup: ~/.claude/CLAUDE.md.bak-pre-distill
Restore: cp ~/.claude/CLAUDE.md.bak-pre-distill ~/.claude/CLAUDE.md && cp ~/.claude/rules.bak-pre-distill/* ~/.claude/rules/
```

---

## Compression Examples

Before (human prose, 156 bytes):
```
**No Guessing Rule:** Don't guess. Before editing any file, use Read/Grep to get
line numbers. Before saying "the problem is X", paste evidence first.
Only change one thing at a time, verify immediately.
```

After (AI-native, 91 bytes):
```
EVIDENCE: no-fabricate no-guess unsure=say-so | all-claims-need-proof(data/line#/source) | one-change-then-verify
```

43% fewer bytes. Same AI compliance.

---

## The Learning Loop (optional)

Add a `<learn>` section so the system gets wiser, not fatter:

```xml
<learn>
new-preference → update profile.md
3x-repeated-task → automate
told-once → never-ask-again
goal: rules→fewer rapport→deeper
</learn>
```

New lessons → absorbed into existing principles, not appended to CLAUDE.md.

---

## Anti-patterns

| Don't | Why |
|-------|-----|
| Write CLAUDE.md as human prose | Token waste |
| Same rule in CLAUDE.md + rules/ | Paying double every turn |
| Rarely-used info in always-loaded files | Paying for unused context |
| 20+ rules | 5 compressed principles > 20 specific rules |
| Describe hook behavior in text | Hooks auto-run, text is redundant |
| Add rules without compressing | System grows fatter, not wiser |

## Expected Results

| Metric | Before | After |
|--------|--------|-------|
| Auto-loaded size | 15-30 KB | 2-3 KB |
| Tokens per turn | 4,000-8,000 | 500-800 |
| 50-turn session waste | 200-400K tokens | 25-40K tokens |
| Behavioral compliance | baseline | same or better |
| Effective session capacity | 1x | ~2x |
