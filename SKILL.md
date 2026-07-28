---
name: jobmatch
description: Daily job hunt. Search all platforms for US jobs, strictly evaluate the candidate's real abilities (a facts ledger) against each full JD ignoring seniority labels, and deliver EVERY genuinely applyable posting (no upper cap; 10 is a floor, not a target) with fact-grounded match evidence. Use when the user invokes /jobmatch, asks to find jobs to apply to today, or asks whether a specific posting matches their abilities.
---

# jobmatch: daily strict-match job hunt

Find **every** US job posting the candidate can genuinely do, judged **only** on ability-vs-JD evidence. Titles and seniority words carry zero weight; unverifiable claims about abilities are forbidden.

**No upper cap on results.** The candidate is job-hunting and wants a shot at every posting they qualify for. Present *all* jobs that clear the qualify bar (Tier S and Tier A alike). 10 is the minimum to aim for, never a ceiling. If 25 qualify, report 25. The only reasons a qualifying job is left out are the strict rubric (a required item missing, or a hard exclusion), never "we already have enough" or "this family is over-represented today." Strictness gates *quality*, not *quantity*.

## Fixed paths (adapt these to your own setup)

- **Facts ledger (single source of truth for ability claims)**: `<path-to-your-facts-ledger>/facts.md`. Every skill/project/experience claim must trace to an ID in this file. See [references/evaluation-rubric.md](references/evaluation-rubric.md) for how IDs get cited.
- **Role keywords**: same directory, `JD_KEYWORDS.md`. Your role families and the real-world search terms per family.
- **Workspace**: `<path-to-your-workspace>/`. Contains `seen.tsv` (dedupe ledger) and `reports/YYYY-MM-DD.md` (daily output).

## Workflow

### 1. Load ground truth

Read `facts.md` in full: every skill, project, and experience entry, each with a stable ID (this template's examples use prefixes like `EVG-xx`, `RAG-xx`, `MAE-xx`, `SK-xx` for one candidate's ledger; substitute your own ledger's ID scheme). Also note the candidate's own work-authorization status (citizenship, visa, green card, or sponsorship needs) and location constraints, since Step A of the rubric depends on this being accurate, not assumed. Read `JD_KEYWORDS.md` for the role families and their real-world search terms. Read `seen.tsv` if it exists; every URL and company+title already there is off-limits for re-recommendation (re-evaluation allowed only if the user asks).

### 2. Search broadly

Follow [references/search-playbook.md](references/search-playbook.md). Requirements:

- Cover **at least 3 distinct source types** per run (e.g. ATS APIs, web search, HN/aggregators). Rotate role families across days **to widen search coverage**, i.e. so under-searched families still get harvested, NOT to cap or balance the output. Every family the candidate qualifies in should be searched; if a family yields 8 qualifying jobs, all 8 are reported.
- Collect **enough candidates to surface every qualifying job**, not a fixed number. Cast wide (aim for 30 to 50+ raw candidates across families) since strict scoring has heavy attrition. Under-collecting silently drops jobs the candidate could have applied to.
- Prefer sources whose full JD text is fetchable (Greenhouse/Lever/Ashby). A snippet is never enough to evaluate.
- **Freshness is a first-class filter and the primary sort. Job-hunters need current openings; stale postings waste their time.**
  - **Sort every output freshest-first** by best-known date (Greenhouse `first_published`, Ashby `publishedAt`, Lever `createdAt`). The newest jobs go at the top of the report where the candidate looks first. Date-descending order beats tier order for the overall list (still note tier per row).
  - **Drop stale postings**: anything **more than 90 days old** (roughly 3 months), or first-published in a **prior calendar year**, or an undated "evergreen-pipeline" role (e.g. some companies keep reqs open for years). These are almost always filled or ghost. Do NOT recommend them, even if the ability match is perfect. A prior-year posting in the current year is a no-send.
  - **Greenhouse rescue**: if `first_published` is old but `updated_at` is within roughly 21 days, the role is being actively refreshed, so keep it and show the updated date. (Ashby/Lever expose no update signal, so their date is final.)
  - Within the kept set, 14 days or newer is ideal; flag anything 45 to 90 days old as "aging" so the candidate prioritizes the fresh ones.

### 3. Evaluate strictly: the core contract

For each candidate posting, fetch the **full JD**, then apply [references/evaluation-rubric.md](references/evaluation-rubric.md). Summary of the non-negotiables:

- **Ignore completely**: title seniority words (Junior/Senior/Staff/Lead/Principal), years-of-experience numbers. Record YOE in the report but never gate on it.
- **Hard exclusions** (instant reject): work-authorization or clearance requirements the candidate doesn't meet; location outside where the candidate can/will work; a degree or license the candidate lacks as a stated requirement; staffing-agency spam or ghost reposts.
- **Requirement mapping**: score every concrete required skill/responsibility as hit, partial, or missing, with each hit/partial citing fact IDs from the ledger. No fact ID, no credit.
- **Qualify bar**: zero missing among required technical items. Partial credit allowed only with cited adjacent evidence. Preferred-item misses are fine but must be listed.
- Tier the survivors: **S (strong)**, all required items hit. **A (qualified)**, required items all at least partial.

### 4. Report

Write `reports/YYYY-MM-DD.md` using the format in [references/evaluation-rubric.md](references/evaluation-rubric.md) § Report format. **Report every job that clears the qualify bar, with no upper limit.** Both Tier S and Tier A are applyable and both go in the main recommendation list (tier just signals strength). If only 6 qualify, deliver 6 and say so; if 30 qualify, deliver 30. Never pad with sub-bar matches (that wastes an application) and never trim qualifying ones (that costs a shot). Each entry names the recommended resume variant/family and the top gap to prep for. Order by freshness (see §2) so the candidate can triage, but present them all.

### 5. Log

Append every *evaluated* posting (including rejects) to `seen.tsv`:
`date<TAB>company<TAB>title<TAB>url<TAB>tier(S/A/REJECT)<TAB>one-line-reason`

### 6. Hand off

End the report and the chat summary with: pick jobs, then run your resume-tailoring workflow (e.g. a companion "tailor résumé" skill) with the posting URL. Never auto-apply, never upload the candidate's resume or personal data anywhere.

## Guardrails

1. Ability claims must trace to facts-ledger IDs. Entries the ledger itself marks unconfirmed/unverified are unusable.
2. Judge from full JD text only; if a JD can't be fetched, drop the posting rather than guess.
3. Honesty over padding: never inflate a sub-bar match to hit a number, and report gaps un-sugarcoated. But this is not a reason to withhold *qualifying* jobs; every job that clears the bar gets reported, however many that is. The cap is on weak matches, never on strong ones.
4. Read-only toward the outside world: search and fetch, nothing else.
