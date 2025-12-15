# Unblocking Playbook

> 10 concrete tactics when you're stuck

---

## When You're Stuck, Try:

### 1. Re-read the error message literally
The parameter name or method causing the issue is often stated explicitly in the error. Don't skim—read every word.

**Example:** `AsyncCompletions.create() got an unexpected keyword argument 'system'` tells you exactly which argument is wrong.

---

### 2. Grep for the error string
Search the codebase for the exact error message or key terms from it. This often leads directly to the offending code.

```bash
grep -r "unexpected keyword argument" .
grep -r "params\[.system.\]" .
```

---

### 3. Widen the search radius
If the obvious location didn't have the problem, check:
- **Test files** - They might assert the old behavior
- **Config files** - Values might be overridden
- **Sibling files** - Same pattern might exist elsewhere
- **Lockfiles** - Different version than expected

---

### 4. Ask "what else could cause this?"
Generate 3 alternative hypotheses before committing to one:

1. Maybe it's in the code I looked at
2. Maybe it's in the tests
3. Maybe it's a version mismatch

Don't tunnel-vision on the first hypothesis.

---

### 5. Check version mismatches
The installed version may not match the expected behavior. Verify:

```bash
# Check lockfile
grep -A 5 "name = \"package\"" uv.lock

# Check installed
pip show package

# Check what's actually imported
python -c "import package; print(package.__version__)"
```

---

### 6. Create standalone validation
If dependencies block running the full test suite, create a minimal test that validates the specific change:

```python
# Don't import the blocked module
# Instead, read the source file directly
with open("module.py") as f:
    source = f.read()

# Check for the specific pattern
assert "old_pattern" not in source
assert "new_pattern" in source
```

---

### 7. Examine sibling implementations
How do similar projects handle this? Check:
- Other modules in the same repo
- Sibling projects with similar functionality
- The original library's examples

```bash
grep -r "same_function" ../sibling-project/
```

---

### 8. Reduce scope
If you're trying to fix multiple things, fix one first:
- Comment out the complex part
- Get the simple case working
- Then re-add complexity

---

### 9. Restate the goal in one sentence
When you're lost, clarify what "done" actually means:

> "This is done when [specific observable outcome]."

If you can't state it clearly, you need to clarify before continuing.

---

### 10. Generate a concrete example
Abstract problems become clearer with specific instances:

- What exact input causes the error?
- What exact output do you expect?
- What exact output do you get instead?

Write it down. Often the act of articulating the example reveals the bug.

---

## Unblocking Checklist

Copy this checklist when you're stuck:

```
[ ] Re-read the error message literally
[ ] Grep for the error string in codebase
[ ] Widen search: tests, configs, sibling files
[ ] List 3 alternative hypotheses
[ ] Check version mismatches
[ ] Create standalone validation if blocked
[ ] Check sibling implementations
[ ] Reduce scope to simplest case
[ ] Restate goal in one sentence
[ ] Generate concrete input/output example
```

---

## When to Escalate

If you've tried 5+ tactics and are still stuck:

1. **Document what you've tried** - This prevents repeating dead ends
2. **Ask for help with specifics** - Include error, hypotheses, and what you ruled out
3. **Take a break** - Sometimes stepping away reveals the obvious

---

## Quick Reference

| Stuck Because... | Try This |
|------------------|----------|
| Can't find the bug | Grep for error string, widen search |
| Fix didn't work | Check tests, check versions, widen search |
| Can't run tests | Create standalone validation |
| Too many options | Check sibling implementations |
| Problem is vague | Restate goal, generate concrete example |
| Overwhelmed | Reduce scope to simplest case |
