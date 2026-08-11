---
name: jobmatch
description: Daily job hunt. Search all platforms for US jobs, strictly evaluate the candidate's real abilities (a facts ledger) against each full JD ignoring seniority labels, and deliver EVERY genuinely applyable posting (no upper cap; 10 is a floor, not a target) with fact-grounded match evidence. Accepts a freshness window argument such as 24h, 7d, 30d, or all; defaults to the gap since the last run. Use when the user invokes /jobmatch, asks to find jobs to apply to today or this week, or asks whether a specific posting matches their abilities.
---

# jobmatch: daily strict-match job hunt

Find **every** US job posting the candidate can genuinely do, judged **only** on ability-vs-JD evidence. Titles and seniority words carry zero weight; unverifiable claims about abilities are forbidden.

**No upper cap on results.** The candidate is job-hunting and wants a shot at every posting they qualify for. Present *all* jobs that clear the qualify bar (Tier S and Tier A alike). 10 is the minimum to aim for, never a ceiling. If 25 qualify, report 25. The only reasons a qualifying job is left out are the strict rubric (a required item missing, or a hard exclusion), never "we already have enough" or "this family is over-represented today." Strictness gates *quality*, not *quantity*.

## Time window (how far back to look)

The window is an argument the user passes when invoking the skill, e.g. `/jobmatch 24h`.

| Argument | Window | Use when |
|---|---|---|
| `24h` or `1d` | last 24 hours | running it daily; you only want what's brand new |
| `3d` | last 3 days | checking every few days |
| `7d` or `1w` | last 7 days | the common case, a weekly sweep |
| `14d` / `30d` | 2 / 4 weeks | catching up after a break |
| `90d` | 3 months | the widest useful net |
| `all` | no window | exhaustive audit; ghost-job floor below still applies |

**Default when no argument is given**: infer it from `seen.tsv`. Use the gap since the last recorded run, so a daily user gets 24h and someone returning after 8 days gets 8d. If `seen.tsv` is missing or empty, default to `7d`. This works because everything older was already reported and logged on a previous run.

**State the window on the first line of the report**, including whether it was inferred, e.g. `Window: last 24h (inferred, last run 2026-07-27)`. A saved report should be self-documenting.

**Never silently exceed the window.** A posting outside it does not belong in the main list, however good the ability match.

**Thin windows are expected, not a failure.** Strict scoring plus a 24-hour window will often yield 0 to 3 jobs. That is the correct answer for a quiet day. Report the real number plainly and never loosen the bar to fill space. But when the window yields **fewer than 5 qualifying jobs**, also evaluate the next window up and report those under a clearly separated heading, `Just outside your window (N to M days old)`. Never merge them into the main list. This keeps a quiet day honest and still useful, and leaves the choice with the candidate.

**Ghost-job floor, regardless of window**: even under `all`, still drop postings first-published in a prior calendar year and undated evergreen-pipeline reqs (some companies leave the same req open for years). That is not a freshness preference, it is a filled-or-fake filter.

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
  - **Apply the requested time window** (see the Time window section above) against the true post date: Greenhouse `first_published`, Ashby `publishedAt`, Lever `createdAt`. Anything older is out of the main list.
  - **Filter by date before paying for full JDs.** Ashby and Lever both expose the post date in their list response, so filter there directly. Greenhouse's list endpoint exposes `updated_at` but not `first_published`, so use `updated_at >= window start` as a cheap pre-filter (a strict superset, since anything published inside the window was necessarily updated inside it), then confirm the true date per-job only for survivors. Tight windows therefore run *faster* and cheaper than wide ones, which is what makes a daily 24h run practical. Details in [references/search-playbook.md](references/search-playbook.md).
  - **Sort every output freshest-first.** The newest jobs go at the top of the report where the candidate looks first. Date-descending order beats tier order for the overall list (still note tier per row).
  - **Greenhouse refreshed-but-older roles**: if `first_published` predates the window but `updated_at` falls inside it, the req is being actively refreshed and is plausibly live, but it is not new. Do not put it in the main list. Report it under `Recently refreshed (originally posted <date>)` and show both dates so the candidate can judge. (Ashby/Lever expose no update signal, so their date is final.)
  - Within a wide window (30d or more), flag anything older than 45 days as "aging" so the candidate prioritizes the fresh ones.

### 3. Evaluate strictly: the core contract

For each candidate posting, fetch the **full JD**, then apply [references/evaluation-rubric.md](references/evaluation-rubric.md). Summary of the non-negotiables:

- **Ignore completely**: title seniority words (Junior/Senior/Staff/Lead/Principal), years-of-experience numbers. Record YOE in the report but never gate on it.
- **Hard exclusions** (instant reject): work-authorization or clearance requirements the candidate doesn't meet; location outside where the candidate can/will work; a degree or license the candidate lacks as a stated requirement; staffing-agency spam or ghost reposts.
- **Requirement mapping**: score every concrete required skill/responsibility as hit, partial, or missing, with each hit/partial citing fact IDs from the ledger. No fact ID, no credit.
- **Qualify bar**: zero missing among required technical items. Partial credit allowed only with cited adjacent evidence. Preferred-item misses are fine but must be listed.
- Tier the survivors: **S (strong)**, all required items hit. **A (qualified)**, required items all at least partial.

### 4. Report

Write `reports/YYYY-MM-DD.md` using the format in [references/evaluation-rubric.md](references/evaluation-rubric.md) § Report format. Open with the time window used, then **report every job that clears the qualify bar inside that window, with no upper limit.** Both Tier S and Tier A are applyable and both go in the main recommendation list (tier just signals strength). If only 6 qualify, deliver 6 and say so; if 30 qualify, deliver 30. Never pad with sub-bar matches (that wastes an application) and never trim qualifying ones (that costs a shot). Each entry names the recommended resume variant/family and the top gap to prep for. Order by freshness (see §2) so the candidate can triage, but present them all. If the window produced fewer than 5, append the `Just outside your window` section described in the Time window section.

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
