# Evaluation Rubric: Strict Match Protocol

The whole point of this skill is that a recommendation means "the candidate can actually do this job." Every shortcut here produces a wasted application, so none are allowed.

## Step A: Hard exclusions (before any scoring)

Reject immediately if the JD states any of:

| Condition | Why | Notes |
|---|---|---|
| A work-authorization/status requirement the candidate doesn't meet (e.g. citizenship required, active security clearance, clearable) | Depends on the candidate's actual status. Check the facts ledger, don't assume. | ITAR "US person" language is often fine for green-card holders even though plain "citizenship required" isn't; verify per-candidate |
| Location outside where the candidate is eligible/willing to work, and not remote-eligible for them | The candidate applies within their own stated geography | Confirm the eligible geography (and relocation willingness) from the ledger before rejecting or accepting on this basis |
| "No sponsorship" clauses | NOT automatically an exclusion | Only matters if the candidate needs sponsorship, so check the ledger. If they don't need it, this is actually an advantage; note it in the report. |
| A degree level required "and not equivalent" that the candidate doesn't hold | Compare against the candidate's actual highest degree | "X or equivalent experience" / "X preferred" is fine even if the candidate holds a lower or different degree |
| Professional license/cert the candidate lacks (CPA, PE, RN...) | Unfixable short-term | Check the ledger's certifications section first |
| Staffing-agency reposts, "confidential client", pay-to-apply | Spam signal | Prefer the direct employer posting |
| **Outside the requested time window** (see the Time window section of SKILL.md; default is the gap since the last run, else 7d) | The candidate asked for openings this fresh; older ones belong in the `Just outside your window` section, not the main list | Filter on true post date: Greenhouse `first_published`, Ashby `publishedAt`, Lever `createdAt` |
| **Ghost-job floor, applies even under `all`: first-published in a prior calendar year, or an undated evergreen-pipeline req** | Almost always filled or fake, a no-send even if the ability match is perfect | Greenhouse reqs whose `first_published` is old but `updated_at` is recent are plausibly live: report separately as `Recently refreshed`, showing both dates |

## Step B: What to ignore (record but never gate on)

- **Seniority words in titles**: Junior, Senior, Staff, Lead, Principal, I/II/III. A "Senior X" JD whose actual requirements the candidate meets is IN.
- **Years-of-experience numbers**: "5+ years" is a proxy, not an ability. Record it in the report's YOE column so the candidate knows the framing risk, but never reject on it.
- Vague virtues ("self-starter", "fast-paced environment"). Not scoreable.

## Step C: Requirement mapping (the strict part)

1. Extract from the full JD: every **required** concrete item (skills, tools, task types, responsibilities) and every **preferred** item. Concrete means a thing one can have evidence for ("build RAG pipelines", "Python", "design evals", "customer-facing demos").
2. Map each item to the facts ledger:
   - **Hit**: a fact entry is direct evidence the candidate has done this. Cite ID(s).
   - **Partial**: adjacent evidence, meaning the candidate has done a close neighbor of it. Cite ID(s) **and** name the delta (e.g., "Kubernetes missing, but Docker Compose deployment [PROJECT-ID] gives partial credit").
   - **Missing**: no entry supports it. No credit for plausibility, transferability hand-waving, or "could learn it fast".
3. Anti-inflation rules:
   - A tool name-drop in the ledger counts only if tied to usage evidence. If your ledger's own convention is "every skill must point to evidence of use," honor that: an unconfirmed/detail-pending skill entry is partial at best, never a hit.
   - Entries the ledger flags as unconfirmed/unverified: unusable entirely.
   - One fact ID can support multiple requirements, but stretchy mappings (e.g. "led a negotiation" counted as "enterprise sales experience") must be labeled partial, not hit.
   - When honestly uncertain whether partial or missing, choose missing. Strictness bias is the contract.

## Step D: Verdict

- **Tier S (strong)**: every required item is a hit; preferred items mostly at least partial. Apply today.
- **Tier A (qualified)**: every required item at least partial, zero required items missing. Worth applying; gaps listed.
- **REJECT**: any required technical item missing, or Step A tripped. Goes to `seen.tsv` with reason, never to the report's recommendation list.

**Both S and A are applyable and both belong in the report's main list. There is NO cap on how many.** Tier is a strength signal for triage, not a filter. Do not create a "passed but not featured" bucket that hides qualifying jobs; if it cleared the bar and falls inside the requested time window (see Step A), it goes in the list. Only REJECTs and out-of-window postings stay out of the main list; REJECTs go to `seen.tsv` plus an optional sample section, and out-of-window qualifiers go to the `Just outside your window` section when the window ran thin. **Order the list freshest-first** (most recent post date at the top, since job-hunters think date-first); show the tier per row so the candidate can still weigh strength, but freshness drives the ordering. Never truncate the fresh-and-qualifying set.

Domain-experience requirements ("3+ years in fintech") stated as *required* count as a required item, so the candidate will usually be missing it and get rejected. If merely *preferred*, list as a gap instead.

## Report format (`reports/YYYY-MM-DD.md`)

```markdown
# Job Matches: YYYY-MM-DD

Window: last <24h / 7d / ...> (<requested, or inferred from last run YYYY-MM-DD>)
Scope: <role families covered> · <sources used> · N candidates, M qualify (all shown, S+A both applyable)

## Overview (every job that clears the bar goes in this one table, S and A together, no cap)

| # | Tier | Company | Title | Location | Posted | Last updated | YOE wording | Resume variant | Link |
|---|------|---------|-------|----------|--------|---------------|-------------|-----------------|------|

> Date convention (use the **true first-publish date**; don't dress up "recently refreshed" as "just posted"): **Ashby** = `publishedAt`; **Lever** = `createdAt`; **Greenhouse** = `first_published` (note: this field is only returned by the **single-job endpoint** `/boards/{slug}/jobs/{id}`, not the list endpoint `/jobs`, so fetch it per-job). Keep Greenhouse's `updated_at` as a separate "last updated" column. First-published more than 3 months ago and still open means an evergreen pipeline; flag it.

## Per-job detail

### 1. <Company>: <Title> (Tier S)
- **Link**: <apply URL>  **Location**: ...  **Posted**: ...
- **Why the candidate can do this**: 2 to 3 sentences, citing fact ID(s)
- **Requirement mapping**: `req item → hit (PROJECT-01)` / `req item → partial (PROJECT-15, delta: ...)` ...
- **Gap / interview prep**: ...
- **Recommended resume**: `<YourName>_<family>_2026` plus a one-line reason

## Just outside your window (only when the window yielded fewer than 5)
Qualifying jobs from the next window up, N to M days old, listed separately so the
main list stays true to what was asked for. Same columns as the overview table.

## Recently refreshed (optional)
Greenhouse reqs whose `first_published` predates the window but whose `updated_at`
falls inside it. Show both dates; plausibly live, but not new.

## Today's rejects (optional, 3 to 5 representative REJECTs with reasons)

## Next steps
Pick jobs, then run your resume-tailoring workflow with the posting URL.
```

Chat summary after writing the report: the overview table (or its gist), where the report file is, and the resume-tailoring handoff line. Keep it short.
