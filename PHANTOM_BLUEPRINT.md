# PHANTOM: Master Technical Blueprint

## Context
The $15B competitive intelligence market is dominated by manual processes and reactive tools (Crayon, Klue, Kompyte) that surface changes *after* they happen. PHANTOM flips this — it continuously collects weak signals from non-obvious sources (job postings, GitHub, patents, pricing pages, domains) and uses LLM reasoning to *predict* competitor moves 60-90 days before announcement. This is a hackathon project designed for a compelling 5-minute demo with a path to production SaaS.

---

## SECTION 1: PROBLEM DEFINITION & VISION

### The Market Gap
- **Current CI tools** (Crayon ~$25K/yr, Klue ~$30K/yr, Kompyte ~$20K/yr): Monitor websites for changes, aggregate news, provide battlecards. All are **reactive** — they tell you what happened, not what's coming.
- **Enterprise CI teams**: Analysts manually trawl LinkedIn, patent databases, earnings calls. Costs $200K+/yr per analyst. Coverage is spotty.
- **The insight**: Weak signals from public sources are **leading indicators**. When a competitor posts 15 ML engineer roles in 3 weeks, files a patent for "real-time recommendation system," and registers `competitor-ai-studio.com` — they're launching an AI product. PHANTOM detects this pattern automatically.

### Vision
Every company has a silent satellite watching every competitor, 24/7. Not scraping their homepage — reading the tea leaves of their hiring, engineering, IP, and pricing activity. Then predicting the play before it's announced.

---

## SECTION 2: CORE TECHNICAL ARCHITECTURE

### 2.1 System Pipeline
```
Signal Sources → Collectors → Normalization → Signal Store → Pattern Engine → Prediction Engine → Alert System → Dashboard/Briefs
       |              |              |              |               |                |               |
   [10+ sources] [Cloud Run]  [Unified Schema] [PostgreSQL]  [Gemini 1.5]   [Scoring Model]  [Next.js + Slack]
                  [Scheduled]                   [+ Redis]     [+ Rules]     [+ Evidence]      [+ Email]
```

### 2.2 Signal Collection Network

| Source | Method | Signals Extracted | Update Frequency |
|--------|--------|-------------------|------------------|
| **Job Boards** (LinkedIn, Indeed, Glassdoor, Greenhouse, Lever) | RapidAPI aggregators (JSearch, Indeed API) + direct Greenhouse/Lever JSON APIs | Tech stack, team growth, geo expansion, product direction, M&A prep | Every 6 hours |
| **GitHub** | GitHub REST/GraphQL API | New repos, commit velocity, contributor changes, dependency additions, language shifts | Every 4 hours |
| **Patents** | USPTO PatentsView API, EPO OPS REST API | New filings, CPC classifications, inventor networks, claim analysis | Daily |
| **Pricing Pages** | Puppeteer headless screenshots + pixel diff + text extraction | Price changes, tier restructuring, feature additions/removals | Every 12 hours |
| **News & Press** | Google Custom Search JSON API + NewsAPI | Product launches, partnerships, funding, leadership changes | Every 2 hours |
| **Google Trends** | SerpAPI Google Trends endpoint | Brand search volume, product term trends, geographic interest shifts | Daily |
| **App Stores** | App Store Connect API / Google Play Scraper | Update notes, rating changes, review sentiment shifts | Daily |
| **Domain Registration** | WhoisXML API | New domains suggesting product launches, brand pivots | Daily |
| **Social/Blogs** | RSS feeds + Google Custom Search | Engineering blog posts, tech talks, conference presentations | Every 6 hours |
| **SEC/Regulatory** | SEC EDGAR XBRL API | 10-K/10-Q filings, risk factor changes, segment revenue shifts | On filing |

### 2.3 Signal Normalization Layer

**Unified Signal Schema:**
```typescript
interface Signal {
  id: string;                    // UUID v4
  competitor_id: string;         // FK to competitors table
  source: SignalSource;          // enum: JOB_BOARD | GITHUB | PATENT | PRICING | NEWS | TRENDS | APP_STORE | DOMAIN | SOCIAL | SEC
  source_url: string;            // Original source URL
  signal_type: SignalType;       // enum: HIRING_SURGE | TECH_ADOPTION | PATENT_FILING | PRICE_CHANGE | PRODUCT_LAUNCH | GEO_EXPANSION | LEADERSHIP_CHANGE | PARTNERSHIP | FUNDING | ACQUISITION_SIGNAL
  title: string;                 // Human-readable signal title
  raw_data: JSON;                // Original collected data
  extracted_insights: string[];  // Gemini-extracted key insights
  confidence: number;            // 0.0 - 1.0
  relevance_score: number;       // 0.0 - 1.0 (to user's company)
  tags: string[];                // Taxonomy tags
  timestamp: Date;               // When signal was detected
  source_timestamp: Date;        // When the original event occurred
  fingerprint: string;           // SHA256 for deduplication
}
```

