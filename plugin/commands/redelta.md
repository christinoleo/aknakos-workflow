---
name: redelta
description: |
  Audit delivered work coverage against conversation requirements and produce exact changes needed to reach 100% coverage.
---

# ReDelta: Coverage Audit + 100% Rewrite

Audit what was requested vs what was delivered, then rewrite the work plan so every requirement is fully covered.
Be strict, concrete, and skeptical. No hand-waving.
Example: 
- if we had a plan, check the plan against what was asked for
- if we wrote a file, check the file against what was asked for (in plan or conversation)
- if we implemented a feature, check it against what was asked for (in plan or conversation)
If applicable, make a plan to get there.

## Phase 1 — Coverage Audit

Read the entire conversation and extract all requirements (explicit + inferred).

Include:
- Features, behaviors, constraints/preferences, edge cases
- Non-functional expectations (UX, performance, reliability, error handling)
- Acceptance criteria mentioned by user

Score each requirement against what was actually delivered:

| Status | Meaning |
|---|---|
| ✅ Covered | Explicit, verifiable delivered outcomes exist |
| ⚠️ Partial | Mentioned/delivered but vague, incomplete, or weakly evidenced |
| ❌ Missing | Not delivered |

Output:

```
| # | Requirement | Status | Score (0-10) | Minimal change to reach 10/10 |
|---|-------------|--------|--------------|--------------------------------|
| 1 | ... | ✅/⚠️/❌ | 0-10 | exact step edit/addition needed |
```

Then:

```
## Coverage Report
Total Requirements: N
Covered: X (N%)
Partial: Y (N%)
Missing: Z (N%)
Overall Grade: 0-100% + one-line judgment
```

Rule: for every ⚠️/❌ row, the `Minimal change to reach 10/10` cell is mandatory and must be concrete.
If a decision is required, end that cell with one precise blocking question.

## Phase 2 — Rewrite to 100%

Produce an updated minimal delta plan that:
1. Keeps already-good delivered outcomes intact
2. Expands partial items into concrete executable steps
3. Adds missing steps in correct order
4. Keeps dependencies/logical sequencing clear
5. Is specific enough to execute without guessing

After the updated delta plan, if score was low, rerun the coverage audit on the new plan to confirm 100% coverage.

Hard rules:
- Every requirement must reach ✅ 10/10
- If 100% is impossible without input, ask one precise blocking question
- No scope creep: do not add requirements not requested
- No vague language (avoid "maybe", "consider", "etc")
- Modify the original work in as minimal a way as possible
- Ask questions when in doubt
