# jobmatch

> **Other tools match the keywords and title-eligibility on your resume. This skill matches what you've actually done and can prove, your real ability and level: neither scared off by seniority labels nor fooled by them.**
>
> *别的工具匹配你简历上的关键词和 title 资格；这个 skill 匹配你真实做过、能拿证据的能力和水平——既不被职级字眼吓退，也不被它蒙骗。*

A [Claude Code Skill](https://docs.claude.com/en/docs/claude-code/skills) for daily, brutally strict job-hunt matching. It searches multiple job platforms for openings, evaluates each one against your own evidence-backed "facts ledger" (not vibes, not the job title), and reports back **every** posting that genuinely clears the bar, with no artificial cap and no padding.

## What's new

**2026-08-10**
- **Time windows.** Pass `24h`, `7d`, `30d`, or `all` to control how far back the search looks. Defaults to the gap since your last run.
- **Works outside tech.** Added a per-industry ATS guide (Workday, USAJobs, iCIMS/Taleo) and a verified Workday recipe, since the shipped example roster only covers tech and AI employers.
- **Roster correction.** Found that a "company not found" (404) usually meant a wrong slug or the wrong ATS, not that the company stopped hiring. Roughly 70% of previously written-off companies were alive under a different one. The bulk-probe method now documented in the playbook fixed this and nearly doubled the example roster.
- **Search-term correction.** Pass-rate analysis of 197 evaluated postings found that ambiguous role titles matter a lot: "Sales Engineer" and "Pre-Sales" postings cleared the bar far less often than functionally identical, client-facing technical work titled "Applied AI Architect", "Deployment Strategist", or "Forward Deployed Engineer" (see § What makes this different for the numbers). If your candidate is a lateral mover without a professional sales career, favor the builder-titled search terms.

## What makes this different

Most "find me jobs" prompting either dumps a wall of loosely-relevant listings or arbitrarily caps the output at a round number (10 jobs, whether 3 or 30 actually qualify). Neither is useful when you're job-hunting. You want *every* opening you can honestly apply to, ranked by how fresh it is, with an honest gap list, not a curated highlight reel.

**Evidence you can audit, not a match score.** Every "you're qualified for this" statement cites an ID from your own facts ledger. You can check each one. The skill structurally cannot invent a qualification for you: no fact ID, no credit.

**It ignores the theater.** Job titles and "5+ years required" get recorded but never filter. A "Senior Engineer" posting whose real requirements you meet is in scope. A "Junior" posting that quietly demands 8 years of domain experience is not.

**No output cap, no padding.** If 6 jobs qualify, you get 6. If 40 qualify, you get 40. The strictness lives in the evaluation, never in a quota.

**The reject log becomes a targeting map, not a study list.** Every posting evaluated, including the ones thrown out, is logged with why. Run it for a while and a pattern shows up: the same underlying work, titled differently, passes at wildly different rates. One real example from 197 evaluations: postings titled "Sales Engineer" or "Pre-Sales" cleared the bar 14% of the time (5 of 36); the same client-facing, technical work titled "Applied AI Architect" or "Deployment Strategist" cleared 50% (9 of 18); titled "Forward Deployed Engineer", 77% (20 of 26). The first requires a professional sales career history no project can substitute for. The other two get scored on builder ability. Be honest about the limits here: most rejection reasons (people-management experience, a specific industry vertical, a professional sales background) require *already having a different job*, not studying harder. What the log is actually good for is telling you where to aim your applications, not what to go learn.

**Every match ships with its gap.** Each recommendation names the one thing to prepare before the interview, not just a yes.

**It never applies for you.** Read-only by design: it searches, reads, and evaluates. No auto-apply, no accounts created, no resume uploaded anywhere.

## Choosing a time window

Pass the window as an argument:

```
/jobmatch 24h     last 24 hours, for a daily run
/jobmatch 7d      last week, the common case
/jobmatch 30d     catching up after a break
/jobmatch all     exhaustive audit
```

With no argument it infers the window from your last run recorded in `seen.tsv`, so a daily user gets 24h and someone returning after 8 days gets 8d. The window used is printed at the top of every report.

Two things worth knowing:

- **A tight window runs faster, not slower.** The date filter is applied at the job-board list stage, before the expensive full-JD fetches, so a daily 24h sweep is cheap to run.
- **A quiet day is a real answer.** Strict scoring plus a 24-hour window will sometimes return 0 to 3 jobs. The skill reports that number honestly instead of loosening the bar, and when a window yields fewer than 5 it appends a clearly separated "just outside your window" section so you can still see what's nearby.

## Cost, and choosing a model

Reading full job descriptions into context to evaluate them is the expensive part of a run, not the searching. Two levers matter more than which model you run it with:

1. **Window size.** A `24h` run might evaluate 5 to 15 candidates; an `all` run can hit 100+. Pick the narrowest window that fits how often you actually run it.
2. **Requirements-only extraction.** When a run needs to triage a large batch before full evaluation, pulling just the requirements section out of each JD (not the whole page, careers boilerplate included) cuts what gets read by 60 to 75% with no loss to evaluation quality.

**On splitting models**: a single invocation runs end to end on one model, there's no built-in way to run the search on a cheap model and the evaluation on an expensive one within the same command. If you want that split, do it manually: switch to a cheaper model, let it run the search step (mostly executing scripts and producing a candidate list, not judgment calls), then switch to a stronger model before evaluation begins.

Whichever way you run it, don't run the *evaluation* step on a weak model to save cost. The whole point of this skill is that "you qualify" is a claim someone can rely on. A weaker model is more likely to award credit for a "close enough" skill the ledger doesn't actually support, or to misread a borderline exclusion. Two real examples from actual runs: a green-card holder reads differently under an export-control clause than under a citizenship requirement, and a job's location field split across a primary field and a secondary-locations field needs an actual check, not a guess from the primary field alone. Getting either wrong, in either direction, costs more than running slowly.

## How it works

1. **Load your ground truth**: reads your facts ledger (skills, projects, experience, work-authorization status) and your role-keyword list.
2. **Search broadly**: hits ATS APIs (Greenhouse/Lever/Ashby) directly for full JD text, plus web search and aggregator discovery for companies you don't already track. See [`references/search-playbook.md`](references/search-playbook.md).
3. **Evaluate strictly**: every posting gets the full rubric in [`references/evaluation-rubric.md`](references/evaluation-rubric.md). Hard exclusions come first (location, work authorization, staleness), then a requirement-by-requirement mapping against your ledger (hit / partial / missing), then a tier verdict.
4. **Report everything that qualifies**: one table, freshest first, both tiers included, no cap.
5. **Log** every evaluated posting (including rejects) to a dedupe ledger, so nothing gets re-recommended.

## Scope: what ships configured, and what you swap

The **evaluation engine is field-agnostic**: a facts ledger, requirement-by-requirement mapping, tier verdicts, and freshness windows all work the same for a nurse, an accountant, or an ML engineer. What is *not* field-agnostic is where it looks for jobs.

**What ships configured**: an AI/tech roster (~140 employers on Greenhouse, Ashby, and Lever). If you are hunting AI/tech roles, it works out of the box.

**If you are not in tech, you must swap the sources.** Measured 2026-08-10: of 14 major non-tech employers (Mayo Clinic, Kaiser, JPMorgan, Goldman, Target, Walmart, McKinsey, Deloitte, Boeing, Harvard, Red Cross, Pfizer, NY Times, Cleveland Clinic), **zero** are reachable on Greenhouse/Ashby/Lever. Those three cover tech startups and little else. Your industry's employers are most likely on **Workday** (healthcare, finance, retail, manufacturing, education), **USAJobs** (US government), or iCIMS/Taleo.

[`references/search-playbook.md`](references/search-playbook.md) opens with a per-industry ATS table, a verified Workday recipe (including how to parse its dates into the freshness window), and a bulk-probe method for building your own roster of 100+ employers in a few minutes. Swap the roster and rewrite `JD_KEYWORDS.md` with your own role titles, and the rest of the machinery carries over unchanged.

## Setup

This is a template. It references placeholder paths you need to fill in for your own use:

- A **facts ledger** (`facts.md`): a personal file where every skill/project/experience claim has a stable ID (e.g. `PROJ-01`, `SK-03`) that you can cite as evidence. This is the single source of truth the skill is not allowed to go beyond. If a requirement isn't backed by an ID in this file, it's scored as missing.
- A **role-keywords file** (`JD_KEYWORDS.md`): your target role families and the actual job-title strings recruiters use for each, so search terms aren't guessed from scratch every run.
- A **workspace** for `seen.tsv` (dedupe ledger) and `reports/YYYY-MM-DD.md` (daily output).

Update the `Fixed paths` section in [`SKILL.md`](SKILL.md) to point at your own files, then invoke the skill (e.g. `/jobmatch`) from a Claude Code session with file access to those paths.

## Guardrails

- Read-only toward the outside world: searches and fetches job postings, never applies on your behalf or uploads your resume/personal data anywhere.
- Every claim traces to a fact ID; unconfirmed ledger entries are never used as evidence.
- Honesty over padding. Gaps are reported plainly, and a sub-bar match is never inflated to pad the count.

## License

MIT, see [LICENSE](LICENSE).
