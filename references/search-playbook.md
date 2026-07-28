# Search Playbook — 全平台搜索战术

Goal: 20–30 fresh candidates per run whose **full JD text is fetchable**. Mix at least 3 source types. Search terms come from `JD_KEYWORDS.md` (the 实际岗位名 lists per role family) — don't invent new ones from scratch.

## Source type 1 — ATS APIs (highest yield, full JD, no blocking)

These return clean JSON including full descriptions. Iterate a company list (see below) rather than crawling.

- **Greenhouse**: list at `https://boards-api.greenhouse.io/v1/boards/<company>/jobs?content=true` — full HTML JD in `content`. Filter client-side by title/keyword/location. **Real post date**: fetch the single-job endpoint `.../jobs/<id>` — it returns `first_published` (the list endpoint does NOT). Use `first_published` as the true post date, `updated_at` as last-refreshed.
- **Lever**: `https://api.lever.co/v0/postings/<company>?mode=json` — JD in `description`/`lists`.
- **Ashby**: `https://api.ashbyhq.com/posting-api/job-board/<company>?includeCompensation=true`
- **SmartRecruiters**: `https://api.smartrecruiters.com/v1/companies/<company>/postings`
- **Workable**: `https://apply.workable.com/api/v1/widget/accounts/<company>` (JD needs per-job fetch)

