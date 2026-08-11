# jobmatch

> **Other tools match the keywords and title-eligibility on your resume. This skill matches what you've actually done and can prove, your real ability and level: neither scared off by seniority labels nor fooled by them.**
>
> *别的工具匹配你简历上的关键词和 title 资格；这个 skill 匹配你真实做过、能拿证据的能力和水平——既不被职级字眼吓退，也不被它蒙骗。*

A [Claude Code Skill](https://docs.claude.com/en/docs/claude-code/skills) for daily, brutally strict job-hunt matching. It searches multiple job platforms for openings, evaluates each one against your own evidence-backed "facts ledger" (not vibes, not the job title), and reports back **every** posting that genuinely clears the bar, with no artificial cap and no padding.

## Why this exists

Most "find me jobs" prompting either dumps a wall of loosely-relevant listings or arbitrarily caps the output at a round number (10 jobs, whether 3 or 30 actually qualify). Neither is useful when you're job-hunting. You want *every* opening you can honestly apply to, ranked by how fresh it is, with an honest gap list, not a curated highlight reel.

This skill enforces:

- **Evidence-only claims**: every "you're qualified for this" statement must cite an ID in your own facts ledger. No fact ID, no credit.
- **No seniority/YOE gatekeeping**: a "Senior Engineer" posting whose actual requirements you meet is in scope. Job titles and "5+ years" numbers are noise, not filters.
- **No output cap**: if 6 jobs qualify, you get 6. If 40 qualify, you get 40. The strictness is in the evaluation, not in an arbitrary quota.
- **Freshness-first, and you pick the window**: run it with `24h` for just-posted roles, `7d` for a weekly sweep, up to `90d` or `all`. Postings are sorted newest-first, and dead listings (prior-year reqs, undated evergreen pipelines) are dropped at every setting.

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
