# Search Playbook

Goal: enough fresh candidates per run to surface every qualifying job, all with **fetchable full JD text**. Mix at least 3 source types. Search terms come from `JD_KEYWORDS.md` (your role families and the real job titles used for each); don't invent new ones from scratch.

---

## ⚠️ Read this first if you are not in tech

**The company roster shipped in this file is AI/tech only, and the three ATSes it leans on cover almost nothing outside tech.**

Measured 2026-08-10: of 14 major non-tech employers (Mayo Clinic, Cleveland Clinic, Kaiser, JPMorgan, Goldman Sachs, Target, Walmart, McKinsey, Deloitte, Boeing, Harvard, Red Cross, Pfizer, NY Times), **zero** were reachable on Greenhouse, Ashby, or Lever. Those three dominate tech startups and very little else. If you work in healthcare, finance, retail, government, education, manufacturing, logistics, legal, or nonprofit, source type 1 as configured below will return an empty pool, and that is a roster problem, not a bug in the skill.

The evaluation engine (facts ledger, requirement mapping, freshness windows) is field-agnostic. Only the **sources** and the **role keywords** need swapping. Do this:

1. **Find where your industry's employers actually post** (table below), and use those endpoints as your source type 1.
2. **Build your own roster** with the bulk-probe method in § Building your roster.
3. **Rewrite `JD_KEYWORDS.md`** with your own role families and their real-world job titles.

### Which ATS does your field use

