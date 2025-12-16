# Session Debugging Protocol

> Evidence-driven approach to diagnosing and repairing corrupted Amplifier session transcripts

---

## When to Use

Apply this protocol when:
- Amplifier sessions fail to load with provider errors
- Error mentions "tool_use ids were found without tool_result blocks"
- Session appears stuck or corrupted
- Need to recover conversation history

---

## Core Framework: The 5-Phase Approach

### Phase 1: Locate & Examine

**Given:** Session ID or error logs

**Actions:**
1. Find the transcript file
2. Check basic health (line count, tail)
3. Verify JSON structure

```bash
# Find transcript
find ~/.amplifier -name "transcript.jsonl" -path "*SESSION_ID*"

# Quick health check
wc -l transcript.jsonl
tail -10 transcript.jsonl
```

### Phase 2: Error Pattern Recognition

**Key principle:** Let the error guide you to the evidence

**Pattern 1: Orphaned Tool Calls** (Most common)
- **Signature:** "tool_use ids were found without tool_result blocks"
- **Root cause:** Session crashed before tool execution completed
- **Evidence:** Grep for tool IDs mentioned in error

```bash
grep -n "toolu_01ABC..." transcript.jsonl
```

**Pattern 2: Duplicate Messages**
- **Signature:** Same message repeated
- **Detection:** Compare consecutive lines

**Pattern 3: Invalid JSON**
- **Signature:** Can't parse file
- **Detection:** JSON validation per line

### Phase 3: Scope Assessment

**Critical question:** "Remove just the bad line, or everything after?"

**Decision framework:**
1. Examine lines after corruption point
2. If they reference broken state ("continue", "stuck") → Remove them
3. If independent → Consider keeping (but risky)
4. **Default:** Clean break (remove everything from corruption onward)

**Always show user:**
- What will be removed
- Why it needs removal
- Get confirmation on scope

### Phase 4: Safe Repair

**Golden rule:** Backup first, always

```bash
# Backup
cp transcript.jsonl transcript.jsonl.backup

# Remove corrupted lines (keep 1 through LAST_GOOD)
sed -n '1,NUMBERp' transcript.jsonl.backup > transcript.jsonl
```

### Phase 5: Multi-Channel Validation

**Don't assume it worked - verify through multiple channels:**

```bash
# Channel 1: Line count
wc -l transcript.jsonl

# Channel 2: Grep for bad tool IDs (should be 0)
grep -c "TOOL_ID" transcript.jsonl || echo "0 - cleaned!"

# Channel 3: Inspect tail
tail -3 transcript.jsonl

# Channel 4: JSON validity
tail -1 transcript.jsonl | python3 -m json.tool >/dev/null
```

**Report clearly:**
- What was removed
- What changed (line count, etc.)
- Where backup is stored
- Expected outcome

---

## Thinking Patterns Applied

### 1. Error-First Tracing (PATTERNS.md #1)
Start with exact error message → Extract tool IDs → Grep to source

### 2. Grep-Based Evidence (PATTERNS.md #15)
Use grep with `-n` for line numbers, not guessing

### 3. Context Examination (UNBLOCKING-PLAYBOOK.md #3)
Don't just look at the bad line - widen search to surrounding context

### 4. Iterative Validation (PATTERNS.md #13)
After first fix, validate. If user says "still broken" → Run diagnostics again

### 5. Multi-Channel Verification (PATTERNS.md #13)
Line count + grep + tail + JSON check = comprehensive validation

### 6. Explicit Problem Restatement (PATTERNS.md #3)
Before fixing: "Line X contains Y, which causes Z. I will remove A-B."

---

## Real Example Walkthrough

### User Report:
```
Error: tool_use ids were found without tool_result blocks:
toolu_01VY5pTh2mqfRrtoUyHtXrEr, toolu_01PM1h3FfiQPSuQhBuU7PCoS
```

### Phase 1: Locate
```bash
find ~/.amplifier -name "transcript.jsonl" -path "*360404ec*"
# Found: ~/.amplifier/projects/.../360404ec.../transcript.jsonl
```

### Phase 2: Recognize Pattern
```bash
grep -n "toolu_01VY5p" transcript.jsonl
# Line 137: {"role": "assistant", "content": [...tool_call blocks...]}
```

**Pattern identified:** Orphaned tool calls at line 137

### Phase 3: Assess Scope
```bash
sed -n '136,140p' transcript.jsonl | cat -n
```

**Found:**
- Line 136: Valid tool result (last good line)
- Line 137: Assistant with orphaned tools
- Line 138-139: User saying "continue, you were stuck"

**Decision:** Remove 137-139 (they reference the broken state)

### Phase 4: Repair
```bash
cp transcript.jsonl transcript.jsonl.backup
sed -n '1,136p' transcript.jsonl.backup > transcript.jsonl
```

### Phase 5: Validate
```bash
wc -l transcript.jsonl  # 136 (was 139) ✓
grep -c "toolu_01VY5p" transcript.jsonl  # 0 ✓
tail -3 transcript.jsonl  # Shows clean ending ✓
```

**Result:** Session fixed, reduced from 139 → 136 lines

### User Says: "Still broken"
**Don't repeat same fix - new diagnostics:**

```bash
grep -n "toolu_01TNv1" transcript.jsonl
# Line 138: Found another orphaned tool!
```

**Lesson:** Iterative validation caught second corruption

---

## Key Principles

| Principle | Application |
|-----------|-------------|
| **Evidence over intuition** | Grep for tool IDs, don't guess locations |
| **Show before change** | Display what will be removed, get confirmation |
| **Validate iteratively** | After each fix, check if user reports success |
| **Backup always** | Never modify without .backup file |
| **Widen when blocked** | If first fix fails, look at broader context |
| **Multi-channel verification** | One check isn't enough |

---

## Anti-Patterns

| Trap | Signal | Fix |
|------|--------|-----|
| **Modify without backup** | No safety net | Always `cp` first |
| **Remove just bad line** | Leaves inconsistent state | Remove dependent messages too |
| **Assume one fix enough** | User reports "still broken" | Run fresh diagnostics |
| **Skip validation** | Don't know if it worked | All 4 channels every time |
| **Manual JSON editing** | Risk syntax errors | Use `sed` to remove whole lines |

---

## Quick Reference Commands

```bash
# === LOCATE ===
find ~/.amplifier -name "transcript.jsonl" -path "*SESSION_ID*"

# === DIAGNOSE ===
wc -l transcript.jsonl
grep -n "TOOL_ID" transcript.jsonl
sed -n 'LINE-2,LINE+2p' transcript.jsonl | cat -n

# === REPAIR ===
cp transcript.jsonl transcript.jsonl.backup
sed -n '1,LAST_GOOD_LINEp' transcript.jsonl.backup > transcript.jsonl

# === VALIDATE ===
wc -l transcript.jsonl
grep -c "TOOL_ID" transcript.jsonl || echo "0 - cleaned!"
tail -3 transcript.jsonl | cat -n
tail -1 transcript.jsonl | python3 -m json.tool >/dev/null && echo "Valid"
```

---

## Success Checklist

After repair, verify:
- [ ] Session loads without provider errors
- [ ] No orphaned tool_use blocks remain (grep returns 0)
- [ ] Transcript ends at valid message boundary
- [ ] JSON structure is valid (python3 -m json.tool passes)
- [ ] Backup exists at transcript.jsonl.backup
- [ ] User understands what was removed and why
- [ ] If user says "still broken", run fresh diagnostics
