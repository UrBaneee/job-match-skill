# jobmatch

A [Claude Code Skill](https://docs.claude.com/en/docs/claude-code/skills) for daily, brutally strict job-hunt matching. It searches multiple job platforms for openings, evaluates each one against your own evidence-backed "facts ledger" (not vibes, not the job title), and reports back **every** posting that genuinely clears the bar — no artificial cap, no padding.

## Why this exists

Most "find me jobs" prompting either dumps a wall of loosely-relevant listings or arbitrarily caps the output at a round number (10 jobs, whether 3 or 30 actually qualify). Neither is useful when you're job-hunting: you want *every* opening you can honestly apply to, ranked by how fresh it is, with an honest gap list — not a curated highlight reel.

This skill enforces:

- **Evidence-only claims** — every "you're qualified for this" statement must cite an ID in your own facts ledger. No fact ID, no credit.
- **No seniority/YOE gatekeeping** — a "Senior Engineer" posting whose actual requirements you meet is in scope; job titles and "5+ years" numbers are noise, not filters.
- **No output cap** — if 6 jobs qualify, you get 6; if 40 qualify, you get 40. The strictness is in the evaluation, not in an arbitrary quota.
- **Freshness-first** — postings are sorted newest-first and anything stale (>90 days old, or posted in a prior year, or an undated evergreen pipeline) is dropped, because applying to a dead listing wastes your time.

## How it works

1. **Load your ground truth** — reads your facts ledger (skills, projects, experience, work-authorization status) and your role-keyword list.
2. **Search broadly** — hits ATS APIs (Greenhouse/Lever/Ashby) directly for full JD text, plus web search and aggregator discovery for companies you don't already track. See [`references/search-playbook.md`](references/search-playbook.md).
3. **Evaluate strictly** — every posting gets the full rubric in [`references/evaluation-rubric.md`](references/evaluation-rubric.md): hard exclusions first (location, work authorization, staleness), then a requirement-by-requirement mapping against your ledger (hit / partial / missing), then a tier verdict.
4. **Report everything that qualifies** — one table, freshest first, both tiers included, no cap.
5. **Log** every evaluated posting (including rejects) to a dedupe ledger so nothing gets re-recommended.

## Setup

This is a template — it references placeholder paths you need to fill in for your own use:

- A **facts ledger** (`facts.md`): a personal file where every skill/project/experience claim has a stable ID (e.g. `PROJ-01`, `SK-03`) that you can cite as evidence. This is the single source of truth the skill is not allowed to go beyond — if a requirement isn't backed by an ID in this file, it's scored as missing.
- A **role-keywords file** (`JD_KEYWORDS.md`): your target role families and the actual job-title strings recruiters use for each, so search terms aren't guessed from scratch every run.
- A **workspace** for `seen.tsv` (dedupe ledger) and `reports/YYYY-MM-DD.md` (daily output).

Update the `Fixed paths` section in [`SKILL.md`](SKILL.md) to point at your own files, then invoke the skill (e.g. `/jobmatch`) from a Claude Code session with file access to those paths.

## Guardrails

- Read-only toward the outside world: searches and fetches job postings, never applies on your behalf or uploads your resume/personal data anywhere.
- Every claim traces to a fact ID; unconfirmed ledger entries are never used as evidence.
- Honesty over padding — gaps are reported plainly, and a sub-bar match is never inflated to pad the count.

## License

MIT — see [LICENSE](LICENSE).