| Field | Dominant ATS | Reachable? |
|---|---|---|
| Tech startups, AI labs | Greenhouse, Ashby, Lever | Yes, clean public JSON (this file's default) |
| Healthcare systems, hospitals, pharma | **Workday**, iCIMS, Taleo | Workday yes (recipe below); iCIMS/Taleo via web search |
| Banking, insurance, finance | **Workday**, Taleo | Same |
| Retail, hospitality, logistics | **Workday**, iCIMS | Same |
| Manufacturing, industrial | Workday, SuccessFactors (SAP) | Workday yes; SuccessFactors via web search |
| Universities, education | Workday, PageUp, Interfolio | Workday yes; others via web search |
| US federal / state government | **USAJobs** | Yes, official public API at `developer.usajobs.gov` |
| Nonprofits, small orgs | Workable, BambooHR, Greenhouse | Workable yes (endpoint already listed below) |

### Workday recipe (the enterprise default, works across most non-tech fields)

```
POST https://<tenant>.<host>.myworkdayjobs.com/wday/cxs/<tenant>/<site>/jobs
Content-Type: application/json
{"appliedFacets":{},"limit":20,"offset":0,"searchText":"nurse"}
```

- `<host>` is `wd1` / `wd3` / `wd5` / `wd12`, and `<tenant>` / `<site>` come straight from the employer's careers URL. Open their careers page and copy the pieces out of the address bar.
- **`searchText` does server-side keyword search**, which is actually better than Greenhouse/Ashby/Lever, where you pull the whole board and filter client-side. Query your role titles directly.
- **Dates**: `postedOn` is a human-readable string that is day-precise up to a month (`"Posted Today"`, `"Posted 20 Days Ago"`) and then buckets into `"Posted 30+ Days Ago"`. So it supports the `24h` / `7d` / `30d` windows fine; only `all` loses precision. Parse the string into a day count.
- Verified live 2026-08-10: CVS Health (`cvshealth` / `CVS_Health_Careers` / `wd1`, 1,043 hits for "nurse"), Target (`target` / `targetcareers` / `wd5`), Salesforce (`salesforce` / `External_Career_Site` / `wd12`).

Everything below this line is the tech/AI configuration. Treat the company lists as a worked example of the method, not as the method itself.

---

## Source type 1: ATS APIs (highest yield, full JD, no blocking)

These return clean JSON including full descriptions. Iterate a company list (see below) rather than crawling. **The list below is the AI/tech example roster.** Swap it for your own field's employers, built with § Building your roster.

- **Greenhouse**: list at `https://boards-api.greenhouse.io/v1/boards/<company>/jobs?content=true`, full HTML JD in `content`. Filter client-side by title/keyword/location. **Real post date**: fetch the single-job endpoint `.../jobs/<id>`, which returns `first_published` (the list endpoint does NOT). Use `first_published` as the true post date, `updated_at` as last-refreshed.
- **Lever**: `https://api.lever.co/v0/postings/<company>?mode=json`, JD in `description`/`lists`.
- **Ashby**: `https://api.ashbyhq.com/posting-api/job-board/<company>?includeCompensation=true`
- **SmartRecruiters**: `https://api.smartrecruiters.com/v1/companies/<company>/postings`
- **Workable**: `https://apply.workable.com/api/v1/widget/accounts/<company>` (JD needs per-job fetch)

Company slugs verified working 2026-07-24: **GH** `anthropic`, `scaleai`, `databricks`, `togetherai`, `vercel`, `brex`. **Ashby** `openai`, `cohere`, `harvey`, `sierra`, `decagon`, `writer`, `langchain`, `elevenlabs`, `abridge`. **Lever** `palantir`. Each run, also try a few new companies found via web search and record working slugs here.

**Batch added 2026-08-10** via bulk slug probing (see § Building your roster). 144 candidates probed, **97 hit (67%)**, adding 338 role-family US jobs and roughly doubling the pool:

- **Greenhouse**: `stripe`, `waymo`, `cloudflare`, `coreweave`, `elastic`, `affirm`, `fivetran`, `block`, `figma`, `asana`, `figureai`, `wizinc`, `intercom`, `nuro`, `gusto`, `chainguard`, `chime`, `mercury`, `tailscale`, `lightningai`, `marqeta`, `truveta`, `algolia`, `amplitude`, `arizeai`, `turing`, `heygen`, `alloy`, `airtable`, `observeai`, `typeface`, `descript`, `planetscale`, `labelbox`, `galileo`, `udio`, `stabilityai`, `invisible`, `comet`, `netlify`, `gleanwork`, `cresta`, `duolingo`, `peloton`
- **Ashby**: `snowflake`, `cerebras`, `cursor`, `plaid`, `vanta`, `handshake`, `rogo`, `synthesia`, `lambda`, `baseten`, `fireworks`, `supabase`, `sentilink`, `lumaai`, `attio`, `render`, `sardine`, `modal`, `linear`, `physicalintelligence`, `hex`, `confluent`, `workos`, `socket`, `braintrust`, `dust`, `assembledhq`, `middesk`, `persona`, `midjourney`, `semgrep`, `anyscale`, `nabla`, `lancedb`, `airbyte`, `pika`, `pinecone`, `railway`, `langfuse`, `atlan`, `stytch`, `unit`, `knock`, `cedar`, `weaviate`, `notion`, `ramp`, `perplexity`, `runway`
- **Lever**: `zoox`, `includedhealth`, `neon`, `zilliz`, `unify`, `spotify`
- **Workable**: `huggingface`

Highest-yield additions (role-family US jobs each): `waymo` 64, `zoox` 20, `block` 15, `snowflake` 15, `figureai` 14, `stripe` 14, `handshake` 12, `asana` 10.

**Probed but deliberately excluded**: `shieldai` (434 jobs), `saronic` (247), `applied` (Applied Intuition, 264), `andurilindustries`. Defense-heavy rosters where most roles demand US citizenship or an active clearance, which is a Step A hard exclusion for many candidates. Include them only if that exclusion does not apply to you.

**Not found on any of the five ATS APIs** (likely self-hosted careers pages or Workday/iCIMS): `retool`, `weightsandbiases`/`wandb`. Fall back to source type 2 for these.

**Batch added 2026-07-25** via fastaijobs.com discovery (source type 3), verified live, AI-native/LLM-relevant (filtered out generic IT-support/non-AI-core companies and defense-hardware firms whose roles mostly require citizenship/clearance, e.g. Anduril, Armada gov roles, which are hard-exclusion territory per rubric):
- **Ashby**: `mercor` (AI hiring/staffing, FDE-heavy, 73 jobs), `basiccapital` (fintech+AI, FDE, 18), `credal` (enterprise LLM governance, FDE, 4), `astronomer` (data orchestration, SE, 27), `blitzy` (AI codegen, solutions, 50), `chalk` (ML feature platform, 15), `coderabbit` (AI code review, 53), `cartesia` (voice AI foundation models, 30), `synthesia` (AI video gen, 76), `replit` (AI coding platform, has DS Trust&Safety, 92), `snorkelai` (note: this one is actually GH not Ashby, see below), `suno` (AI music gen, 69), `happyrobot.ai` (voice AI agents, 81), `trm-labs` (blockchain intelligence/ML, 122), `horizon3ai` (autonomous AI pentesting, 98), `robco` (AI robotics, 38), `Crusoe` (note capital C, GPU/AI cloud infra, 359)
- **Greenhouse**: `formationbio` (biotech drug-dev AI, 22), `moloco` (ML-driven adtech, 45), `snorkelai` (data-centric AI/weak supervision, 42)

Deprioritized (verified live but low relevance: generic ops/support roles, not AI-core, or defense/clearance-gated): `andurilindustries`, `armada`, `pomelocare`, `tailscale`, `alphasense`, `midihealth`, `bluefishai`, `platacard`, `engine`, `clair`, `leland`, `recharge`, `span.app`, `airwallex`, `rillet`, `axle-careers`. Skip these in default rotation; revisit only if a specific posting surfaces via web search with a strong title match.

**Batch added 2026-07-25**: small AI-native companies discovered via targeted web search (`"evaluation engineer" OR "LLM evaluation" hiring site:jobs.ashbyhq.com OR ...`), not fastaijobs. These punch far above their size for LLM_Eval/DS_Safety fit; their JDs read almost like they were written against this candidate's eval-heavy project portfolio:
- **Lever**: `apolloresearch` (AI alignment/scheming pre-deployment evals, SF+London, exceptional LLM_Eval fit), `epoch-ai` (AI capability benchmarking, fully remote, exceptional LLM_Eval fit)
- **Ashby**: `nous-research` (open frontier-lab agent/eval engineering, US timezones remote, best single match found to date), `paradigm-health` (clinical-AI eval + analytics engineering, US remote, strong DS_Safety/LLM_Eval hybrid; note: requires dbt/Databricks which isn't in facts.md yet), `thirdlaw` (LLM eval/guardrails engineering; content is a strong match, but **this specific posting was about 20 months stale as of 2026-07-25, treat as likely ghost repost; re-verify freshness before reusing the slug**)
- Also verified live but deprioritized after evaluation: `moloco` (GH, adtech ML, Staff-level advertiser-strategy DS role, domain too specialized), `aircall` (Lever, has FDE-titled postings but explicitly commercial/low-code, not a SWE role), `formationbio` (GH, DS roles gate hard on healthcare/EHR domain), `snorkelai` (GH, good FDE/eval fit, keep for rotation), `trm-labs`/`horizon3ai`/`Crusoe`/`replit`/`suno`/`happyrobot.ai`/`mercor`/`basiccapital`/`credal`/`astronomer`/`blitzy`/`coderabbit`/`cartesia` all confirmed good yield 2026-07-25, keep in main rotation (see 2026-07-24 batch above for verified job counts).

**Duplicate-posting trap**: some ATS boards list the same req twice under slightly different job IDs (e.g. TRM Labs "Agent Engineer" and "AI Agent Engineer" postings on 2026-07-25 had identical requirements text). Dedupe by requirements content, not just title, before evaluating both.

Use `curl -s <url> | python3 -c ...` or WebFetch; cache nothing, since postings change daily.

### Date-filter before paying for full JDs

Full-JD fetches are the expensive part of a run, so apply the time window at the *list* stage, before pulling them. Done this way, a tight window makes a run cheaper rather than costlier, which is what makes a daily 24h sweep practical.

- **Ashby**: the job-board list response already carries `publishedAt` per job. Filter directly, no extra calls.
- **Lever**: the list response already carries `createdAt`. Filter directly, no extra calls.
- **Greenhouse**: the list endpoint exposes `updated_at` but not `first_published`. Filter on `updated_at >= window_start` as a cheap pre-filter. This is a strict superset and therefore lossless: any req first published inside the window was necessarily also updated inside it. Then call the single-job endpoint for survivors only, to confirm the true `first_published` and to separate genuinely new reqs from merely refreshed old ones.

Rough shape for a Greenhouse pre-filter:

```python
# cheap: one list call per company, no per-job fetches yet
jobs = get(f"https://boards-api.greenhouse.io/v1/boards/{co}/jobs")["jobs"]
maybe_fresh = [j for j in jobs if j["updated_at"] >= window_start]   # lossless superset
# expensive: only now, and only for survivors
for j in maybe_fresh:
    detail = get(f"https://boards-api.greenhouse.io/v1/boards/{co}/jobs/{j['id']}")
    true_date = detail["first_published"]        # the real post date
```

Titles still get filtered against `JD_KEYWORDS.md` as before; do that in the same cheap pass.

## Source type 2: Web search (discovery of companies you don't know)

WebSearch with role terms from JD_KEYWORDS.md plus freshness and ATS site filters:

- `"forward deployed engineer" site:jobs.ashbyhq.com OR site:boards.greenhouse.io OR site:jobs.lever.co`
- `"AI evaluation engineer" OR "LLM evaluation" hiring 2026`
- `"solutions engineer" AI startup remote US posted`
- Vary family per day (rotation note in seen.tsv header, or just check yesterday's report).

Search hits from LinkedIn/Indeed/Glassdoor usually 403 on fetch. **Chase the same posting to the employer's own careers page / ATS URL** and evaluate there. If no fetchable version exists, drop it.

## Source type 3: Aggregators and communities

**These are discovery layers only. Never evaluate or report a posting straight from an aggregator.** Aggregators re-host/re-index; the JD text may be stale or truncated, and the "Apply" link is what matters. Use them to surface company names and role families you don't already track, then chase the link back to the employer's own ATS (source type 1) and fetch/evaluate the full JD from there.

- **Hacker News Who's Hiring** (current month): fetch via Algolia at `https://hn.algolia.com/api/v1/search_by_date?tags=story,author_whoishiring&query=hiring` to find the thread id, then `https://hn.algolia.com/api/v1/items/<id>`. Grep comments for role keywords plus US/remote.
- **fastaijobs.com**: AI-specific job aggregator (Supabase-backed). Its `robots.txt` blocks `/api/` and default crawlers (`Disallow: /` for `User-Agent: *`) but explicitly allows named bots including ClaudeBot on the rendered pages. **Never call its API directly, only fetch the allowed HTML pages.** Useful filtered entry points (WebFetch, since it's a JS app; the raw HTML won't have content):
  - `https://www.fastaijobs.com/jobs/role/forward-deployed-engineer`
  - `https://www.fastaijobs.com/jobs/role/solutions-engineer`
  - `https://www.fastaijobs.com/jobs/role/customer-engineer`
  - `https://www.fastaijobs.com/jobs/role/ai-engineer`
  - `https://www.fastaijobs.com/jobs/role/data-scientist`
  - `https://www.fastaijobs.com/jobs/role/ml-engineer`
  - `https://www.fastaijobs.com/jobs/role/mlops-infra-engineer`
  - (full role list at `https://www.fastaijobs.com/sitemap.xml`)
  Each page renders company, title, location, post date, and a direct ATS "Apply" link (greenhouse/lever/ashby). WebFetch only surfaces the first handful of rendered rows (no pagination/infinite-scroll support), so treat each page as a "what's newest today" skim, not exhaustive coverage. Pull out companies not already in the verified-slug list above, then fetch their board via source type 1.
- **Wellfound (AngelList)**: `https://wellfound.com/role/r/<slug>` browse pages sometimes fetchable; otherwise use Browser pane.
- **Work at a Startup (YC)**: `https://www.workatastartup.com/companies?query=...`, use Browser pane if fetch is blocked.
- **RemoteOK / WeWorkRemotely**: fetchable HTML/JSON (`https://remoteok.com/api`), filter US-eligible.

## Source type 4: Browser pane (last resort, JS-heavy boards)

For LinkedIn Jobs / Indeed / a specific company portal that blocks curl: `preview_start`/`navigate` plus `get_page_text`. Costlier, so use for high-value leads only, not bulk discovery.

## Practical notes

- Dedupe against `seen.tsv` **before** spending fetches on full JDs.
- Date filter: apply the run's time window at the list stage (see "Date-filter before paying for full JDs" above) rather than after fetching everything.
- On a tight window (24h/3d), the ATS APIs carry the run. Web search and aggregators are tuned for discovery, not recency, so they mostly surface older reqs; keep them for widening the company roster, and expect their hits to land in the out-of-window section.
- Location filter: US city or "Remote (US)". "Remote" with no country plus a US company means you should verify inside the JD text.
- Keep per-run fetch volume sane: roughly 10–15 ATS board pulls plus 5–8 web searches plus 1 HN pull is plenty for 20–30 candidates.
- 403/404/empty board means skip silently, note nothing. Never fabricate a posting or URL from memory; every reported link must have been fetched this run.

## Building your roster (do this when results feel thin)

**Diagnose before you loosen anything.** Thin output is almost always too few employers, not too strict a filter. Measure the funnel first:

```
all open jobs → title matches a role family → located where you can work → inside the time window
```

Measured baseline 2026-08-10 on the AI/tech roster: 45 companies, 6,067 open jobs, but only **~8 jobs per company** survive "matches a role family AND is US-based." That ratio is what caps your results. Adding employers scales output linearly; loosening the rubric just fills the report with jobs you cannot actually get.

**Bulk slug probing** (validates 100+ employers in a few minutes):

1. List 100–200 target employers in your field.
2. Auto-generate slug variants per name: lowercase-no-spaces, camelCase→hyphenated, plus `ai` / `hq` / `inc` suffixes.
3. Fire concurrent GETs for every variant against each ATS you care about (~24 workers). A non-empty jobs array means a hit.
4. Expect roughly a **two-thirds hit rate**. Record the winning slug *and* which ATS it belongs to, since date fields differ per ATS.
5. For misses, try the remaining endpoints (SmartRecruiters, Workable, Workday), then fall back to web search.

A 404 only means *that slug on that ATS* does not exist. It does **not** mean the employer is not hiring. Measured 2026-08-10: of 10 companies previously written off as dead, **7 were alive** under a different slug or a different ATS (Notion, Ramp, Perplexity, Glean, Cresta, Runway, HuggingFace; several with 90+ open roles each). Always try 2–3 variants across every ATS before recording a miss, or you will silently drop entire employers.

**While probing, record** the ATS per slug and how many role-family jobs each employer yields, so future runs can prioritize the productive ones.
