# Session README — Vibe Coding Workshop (Civic Tech Edition)

Participant branch: `participant/participant-pune` (city: **Pune**)

This file is the participant's session log. It does not replace the workshop
`README.md` — it documents what was built, how to run each piece, and progress
across the four use cases.

---

## Repo status

| UC | Deliverable | Status |
|----|-------------|--------|
| UC-0A | Complaint Classifier | ✅ Built · `results_[city].csv` produced for all 4 cities |
| UC-0B | Summary That Changes Meaning | ✅ Built · `summary_hr_leave.txt` produced |
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
  plus `results_hyderabad.csv`, `results_kolkata.csv`, `results_ahmedabad.csv`

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

## UC-0B — Summary That Changes Meaning

Files:
- `uc-0b/agents.md` — RICE spec
- `uc-0b/skills.md` — `retrieve_policy` + `summarize_policy`
- `uc-0b/app.py` — implementation
- `uc-0b/summary_hr_leave.txt` — generated output (all 29 clauses)

**How to run:**
```bash
cd uc-0b
python app.py --input ../data/policy-documents/policy_hr_leave.txt --output summary_hr_leave.txt
```

**Enforcement (verified):**
- Clause coverage — every numbered clause (1.1–8.2 = 29) must appear in output; missing summaries raise a fatal error
- Condition preservation — per-clause token check (timeframes, approvers, amounts, prohibitions) fails → clause written verbatim and flagged `[VERBATIM]`
- No invented info — scope-bleed phrases (`as is standard practice`, `typically in government`, ...) are detected and refused
- Binding verbs (`must`/`will`/`requires`/`not permitted`) are never softened

**Verification:** checker confirms 29/29 source clauses present, zero extra
clauses, zero scope-bleed phrases, zero `[VERBATIM]` fallbacks needed, and no
softened binding verbs.

**CRAFT loop — what failed on the naive prompt → what changed:**
- *Clause omission:* the first section parser rejected `5. LEAVE WITHOUT PAY (LWP)`
  because the title regex only allowed letters — sections 5.1–5.4 were silently
  dropped (25 of 29 clauses). → Broadened the title regex to allow parentheses
  and digits; coverage now forces all 29 clauses.
- *Condition dropping:* a plain rephrase could drop "Department Head AND HR
  Director" or "within 48 hours". → Added a token-preservation pass that proves
  every mandatory condition survived; violations are quoted verbatim with
  `[VERBATIM]` instead of silently weakening.

---

## Commit log (workshop formula)

Every change below follows the official formula:
`[UC-ID] Fix [what]: [why it failed] → [what you changed]`

```
[UC-0A] Fix ambiguity blindness: heritage-context and category ties were
classified with false confidence → added NEEDS_REVIEW flagging + strict
severity keyword enforcement

[UC-0A] Fix missing city coverage: only Pune output existed → ran the
classifier on Hyderabad, Kolkata, and Ahmedabad test files; all rows pass
the rule check

[UC-0A] Fix session log: commit history not documented in README → added
commit log section using the workshop formula with both change entries

[UC-0B] Fix clause omission: section-title regex rejected "(LWP)" so
sections 5.1-5.4 were silently dropped → broadened the regex and added a
29-clause coverage check plus a per-clause token-preservation pass
(conditions dropped → verbatim + [VERBATIM] flag)
```