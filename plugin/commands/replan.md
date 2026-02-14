---
name: replan
description: |
  Audit plan coverage against conversation requirements and rewrite the plan to 100% coverage.
---

# Replan: Coverage Audit + 100% Rewrite

Audit what was requested vs what was planned, then rewrite the plan so every requirement is fully covered.
Be strict, concrete, and skeptical. No hand-waving.

## Phase 1 — Coverage Audit

Read the entire conversation and extract all requirements (explicit + inferred).

Include:
- Features, behaviors, constraints/preferences, edge cases
- Non-functional expectations (UX, performance, reliability, error handling)
- Acceptance criteria mentioned by user

Score each requirement against the current plan:

| Status | Meaning |
|---|---|
| ✅ Covered | Explicit, actionable plan steps exist |
| ⚠️ Partial | Mentioned but vague/incomplete |
| ❌ Missing | Not addressed |

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

## Phase 2 — Rewrite Plan to 100%

Produce an updated plan that:
1. Keeps already-good covered steps
2. Expands partial items into concrete executable steps
3. Adds missing steps in correct order
4. Keeps dependencies/logical sequencing clear
5. Is specific enough to execute without guessing

After the updated plan, if score was low, rerun the coverage audit on the new plan to confirm 100% coverage.

Hard rules:
- Every requirement appears once in the final checklist
- Final checklist must be 100% ✅ 10/10
- If 100% is impossible without input, ask one precise blocking question
- No scope creep: do not add requirements not requested
- No vague language (avoid "maybe", "consider", "etc")
- Modify the plan in as minimal a way as possible
