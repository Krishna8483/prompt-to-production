# Session README — Vibe Coding Workshop (Civic Tech Edition)

Participant branch: `participant/participant-pune` (city: **Pune**)

This file is the participant's session log. It does not replace the workshop
`README.md` — it documents what was built, how to run each piece, and progress
across the four use cases.

---

## Repo status

| UC | Deliverable | Status |
|----|-------------|--------|
| UC-0A | Complaint Classifier | ✅ Built · `results_pune.csv` produced |
| UC-0B | Summary That Changes Meaning | ⏳ Not started |
| UC-0C | Number That Looks Right | ⏳ Not started |
| UC-X | Ask My Documents | ⏳ Not started |

Data files confirmed present:
`data/city-test-files/`, `data/policy-documents/`, `data/budget/ward_budget.csv`.

Environment: Python 3.14.7 · Git 2.55.0 · `csv`/`json` import OK.

---

## UC-0A — Complaint Classifier

Files:
- `uc-0a/agents.md` — RICE spec (role, intent, context, enforcement)
- `uc-0a/skills.md` — `classify_complaint` + `batch_classify`
- `uc-0a/classifier.py` — implementation
- `uc-0a/results_pune.csv` — generated output (15 rows)

**How to run:**
```bash
cd uc-0a
python classifier.py --input ../data/city-test-files/test_pune.csv --output results_pune.csv
```

**Classification rules (enforced):**
- `category` — exact strings only: Pothole · Flooding · Streetlight · Waste ·
  Noise · Road Damage · Heritage Damage · Heat Hazard · Drain Blockage · Other
- `priority` — `Urgent` iff a severity keyword is present
  (`injury, child, school, hospital, ambulance, fire, hazard, fell, collapse`);
  `Low` only for minor markers; otherwise `Standard`
- `reason` — one sentence quoting the exact words that drove the decision
- `flag` — `NEEDS_REVIEW` on genuine ambiguity (category tie, no match, or
  heritage-context streetlight complaints), otherwise blank

**CRAFT loop — what failed on the naive prompt → what changed:**
- *False confidence on ambiguity:* the heritage-street complaint (PM-202430,
  "Heritage street, lights out") was classified confidently with no flag.
  → Added heritage-context conflict detection; it now yields `Streetlight` +
  `NEEDS_REVIEW`.
- *Category tie blindness:* flooding + drain-blocked complaints were resolved
  without any review marker. → Ties now force `NEEDS_REVIEW`.
- *Severity blindness:* priority was detached from severity keywords.
  → Priority is now strictly a function of the severity keyword list.

**Verification:** a rule-checker (categories, priorities, urgent-trigger
consistency, reason presence, flag validity) passes all 15 rows with 0 errors.

---

## Suggested commit (per workshop standard)

```
[UC-0A] Fix ambiguity blindness: heritage-context and category ties were resolved
with false confidence → added NEEDS_REVIEW flagging + strict severity enforcement
```