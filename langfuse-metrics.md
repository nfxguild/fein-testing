# NFX Langfuse metrics extraction

- **Project:** `internal-services` (org `nfx`), Langfuse Cloud US (`https://us.cloud.langfuse.com`)
- **Extracted:** 2026-11-17, about 22:50 UTC. All API calls were read-only `GET`s.
- **Data window:** 2026-08-19 22:58 UTC → 2026-11-17. This is the **entire retained history** (see Caveat 1). In this project, "last 90 days" and "since the earliest trace" cover the same data.
- **Environments:** about 23% of traces come from `staging`. Headline numbers are **production-only** unless marked *(all envs)*.

> ⚠️ **Read Caveat 1 before using any date.** The earliest trace is exactly 90 days old to within about 6 minutes. That points to a 90-day data-retention cutoff, not the date the rollout happened. The real rollout date is **NOT AVAILABLE** from this API. Prompt history shows Langfuse in use since at least **2026-04-28**.

---

## A. One-page summary: strongest numbers

Window for all rows: **2026-08-19 → 2026-11-17 (90 days, production)** unless stated. Query IDs (Q#) are spelled out in the [Query appendix](#query-appendix).

| # | Metric | Value | Date range | Source |
|---|---|---|---|---|
| 1 | Production AI runs (traces) | **282,576** traces; **277,240** LLM generations; **1.13B tokens** (1.07B in / 62.8M out) | 2026-08-19 → 2026-11-17 | Q4 (traces, env=production); Q5 (observations by model, env=production) |
| 2 | Breadth of AI surface | **35** production trace names: **22** LLM features with cost + **11** MCP tools (5,240 tool calls) | same | Q4 |
| 3 | Prompt governance coverage | **100%** of production LLM generations (277,241 / 277,240; the 1-row difference is from approximate counts) linked to a **managed, versioned prompt**. **34** managed prompts, **61** versions; **22** prompts served traffic. | prompts created 2026-04-28 → 2026-08-18; usage over the window | Q15, Q16 |
| 4 | Total LLM spend (production) | **$1,538.17** total; **$5.44 per 1,000 runs**; trailing 30 days **$402.45** on 52,133 runs | 90 days; trailing 30 = 2026-10-19 → 2026-11-17 | Q4, Q1b |
| 5 | Highest-volume feature cost efficiency | `founder-score`: **123,296** runs (43.6% of volume) at **$2.75 per 1,000 runs** on gpt-4.1-mini; 22.0% of spend | 90 days | Q4 |
| 6 | Reliability | **99.95%** of production traces had no ERROR-level observation (149 errored / 282,576). **99.96%** excluding the `agent:example-cron` demo job (102 / 282,529). | 90 days | Q7, Q4 |
| 7 | Cost concentration | `gpt-4o-2024-08-06` = **56.2% of spend** on 28.9% of generations. `summarize-comments` = **20.6% of spend** on 4.4% of runs ($25.64 per 1k). | 90 days | Q5, Q4 |
| 8 | Founder-score latency | p50 **4.54s → 2.88s (-37%)** and p95 **7.56s → 4.67s (-38%)**, Aug → Oct, at a flat ~$2.6–2.8 per 1k. **Cause unknown; see Caveat 6.** | Aug 2026 vs Oct 2026 | Q10 |
| 9 | AI-drafted pass emails | **65.4%** sent with no edits; mean normalized edit distance **0.156**; subject line changed **14.0%** (n = 136 each) | 90 days | Q12 |
| 10 | Peak throughput | Peak day **13,568** traces on 2026-08-25 *(all envs)*. Peak month **September: 107,783** production traces. | 2026-08-19 → 2026-11-17 | Q3, Q1 |

---

## B. Full tables

### 1. Rollout and volume

**Earliest trace (rollout date): NOT AVAILABLE as a rollout date.**
- The earliest retained trace is `2026-08-19T22:58:27.900Z` (`summarize-comments`), from Q2.
- The extraction ran at 2026-11-17 22:52 UTC, so that trace is 90 days old minus about 6 minutes. Days 1–2 already show 7,000–9,000 traces/day. Both facts fit a rolling retention cutoff, not a launch.
- **Evidence of earlier use:**
  - Earliest managed prompt version: `hello-world` v1, created 2026-04-28.
  - Earliest score config: `helpfulness`, created 2026-05-08.
  - Production prompts were migrated in June–July 2026 (commit messages read "Migrate from Cantina/LangSmith").

**Traces per month (Q1, v1 metrics `traces` view, month granularity):**

| Month | Days with data | Prod traces | Prod avg/day | Prod cost | Prod cost / 1k traces | All-env traces | All-env cost |
|---|---:|---:|---:|---:|---:|---:|---:|
| Aug 2026 (from 08-19 22:58 UTC) | 13 | 86,580 | 6,660 | $328.50 | $3.79 | 103,522 | $406.21 |
| Sep 2026 | 30 | 107,783 | 3,593 | $567.13 | $5.26 | 134,157 | $721.99 |
| Oct 2026 | 31 | 62,229 | 2,007 | $434.01 | $6.97 | 91,418 | $591.30 |
| Nov 2026 (1–17, MTD) | 17 | 25,984 | 1,528 | $208.53 | $8.03 | 37,750 | $273.22 |
| **Total** | 91 | **282,576** | | **$1,538.17** | **$5.44** | **366,847** | **$1,992.72** |

- **Peak month:** September 2026, with 107,783 prod traces (134,157 all envs). August's daily average is higher, but August is a partial month.
- **Peak day** *(all envs, Q3 daily endpoint)*: 2026-08-25, 13,568 traces.
- **Current monthly run rate (production):** 52,133 traces and $402.45 in the trailing 30 days (2026-10-19 → 2026-11-17).
  - The last 7 days ran 12,955 traces *(all envs)*.
- **Growth, first full month → latest full month (production):** Sep → Oct = **-42.3%** (all envs: -31.9%). **Volume has declined, not grown.**
  - Daily volume fell around 2026-09-07/08. Production averaged **7,430/day** from 2026-08-20 → 09-06 and **2,024/day** from 2026-09-08 → 11-17 (-73%).
  - Most of the drop is `founder-score`: 62,096 prod runs in Aug versus 8,167 in Oct. That pattern suggests a backfill or batch that finished. **[CONFIRM]**
- Cost per trace rose each month ($3.79 → $8.03 per 1k) because the mix shifted away from cheap, high-volume `founder-score` toward pricier features.

**Trace count by trace name, last 90 days, top 10, production (Q4):**

| # | Trace name | Traces | % share |
|---|---|---:|---:|
| 1 | `founder-score` | 123,296 | 43.6% |
| 2 | `headline-extractor` | 65,315 | 23.1% |
| 3 | `c3-team-grade-linkedin` | 30,513 | 10.8% |
| 4 | `company-relevance-checker` | 19,696 | 7.0% |
| 5 | `summarize-comments` | 12,351 | 4.4% |
| 6 | `auto-tug-comments` | 7,352 | 2.6% |
| 7 | `company-data-extractor` | 4,608 | 1.6% |
| 8 | `meeting-prep/meeting-categorization` | 4,205 | 1.5% |
| 9 | `company-referrer-extractor` | 2,384 | 0.8% |
| 10 | `mcp.tool.companies-structured-search` | 2,191 | 0.8% |
| | **Top 10 combined** | 271,911 | 96.2% |
| | **All production traces** | 282,576 | 100% |

**Same, all environments (Q4 without env filter):** `founder-score` 153,052 (41.7%), `headline-extractor` 68,368 (18.6%), `c3-team-grade-linkedin` 48,462 (13.2%), `auto-tug-comments` 24,810 (6.8%), `company-relevance-checker` 19,697 (5.4%), `summarize-comments` 13,232 (3.6%), `meeting-prep/meeting-categorization` 12,521 (3.4%), `company-data-extractor` 4,628 (1.3%), `meeting-prep/meeting-prep-followup` 4,508 (1.2%), `meeting-prep/meeting-prep-first` 4,132 (1.1%). Top 10 = 96.3% of 366,847.

**Staging share of selected features** *(Q4b, traces by name × environment)*:

| Feature | Staging traces | Production traces | Staging share |
|---|---:|---:|---:|
| `auto-tug-comments` | 17,458 | 7,352 | 70% |
| `meeting-prep/meeting-categorization` | 8,316 | 4,205 | 66% |
| `c3-team-grade-linkedin` | 17,949 | 30,513 | 37% |
| `founder-score` | 29,756 | 123,297 | 19% |

### 2. Prompt management

Source: Q15 (`GET /api/public/v2/prompts`, then `GET /api/public/v2/prompts/{name}?version=N` for every version) and Q16 (observations grouped by `promptName` × `promptVersion`).

- **Managed prompts:** 34
- **Total prompt versions:** 61. The listing returned 61 version entries. `draft-pass-email/with-style-v2-copy` lists versions 1 and 3; version 2 was likely deleted.
- **Prompts that served traffic in the window:** 22 of 34. Each served **only one version**, its current `production` version.
- **Prompts with 0 traffic in the window:** 12. These are: the spike/test prompts, `hello-world`, `agent-platform/*`, `lando/agent`, 4 `draft-pass-email/*-copy` variants, `pinecone-company-search-planner`, `summarize-attachment-pages`, and `whatsapp/whatsapp-intent-parser`.
- **Prompt-to-generation linkage:** 100% of LLM generations carry a `promptName`, in production and in all envs (361,312 / 361,312).
- **Migration evidence (commit messages):** 6 prompts have "Migrate … from Cantina/LangSmith" commit messages, dated 2026-06-25 → 2026-07-08:
  - `company-referrer-extractor`
  - `lightscribe-pitch-meeting`
  - `summarize-comments`
  - `company-relevance-checker`
  - `analyze-sender-style-profile`
  - `company-data-extractor`

**Per-prompt detail** (sorted by version count):

| Prompt | Type | Versions | First version | Latest version | Labels (version → labels) | Generations served in window (all envs) |
|---|---|---:|---|---|---|---:|
| `lando/scoring` | text | 4 | 2026-05-15 | 2026-05-26 | v4: latest, production | 366 (v4) |
| `draft-pass-email/from-granola-object-copy` | text | 3 | 2026-05-27 | 2026-07-09 | v3: latest, production | 0 |
| `draft-pass-email/from-granola-with-style-copy` | text | 3 | 2026-05-27 | 2026-07-09 | v3: latest, production | 0 |
| `draft-pass-email/with-style-v3` | text | 3 | 2026-08-18 | 2026-08-18 | v3: latest, production | 401 (v3) |
| `lando/agent` | text | 3 | 2026-05-15 | 2026-05-26 | v3: latest, production | 0 |
| `meeting-prep/meeting-prep-first` | chat | 3 | 2026-06-17 | 2026-06-20 | v3: latest, production | 4,129 (v3) |
| `spike/personalization-test` | text | 3 | 2026-07-23 | 2026-07-23 | v1: production; v3: latest, user-test-123 | 0 |
| `c3-team-grade-linkedin` | chat | 2 | 2026-05-25 | 2026-05-27 | v2: latest, production | 48,458 (v2) |
| `company-referrer-extractor` | text | 2 | 2026-06-25 | 2026-06-26 | v2: latest, production | 2,392 (v2) |
| `deck/datasheet-full` | chat | 2 | 2026-05-27 | 2026-06-03 | v2: latest, production | 1,217 (v2) |
| `deck/text-extraction` | chat | 2 | 2026-05-27 | 2026-06-01 | v2: latest, production | 1,193 (v2) |
| `draft-pass-email/from-company-comments-copy` | text | 2 | 2026-05-27 | 2026-07-09 | v2: latest, production | 14 (v2) |
| `draft-pass-email/with-style-v2-copy` | text | 2 | 2026-05-27 | 2026-07-09 | v3: latest, production | 0 |
| `founder-score` | text | 2 | 2026-06-29 | 2026-06-30 | v2: latest, production | 153,039 (v2) |
| `lightscribe` | text | 2 | 2026-07-01 | 2026-07-02 | v2: latest, production | 92 (v2) |
| `meeting-prep/meeting-categorization` | chat | 2 | 2026-06-17 | 2026-07-20 | v2: latest, production | 12,513 (v2) |
| `meeting-prep/meeting-prep-followup` | chat | 2 | 2026-06-17 | 2026-06-20 | v2: latest, production | 4,506 (v2) |
| `whatsapp/intent-parser` | chat | 2 | 2026-07-07 | 2026-07-08 | v2: latest, production | 14 (v2) |
| `whatsapp/whatsapp-intent-parser` | chat | 2 | 2026-07-04 | 2026-07-04 | v1: production; v2: latest | 0 |
| `agent-platform/example-slack` | text | 1 | 2026-06-02 | 2026-06-02 | v1: latest, production | 0 |
| `agent-platform/spanish-translator` | text | 1 | 2026-06-08 | 2026-06-08 | v1: latest, production | 0 |
| `ai_based_business_category` | text | 1 | 2026-07-09 | 2026-07-09 | v1: latest, production | 785 (v1) |
| `analyze-sender-style-profile` | text | 1 | 2026-07-07 | 2026-07-07 | v1: latest, production | 593 (v1) |
| `auto-tug-comments` | text | 1 | 2026-07-03 | 2026-07-03 | v1: latest, production | 24,809 (v1) |
| `company-data-extractor` | text | 1 | 2026-07-08 | 2026-07-08 | v1: latest, production | 4,623 (v1) |
| `company-relevance-checker` | text | 1 | 2026-07-07 | 2026-07-07 | v1: latest, production | 19,697 (v1) |
| `headline-extractor` | text | 1 | 2026-07-14 | 2026-07-14 | v1: latest, production | 68,368 (v1) |
| `hello-world` | chat | 1 | 2026-04-28 | 2026-04-28 | v1: latest, production | 0 |
| `lightscribe-pitch-meeting` | chat | 1 | 2026-06-30 | 2026-06-30 | v1: latest, production | 868 (v1) |
| `pinecone-company-search-planner` | text | 1 | 2026-06-26 | 2026-06-26 | v1: latest, production | 0 |
| `process-linkedin-html` | text | 1 | 2026-07-07 | 2026-07-07 | v1: latest, production | 4 (v1) |
| `spike/composition-test` | text | 1 | 2026-07-23 | 2026-07-23 | v1: latest, production | 0 |
| `summarize-attachment-pages` | text | 1 | 2026-07-28 | 2026-07-28 | v1: latest, production | 0 |
| `summarize-comments` | text | 1 | 2026-07-02 | 2026-07-02 | v1: latest, production | 13,231 (v1) |

**Prompts with the most versions (top 5):**
1. `lando/scoring`: 4 versions
2. Five prompts tie at 3 versions:
   - `draft-pass-email/from-granola-object-copy`
   - `draft-pass-email/from-granola-with-style-copy`
   - `draft-pass-email/with-style-v3`
   - `lando/agent`
   - `meeting-prep/meeting-prep-first`
3. `spike/personalization-test` also has 3 versions, but it is a test prompt.

### 3. Cost

Cost is Langfuse's computed cost from model price tables, not an invoice (Caveat 7).

**Total cost per month:** see the section 1 monthly table.
- Production: $328.50 / $567.13 / $434.01 / $208.53 (Nov MTD). Total **$1,538.17**.
- All envs: $406.21 / $721.99 / $591.30 / $273.22. Total **$1,992.72**.
- The all-env figure matches the daily endpoint (Q3) within $0.42 ($1,993.14).

**Cost per trace by trace name, top 10 by volume, last 90 days, production (Q4):**

| # | Trace name | Traces | Cost (USD) | % of prod cost | Cost / trace | Cost / 1k traces |
|---|---|---:|---:|---:|---:|---:|
| 1 | `founder-score` | 123,296 | $338.91 | 22.0% | $0.0027 | $2.75 |
| 2 | `headline-extractor` | 65,315 | $210.57 | 13.7% | $0.0032 | $3.22 |
| 3 | `c3-team-grade-linkedin` | 30,513 | $259.42 | 16.9% | $0.0085 | $8.50 |
| 4 | `company-relevance-checker` | 19,696 | $45.74 | 3.0% | $0.0023 | $2.32 |
| 5 | `summarize-comments` | 12,351 | $316.68 | 20.6% | $0.0256 | $25.64 |
| 6 | `auto-tug-comments` | 7,352 | $42.74 | 2.8% | $0.0058 | $5.81 |
| 7 | `company-data-extractor` | 4,608 | $80.90 | 5.3% | $0.0176 | $17.56 |
| 8 | `meeting-prep/meeting-categorization` | 4,205 | $15.61 | 1.0% | $0.0037 | $3.71 |
| 9 | `company-referrer-extractor` | 2,384 | $19.32 | 1.3% | $0.0081 | $8.10 |
| 10 | `mcp.tool.companies-structured-search` | 2,191 | $0.00 | 0.0% | n/a (no LLM cost) | n/a |

Not in the top 10 by volume but notable: `deck-datasheet-extraction` cost **$127.13** on 1,266 runs (**$100.42 per 1k**, 8.3% of prod spend).

**Cost by model, last 90 days, production, with token counts (Q5):**

| Model | Generations | % gens | Input tokens | Output tokens | Total tokens | Cost (USD) | % cost | Cost / 1k gens |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| `gpt-4.1-mini-2025-04-14` | 123,284 | 44.5% | 669,210,977 | 44,516,169 | 713,727,146 | $338.91 | 22.0% | $2.75 |
| `gpt-4o-2024-08-06` | 80,126 | 28.9% | 287,009,324 | 14,646,054 | 301,655,378 | $863.98 | 56.2% | $10.78 |
| `gpt-4o` | 65,397 | 23.6% | 92,936,387 | 1,163,260 | 94,099,647 | $243.97 | 15.9% | $3.73 |
| `claude-sonnet-4-6` | 7,449 | 2.7% | 13,028,579 | 2,130,323 | 15,158,902 | $71.04 | 4.6% | $9.54 |
| `claude-sonnet-4-5-20250929` | 527 | 0.2% | 4,391,921 | 206,885 | 4,598,806 | $16.28 | 1.1% | $30.89 |
| `gpt-4o-mini-2024-07-18` | 453 | 0.2% | 2,206,715 | 103,696 | 2,310,411 | $0.39 | 0.0% | $0.87 |
| `gpt-5.5-2026-04-23` | 4 | 0.0% | 686,464 | 5,322 | 691,786 | $3.59 | 0.2% | $898.00 |
| **Total** | **277,240** | | **1,069,470,367** | **62,771,709** | **1,132,242,076** | **$1,538.17** | | **$5.55** |

**All-env totals:** 361,312 generations; 1,330,263,249 input / 80,985,939 output / 1,411,249,188 total tokens; $1,992.72. `gpt-4o-2024-08-06` alone = $1,064.51 (53.4%).

Another 400,971 production observations (498,435 all envs) have no model. These are spans and events with zero cost.

**Cost per 1,000 traces, top 3 features by production volume:**

| Feature | Model | Cost / 1k traces |
|---|---|---:|
| `founder-score` | gpt-4.1-mini | **$2.75** |
| `headline-extractor` | gpt-4o (alias) | **$3.22** |
| `c3-team-grade-linkedin` | gpt-4o-2024-08-06 | **$8.50** |

### 4. Latency

Latency is trace duration (end − start). It includes any non-LLM work inside the trace. Percentiles come from Langfuse (ClickHouse quantiles) and are approximate (Caveat 9).

**p50 / p95 per top-10 trace name, last 90 days, production (Q4):**

| Trace name | p50 (s) | p95 (s) |
|---|---:|---:|
| `founder-score` | 4.20 | 7.11 |
| `headline-extractor` | 0.67 | 1.25 |
| `c3-team-grade-linkedin` | 1.17 | 2.51 |
| `company-relevance-checker` | 1.26 | 2.71 |
| `summarize-comments` | 2.86 | 7.67 |
| `auto-tug-comments` | 1.68 | 4.78 |
| `company-data-extractor` | 3.69 | 13.58 |
| `meeting-prep/meeting-categorization` | 1.38 | 7.78 |
| `company-referrer-extractor` | 1.74 | 3.63 |
| `mcp.tool.companies-structured-search` | 0.22 | 1.61 |

Other notable latencies (production):
- `meeting-prep/meeting-prep-followup`: p50 25.1s / p95 33.6s
- `meeting-prep/meeting-prep-first`: p50 8.9s / p95 15.7s
- `deck-datasheet-extraction`: p50 46.7s / p95 118.6s

**p50 / p95 trend by month for the highest-volume feature, `founder-score`, production (Q10):**

| Month | Traces | p50 (s) | p95 (s) | Cost / 1k traces |
|---|---:|---:|---:|---:|
| Aug 2026 (19–31) | 62,096 | 4.54 | 7.56 | $2.76 |
| Sep 2026 | 49,006 | 4.05 | 6.81 | $2.76 |
| Oct 2026 | 8,167 | 2.88 | 4.67 | $2.61 |
| Nov 2026 (1–17) | 4,028 | 3.08 | 5.23 | $2.69 |

All-env version: p50 4.58 / 4.15 / 2.85 / 3.00s; p95 7.77 / 6.70 / 4.56 / 4.98s.

**Project-wide monthly latency, production (Q11):**

| Month | p50 (s) | p95 (s) |
|---|---:|---:|
| Aug 2026 | 4.03 | 7.56 |
| Sep 2026 | 2.71 | 6.77 |
| Oct 2026 | 1.07 | 4.61 |
| Nov 2026 | 1.35 | 6.08 |

The project-wide drop mostly reflects the feature mix moving away from `founder-score`. Don't cite it as a speed-up.

### 5. Reliability

**Error definition:** a trace counts as errored if it contains at least 1 observation with `level = ERROR`.
- The "status message set" half of the requested definition is **NOT AVAILABLE**. `statusMessage` is not a filterable column in the metrics API (the API returned 400: "Invalid filter column statusMessage"). Counting it would mean paging through about 860K raw observations. These rates are therefore a **lower bound**.

**Error rate per month, production (Q8 errored traces ÷ Q1 traces):**

| Month | Errored traces | ERROR observations | Traces | Trace error rate |
|---|---:|---:|---:|---:|
| Aug 2026 (19–31) | 24 | 34 | 86,580 | 0.028% |
| Sep 2026 | 77 | 114 | 107,783 | 0.071% |
| Oct 2026 | 36 | 65 | 62,229 | 0.058% |
| Nov 2026 (1–17) | 12 | 20 | 25,984 | 0.046% |
| **Total** | **149** | **233** | **282,576** | **0.053%** |

All envs: 30 / 86 / 42 / 13 errored traces. Total 171 of 366,847 (0.047%). Staging contributed 22 of them.

**Error rate by trace name, production, all features with ≥1 error (Q7):**

| Trace name | Errored traces | ERROR obs | Traces | Error rate |
|---|---:|---:|---:|---:|
| `agent:example-cron` | 47 | 47 | 47 | 100.00% |
| `deck-datasheet-extraction` | 51 | 100 | 1,266 | 4.03% |
| `draft-pass-email/draft-pass-email-generator-from-company-comments-styled` | 6 | 12 | 395 | 1.52% |
| *(unnamed / null trace name)* | 5 | 5 | 561 | 0.89% |
| `mcp.tool.deck-get-deck-datasheet` | 2 | 2 | 236 | 0.85% |
| `company-referrer-extractor` | 7 | 14 | 2,384 | 0.29% |
| `lando.score` | 1 | 2 | 367 | 0.27% |
| `lightscribe-pitch-meeting` | 1 | 2 | 528 | 0.19% |
| `company-data-extractor` | 5 | 10 | 4,608 | 0.11% |
| `meeting-prep/meeting-categorization` | 3 | 3 | 4,205 | 0.07% |
| `meeting-prep/meeting-prep-first` | 1 | 1 | 1,465 | 0.07% |
| `mcp.tool.companies-structured-search` | 1 | 1 | 2,191 | 0.05% |
| `auto-tug-comments` | 1 | 2 | 7,352 | 0.01% |
| `c3-team-grade-linkedin` | 4 | 4 | 30,513 | 0.01% |
| `founder-score` | 13 | 26 | 123,296 | 0.01% |
| `summarize-comments` | 1 | 2 | 12,351 | 0.01% |

**Top 5 worst, excluding the demo cron and unnamed traces:**
1. `deck-datasheet-extraction`: 4.03%
2. `draft-pass-email/…-styled`: 1.52%
3. `mcp.tool.deck-get-deck-datasheet`: 0.85%
4. `company-referrer-extractor`: 0.29%
5. `lando.score`: 0.27%

`agent:example-cron` errors on every run (47/47). It looks like an example or demo job **[CONFIRM]**.

### 6. Model changes

**Every model used, per trace name, with first and last seen dates** (Q9, observations, day granularity, all envs).

Dates of 2026-08-19/20 are the retention edge, and 2026-11-17 is the extraction date. Both are therefore *censored*: actual usage may extend beyond them.

| Trace name | Model | First seen | Last seen | Generations | Cost (USD) |
|---|---|---|---|---:|---:|
| `ai_based_business_category` | `gpt-4o-2024-08-06` | 2026-08-20 | 2026-11-17 | 785 | $2.33 |
| `analyze-sender-style-profile` | `gpt-4o-2024-08-06` | 2026-08-23 | 2026-11-15 | 593 | $13.97 |
| `auto-tug-comments` | `gpt-4o-2024-08-06` | 2026-08-20 | 2026-11-17 | 24,809 | $126.34 |
| `c3-team-grade-linkedin` | `gpt-4o-2024-08-06` | 2026-08-20 | 2026-11-17 | 48,458 | $345.50 |
| `company-data-extractor` | `gpt-4o-2024-08-06` | 2026-08-20 | 2026-11-17 | 4,623 | $81.04 |
| `company-referrer-extractor` | `gpt-4o-2024-08-06` | 2026-08-20 | 2026-11-17 | 2,392 | $19.41 |
| `company-relevance-checker` | `gpt-4o-2024-08-06` | 2026-08-20 | 2026-11-17 | 19,697 | $45.74 |
| `deck-datasheet-extraction` | `gpt-4o-2024-08-06` | 2026-08-20 | 2026-11-17 | 2,406 | $127.13 |
| `draft-pass-email/draft-pass-email-generator-from-company-comments` | `gpt-4o-mini-2024-07-18` | 2026-08-20 | 2026-10-05 | 14 | $0.05 |
| `draft-pass-email/draft-pass-email-generator-from-company-comments-styled` | `gpt-4o-mini-2024-07-18` | 2026-08-20 | 2026-11-17 | 401 | $0.34 |
| `founder-score` | `gpt-4.1-mini-2025-04-14` | 2026-08-19 | 2026-11-17 | 153,034 | $418.11 |
| `headline-extractor` | `gpt-4o` | 2026-08-19 | 2026-11-16 | 68,368 | $223.41 |
| `lando.score` | `claude-sonnet-4-6` | 2026-08-20 | 2026-10-01 | 366 | $1.94 |
| `lightscribe` | `gpt-4o-mini-2024-07-18` | 2026-08-20 | 2026-11-09 | 92 | $0.11 |
| `lightscribe-pitch-meeting` | `claude-sonnet-4-5-20250929` | 2026-08-20 | 2026-11-17 | 868 | $28.43 |
| `meeting-prep/meeting-categorization` | `claude-sonnet-4-6` | 2026-08-20 | 2026-11-17 | 12,513 | $46.91 |
| `meeting-prep/meeting-prep-first` | `claude-sonnet-4-6` | 2026-08-20 | 2026-11-17 | 4,129 | $41.45 |
| `meeting-prep/meeting-prep-followup` | `claude-sonnet-4-6` | 2026-08-20 | 2026-11-17 | 4,506 | $129.22 |
| `process-linkedin-html` | `gpt-5.5-2026-04-23` | 2026-08-21 | 2026-09-03 | 4 | $3.59 |
| `summarize-comments` | `gpt-4o-2024-08-06` | 2026-08-19 | 2026-11-17 | 13,146 | $302.87 |
| `summarize-comments` | `gpt-4o` | 2026-08-20 | 2026-11-17 | 85 | $34.63 |
| `whatsapp/intent-parser` | `gpt-4o-mini-2024-07-18` | 2026-08-25 | 2026-09-14 | 14 | $0.00 |
| *(null trace name)* | `gpt-4o-2024-08-06` | 2026-08-20 | 2026-09-24 | 4 | $0.19 |
| *(null trace name)* | `gpt-4.1-mini-2025-04-14` | 2026-08-21 | 2026-08-26 | 5 | $0.01 |

**Model switches: none in the retained window. The 30-day before/after comparison is NOT AVAILABLE.**
- Every feature used a single model for the whole window, with one exception: `summarize-comments`.
- `summarize-comments` ran `gpt-4o-2024-08-06` and the `gpt-4o` alias **side by side** every month:
  - Aug: 1,504 vs 8
  - Sep: 5,108 vs 28
  - Oct: 4,188 vs 32
  - Nov: 2,346 vs 17
- That is a parallel path, not a switch. The 85 `gpt-4o` alias calls cost $0.41 each versus $0.023 for the pinned model, which points to much larger inputs on that path **[CONFIRM]**.
- Prompt configs (Q15) record the same model in every version of each prompt. For example, `founder-score` v1 and v2 both specify `gpt-4.1-mini`. Prompt history therefore shows no switch before the window either.

### 7. Scores and feedback

**All score names** (Q12; v2 `scores-numeric`, `scores-categorical` and `scores-boolean` views; no date filter beyond retention):

| Score name | Data type | Source | Count | Average | Attached to |
|---|---|---|---:|---:|---|
| `draft_pass_email_sent_unedited` | boolean | API | 136 | 0.654 (65.4% true) | draft-pass-email traces (134 styled, 2 unstyled) |
| `draft_pass_email_edit_distance` | numeric | API | 136 | 0.156 | same |
| `draft_pass_email_subject_changed` | boolean | API | 136 | 0.140 (14.0% true) | same |
| `draft_pass_email_time_to_send_seconds` | numeric | API | 136 | **0** (max 0; looks broken, see Caveat 10) | same |
| `draft_pass_email_thumbs` | categorical | API | 2 | n/a (2 × "down") | same |
| **Total** | | | **546** | | All production environment |

- Sources: **100% API**. There are **0** `ANNOTATION` scores and **0** `EVAL` (LLM-as-judge) scores.
- **Score configs defined but unused in the window** (`GET /api/public/score-configs`):
  - `lando.human_review`: categorical pass/pursue, created 2026-05-20
  - `helpfulness`: boolean, created 2026-05-08
  - Both have 0 values in the retained window.

**Monthly trend for scores with 50+ values (Q13):**

| Month | n | `sent_unedited` avg | `edit_distance` avg | `subject_changed` avg | `time_to_send_seconds` avg |
|---|---:|---:|---:|---:|---:|
| Aug 2026 (19–31) | 26 | 0.731 | 0.117 | 0.115 | 0 |
| Sep 2026 | 43 | 0.628 | 0.190 | 0.302 | 0 |
| Oct 2026 | 58 | 0.672 | 0.130 | 0.052 | 0 |
| Nov 2026 (1–17) | 9 | 0.444 | 0.267 | 0.000 | 0 |

The November sample (n = 9) is too small to support a trend claim.

**Human annotations: 0.**
- No `ANNOTATION`-source scores exist (`GET /api/public/scores?source=ANNOTATION`, totalItems = 0).
- No annotation queues exist (`GET /api/public/annotation-queues`, totalItems = 0).
- `draft_pass_email_*` scores are implicit human feedback: they record whether a user edited or sent the draft. They were logged via the API, not through annotation.

### 8. Prompt-change regressions

**NOT AVAILABLE. No production prompt version was created inside the retained trace window, so no 14-day before/after comparison exists.**
- All 61 prompt versions were created between 2026-04-28 and 2026-08-18. The newest is `draft-pass-email/with-style-v3` v3, created 2026-08-18 20:37 UTC.
- Retained trace data starts 2026-08-19 22:58 UTC, about 26 hours after that newest version. No "before" period is retained for any version change.
- Within the window, every prompt served exactly one version (Q16). There were no promotions or rollbacks to compare.
- **Flags:** none can be raised (no error-rate, latency or score regression is computable). Reversals: not applicable.
- Reconstructing this would require trace data from before 2026-08-19, for example from a Langfuse export, a blob-storage export or a higher retention tier, if any of those exist **[CONFIRM]**.

---

## C. Caveats

1. **Retention truncation, the biggest caveat.**
   - The earliest trace (2026-08-19 22:58 UTC) is 90 days before extraction to within about 6 minutes. That is consistent with a 90-day retention policy on this Langfuse plan.
   - "Rollout date", "growth since rollout" and anything before 2026-08-19 are therefore **NOT AVAILABLE**.
   - Prompts and score configs are not purged by retention, and they date use back to at least 2026-04-28.
   - If you re-run this later, the window will slide and the figures will change.
2. **August is partial and likely includes a batch or backfill.** Production volume fell 73% around 2026-09-07/08 (7,430 → 2,024 traces/day), mostly in `founder-score`. Any "growth" bullet would be negative. "Peak" numbers reflect that batch period. **[CONFIRM what ran in Aug–early Sep]**
3. **Staging is mixed in.** 84,358 of 366,847 traces (23%) and $454.55 of $1,992.72 (22.8%) are `staging`.
   - Headline numbers use `environment = production`.
   - The daily endpoint (Q3) cannot filter by environment, so peak day and last-7-day figures are all-env.
   - Some features are mostly staging traffic: `auto-tug-comments` 70%, `meeting-categorization` 66%.
4. **Deprecated endpoints.**
   - `/api/public/metrics/daily`, `/api/public/metrics` (v1), `/api/public/traces` and `/api/public/scores` all returned a notice saying they were removed on 2026-11-16. They still answered on 2026-11-17, but they may stop at any time.
   - The v2 metrics API has no `traces` view. Trace counts, trace cost and trace latency therefore depend on v1. Observation, model, error and score figures use v2.
   - **Reproducing the trace-level numbers later may not be possible.**
5. **Observation counts disagree between endpoints.** The daily endpoint reports 601,918 observations; v2 metrics reports about 860K, including spans and events with no model. I avoided raw "observation" counts and used traces and LLM generations (models non-null) instead.
6. **The founder-score latency drop has no recorded cause.**
   - p50 fell 37% from Aug to Oct, but no prompt version or model change happened in the window.
   - The drop coincides with a roughly 87% fall in that feature's volume, so it could reflect lower concurrency or rate-limit pressure, or provider-side speedups, rather than an engineering change.
   - Do not claim it as an optimization without context **[CONFIRM]**.
7. **Cost is Langfuse-computed, not billed.** It uses Langfuse's model price tables, which matters for the unpinned `gpt-4o` alias. It has not been reconciled with OpenAI or Anthropic invoices. MCP tool traces and non-LLM spans carry $0.
8. **Error rate is a lower bound.**
   - Only `level = ERROR` observations count. Status-message-only failures could not be filtered via aggregates.
   - Application failures never logged to Langfuse are invisible.
   - Errored-trace counts use `uniq(traceId)`, which is approximate (HyperLogLog-style; for example, 154,022 unique traces vs 153,039 observations for one prompt). At these small counts the error is negligible but non-zero.
9. **Percentiles are approximate.** Two identical runs of the founder-score p95 query returned 7.01s and 7.09s. Treat latency to about ±0.1s.
10. **Score data is thin.**
    - Only 136 scored events, all on one feature (`draft-pass-email`).
    - `draft_pass_email_time_to_send_seconds` is 0 on every record, which looks like an instrumentation bug. Don't use it.
    - There are only 2 thumbs ratings, both "down".
    - There are no human annotations and no automated evals.
11. **`userId` is not a human user.** `uniqueUserIds` = 455,943, which exceeds the trace count, so `userId` holds entity IDs (companies or founders), not people. No "users served" metric can be derived.
12. **Unnamed traces.** 561 production traces (575 all-env) have an empty or null name. They are included in totals and excluded from per-feature tables.
13. **Censored model dates.** "First seen" is bounded by retention and "last seen" by the extraction date. Neither marks a real start or end of a model's use.
14. **Prompt version count:** `draft-pass-email/with-style-v2-copy` lists versions [1, 3], so the 61-version total excludes 1 apparently deleted version.

---

## D. Draft résumé bullets

Ordered by business impact. Every number comes from this extraction.

1. **Built and scaled NFX's production LLM platform** on Langfuse, serving **282,576 production AI runs** (277,240 model calls, 1.13B tokens) across **22 LLM-powered features and 11 MCP tools** in 90 days (Aug–Nov 2026). **[CONFIRM: your ownership scope; data covers only the 90-day retention window, and the platform ran from at least Apr 2026]**
2. **Put 100% of production LLM calls under versioned prompt management**: **34 managed prompts / 61 versions**, including migrating **6 production prompts from Cantina/LangSmith** in Jun–Jul 2026. Every one of 277,240 generations is traceable to a specific prompt version. **[CONFIRM: you led the migration]**
3. **Ran production AI at $5.44 per 1,000 runs** ($1,538 total over 90 days, Aug–Nov 2026). The highest-volume workload, founder scoring (123K runs, 44% of volume), ran on gpt-4.1-mini at **$2.75 per 1,000 runs**. **[CONFIRM: you chose the model or cost design; costs are Langfuse-computed, not invoiced]**
4. **Kept 99.95% of 282,576 production AI runs error-free** over 90 days (149 errored traces; 99.96% excluding a demo job), across 22 LLM features (Aug–Nov 2026). **[CONFIRM: count is limited to ERROR-level logs]**
5. **Shipped AI-drafted pass emails that users sent unedited 65% of the time** (n = 136; mean edit distance 0.16; subject changed 14%), with acceptance tracked automatically through Langfuse feedback scores (Aug–Nov 2026). **[CONFIRM: small sample; describe the feature's purpose]**

*Not recommended without context:* the founder-score latency reduction (p50 -37%, p95 -38%, Aug → Oct 2026 at flat cost). No engineering change is recorded, and it coincides with a volume drop (Caveat 6).

---

## Query appendix

Base URL: `https://us.cloud.langfuse.com`, Basic auth with the public and secret key.

- `F` = `2026-08-19T00:00:00Z`; some queries used `2020-01-01` or `2026-01-01`, which returns the same data because of retention.
- `T` = `2026-11-18T00:00:00Z`.
- `PROD` = `{"column":"environment","operator":"=","value":"production","type":"string"}`.
- All metrics queries are sent as `?query=<urlencoded JSON>`.

| ID | Endpoint | Query body / params |
|---|---|---|
| Q1 | `GET /api/public/metrics` (v1) | `{"view":"traces","dimensions":[],"metrics":[{"measure":"count","aggregation":"count"},{"measure":"totalCost","aggregation":"sum"},{"measure":"latency","aggregation":"p50"},{"measure":"latency","aggregation":"p95"}],"filters":[PROD],"timeDimension":{"granularity":"month"},"fromTimestamp":F,"toTimestamp":T}` (all-env: `filters:[]`) |
| Q1b | `GET /api/public/metrics` (v1) | Same as Q1 with no timeDimension, `fromTimestamp:"2026-10-19T00:00:00Z"`. The same pattern for 08-20→09-07 and 09-08→11-18 gave the daily-average comparison. |
| Q2 | `GET /api/public/traces` | `orderBy=timestamp.asc&limit=1&fields=core` |
| Q3 | `GET /api/public/metrics/daily` | `page=1&limit=100` (91 days returned) |
| Q4 | `GET /api/public/metrics` (v1) | `{"view":"traces","dimensions":[{"field":"name"}],"metrics":[count/count, totalCost/sum, latency/p50, latency/p95],"filters":[PROD],"fromTimestamp":F,"toTimestamp":T,"orderBy":[{"field":"count_count","direction":"desc"}],"config":{"row_limit":100}}` |
| Q4b | `GET /api/public/metrics` (v1) | Q4 with `dimensions:[{"field":"name"},{"field":"environment"}]`, `filters:[]` |
| Q5 | `GET /api/public/v2/metrics` | `{"view":"observations","dimensions":[{"field":"providedModelName"}],"metrics":[count/count, totalCost/sum, inputTokens/sum, outputTokens/sum, totalTokens/sum],"filters":[PROD],"fromTimestamp":F,"toTimestamp":T}` |
| Q6 | `GET /api/public/v2/metrics` | `{"view":"observations","dimensions":[{"field":"environment"}],"metrics":[{"measure":"traceId","aggregation":"uniq"},{"measure":"totalCost","aggregation":"sum"}],...}` |
| Q7 | `GET /api/public/v2/metrics` | `{"view":"observations","dimensions":[{"field":"traceName"}],"metrics":[{"measure":"traceId","aggregation":"uniq"},{"measure":"count","aggregation":"count"}],"filters":[PROD,{"column":"level","operator":"=","value":"ERROR","type":"string"}],...}` |
| Q8 | `GET /api/public/v2/metrics` | Q7 with `dimensions:[]`, `timeDimension:{"granularity":"month"}` |
| Q9 | `GET /api/public/v2/metrics` | `{"view":"observations","dimensions":[{"field":"traceName"},{"field":"providedModelName"}],"metrics":[count/count,totalCost/sum],"filters":[{"column":"providedModelName","operator":"=","value":<model>,"type":"string"}],"timeDimension":{"granularity":"day"},"config":{"row_limit":1000}}`, run once per model |
| Q10 | `GET /api/public/metrics` (v1) | Q1 with an added filter `{"column":"name","operator":"=","value":"founder-score","type":"string"}` |
| Q11 | `GET /api/public/metrics` (v1) | Q1 (latency p50/p95 columns) |
| Q12 | `GET /api/public/v2/metrics` | `{"view":"scores-numeric" \| "scores-boolean" \| "scores-categorical","dimensions":[{"field":"name"},{"field":"source"}],"metrics":[count/count, value/avg]}`; `[{"field":"name"},{"field":"traceName"}]` and `[{"field":"name"},{"field":"environment"}]` variants |
| Q13 | `GET /api/public/v2/metrics` | `{"view":"scores-numeric","dimensions":[{"field":"name"}],"metrics":[count/count,value/avg],"timeDimension":{"granularity":"month"}}` |
| Q15 | `GET /api/public/v2/prompts` then `GET /api/public/v2/prompts/{name}?version={n}` | 34 prompts, 61 version fetches |
| Q16 | `GET /api/public/v2/metrics` | `{"view":"observations","dimensions":[{"field":"promptName"},{"field":"promptVersion"}],"metrics":[count/count, traceId/uniq]}` (also the PROD + `providedModelName is not null` variant) |
| Q17 | `GET /api/public/score-configs`, `GET /api/public/annotation-queues`, `GET /api/public/scores?source=ANNOTATION&limit=1` | totals only |