Company slugs verified working 2026-07-24: **GH** `anthropic`, `scaleai`, `databricks`, `togetherai`, `vercel`, `brex` · **Ashby** `openai`, `cohere`, `harvey`, `sierra`, `decagon`, `writer`, `langchain`, `elevenlabs`, `abridge` · **Lever** `palantir`. Dead as of 2026-07-24 (don't retry blindly; find their current ATS via web search): huggingface/glean/ramp/runwayml/retool/notion/weightsandbiases (GH 404), cresta (Lever 404), perplexity-ai + together-ai (Ashby 404). Each run, also try 2–3 new companies found via web search/HN and record working slugs here.

**Batch added 2026-07-25** via fastaijobs.com discovery (source type 3) — verified live, AI-native/LLM-relevant (filtered out generic IT-support/non-AI-core companies and defense-hardware firms whose roles mostly require citizenship/clearance, e.g. Anduril, Armada gov roles — hard-exclusion territory per rubric):
- **Ashby**: `mercor` (AI hiring/staffing, FDE-heavy — 73 jobs), `basiccapital` (fintech+AI, FDE — 18), `credal` (enterprise LLM governance, FDE — 4), `astronomer` (data orchestration, SE — 27), `blitzy` (AI codegen, solutions — 50), `chalk` (ML feature platform — 15), `coderabbit` (AI code review — 53), `cartesia` (voice AI foundation models — 30), `synthesia` (AI video gen — 76), `replit` (AI coding platform, has DS Trust&Safety — 92), `snorkelai`→note this one is actually GH not Ashby (see below), `suno` (AI music gen — 69), `happyrobot.ai` (voice AI agents — 81), `trm-labs` (blockchain intelligence/ML — 122), `horizon3ai` (autonomous AI pentesting — 98), `robco` (AI robotics — 38), `Crusoe` (note capital C — GPU/AI cloud infra — 359)
- **Greenhouse**: `formationbio` (biotech drug-dev AI — 22), `moloco` (ML-driven adtech — 45), `snorkelai` (data-centric AI/weak supervision — 42)

Deprioritized (verified live but low relevance — generic ops/support roles, not AI-core, or defense/clearance-gated): `andurilindustries`, `armada`, `pomelocare`, `tailscale`, `alphasense`, `midihealth`, `bluefishai`, `platacard`, `engine`, `clair`, `leland`, `recharge`, `span.app`, `airwallex`, `rillet`, `axle-careers`. Skip these in default rotation; revisit only if a specific posting surfaces via web search with a strong title match.

**Batch added 2026-07-25** — small AI-native companies discovered via targeted web search (`"evaluation engineer" OR "LLM evaluation" hiring site:jobs.ashbyhq.com OR ...`), not fastaijobs. These punch far above their size for LLM_Eval/DS_Safety fit — their JDs read almost like they were written against this candidate's eval-heavy project portfolio:
- **Lever**: `apolloresearch` (AI alignment/scheming pre-deployment evals, SF+London — exceptional LLM_Eval fit), `epoch-ai` (AI capability benchmarking, fully remote — exceptional LLM_Eval fit)
- **Ashby**: `nous-research` (open frontier-lab agent/eval engineering, US timezones remote — best single match found to date), `paradigm-health` (clinical-AI eval + analytics engineering, US remote — strong DS_Safety/LLM_Eval hybrid, note: requires dbt/Databricks which isn't in facts.md yet), `thirdlaw` (LLM eval/guardrails engineering — content is a strong match but **this specific posting was ~20 months stale as of 2026-07-25, treat as likely ghost repost; re-verify freshness before reusing the slug**)
- Also verified live but deprioritized after evaluation: `moloco` (GH, adtech ML — Staff-level advertiser-strategy DS role, domain too specialized), `aircall` (Lever — has FDE-titled postings but explicitly commercial/low-code, not a SWE role), `formationbio` (GH — DS roles gate hard on healthcare/EHR domain), `snorkelai` (GH — good FDE/eval fit, keep for rotation), `trm-labs`/`horizon3ai`/`Crusoe`/`replit`/`suno`/`happyrobot.ai`/`mercor`/`basiccapital`/`credal`/`astronomer`/`blitzy`/`coderabbit`/`cartesia` all confirmed good yield 2026-07-25, keep in main rotation (see 2026-07-24 batch above for verified job counts).

**Duplicate-posting trap**: some ATS boards list the same req twice under slightly different job IDs (e.g. TRM Labs "Agent Engineer" and "AI Agent Engineer" postings on 2026-07-25 had identical requirements text) — dedupe by requirements content, not just title, before evaluating both.

Use `curl -s <url> | python3 -c ...` or WebFetch; cache nothing — postings change daily.

## Source type 2 — Web search (discovery of companies you don't know)

WebSearch with role terms from JD_KEYWORDS.md + freshness + ATS site filters:

- `"forward deployed engineer" site:jobs.ashbyhq.com OR site:boards.greenhouse.io OR site:jobs.lever.co`
- `"AI evaluation engineer" OR "LLM evaluation" hiring 2026`
- `"solutions engineer" AI startup remote US posted`
- Vary family per day (rotation note in seen.tsv header or just check yesterday's report).

Search hits from LinkedIn/Indeed/Glassdoor usually 403 on fetch — **chase the same posting to the employer's own careers page / ATS URL** and evaluate there. If no fetchable version exists, drop it.

## Source type 3 — Aggregators & communities

**These are discovery layers only — never evaluate or report a posting straight from an aggregator.** Aggregators re-host/re-index; the JD text may be stale or truncated, and the "Apply" link is what matters. Use them to surface company names + role families you don't already track, then chase the link back to the employer's own ATS (source type 1) and fetch/evaluate the full JD from there.

- **Hacker News Who's Hiring** (current month): fetch via Algolia — `https://hn.algolia.com/api/v1/search_by_date?tags=story,author_whoishiring&query=hiring` to find the thread id, then `https://hn.algolia.com/api/v1/items/<id>`. Grep comments for role keywords + US/remote.
- **fastaijobs.com** — AI-specific job aggregator (Supabase-backed). Its `robots.txt` blocks `/api/` and default crawlers (`Disallow: /` for `User-Agent: *`) but explicitly allows named bots including ClaudeBot on the rendered pages — **never call its API directly, only fetch the allowed HTML pages**. Useful filtered entry points (WebFetch, since it's a JS app — the raw HTML won't have content):
  - `https://www.fastaijobs.com/jobs/role/forward-deployed-engineer`
  - `https://www.fastaijobs.com/jobs/role/solutions-engineer`
  - `https://www.fastaijobs.com/jobs/role/customer-engineer`
  - `https://www.fastaijobs.com/jobs/role/ai-engineer`
  - `https://www.fastaijobs.com/jobs/role/data-scientist`
  - `https://www.fastaijobs.com/jobs/role/ml-engineer`
  - `https://www.fastaijobs.com/jobs/role/mlops-infra-engineer`
  - (full role list at `https://www.fastaijobs.com/sitemap.xml`)
  Each page renders company + title + location + post date + a direct ATS "Apply" link (greenhouse/lever/ashby). WebFetch only surfaces the first handful of rendered rows (no pagination/infinite-scroll support), so treat each page as a "what's newest today" skim, not exhaustive coverage. Pull out companies not already in the verified-slug list below, then fetch their board via source type 1.
- **Wellfound (AngelList)**: `https://wellfound.com/role/r/<slug>` browse pages sometimes fetchable; otherwise use Browser pane.
- **Work at a Startup (YC)**: `https://www.workatastartup.com/companies?query=...` — Browser pane if fetch blocked.
- **RemoteOK / WeWorkRemotely**: fetchable HTML/JSON (`https://remoteok.com/api`), filter US-eligible.

## Source type 4 — Browser pane (last resort, JS-heavy boards)

For LinkedIn Jobs / Indeed / a specific company portal that blocks curl: `preview_start`/`navigate` + `get_page_text`. Costlier — use for high-value leads only, not bulk discovery.

## Practical notes

- Dedupe against `seen.tsv` **before** spending fetches on full JDs.
- Date filter: prefer `first_published`/`createdAt` within 14 days when the API exposes it.
- Location filter: US city or "Remote (US)". "Remote" with no country + US company → verify inside the JD text.
- Keep per-run fetch volume sane: ~10–15 ATS board pulls + 5–8 web searches + 1 HN pull is plenty for 20–30 candidates.
- 403/404/empty board = skip silently, note nothing. Never fabricate a posting or URL from memory — every reported link must have been fetched this run.