**Normalization Pipeline:**
1. Raw data ingestion from collector
2. Deduplication via fingerprint (SHA256 of source_url + key content fields)
3. Gemini Flash classification → signal_type + tags
4. Confidence scoring (source reliability * content specificity * recency)
5. Relevance scoring (semantic similarity to user's tracked competitive dimensions)
6. Store in PostgreSQL with full-text search index

### 2.4 Threat Intelligence Engine

**How Gemini reasons about signal combinations:**

```
Input: Signal cluster for Competitor X over past 30 days
  - 12 new ML engineer job postings (3x normal rate)
  - New GitHub repo: "competitor-x/recommendation-engine" (Python, TensorFlow)
  - Patent filed: "System for real-time personalized content ranking"
  - New domain registered: "compx-ai.com"
  - Pricing page: New "AI-Powered" tier added (empty/coming soon)

Gemini Reasoning Chain:
  1. Hiring surge in ML → building AI team (confidence: 0.85)
  2. GitHub repo confirms recommendation system development (confidence: 0.90)
  3. Patent filing confirms strategic investment, not experiment (confidence: 0.88)
  4. Domain registration → preparing public launch (confidence: 0.75)
  5. Pricing page → monetization planned within 60 days (confidence: 0.80)

Combined Prediction:
  "Competitor X will launch an AI-powered recommendation/personalization product
   within 45-75 days, likely as a premium tier. Confidence: 0.87"
  Evidence: [links to all 5 signals]
```

**The Move Prediction Model:**
- Signal clusters are grouped by competitor + time window (rolling 30/60/90 days)
- Gemini 1.5 Pro receives the cluster + historical patterns + industry context
- Outputs: predicted_action, timeframe, confidence, evidence_chain, recommended_response
- Each prediction is stored with full provenance for accuracy tracking

### 2.5 Alert Tiering System

| Tier | Criteria | Response Time | Channel |
|------|----------|---------------|---------|
| **Tier 1: CRITICAL** | Competitor entering your core market, major product launch imminent, pricing undercut detected | Immediate | Slack DM + Email + SMS + Dashboard banner |
| **Tier 2: WATCH** | Capability buildup detected, 60-90 day horizon, hiring patterns suggest new initiative | Within 4 hours | Slack channel + Email digest + Dashboard card |
| **Tier 3: MONITOR** | Ongoing awareness signals, minor changes, background intelligence | Daily digest | Email weekly brief + Dashboard feed |

**Tier assignment logic:**
```
tier = 1 if (prediction.confidence > 0.80 AND prediction.timeframe < 30 days AND prediction.impacts_core_market)
tier = 2 if (prediction.confidence > 0.60 AND prediction.timeframe < 90 days)
tier = 3 otherwise
```

### 2.6 Weekly Intelligence Brief Generation

Gemini 1.5 Pro generates an analyst-quality PDF report every Monday at 6am:

**Brief Structure:**
1. **Executive Summary** — Top 3 competitive developments this week
2. **Threat Dashboard** — Current predictions with confidence levels
3. **Signal Highlights** — Most significant new signals per competitor
4. **Trend Analysis** — How signal patterns are evolving over time
5. **Recommended Actions** — What the user should consider doing
6. **Appendix** — Full signal log for the week

Generated as Markdown → rendered to PDF via Puppeteer.

### 2.7 Google Search API + Vertex AI Grounding

- **Google Custom Search JSON API**: Real-time web search for competitor mentions, product announcements, press releases. 100 free queries/day, $5/1000 after.
- **Vertex AI Grounding**: Grounds Gemini's predictions in verifiable web sources. Prevents hallucinated predictions. Each prediction must cite real, retrievable evidence.

---

## SECTION 3: GOOGLE API INTEGRATION PLAN

| API | Purpose | Config | Pricing |
|-----|---------|--------|---------|
| **Gemini 1.5 Pro** | Strategic reasoning, prediction generation, weekly briefs | `gemini-1.5-pro-latest`, temp 0.3, max_tokens 8192 | $3.50/M input, $10.50/M output |
| **Gemini 1.5 Flash** | High-speed signal classification, tagging, dedup | `gemini-1.5-flash-latest`, temp 0.1, max_tokens 1024 | $0.075/M input, $0.30/M output |
| **Google Custom Search JSON API** | Real-time news/web monitoring | Programmable Search Engine ID + API key | 100 free/day, $5/1K queries |
| **Vertex AI (Grounding)** | Fact-checking predictions against web sources | `google-search` grounding tool in Gemini API | Included with Gemini calls |
| **Google Trends (via SerpAPI)** | Search volume tracking for competitor brands/products | SerpAPI key, `engine=google_trends` | $50/mo (5000 searches) |
| **BigQuery** | Signal storage, trend analysis, historical queries | Dataset: `phantom_signals`, partitioned by `timestamp` | 1TB free/mo query, 10GB free storage |
| **Cloud Pub/Sub** | Signal ingestion queue, event-driven processing | Topic: `raw-signals`, Sub: `signal-processor` | 10GB free/mo |
| **Cloud Scheduler** | Cron jobs for all collectors | Jobs per collector with appropriate intervals | 3 free jobs, $0.10/job/mo |
| **Cloud Run** | Signal collector workers, API backend | Min instances: 0, Max: 10, 1 vCPU, 512MB | 2M free requests/mo |
| **Firebase (FCM)** | Push notification alerts | Project linked to GCP | Free tier sufficient |
| **Looker Studio** | Executive CI dashboards (optional) | Connected to BigQuery | Free |

**Auth Setup:**
```bash
# Service account with roles:
# - roles/aiplatform.user (Vertex AI / Gemini)
# - roles/bigquery.dataEditor
# - roles/pubsub.publisher + subscriber
# - roles/run.invoker
# - roles/cloudscheduler.admin
gcloud iam service-accounts create phantom-sa --display-name="PHANTOM Service Account"
```

---

## SECTION 4: JOB POSTING INTELLIGENCE LAYER

### 4.1 What Job Postings Reveal

| Signal Pattern | Intelligence Extracted |
|---------------|----------------------|
| 15+ ML/AI roles in 30 days | AI product development, 60-90 day launch horizon |
| "Kubernetes", "Terraform" appearing in DevOps roles | Infrastructure modernization / cloud migration |
| Sales roles in new geographies | Geographic expansion planned |
| "M&A Integration", "Due Diligence" in finance roles | Acquisition activity |
| Senior PM roles mentioning specific product areas | New product line development |
| "Series B", "IPO readiness" in finance roles | Funding/IPO preparation |
| Sudden 3x hiring rate across all functions | Major growth initiative or funding received |

### 4.2 Job Posting Analysis Prompt

```
You are a competitive intelligence analyst. Analyze this job posting and extract:

1. TECHNOLOGY SIGNALS: What technologies, frameworks, tools are mentioned?
   Rate each as: [CURRENT_STACK] or [NEW_ADOPTION] based on language cues.

2. PRODUCT SIGNALS: What product areas does this role support?
   What can you infer about upcoming features or products?

3. TEAM SIGNALS: Is this a new team/function or backfill?
   What seniority level suggests about initiative maturity?

4. STRATEGIC SIGNALS: Geographic expansion? New market entry?
   Partnership indicators? M&A preparation language?

Output as JSON with confidence scores for each signal.
```

### 4.3 Technology Stack Inference
- Parse "Requirements" and "Nice to have" sections separately
- "Requirements" = current stack; "Nice to have" = planned adoption
- Track tech mentions over time → detect stack migrations (e.g., Java → Go transition)

### 4.4 Team Growth Signal Detection
- Baseline: average job postings per month per function over past 6 months
- Alert when current month > 2x baseline for any function
- Track ratio of senior:junior roles → mature team vs. building from scratch

### 4.5 Hiring Surge Detector
```python
# Statistical anomaly detection
z_score = (current_month_postings - rolling_6mo_mean) / rolling_6mo_std
if z_score > 2.0:
    signal_type = "HIRING_SURGE"
    confidence = min(0.95, 0.5 + (z_score - 2.0) * 0.15)
```

---

## SECTION 5: PREDICTION ENGINE

### 5.1 Signal Taxonomy → Predicted Action Mapping

| Signal Cluster | Predicted Action | Typical Lead Time |
|---------------|-----------------|-------------------|
| ML hiring surge + AI patent + new AI repo | AI product launch | 60-90 days |
| Sales roles in new geo + domain registration | Geographic expansion | 45-60 days |
| Pricing page changes + new tier added | Pricing restructure | 15-30 days |
| M&A language in roles + due diligence hiring | Acquisition activity | 90-180 days |
| Security/compliance hiring surge | Enterprise market push | 60-120 days |
| Mobile dev hiring + app store activity | Mobile product launch | 45-75 days |
| Infrastructure/DevOps surge + cloud tech mentions | Platform migration/rewrite | 90-180 days |

### 5.2 Evidence Quality Weighting

| Source | Base Weight | Rationale |
|--------|------------|-----------|
| Patent filing | 0.90 | High cost to file = strong commitment signal |
| SEC filing / regulatory | 0.90 | Legal obligation = highly reliable |
| Pricing page change | 0.85 | Direct business action |
| Job posting (single) | 0.30 | One post = noise; pattern = signal |
| Job posting (cluster 10+) | 0.75 | Clear hiring initiative |
| GitHub repo (new, active) | 0.70 | Active development confirmed |
| GitHub repo (empty/stale) | 0.20 | May be experimental |
| Domain registration | 0.50 | Low cost, but intentional |
| News/press mention | 0.60 | May be speculative reporting |
| Google Trends shift | 0.40 | Correlational, not causal |

### 5.3 Combination Confidence Model
```
combined_confidence = 1 - ∏(1 - signal_i.confidence * signal_i.weight)

Example: 5 weak signals each at 0.30 confidence, 0.40 weight
= 1 - (1-0.12)^5 = 1 - 0.88^5 = 1 - 0.528 = 0.472

Example: 3 strong signals at 0.80 confidence, 0.85 weight
= 1 - (1-0.68)^3 = 1 - 0.32^3 = 1 - 0.033 = 0.967
```

### 5.4 Prediction Scorecard
- Track all predictions made with timestamps
- When a predicted event occurs (or doesn't within the window), score it
- Rolling accuracy metric displayed on dashboard
- Feed accuracy data back to improve confidence calibration

---

## SECTION 6: FRONTEND ARCHITECTURE

**Stack: Next.js 14 (App Router) + Tailwind CSS + shadcn/ui + Recharts**

### 6.1 Pages & Components

| Page | Route | Key Components |
|------|-------|----------------|
| **Dashboard** | `/` | CompetitorCards, ThreatScoreGauge, RecentSignalsFeed, ActivePredictions |
| **Competitor Detail** | `/competitor/[id]` | SignalTimeline, ThreatHistory, PredictionPanel, HiringChart, TechStackMap |
| **Signal Feed** | `/signals` | SignalList (filterable by type/competitor/date), SignalDetailModal |
| **Predictions** | `/predictions` | PredictionCards, EvidenceChain, ConfidenceGauge, AccuracyScorecard |
| **Competitive Map** | `/map` | PositioningMatrix (2D), FeatureComparisonTable, MarketOverlap Venn |
| **Intelligence Briefs** | `/briefs` | BriefList, BriefViewer (rendered PDF), BriefArchive |
| **Settings** | `/settings` | CompetitorSetup, AlertConfig, IntegrationSettings (Slack/Email), UserProfile |

### 6.2 Real-Time Updates
- Server-Sent Events (SSE) from Next.js API routes for live signal feed
- Optimistic UI updates with React Query for mutations
- WebSocket fallback for Tier 1 alerts

### 6.3 Key UI Components
- **ThreatScoreGauge**: Animated circular gauge showing 0-100 threat level per competitor
- **SignalTimeline**: Chronological view with signal type icons, expandable cards
- **EvidenceChain**: Visual flowchart connecting signals to predictions
- **HiringChart**: Recharts area chart showing job posting volume over time with anomaly highlights
- **CompetitiveMap**: D3.js-powered 2D positioning matrix (customizable axes)

---

## SECTION 7: FILE & FOLDER STRUCTURE

```
phantom/
├── package.json
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── .env.local                          # API keys (Gemini, Google Search, SerpAPI, etc.)
├── .env.example
├── docker-compose.yml                  # PostgreSQL + Redis local dev
├── prisma/
│   ├── schema.prisma                   # DB schema (Competitor, Signal, Prediction, Alert, Brief)
│   └── migrations/
│       └── 001_init/
├── src/
│   ├── app/                            # Next.js App Router
│   │   ├── layout.tsx                  # Root layout with sidebar nav
│   │   ├── page.tsx                    # Dashboard
│   │   ├── competitor/
│   │   │   └── [id]/
│   │   │       └── page.tsx            # Competitor detail
│   │   ├── signals/
│   │   │   └── page.tsx                # Signal feed
│   │   ├── predictions/
│   │   │   └── page.tsx                # Predictions view
│   │   ├── map/
│   │   │   └── page.tsx                # Competitive map
│   │   ├── briefs/
│   │   │   └── page.tsx                # Intelligence briefs
│   │   ├── settings/
│   │   │   └── page.tsx                # Settings & config
│   │   └── api/
│   │       ├── signals/
│   │       │   ├── route.ts            # GET signals, POST new signal
│   │       │   └── stream/
│   │       │       └── route.ts        # SSE endpoint for live signals
│   │       ├── predictions/
│   │       │   └── route.ts            # GET predictions, trigger analysis
│   │       ├── competitors/
│   │       │   └── route.ts            # CRUD competitors
│   │       ├── briefs/
│   │       │   ├── route.ts            # GET briefs list
│   │       │   └── generate/
│   │       │       └── route.ts        # POST trigger brief generation
│   │       ├── alerts/
│   │       │   └── route.ts            # Alert config CRUD
│   │       ├── collect/
│   │       │   └── route.ts            # POST trigger manual collection
│   │       └── cron/
│   │           └── route.ts            # Vercel Cron / Cloud Scheduler webhook
│   ├── lib/
│   │   ├── db.ts                       # Prisma client singleton
│   │   ├── gemini.ts                   # Gemini API client (Pro + Flash)
│   │   ├── google-search.ts            # Custom Search API client
│   │   ├── redis.ts                    # Redis client (upstash or local)
│   │   └── utils.ts                    # Shared utilities
│   ├── collectors/
│   │   ├── index.ts                    # Collector orchestrator
│   │   ├── jobs.ts                     # Job board collector (JSearch/RapidAPI)
│   │   ├── github.ts                   # GitHub API collector
│   │   ├── patents.ts                  # USPTO PatentsView + EPO OPS
│   │   ├── pricing.ts                  # Puppeteer pricing page monitor
│   │   ├── news.ts                     # Google Custom Search + NewsAPI
│   │   ├── trends.ts                   # SerpAPI Google Trends
│   │   ├── appstore.ts                 # App Store / Play Store monitor
│   │   ├── domains.ts                  # WhoisXML domain monitor
│   │   └── social.ts                   # RSS + blog/social monitor
│   ├── engine/
│   │   ├── normalizer.ts               # Raw data → Signal schema
│   │   ├── deduplicator.ts             # Fingerprint-based dedup
│   │   ├── classifier.ts              # Gemini Flash signal classification
│   │   ├── scorer.ts                   # Confidence + relevance scoring
│   │   ├── pattern-detector.ts         # Signal cluster identification
│   │   ├── predictor.ts               # Gemini Pro prediction generation
│   │   ├── alert-router.ts            # Tier assignment + dispatch
│   │   └── brief-generator.ts         # Weekly intelligence brief
│   ├── components/
│   │   ├── ui/                         # shadcn/ui components
│   │   ├── dashboard/
│   │   │   ├── competitor-card.tsx
│   │   │   ├── threat-gauge.tsx
│   │   │   ├── signal-feed.tsx
│   │   │   └── prediction-summary.tsx
│   │   ├── signals/
│   │   │   ├── signal-list.tsx
│   │   │   ├── signal-detail.tsx
│   │   │   └── signal-filters.tsx
│   │   ├── predictions/
│   │   │   ├── prediction-card.tsx
│   │   │   ├── evidence-chain.tsx
│   │   │   └── confidence-gauge.tsx
│   │   ├── charts/
│   │   │   ├── hiring-chart.tsx
│   │   │   ├── signal-volume-chart.tsx
│   │   │   └── threat-history-chart.tsx
│   │   ├── competitive-map.tsx
│   │   ├── brief-viewer.tsx
│   │   └── layout/
│   │       ├── sidebar.tsx
│   │       ├── header.tsx
│   │       └── alert-banner.tsx
│   ├── hooks/
│   │   ├── use-signals.ts              # Signal data fetching + SSE
│   │   ├── use-predictions.ts
│   │   └── use-competitors.ts
│   └── types/
│       ├── signal.ts                   # Signal, SignalSource, SignalType
│       ├── prediction.ts               # Prediction, Evidence, Confidence
│       ├── competitor.ts               # Competitor, ThreatScore
│       └── brief.ts                    # Brief, BriefSection
├── scripts/
│   ├── seed-competitors.ts             # Seed demo competitors
│   ├── backfill-signals.ts             # Backfill historical signals for demo
│   └── run-collectors.ts               # Manual collector trigger
└── tests/
    ├── collectors/
    │   └── jobs.test.ts
    ├── engine/
    │   ├── normalizer.test.ts
    │   └── predictor.test.ts
    └── api/
        └── signals.test.ts
```

---

## SECTION 8: HACKATHON DEMO PLAN

### Pre-Hackathon (48+ hours before)
1. Deploy signal collectors monitoring 3 well-known companies (e.g., Stripe, Figma, Notion)
2. Run all collectors every 2 hours for 48 hours → accumulate 500+ real signals
3. Run prediction engine on accumulated signals → generate initial predictions
4. Generate one intelligence brief as sample

### Live Demo Script (5 minutes)

| Time | Action | Impact |
|------|--------|--------|
| 0:00 | Open dashboard — 3 competitor cards with live threat scores | "This has been watching silently for 48 hours" |
| 0:30 | Click into Stripe — show signal timeline with real job postings, GitHub activity | "Every signal is real, collected automatically" |
| 1:00 | Show hiring surge detection — chart with anomaly highlighted | "Stripe posted 18 ML roles this month — 3x their baseline" |
| 1:30 | Show prediction panel — "Stripe likely launching AI-powered fraud detection upgrade" with evidence chain | "PHANTOM predicted this from the pattern" |
| 2:00 | **The verification moment** — Google the prediction, show it matches a recent announcement or credible rumor | "This was predicted 3 weeks before the announcement" |
| 2:30 | Show weekly intelligence brief — PDF-quality automated report | "Every Monday, your team gets this. Zero analyst hours." |
| 3:00 | Live: Add a NEW competitor (e.g., Linear) — watch signals populate in real-time via SSE | "30 seconds to first intelligence on any company" |
| 3:30 | Show alert system — configure a Slack alert, trigger it | "Tier 1 threats hit your Slack instantly" |
| 4:00 | Show competitive map — positioning visualization | "The full competitive landscape, always current" |
| 4:30 | Pitch: market size, pricing model, roadmap | Close strong |

### Demo Safeguards
- Pre-cache all API responses for demo companies (Redis) so demo never waits on APIs
- Fallback to pre-collected data if any API is down
- Have 2 verified predictions ready as backup "verification moments"

---

## SECTION 9: 24-HOUR BUILD SPRINT

### T-48h: Pre-Collection Phase (before hackathon)
- [ ] Deploy PostgreSQL + Redis (Docker or cloud)
- [ ] Implement job board + GitHub + news collectors
- [ ] Start collecting signals for 3 demo competitors
- [ ] Verify signal storage and dedup working

### Hour 0-3: Foundation
- [ ] `npx create-next-app@latest phantom --typescript --tailwind --app`
- [ ] Prisma schema + migrations (Competitor, Signal, Prediction, Alert, Brief)
- [ ] Gemini client library setup (Pro + Flash)
- [ ] Environment variables and config

### Hour 3-6: Signal Pipeline
- [ ] Signal normalizer (raw → unified schema)
- [ ] Gemini Flash classifier integration
- [ ] Confidence + relevance scorer
- [ ] API routes: GET/POST signals, GET competitors

### Hour 6-10: Intelligence Engine
- [ ] Pattern detector (signal clustering by competitor + time window)
- [ ] Gemini Pro predictor (cluster → prediction with evidence chain)
- [ ] Alert tier assignment logic
- [ ] Prediction API routes

### Hour 10-15: Frontend
- [ ] Dashboard layout with sidebar
- [ ] Competitor cards with threat gauge
- [ ] Signal feed with SSE real-time updates
- [ ] Prediction panel with evidence chain
- [ ] Hiring surge chart (Recharts)

### Hour 15-18: Polish Features
- [ ] Weekly brief generator (Gemini Pro → Markdown → PDF)
- [ ] Alert configuration UI
- [ ] Competitive map visualization
- [ ] Competitor detail page with full signal timeline

### Hour 18-21: Integration & Demo Prep
- [ ] Slack webhook integration for alerts
- [ ] "Add new competitor" flow with live signal population
- [ ] Cache all demo data in Redis
- [ ] End-to-end flow testing

### Hour 21-24: Demo Polish
- [ ] Loading states, animations, error handling
- [ ] Demo script rehearsal
- [ ] Backup data preparation
- [ ] Pitch deck finalization

---

## SECTION 10: PRODUCTION & SCALE

### Infrastructure Costs (estimated monthly at scale)
| Component | Cost | Notes |
|-----------|------|-------|
| Cloud Run (collectors) | $50-150 | Scales to zero when idle |
| PostgreSQL (Cloud SQL) | $50-100 | db-f1-micro to db-g1-small |
| Redis (Memorystore) | $30 | 1GB basic tier |
| Gemini API | $100-500 | Depends on signal volume and prediction frequency |
| Google Custom Search | $25-50 | ~5K-10K queries/month |
| SerpAPI (Trends) | $50 | 5K searches/month |
| RapidAPI (Job boards) | $50-100 | JSearch Pro plan |
| WhoisXML | $20 | Basic plan |
| Vercel (frontend) | $20 | Pro plan |
| **Total** | **$395-1,090/mo** | For ~10 tracked competitors |

### Scaling Considerations
- Horizontal: Each collector is a stateless Cloud Run service → scale independently
- Signal storage: Partition PostgreSQL by month, archive to BigQuery after 6 months
- Prediction cache: Redis TTL-based caching for repeated dashboard loads
- Rate limit budget: Track API quota usage, prioritize high-value sources when nearing limits

### Legal Compliance
- **robots.txt**: All collectors check and respect robots.txt before scraping
- **Rate limiting**: Never exceed 1 request/second to any single domain
- **Public data only**: Only collect publicly accessible information
- **No login bypass**: Never circumvent authentication or paywalls
- **ToS compliance**: Use official APIs where available (GitHub, USPTO, Greenhouse, Lever)
- **GDPR**: No personal data collection — only company/organizational intelligence
- **hiQ v LinkedIn precedent**: Public data scraping is legal, but respect ToS and rate limits

---

## SECTION 11: TECHNICAL RISKS & MITIGATIONS

| Risk | Impact | Probability | Mitigation |
|------|--------|------------|------------|
| LinkedIn anti-bot blocking | Lose major signal source | High | Use RapidAPI aggregators (JSearch) instead of direct scraping; multiple job board sources for redundancy |
| Gemini hallucinating predictions | False intelligence, lost trust | Medium | Vertex AI Grounding enforces citation; require every prediction to link to specific signals; confidence threshold for display |
| API rate limits exhausted during demo | Demo failure | Medium | Pre-cache all demo data in Redis; fallback to cached responses |
| Signal quality variance across sources | Low-confidence predictions | High | Weight signals by source reliability; require multi-source confirmation for Tier 1 alerts |
| Patent data lag (USPTO ~18 months) | Stale patent signals | Medium | Supplement with provisional application monitoring; weight patent signals with recency factor |
| Prediction false positives | Alert fatigue | Medium | Start conservative (high confidence threshold); track accuracy scorecard; tune thresholds over time |
| Cloud Run cold starts | Slow collector execution | Low | Set min-instances=1 for critical collectors; use always-on for demo |
| Cost overrun from Gemini API | Budget exceeded | Medium | Use Flash for classification (cheap), Pro only for predictions; cache repeated analyses |

---

## SECTION 12: BUSINESS MODEL

### Pricing Tiers

| Tier | Price | Includes |
|------|-------|----------|
| **Starter** | $299/mo | 3 competitors, daily signals, weekly brief, email alerts |
| **Professional** | $799/mo | 10 competitors, real-time signals, daily briefs, Slack integration, prediction engine |
| **Enterprise** | $2,499/mo | Unlimited competitors, custom signal sources, API access, dedicated Slack channel, custom reports |
| **Add-on: Analyst Brief** | +$500/mo | Human-reviewed weekly brief with strategic recommendations |

### Unit Economics
- Cost per competitor: ~$40-100/mo (API costs + compute)
- Gross margin at Professional tier: ~87%
- Target: 100 Professional customers = $79,900 MRR = ~$960K ARR

### Go-to-Market
1. Launch on Product Hunt with verified prediction showcase
2. Free tier: 1 competitor, limited signals (lead gen)
3. Content marketing: publish monthly "prediction accuracy report"
4. Enterprise: direct sales to product marketing and strategy teams

---

## SECTION 13: JUDGING CRITERIA + PITCH

### Hackathon Scoring Alignment

| Criterion | How PHANTOM Scores |
|-----------|-------------------|
| **Innovation** | First predictive CI tool using LLM reasoning on multi-source weak signals |
| **Technical Complexity** | 10+ data sources, real-time pipeline, LLM-powered prediction engine, SSE live updates |
| **Completeness** | Full working product: collection → analysis → prediction → alerts → dashboard → reports |
| **Google API Usage** | Gemini Pro + Flash, Custom Search, Vertex AI Grounding, Cloud Run, Pub/Sub, BigQuery, Cloud Scheduler |
| **Impact** | $15B market, every company needs CI, current tools are reactive |
| **Demo Quality** | Pre-collected real data, verified predictions, live "add competitor" flow |

### Pitch Structure (2 minutes)

1. **Hook** (15s): "What if you could predict your competitor's next product launch 60 days before they announce it?"
2. **Problem** (20s): CI is manual, expensive, reactive. Crayon tells you what happened. PHANTOM tells you what's coming.
3. **Solution** (20s): 24/7 silent monitoring of 10+ signal sources. LLM-powered prediction with evidence chains.
4. **Demo** (45s): Show dashboard, signal timeline, prediction with verification, live competitor addition.
5. **Traction** (10s): "In 48 hours of monitoring, PHANTOM correctly predicted [specific verified event]."
6. **Business** (10s): $799/mo per team, 87% gross margin, $15B TAM.

---

## SECTION 14: APPENDICES

### A. Database Schema (Prisma)

```prisma
model Competitor {
  id          String       @id @default(uuid())
  name        String
  domain      String
  description String?
  industry    String?
  logoUrl     String?
  threatScore Float        @default(0)
  signals     Signal[]
  predictions Prediction[]
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt
}

model Signal {
  id              String      @id @default(uuid())
  competitorId    String
  competitor      Competitor  @relation(fields: [competitorId], references: [id])
  source          String      // JOB_BOARD, GITHUB, PATENT, PRICING, NEWS, TRENDS, APP_STORE, DOMAIN, SOCIAL, SEC
  sourceUrl       String
  signalType      String      // HIRING_SURGE, TECH_ADOPTION, PATENT_FILING, PRICE_CHANGE, etc.
  title           String
  rawData         Json
  insights        String[]
  confidence      Float
  relevanceScore  Float
  tags            String[]
  fingerprint     String      @unique
  sourceTimestamp DateTime
  createdAt       DateTime    @default(now())

  @@index([competitorId, createdAt])
  @@index([signalType])
  @@index([fingerprint])
}

model Prediction {
  id            String     @id @default(uuid())
  competitorId  String
  competitor    Competitor @relation(fields: [competitorId], references: [id])
  action        String     // Predicted competitive action
  description   String     // Detailed prediction narrative
  confidence    Float
  timeframeDays Int        // Predicted days until action
  tier          Int        // 1, 2, or 3
  evidenceIds   String[]   // Signal IDs supporting this prediction
  status        String     @default("ACTIVE") // ACTIVE, VERIFIED, EXPIRED, FALSE
  verifiedAt    DateTime?
  createdAt     DateTime   @default(now())
  expiresAt     DateTime

  @@index([competitorId, status])
  @@index([tier])
}

model Alert {
  id           String   @id @default(uuid())
  userId       String
  competitorId String?  // null = all competitors
  minTier      Int      @default(1) // Minimum tier to alert on
  channels     String[] // SLACK, EMAIL, SMS, PUSH
  slackWebhook String?
  email        String?
  active       Boolean  @default(true)
  createdAt    DateTime @default(now())
}

model Brief {
  id           String   @id @default(uuid())
  title        String
  markdownBody String
  pdfUrl       String?
  periodStart  DateTime
  periodEnd    DateTime
  createdAt    DateTime @default(now())
}
```

### B. Sample Signal Output (Job Posting Analysis)
```json
{
  "id": "sig_abc123",
  "competitor": "Stripe",
  "source": "JOB_BOARD",
  "title": "Stripe hiring surge: 12 ML Engineer roles in 14 days",
  "signalType": "HIRING_SURGE",
  "insights": [
    "12 ML Engineer postings detected (baseline: 3/month)",
    "All roles require TensorFlow + real-time inference experience",
    "4 roles specifically mention 'fraud detection' and 'risk scoring'",
    "New team: 'ML Platform' appears for first time in Stripe job listings",
    "Senior/Staff level roles suggest building new capability, not maintaining"
  ],
  "confidence": 0.82,
  "tags": ["machine-learning", "fraud-detection", "hiring-surge", "new-team"]
}
```

### C. Sample Prediction Output
```json
{
  "id": "pred_xyz789",
  "competitor": "Stripe",
  "action": "AI_PRODUCT_LAUNCH",
  "description": "Stripe is highly likely to launch an AI-powered fraud detection and risk scoring upgrade within 45-75 days. Evidence suggests a new 'ML Platform' team is being built from scratch with focus on real-time inference for transaction risk assessment.",
  "confidence": 0.87,
  "timeframeDays": 60,
  "tier": 1,
  "evidenceChain": [
    { "signalId": "sig_abc123", "weight": 0.75, "summary": "12 ML engineer roles in 14 days, 3x baseline" },
    { "signalId": "sig_def456", "weight": 0.70, "summary": "New GitHub repo: stripe/ml-risk-engine (Python, TensorFlow)" },
    { "signalId": "sig_ghi789", "weight": 0.85, "summary": "Patent filed: 'Real-time transaction risk scoring using neural networks'" },
    { "signalId": "sig_jkl012", "weight": 0.50, "summary": "Domain registered: stripe-radar-ai.com" }
  ],
  "recommendedResponse": "Evaluate your own fraud detection capabilities. Consider accelerating ML investment in risk scoring. Prepare competitive positioning for when Stripe announces."
}
```

### D. Key Environment Variables
```env
# Gemini / Google AI
GOOGLE_AI_API_KEY=
GOOGLE_CLOUD_PROJECT_ID=
GOOGLE_CUSTOM_SEARCH_API_KEY=
GOOGLE_CUSTOM_SEARCH_ENGINE_ID=

# Data Sources
RAPIDAPI_KEY=                    # JSearch job board API
GITHUB_TOKEN=                    # GitHub API (fine-grained PAT)
SERPAPI_KEY=                      # Google Trends
WHOISXML_API_KEY=                # Domain monitoring
NEWSAPI_KEY=                      # News aggregation

# Infrastructure
DATABASE_URL=postgresql://...
REDIS_URL=redis://...

# Integrations
SLACK_WEBHOOK_URL=
SENDGRID_API_KEY=               # Email alerts

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
CRON_SECRET=                     # Secure cron webhook endpoint
```

---

## VERIFICATION PLAN

1. **Unit tests**: Normalizer, deduplicator, confidence scorer, tier assignment
2. **Integration tests**: Full pipeline — collector → normalizer → store → pattern detect → predict
3. **E2E test**: Add competitor via UI → verify signals appear in feed → verify prediction generated
4. **Demo rehearsal**: Run full demo script 3x, verify all data loads, all predictions display, verification moment works
5. **Load test**: Simulate 50 competitors with 10K signals → verify dashboard renders < 2s
6. **Manual verification**: Cross-check 5 predictions against real-world public information

---

## IMPLEMENTATION ORDER (Priority)

1. **Prisma schema + DB setup** — Foundation everything builds on
2. **Gemini client library** — Core intelligence capability
3. **Job board collector** (highest signal value) → normalizer → store
4. **GitHub + news collectors** — Second-highest value sources
5. **Signal API routes + SSE** — Enable frontend development
6. **Dashboard + competitor cards** — Visual impact for demo
7. **Pattern detector + predictor** — The "wow factor"
8. **Evidence chain UI** — The "trust factor"
9. **Brief generator** — The "enterprise factor"
10. **Alert system + Slack** — The "real product" factor
11. **Remaining collectors** (patents, pricing, domains, trends, app store)
12. **Polish, caching, demo prep**
