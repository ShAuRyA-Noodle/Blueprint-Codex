# CHRONOS — The Institutional Knowledge Immortality Engine
## Production-Grade Technical & Strategic Master Blueprint

**Tagline:** Every company loses millions when employees leave — taking years of context, decisions, and reasoning with them. CHRONOS makes that knowledge permanent, queryable, and alive.

---

# SECTION 1: PROBLEM DEFINITION & VISION

## 1.1 The $47B Institutional Knowledge Loss Problem

Every year, U.S. companies lose an estimated **$47 billion** in productivity due to inadequate knowledge sharing and institutional memory loss. This number, derived from the Bureau of Labor Statistics' annual turnover rate (44.3M separations in 2023) multiplied by average onboarding productivity loss, **understates** the real damage because it doesn't capture the invisible cost: the loss of *reasoning*.

### What a Senior Engineer's Departure Actually Costs

| Cost Component | Calculation | Amount |
|---|---|---|
| Recruiting replacement | Agency fees (20-25% of salary) | $35,000-$50,000 |
| Onboarding ramp (3-6 months at 50% productivity) | $180K salary × 50% × 4.5 months | $33,750 |
| Lost decision context | Unmeasurable — but real | ??? |
| Repeated mistakes | Average 2.3 incidents per departed employee | $15,000-$200,000 |
| Team disruption | Remaining engineers spend 15-20% of time answering questions | $25,000 |
| **Total per senior departure** | | **$110,000-$310,000+** |

The hidden killer: **decision context loss**. When David Park leaves TechFlow Inc. and takes with him the knowledge of *why* the order normalization logic handles edge cases the way it does, no amount of code comments can replace the reasoning chain that led to those design choices.

### The Onboarding Failure

New employees make the same mistakes their predecessors made 3 years ago because:
- Confluence/Notion has **WHAT** was decided, never **WHY**
- Slack messages containing the actual debate are buried in unreachable history
- The person who could explain it has moved to a competitor
- Meeting recordings exist but nobody watches 47 minutes of video to find the 90-second segment where the VP of Engineering changed their mind

### The M&A Knowledge Destruction

When Company A acquires Company B:
- 30-40% of acquired employees leave within 18 months
- The integration team has no map of what Company B's employees know
- Institutional knowledge of Company B is essentially destroyed within 2 years
- The acquirer paid for the *brain* but only got the *skeleton*

## 1.2 Why Existing Tools Don't Solve This

| Tool | What It Stores | What It Misses |
|---|---|---|
| **Confluence/Notion** | Documents, pages, templates | WHY decisions were made; the debate that preceded them |
| **Slack/Teams** | Messages, threads | Structure; decisions are buried in noise; history expires |
| **Guru/Tettra** | Curated knowledge cards | Only what someone manually writes; dies when the curator leaves |
| **GitHub** | Code, PRs, issues | Business rationale; cross-system decision context |
| **Jira/Linear** | Tickets, status | The human reasoning; why things were prioritized |

**The CHRONOS Insight:** These tools capture the *record*. CHRONOS captures the *reasoning*. The record says "We migrated to PostgreSQL." The reasoning says "We chose PostgreSQL over MongoDB because our user data model has strong relational requirements, three engineers had PostgreSQL expertise vs. one with MongoDB, and our enterprise customers' security reviews required ACID guarantees — and the alternative of DynamoDB was rejected due to vendor lock-in concerns."

## 1.3 The CHRONOS Value Proposition

CHRONOS is the **institutional knowledge graph** that:
1. **Ingests** data from every tool a company already uses (Slack, GitHub, Confluence, Jira, email, meeting transcripts, Google Workspace)
2. **Extracts** the reasoning behind decisions using Gemini 1.5 Pro's 2M context window
3. **Builds** a queryable knowledge graph connecting decisions, people, systems, concepts, and outcomes
4. **Answers** natural language questions: *"Why did we choose X?"* *"Who knows about Y?"* *"What happened after we decided Z?"*
5. **Surfaces** relevant institutional context automatically via a Chrome extension — on Jira tickets, GitHub PRs, email compose windows

**One-line pitch to judges:** *"We turn 2 years of Slack messages, GitHub PRs, and Confluence pages into a queryable brain that tells you WHY your company made every decision — so when people leave, the knowledge doesn't."*

---

# SECTION 2: CORE TECHNICAL ARCHITECTURE

## 2.1 System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CHRONOS SYSTEM ARCHITECTURE                       │
│                                                                             │
│  ┌──────────────────── Data Sources ────────────────────┐                  │
│  │  GitHub  Slack  Confluence  Jira  Gmail  Meet  GDocs │                  │
│  └────────────────────────┬─────────────────────────────┘                  │
│                           │ Webhooks + Polling                              │
│                           ▼                                                 │
│  ┌──────────────── Ingestion Pipeline ──────────────────┐                  │
│  │  Cloud Run Workers (Python)                          │                  │
│  │  ┌──────┐ ┌──────────┐ ┌───────┐ ┌─────┐ ┌───────┐  │                  │
│  │  │Fetch │→│Normalize │→│Privacy│→│Chunk│→│Embed  │  │                  │
│  │  └──────┘ └──────────┘ └───────┘ └─────┘ └───┬───┘  │                  │
│  │       GCS raw storage    Presidio   2K tok   │       │                  │
│  │                          PII det    overlap  │       │                  │
│  └──────────────────────────────────────────────┼───────┘                  │
│                                                 │                          │
│                           ┌─────────────────────▼───────────────┐          │
│                           │   Reasoning Extraction Engine       │          │
│                           │   Gemini 1.5 Pro (2M context)       │          │
│                           │   ┌────────────────────────────┐   │          │
│                           │   │ Decision Detection         │   │          │
│                           │   │ Rationale Extraction       │   │          │
│                           │   │ Stakeholder Attribution    │   │          │
│                           │   │ Context Chain Linking      │   │          │
│                           │   │ Consequence Tracking       │   │          │
│                           │   └────────────┬───────────────┘   │          │
│                           └────────────────┼───────────────────┘          │
│                                            │                              │
│                           ┌────────────────▼───────────────────┐          │
│                           │      Knowledge Graph (Neo4j)       │          │
│                           │  Nodes: Decision, Person, System,  │          │
│                           │  Concept, Policy, Event,           │          │
│                           │  Alternative, Outcome, Document,   │          │
│                           │  Meeting, Thread                   │          │
│                           │  Edges: made_by, influenced_by,    │          │
│                           │  led_to, contradicts, supersedes,  │          │
│                           │  was_because_of, dissented_from... │          │
│                           └────────────────┬───────────────────┘          │
│                                            │                              │
│                           ┌────────────────▼───────────────────┐          │
│                           │         Query Engine               │          │
│                           │  NL → Intent → Cypher → Synthesis  │          │
│                           │  Multi-hop | WHO KNOWS | Timeline  │          │
│                           └──────────┬──────────┬──────────────┘          │
│                                      │          │                          │
│                    ┌─────────────────▼──┐  ┌───▼──────────────────┐       │
│                    │  Next.js Web App   │  │  Chrome Extension    │       │
│                    │  Dashboard, Graph  │  │  Side Panel, Tooltip │       │
│                    │  Search, Timeline  │  │  Context Surfacing   │       │
│                    └────────────────────┘  └──────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2.2 The Multi-Source Ingestion Pipeline

Seven-stage pipeline running on Cloud Run workers (Python):

| Stage | Operation | Technology | Output |
|---|---|---|---|
| 1. Fetch | Source-specific API calls with rate limiting | Python connectors, GCS | Raw documents in `gs://chronos-raw/{org}/{source}/{date}/{id}` |
| 2. Normalize | Map to canonical Document schema | Custom mappers | NormalizedDocument (clean text, metadata) |
| 3. Privacy Filter | PII detection + sensitivity classification | Microsoft Presidio + custom classifiers | Filtered documents (INTERNAL/RESTRICTED/CONFIDENTIAL) |
| 4. Chunk | Split into overlapping segments | Boundary-aware splitter | 2000-token chunks with 200-token overlap |
| 5. Embed | Generate vector embeddings | Vertex AI `text-embedding-004` | 768-dim vectors in Vertex AI Vector Search |
| 6. Extract | Gemini reasoning extraction | Gemini 1.5 Pro structured output | DecisionExtraction JSON |
| 7. Graph Write | Entity resolution + upsert | Neo4j driver, Levenshtein + embedding dedup | Nodes + edges in Neo4j |

Pub/Sub topics connect each stage for decoupled, scalable processing.

### Source Connectors

- **GitHub**: App with `repo`, `read:org` scopes. Fetches PR descriptions, comments, commit messages, issues, ADRs. Webhooks for real-time; `updated_since` for backfill. Decision signals: PRs with "why" in body, `architecture`/`RFC` labels.
- **Slack**: App with `channels:history`, `users:read`. Fetches messages in decision-relevant channels (`#architecture`, `#eng-decisions`, `#incidents`). Cursor-based pagination. Decision signals: "we decided", "going with", `:white_check_mark:` reactions, pinned threads. DMs excluded by default.
- **Confluence**: API token/OAuth 2.0. Fetches pages in whitelisted spaces, page history diffs. CQL `lastmodified > {checkpoint}`. Decision signals: labels `decision`, `adr`, `rfc`, `post-mortem`.
- **Jira**: API token. Fetches DECISION/RFC/SPIKE/INCIDENT issue types + comments. JQL `updated > {checkpoint}`.
- **Email**: Google Workspace service account with domain-wide delegation. Shared inboxes only (`architecture@`, `engineering@`). Gmail `history.list` API. Personal inboxes never ingested.
- **Meeting Transcripts**: Google Meet via Workspace Admin API, Zoom server-to-server OAuth. VTT/SRT → speaker-diarized text → 5-minute chunks.
- **Google Docs/Drive**: Service account with Drive API scope. Shared drives with "Architecture"/"Decisions"/"RFCs" folder names. `changes.list` API.

### Incremental Update Strategy

Each connector maintains a checkpoint in Redis:
```
Key:   chronos:checkpoint:{orgId}:{source}:{channelOrRepoId}
Value: {lastTimestamp, lastCursor, lastPageToken, etag}
```

Sync frequency: GitHub webhooks (real-time, <30s lag) → Slack (15min high-priority, 2hr others) → Jira (30min) → Confluence (1hr) → Email (1hr) → Meeting transcripts (1hr after meeting end + 30min buffer) → Google Docs (2hr).

## 2.3 The Reasoning Extraction Engine

This is CHRONOS's intellectual core. Two-stage classification before expensive extraction:

### Stage A: Decision Detection (cheap, fast)

**A1 — Rule-based pre-filter (~1ms):** Regex scan for signals like `we decided`, `going with`, `rationale:`, `trade-off`, `RFC-\d+`, `we rejected`. Negative signals filter out passwords, credit cards, pure OKR docs.

**A2 — Gemini 1.5 Flash binary classifier:** Asks "does this text contain an explicit organizational decision with stated reasoning?" Returns `{is_decision: bool, confidence: float}`. Threshold: confidence >= 0.65 proceeds to full extraction.

### Stage B: Full Reasoning Extraction (Gemini 1.5 Pro)

Context assembly (priority order, ~50K token budget):
1. Triggering chunk (always)
2. ±2 surrounding chunks (temporal context)
3. Parent document metadata
4. Thread context (up to 50 prior messages)
5. Mentioned person profiles (name, role, department)
6. Top 3 semantically similar previously-extracted decisions

**Master extraction prompt** enforces structured JSON output via Gemini function calling, extracting for each decision:
1. **Decision statement** — what was decided, type, date, confidence
2. **Rationale** (the WHY — primary reasons, constraints, assumptions, verbatim evidence quote)
3. **Stakeholders** — name, role (DECIDER/APPROVER/INFLUENCER/DISSENTER), dissent text
4. **Alternatives considered** — what, why rejected, who proposed
5. **Context chain** — prior decisions, triggering events, shaping constraints
6. **Anticipated consequences** — predicted outcomes, acknowledged risks
7. **Concepts and systems** — referenced patterns, affected services

### Stakeholder Attribution Pipeline

Multi-step resolution: exact name match → alias match → email domain → fuzzy match (Levenshtein < 2) → embedding similarity > 0.92 → context clues ("the CTO", "Sarah from payments") → unresolved creates provisional UNVERIFIED Person node.

### Context Chain Linking

Post-extraction linking pass: temporal proximity (±30 days) → system overlap → semantic similarity > 0.75 → explicit reference detection ("as we decided in ADR-12") → cross-document threading.

### Consequence Tracking

Background job runs every 30 days per Decision node: searches post-decision documents with embedding similarity > 0.70, prompts Gemini to classify outcomes (POSITIVE/NEGATIVE/NEUTRAL/MIXED), creates Outcome nodes with `led_to` edges including lag days. Also detects contradictions for UI alerts.

## 2.4 The Knowledge Graph Architecture

### Node Types (11)

| Node | Key Properties | Purpose |
|---|---|---|
| **Decision** | title, summary, rationale, confidence (0-1), status (PROPOSED\|ACTIVE\|SUPERSEDED\|REVERSED), decisionType, impact, reversibility, embeddingVector | The core entity — a captured decision with its WHY |
| **Person** | fullName, role, roleHistory[], department, team, status (ACTIVE\|DEPARTED\|ACQUIRED), expertise[], communicationIds{slack,github,google,jira} | Identity resolution across all sources |
| **Concept** | name, description, conceptType (PATTERN\|TECHNOLOGY\|PRINCIPLE\|METHODOLOGY\|DOMAIN_TERM), aliases[], mentionCount | Technical and business concepts referenced in decisions |
| **System** | name, systemType (SERVICE\|DATABASE\|LIBRARY\|PLATFORM\|EXTERNAL_API\|INFRASTRUCTURE), status, techStack[], owningTeam, repositoryUrl | Software systems, services, and infrastructure |
| **Policy** | title, content, policyType (SECURITY\|ENGINEERING\|HR\|COMPLIANCE), status, effectiveDate, mandatoryFor[] | Organizational rules and standards |
| **Event** | title, eventType (INCIDENT\|LAUNCH\|ACQUISITION\|REORG\|OFFBOARDING), occurredAt, severity, impact, resolution | Trigger events and milestones |
| **Alternative** | title, description, whyRejected, tradeoffs[], proposedBy | Options considered but not chosen |
| **Outcome** | title, description, outcomeType (POSITIVE\|NEGATIVE\|NEUTRAL\|MIXED), metrics[], lagDays | What happened after a decision |
| **Document** | title, sourceSystem, sourceUrl, contentHash, contentType (PR\|ISSUE\|PAGE\|THREAD\|TRANSCRIPT\|RFC\|ADR), privacyTier | Source material provenance |
| **Meeting** | title, platform, participants[], durationMinutes, actionItems[] | Meeting records with transcripts |
| **Thread** | platform, channelOrList, participantIds[], messageCount | Conversation threads across platforms |

### Edge Types (13)

| Edge | Direction | Properties | Meaning |
|---|---|---|---|
| **made_by** | Decision → Person | role (DECIDER\|APPROVER\|INFLUENCER\|DISSENTER), confidence | Who participated in the decision and how |
| **influenced_by** | Decision → Decision\|Event\|Document\|Concept | influenceType, strength (0-1), description | What shaped this decision |
| **led_to** | Decision\|Event → Decision\|Outcome\|Event\|System | causalityType (DIRECT\|ENABLES\|FORCES\|PREVENTS), confidence, lagDays | Causal consequences |
| **contradicts** | Decision → Decision\|Policy | severity (MINOR\|MAJOR\|FUNDAMENTAL), resolvedAt | Conflicting decisions |
| **supersedes** | Decision → Decision\|Policy | supersededAt, reason | Replacement decisions |
| **was_because_of** | Decision → Event\|Concept\|System\|Policy | causationType (ROOT_CAUSE\|CONTRIBUTING\|TRIGGERING), confidence | Root cause links |
| **dissented_from** | Person → Decision | dissent text, intensity (MILD\|STRONG\|VETO_ATTEMPT), outcome (OVERRULED\|ACCOMMODATED) | Recorded disagreement |
| **participated_in** | Person → Meeting\|Thread\|Event | role, speakingTime | Participation records |
| **references** | Document\|Decision\|Thread → any | referenceType (CITES\|LINKS\|MENTIONS\|QUOTES), context | Citation links |
| **obsoletes** | Event\|Decision → System\|Policy\|Concept | obsoletedAt, reason | Deprecation links |
| **proposed** | Person → Alternative | timestamp | Who proposed rejected alternatives |
| **owns** | Person\|Team → System | since, ownership_type (TECHNICAL\|BUSINESS\|BOTH) | System ownership |
| **part_of** | System → System | relationship (DEPENDS_ON\|CALLS\|STORES_DATA_IN\|BUILT_ON) | System dependencies |

### Temporal Dimension

Every node has `createdAt`/`updatedAt` timestamps. Decisions have `decisionDate` (when the decision was made) and `discoveredAt` (when CHRONOS extracted it). Edge properties include timestamps. This enables timeline queries and temporal graph traversal ("show me the state of knowledge as of June 2024").

### Confidence Scoring

Every extracted fact carries a confidence score (0.0-1.0):
- **0.9-1.0**: Explicitly stated decision with clear rationale and attribution
- **0.7-0.9**: Clearly implied decision, some rationale extracted, partial attribution
- **0.5-0.7**: Probable decision, rationale inferred from context, needs human review
- **<0.5**: Discarded — insufficient evidence

## 2.5 The Query Engine

Five-phase pipeline: **NL → Intent → Cypher → Results → Synthesis**

### Phase 1: Query Understanding (Gemini 1.5 Pro)

Classifies into query types: `DECISION_LOOKUP`, `PERSON_EXPERTISE`, `SYSTEM_HISTORY`, `TIMELINE`, `CAUSAL_CHAIN`, `WHO_KNOWS`, `COMPARISON`, `RATIONALE_DEEP_DIVE`. Extracts entities, time ranges, relationship focus, depth (1-3 hops), filters.

### Phase 2: Entity Resolution

Hybrid scoring: `0.6 × full_text_score + 0.4 × vector_score` against Neo4j full-text index + Vertex AI Matching Engine. Returns top 3 candidates per entity.

### Phase 3: Cypher Query Generation

Parameterized Cypher templates per query type. Examples:

**DECISION_LOOKUP** — "Why did we choose PostgreSQL?":
```cypher
MATCH (d:Decision) WHERE d.orgId = $orgId
WITH d, gds.similarity.cosine(d.embeddingVector, $queryVector) AS score
WHERE score > 0.70
MATCH (d)-[:made_by]->(p:Person)
MATCH (d)-[:was_because_of]->(context)
OPTIONAL MATCH (d)<-[:proposed]-(alt:Alternative)
OPTIONAL MATCH (d)-[:led_to]->(outcome:Outcome)
RETURN d, collect(DISTINCT p) AS stakeholders, collect(DISTINCT alt) AS alternatives, score
ORDER BY score DESC LIMIT 5
```

**WHO_KNOWS** — "Who knows about the payment system?":
```cypher
MATCH (s:System {name: $systemName, orgId: $orgId})
MATCH (p:Person)-[r]-(n) WHERE (n)-[:references|led_to*1..2]-(s)
WITH p, count(DISTINCT n) AS touchpoints
MATCH (p)-[:made_by {role: "DECIDER"}]->(d:Decision) WHERE (d)-[:was_because_of|led_to*1..2]-(s)
RETURN p, touchpoints, count(d) AS decisionsOwned,
       (touchpoints * 0.5 + count(d) * 2.0) AS expertiseScore
ORDER BY expertiseScore DESC LIMIT 10
```

### Phase 4: Result Ranking

`final_score = cypher_score × 0.5 + extraction_confidence × 0.3 + source_recency × 0.2`. Redundancy filtering at embedding similarity > 0.95.

### Phase 5: Answer Synthesis (Gemini)

Synthesizes structured results into natural language: directly answers the question, explains the reasoning chain, names key people, mentions timing, highlights dissenting views, cites source documents. Under 300 words.

### Multi-Hop Reasoning

For queries like "how did the AWS outage eventually affect our pricing strategy?":
1. Resolve both anchor entities
2. Bidirectional BFS, up to 4 hops
3. Path intersection at shortest connection
4. Score paths by edge confidence product × recency decay
5. Gemini narrates the causal chain

### The "Who Knows About X" Algorithm

```
ExpertiseScore(P, X) =
  Σ(decision_weight × impact_score)     // weight: 3.0 DECIDER, 1.5 APPROVER, 0.5 INFLUENCER
  + Σ(document_weight × recency_decay)  // recency_decay = e^(-0.1 × months)
  + Σ(meeting_weight × speaking_ratio)
  + Σ(thread_weight × message_count)

Tenure bonus: >3 decisions in domain → ×1.4
Alumni penalty: departed → ×0.3 (still shown, critical for M&A)
```

## 2.6 Gemini 1.5 Pro's 2M Context Window — The Key Technical Advantage

Gemini 1.5 Pro's 2M token context window is CHRONOS's unfair advantage:
- **Entire decision histories in context**: A typical Slack architecture thread (200 messages) + the resulting Confluence ADR + 3 related Jira issues + the GitHub PR = ~15,000 tokens. CHRONOS can fit ~130 such decision bundles in a single context window.
- **Cross-document reasoning**: Unlike RAG systems limited to retrieved chunks, CHRONOS can load an entire project's communication history and identify decision patterns across conversations that happened weeks apart.
- **Context caching**: Gemini's context caching reduces costs by up to 75% for repeated queries against the same organizational knowledge base.

**Cost optimization**: Use Gemini 1.5 Flash ($0.075/1M input tokens) for Stage A classification. Use Gemini 1.5 Pro ($3.50/1M input tokens for prompts >128K) only for Stage B full extraction and Phase 5 answer synthesis.

## 2.7 The Chrome Extension — In-Context Knowledge Surfacing

The Chrome Extension (MV3) is the "last mile" — surfacing institutional knowledge where employees actually work, without requiring them to open a separate dashboard.

---

# SECTION 3: GOOGLE API INTEGRATION PLAN

## 3.1 Complete API Map

| API | Purpose in CHRONOS | Pricing (2026) | Rate Limits |
|---|---|---|---|
| **Gemini 1.5 Pro** | Reasoning extraction, query understanding, answer synthesis | $1.25/1M input (≤128K), $3.50/1M input (>128K) | 1,000 RPM |
| **Gemini 1.5 Flash** | Decision detection classifier (cheap pre-filter) | $0.075/1M input | 2,000 RPM |
| **Vertex AI Embeddings** (`text-embedding-004`) | Semantic vectors for all nodes + chunks | $0.000025/1K chars | Batch support |
| **Vertex AI Vector Search** (Matching Engine) | Fast approximate nearest neighbor search | Index + query pricing | Low-latency retrieval |
| **Vertex AI Search** | Enterprise search layer (optional, for full-text hybrid) | 10K queries/mo free; enterprise tiers | Per-query |
| **Document AI** | OCR for PDF/image ingestion (contracts, scanned docs) | $1.50/1K pages (1-5M), $0.60/1K pages (5M+) | 200+ languages |
| **Speech-to-Text** | Meeting transcript generation | $0.024/min standard, $0.036/min enhanced | 60 min/mo free |
| **Gmail API** | Shared inbox email ingestion | Free with Workspace license | Per-user quotas |
| **Google Drive API** | Document and shared drive ingestion | Free with Workspace license | 12,000 queries/100s |
| **Google Docs API** | Document content + version history | Free with Workspace license | Standard quotas |
| **Google Calendar API** | Meeting detection + participant resolution | Free with Workspace license | Standard quotas |
| **BigQuery** | Analytics, audit logs, large-scale graph queries (future) | $6.25/TB queried; $0.02/GB stored | Concurrent slots |
| **Cloud Pub/Sub** | Pipeline stage messaging | $40/TB; first 10GB free | 10K msg/sec default |
| **Cloud Run** | Stateless ingestion workers + API hosting | $0.00002400/vCPU-second | Autoscaling |
| **Cloud Storage (GCS)** | Raw ingestion data lake | $0.020/GB/mo (Standard) | Unlimited |
| **Memorystore (Redis)** | Caching, checkpoints, rate limiting | $0.049/GB/hr (Basic) | Instance-based |
| **Firebase Auth** | User authentication + org-level sessions | Free Spark plan (50K MAU) | N/A |

## 3.2 Cost Projection (1000-employee org, first month)

| Component | Estimated Volume | Monthly Cost |
|---|---|---|
| Gemini 1.5 Pro extraction | ~50K documents × 50K tokens avg | ~$8,750 |
| Gemini 1.5 Flash classification | ~200K chunks × 2K tokens | ~$30 |
| Gemini query answering | ~5K queries/mo × 10K tokens | ~$62 |
| Vertex AI Embeddings | ~200K chunks × 500 chars avg | ~$2.50 |
| Vector Search | Index (100K vectors) + queries | ~$150 |
| Speech-to-Text | ~100 hours of meetings | ~$144 |
| Neo4j Aura (Professional) | ~10GB graph | ~$650 |
| Cloud Run (workers + API) | ~1000 vCPU-hours | ~$86 |
| GCS storage | ~50GB raw data | ~$1 |
| Pub/Sub | ~5GB messages | Free tier |
| Redis (Memorystore) | 1GB Basic | ~$35 |
| **Total estimated** | | **~$9,910/mo** |

At $50/seat/month × 1000 seats = $50,000/month revenue → healthy ~80% gross margin after initial ingestion.

---

# SECTION 4: THE INGESTION PIPELINE IN DETAIL

## 4.1 GitHub Integration

**Setup**: GitHub App installed per organization. Permissions: `repo` (read), `read:org`, `read:discussion`. Webhook events subscribed: `pull_request`, `issues`, `discussion`, `push` (for commit messages).

**What is extracted**:
- PR descriptions + body → decision rationale for code changes
- PR review comments → dissenting opinions, approval reasoning
- Commit messages → incremental context (especially conventional commits)
- Issue bodies + comments → requirements, bug reports, design discussions
- ADRs from `docs/adr/` or `docs/architecture/` paths → formal decisions
- GitHub Discussions → open-ended architectural debates

**Decision signals**: PRs with "why" in body, labels `architecture`/`RFC`/`breaking-change`, issues with type `Decision`, discussions in architecture category.

**Rate limit handling**: GitHub webhooks don't consume API rate limits. GraphQL fallback for backfill uses exponential backoff respecting `X-RateLimit-Reset`. Enterprise Cloud: 10,000 points/hour; standard: 5,000 points/hour.

## 4.2 Slack Export Processing

**Setup**: Slack App with `channels:history`, `users:read`, `files:read` scopes. Channel allowlist configured per org (not all channels ingested).

**Threading reconstruction**: Slack's `thread_ts` field links replies to parent messages. CHRONOS reconstructs full conversation threads, preserving temporal order and identifying when the thread topic shifted (new decision point).

**Decision moment detection**: Beyond regex signals, CHRONOS looks for:
- Messages followed by `:white_check_mark:` reaction from a manager
- Pinned messages (these are often decisions)
- Messages in `#decisions`/`#architecture` channels with 5+ replies
- Messages containing links to Confluence ADRs or Jira Decision tickets

**Privacy**: DMs and private channels excluded by default. Admins configure a channel allowlist. Bot messages excluded unless from decision-tracking bots.

## 4.3 Email Threading Analysis

**Only shared/group mailboxes**: `architecture@`, `engineering@`, `decisions@`, `leadership@`. Personal inboxes are **never** ingested.

**Threading**: Gmail API provides `threadId` for message grouping. CHRONOS identifies decision chains: threads with >5 messages, threads with subject lines containing "decision"/"going forward"/"RE: Re: Re:", threads with executive participants.

## 4.4 Meeting Transcript Pipeline

**Input**: Google Meet recordings (auto-transcribed) or Zoom cloud recordings with transcription.

**Processing**:
1. Raw VTT/SRT → speaker-diarized text (identify who said what)
2. Segmented into 5-minute chunks with speaker attribution
3. Action item extraction via Gemini Flash: "Extract action items with assignees and due dates"
4. Decision segment identification: "let's make a call on this", "we've decided", "going with option B"

**Speaker resolution**: Meeting participant names from Calendar API → mapped to Person nodes via email match.

## 4.5 Incremental Update Strategy

- **Webhook-first**: GitHub and Slack use webhooks pushed to Pub/Sub for near real-time processing
- **Polling with checkpoints**: All other sources poll on schedule, storing the last-seen cursor/timestamp in Redis
- **Content-hash dedup**: If a Document's SHA-256 content hash matches an existing node, skip re-processing
- **Re-extraction trigger**: If the extraction prompt version changes, flagged documents queue for re-extraction

## 4.6 The Privacy Layer

**What is NOT ingested (hard rules)**:
- Personal DMs and private 1:1 channels
- Personal email inboxes
- HR-related channels (`#hr-confidential`, `#performance-reviews`)
- Documents containing salary, compensation, or performance review data
- Medical or health information
- Legal privilege communications (attorney-client)

**PII handling**:
- Microsoft Presidio NER detects: PERSON, EMAIL, PHONE, SSN, CREDIT_CARD, IP_ADDRESS
- Custom recognizers for: internal employee IDs, project codenames
- Sensitivity tiers: INTERNAL (pass through), RESTRICTED (redact PII, keep structure), CONFIDENTIAL (metadata only, human review queue)

**Employee opt-out**: Any employee can opt out of having their communications processed. Their messages are still ingested for thread continuity but attributed to "[Opted-Out User]" and their Person node is flagged.

## 4.7 Data Retention and Right-to-be-Forgotten

- **GDPR compliance**: Right to erasure. When a Person exercises RTBF, all edges to their Person node are severed, their communications are anonymized ("Former Employee #X"), and their Person node is pseudonymized.
- **Retention policy**: Raw data in GCS retained for 90 days then deleted. Extracted graph data retained indefinitely (it's the product). Source URLs remain for provenance.
- **Data Processing Agreement**: Enterprise customers sign a DPA specifying data handling, retention, and deletion procedures.

---

# SECTION 5: THE CHROME EXTENSION ARCHITECTURE

## 5.1 Architecture (Manifest V3)

```
chronos-extension/
├── manifest.json          # MV3 manifest with permissions
├── src/
│   ├── background/
│   │   └── service-worker.ts   # Non-persistent; handles auth, message routing
│   ├── content/
│   │   ├── context-detector.ts # Detects page context (GitHub PR? Jira ticket?)
│   │   └── sidebar/
│   │       ├── Sidebar.tsx     # React-rendered side panel
│   │       ├── DecisionPanel.tsx
│   │       └── QueryPanel.tsx
│   ├── popup/
│   │   └── QuickSearch.tsx     # Omnibar-style quick search
│   └── shared/
│       ├── api-client.ts       # Calls CHRONOS API
│       └── storage.ts          # Chrome Storage API for state persistence
```

**MV3 Constraints handled**:
- Service workers are non-persistent: all state stored in Chrome Storage API, rehydrated on wake
- No remote code execution: all logic bundled in extension package
- Side panel (right-side only): used for detailed context display
- Content scripts: lightweight detection + sidebar injection

## 5.2 Trigger Points — When Context Surfaces Automatically

| Page Type | Detection Method | Context Surfaced |
|---|---|---|
| **Jira ticket** | URL match `*.atlassian.net/browse/*` | Past decisions about the ticket's component/epic, related architectural context |
| **GitHub PR** | URL match `github.com/*/pull/*` | Architecture decisions about this codebase, related ADRs, who to ask for review context |
| **GitHub Issue** | URL match `github.com/*/issues/*` | Similar past issues, decisions that might affect this area |
| **Confluence page** | URL match `*.atlassian.net/wiki/*` | Related decisions, contradicting policies, timeline context |
| **Email compose** (Gmail) | URL match `mail.google.com` + compose detection | Context about the recipient's domain of expertise, relevant project history |
| **Any page** (manual) | Extension icon click or keyboard shortcut | Full query interface — "What do we know about X?" |

## 5.3 Side Panel UI Design

```
┌─────────────────────────────┐
│ 🔍 CHRONOS Context          │
│ ─────────────────────────── │
│                              │
│ 📋 Related Decisions (3)     │
│ ┌──────────────────────────┐│
│ │ "PostgreSQL over MongoDB" ││
│ │ May 2023 · Elena Vasquez  ││
│ │ Confidence: 94%           ││
│ │ ▸ View rationale          ││
│ └──────────────────────────┘│
│ ┌──────────────────────────┐│
│ │ "Microservices migration" ││
│ │ Mar 2023 · Priya Nair     ││
│ │ Confidence: 97%           ││
│ │ ▸ View rationale          ││
│ └──────────────────────────┘│
│                              │
│ 👤 Experts on this topic     │
│ Elena Vasquez (Score: 8.4)   │
│ Carlos Torres (Score: 6.1)   │
│                              │
│ ⚠️ Potential Conflicts (1)   │
│ This PR may contradict       │
│ ADR-002 storage policy       │
│                              │
│ ─────────────────────────── │
│ 💬 Ask CHRONOS...            │
│ ┌──────────────────────────┐│
│ │ Why does the user service ││
│ │ use this pattern?         ││
│ └──────────────────────────┘│
└─────────────────────────────┘
```

## 5.4 Manual Query Interface

From any browser tab, the user can click the CHRONOS icon or press `Ctrl+Shift+K` to open a command palette-style search:
- Pre-populated with page context (e.g., the PR title, the Jira ticket summary)
- Typeahead suggestions from recent queries and popular decisions
- Results displayed in the side panel with full source attribution

---

# SECTION 6: FRONTEND ARCHITECTURE

## 6.1 Technology Stack

| Layer | Technology | Rationale |
|---|---|---|
| Framework | Next.js 14 (App Router) | Server components for SEO, streaming for large graphs |
| Styling | Tailwind CSS + shadcn/ui | Rapid prototyping, consistent design system |
| State | Zustand (client) + React Query (server) | Lightweight, no boilerplate; smart caching |
| Graph Visualization | Sigma.js + Graphology | Lighter than D3 for graphs; WebGL rendering |
| Charts/Timeline | Recharts + custom timeline component | |
| Auth | NextAuth.js (Google OAuth) | Easy Google Workspace SSO |

## 6.2 Page Architecture

### Knowledge Graph Explorer (`/graph`)
- Interactive force-directed graph visualization (Sigma.js WebGL)
- Node colors by type: Decision (blue), Person (green), System (orange), Concept (purple), Event (red)
- Click node → right panel shows detail inspector
- Filter by: node type, date range, confidence threshold, team/department
- Zoom to subgraph: "Show me everything connected to the billing-service"
- Mini-map for orientation in large graphs

### Timeline View (`/timeline`)
- Horizontal scrollable decision timeline
- Event markers: Decision (diamond), Event (circle), Outcome (triangle)
- Causal arrows connecting related events (from `led_to` edges)
- Filter by date range, decision type, team, person
- Click event → slide-out panel with summary and links

### Person Knowledge Profile (`/people/[id]`)
- Shows what a person knows: decisions they made, influenced, or dissented from
- Expertise areas with scores
- Timeline of their decision involvement
- **Critical for offboarding**: "Before Sarah leaves, here's everything she knows that nobody else does"
- **Knowledge overlap matrix**: shows which other people share this person's knowledge areas

### Search Interface (`/search`)
- Large search bar with natural language input
- Query suggestions: "Try: 'Why did we choose PostgreSQL?'"
- Results: Gemini-synthesized answer at top, followed by source decision cards
- Each card: decision title, date, key stakeholders, confidence score, source links
- `ReasoningExplainer` component shows HOW the answer was derived (which graph paths were traversed)

### Onboarding Mode (`/onboarding`)
- Structured knowledge transfer for new employees
- Auto-generated "30-60-90 day knowledge guide" based on their team and role
- Key decisions they should understand, in priority order
- People they should talk to (WHO KNOWS results for their domain)
- Recent decisions that may affect their work

### M&A Integration Dashboard (`/mergers/[id]`)
- Side-by-side knowledge graphs of two merging companies
- Knowledge gap analysis: topics where one company has decisions but the other doesn't
- Expert overlap: shared domains of expertise between teams
- Contradiction detection: conflicting decisions between the two organizations
- Integration risk score based on knowledge gaps and contradictions

## 6.3 Key Components

```
components/
├── graph/
│   ├── KnowledgeGraph.tsx         # Main Sigma.js canvas
│   ├── NodeDetail.tsx             # Right-panel node inspector
│   ├── GraphFilters.tsx           # Filter controls
│   └── MiniMap.tsx                # Navigation aid
├── decisions/
│   ├── DecisionCard.tsx           # Summary card
│   ├── DecisionDetail.tsx         # Full detail view
│   ├── DecisionContext.tsx        # WHY chain visualization
│   ├── AlternativesList.tsx       # Rejected options
│   ├── StakeholderBadges.tsx      # People involved
│   └── ConsequenceChain.tsx       # What happened after
├── search/
│   ├── SearchBar.tsx              # NL input with debounce
│   ├── SearchResults.tsx          # Result cards
│   ├── ReasoningExplainer.tsx     # Shows derivation path
│   └── QuerySuggestions.tsx       # Example query chips
├── timeline/
│   ├── TimelineView.tsx           # Horizontal timeline
│   └── TimelineEvent.tsx          # Individual event marker
└── common/
    ├── ConfidenceScore.tsx         # Visual confidence indicator
    ├── SourceBadge.tsx             # GitHub/Slack/Confluence badge
    └── PersonAvatar.tsx            # With expertise tooltip
```

---

# SECTION 7: COMPLETE FILE & FOLDER STRUCTURE

```
chronos/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                            # Lint + test + typecheck
│   │   ├── deploy-frontend.yml               # Vercel/Cloud Run deploy
│   │   ├── deploy-backend.yml                # Cloud Run deploy
│   │   ├── deploy-workers.yml                # Cloud Run deploy
│   │   └── terraform-plan.yml                # Terraform plan on PR
│   └── CODEOWNERS
├── .gitignore
├── .eslintrc.js
├── .prettierrc
├── turbo.json                                # Turborepo build orchestration
├── package.json                              # Root workspace
├── pnpm-workspace.yaml
├── tsconfig.base.json
│
├── packages/
│   ├── shared-types/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── src/
│   │       ├── index.ts
│   │       ├── graph/
│   │       │   ├── nodes.ts                  # All 11 node type interfaces
│   │       │   ├── edges.ts                  # All 13 edge type interfaces
│   │       │   └── properties.ts             # Shared property types
│   │       ├── api/
│   │       │   ├── requests.ts
│   │       │   ├── responses.ts
│   │       │   └── pagination.ts
│   │       ├── ingestion/
│   │       │   ├── sources.ts
│   │       │   ├── pipeline.ts
│   │       │   └── jobs.ts
│   │       ├── extraction/
│   │       │   ├── gemini.ts
│   │       │   └── reasoning.ts
│   │       └── query/
│   │           ├── natural-language.ts
│   │           └── graph-query.ts
│   └── shared-utils/
│       ├── package.json
│       ├── tsconfig.json
│       └── src/
│           ├── index.ts
│           ├── date.ts
│           ├── text.ts
│           ├── crypto.ts
│           └── validation.ts
│
├── apps/
│   ├── web/                                  # Next.js 14 App Router frontend
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── next.config.js
│   │   ├── tailwind.config.ts
│   │   ├── .env.local.example
│   │   └── src/
│   │       ├── app/
│   │       │   ├── layout.tsx
│   │       │   ├── page.tsx
│   │       │   ├── globals.css
│   │       │   ├── (auth)/
│   │       │   │   ├── login/page.tsx
│   │       │   │   └── callback/page.tsx
│   │       │   ├── (dashboard)/
│   │       │   │   ├── layout.tsx
│   │       │   │   ├── page.tsx              # Home / overview
│   │       │   │   ├── search/page.tsx
│   │       │   │   ├── decisions/
│   │       │   │   │   ├── page.tsx
│   │       │   │   │   └── [id]/page.tsx
│   │       │   │   ├── graph/page.tsx
│   │       │   │   ├── people/
│   │       │   │   │   ├── page.tsx
│   │       │   │   │   └── [id]/page.tsx
│   │       │   │   ├── systems/
│   │       │   │   │   ├── page.tsx
│   │       │   │   │   └── [id]/page.tsx
│   │       │   │   ├── timeline/page.tsx
│   │       │   │   ├── onboarding/page.tsx
│   │       │   │   └── settings/
│   │       │   │       ├── page.tsx
│   │       │   │       ├── integrations/page.tsx
│   │       │   │       └── privacy/page.tsx
│   │       │   └── api/auth/[...nextauth]/route.ts
│   │       ├── components/
│   │       │   ├── ui/                       # shadcn/ui primitives
│   │       │   ├── layout/                   # AppShell, Sidebar, TopBar
│   │       │   ├── graph/                    # KnowledgeGraph, NodeDetail, etc.
│   │       │   ├── decisions/                # DecisionCard, DecisionContext, etc.
│   │       │   ├── search/                   # SearchBar, SearchResults, etc.
│   │       │   ├── timeline/                 # TimelineView, TimelineEvent
│   │       │   └── common/                   # ConfidenceScore, SourceBadge, etc.
│   │       ├── lib/
│   │       │   ├── api-client.ts
│   │       │   ├── auth.ts
│   │       │   ├── graph-renderer.ts
│   │       │   └── query-client.ts
│   │       ├── hooks/
│   │       │   ├── useDecision.ts
│   │       │   ├── useGraphData.ts
│   │       │   ├── useSearch.ts
│   │       │   ├── usePerson.ts
│   │       │   └── useTimeline.ts
│   │       ├── stores/
│   │       │   ├── graphStore.ts             # Zustand
│   │       │   ├── searchStore.ts
│   │       │   └── uiStore.ts
│   │       └── types/
│   │           └── next-auth.d.ts
│   │
│   ├── api/                                  # Node.js/TypeScript backend
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── Dockerfile
│   │   ├── .env.example
│   │   └── src/
│   │       ├── index.ts                      # Express entry
│   │       ├── app.ts                        # Express app factory
│   │       ├── config/
│   │       │   ├── index.ts
│   │       │   ├── neo4j.ts
│   │       │   ├── redis.ts
│   │       │   └── pubsub.ts
│   │       ├── routes/
│   │       │   ├── index.ts
│   │       │   ├── decisions.ts
│   │       │   ├── search.ts
│   │       │   ├── graph.ts
│   │       │   ├── people.ts
│   │       │   ├── systems.ts
│   │       │   ├── ingestion.ts
│   │       │   └── health.ts
│   │       ├── controllers/
│   │       │   ├── decisions.controller.ts
│   │       │   ├── search.controller.ts
│   │       │   ├── graph.controller.ts
│   │       │   ├── people.controller.ts
│   │       │   └── systems.controller.ts
│   │       ├── services/
│   │       │   ├── graph.service.ts          # Core Neo4j queries
│   │       │   ├── search.service.ts         # NL → Cypher pipeline
│   │       │   ├── embedding.service.ts
│   │       │   ├── gemini.service.ts
│   │       │   ├── cache.service.ts
│   │       │   └── auth.service.ts
│   │       ├── middleware/
│   │       │   ├── auth.middleware.ts
│   │       │   ├── ratelimit.middleware.ts
│   │       │   ├── cors.middleware.ts
│   │       │   ├── logging.middleware.ts
│   │       │   └── error.middleware.ts
│   │       ├── cypher/
│   │       │   ├── decisions.cypher.ts       # Parameterized Cypher templates
│   │       │   ├── search.cypher.ts
│   │       │   ├── graph.cypher.ts
│   │       │   ├── people.cypher.ts
│   │       │   └── timeline.cypher.ts
│   │       └── utils/
│   │           ├── neo4j-helpers.ts
│   │           ├── pagination.ts
│   │           └── response.ts
│   │
│   ├── workers/                              # Python ingestion pipeline
│   │   ├── pyproject.toml
│   │   ├── requirements.txt
│   │   ├── Dockerfile
│   │   ├── .env.example
│   │   └── src/
│   │       ├── main.py                       # Worker entry point
│   │       ├── config.py                     # Pydantic settings
│   │       ├── connectors/
│   │       │   ├── __init__.py
│   │       │   ├── base.py                   # Abstract connector
│   │       │   ├── github.py
│   │       │   ├── slack.py
│   │       │   ├── confluence.py
│   │       │   ├── jira.py
│   │       │   ├── email.py
│   │       │   ├── google_meet.py
│   │       │   ├── google_docs.py
│   │       │   └── zoom.py
│   │       ├── pipeline/
│   │       │   ├── __init__.py
│   │       │   ├── orchestrator.py
│   │       │   ├── stages/
│   │       │   │   ├── __init__.py
│   │       │   │   ├── fetch.py              # Stage 1
│   │       │   │   ├── normalize.py          # Stage 2
│   │       │   │   ├── privacy_filter.py     # Stage 3
│   │       │   │   ├── chunk.py              # Stage 4
│   │       │   │   ├── embed.py              # Stage 5
│   │       │   │   ├── extract.py            # Stage 6
│   │       │   │   └── graph_write.py        # Stage 7
│   │       │   └── incremental.py
│   │       ├── extraction/
│   │       │   ├── __init__.py
│   │       │   ├── gemini_client.py
│   │       │   ├── prompts/
│   │       │   │   ├── __init__.py
│   │       │   │   ├── decision_detect.py    # Detection prompts
│   │       │   │   ├── rationale.py          # Master extraction prompt
│   │       │   │   ├── stakeholder.py
│   │       │   │   ├── alternatives.py
│   │       │   │   ├── consequences.py
│   │       │   │   └── context_chain.py
│   │       │   ├── classifiers/
│   │       │   │   ├── __init__.py
│   │       │   │   ├── decision_classifier.py
│   │       │   │   └── sentiment_classifier.py
│   │       │   └── entity_resolver.py
│   │       ├── graph/
│   │       │   ├── __init__.py
│   │       │   ├── neo4j_client.py
│   │       │   ├── node_writer.py
│   │       │   ├── edge_writer.py
│   │       │   └── schema_migrations.py
│   │       ├── privacy/
│   │       │   ├── __init__.py
│   │       │   ├── pii_detector.py           # Presidio integration
│   │       │   ├── redactor.py
│   │       │   └── sensitivity_classifier.py
│   │       └── utils/
│   │           ├── __init__.py
│   │           ├── text.py
│   │           ├── date_parser.py
│   │           └── logger.py
│   │
│   └── extension/                            # Chrome Extension MV3
│       ├── manifest.json
│       ├── package.json
│       ├── tsconfig.json
│       ├── webpack.config.js
│       ├── public/
│       │   ├── icon-16.png
│       │   ├── icon-48.png
│       │   └── icon-128.png
│       └── src/
│           ├── background/
│           │   ├── service-worker.ts
│           │   ├── message-handler.ts
│           │   └── auth-handler.ts
│           ├── content/
│           │   ├── index.ts
│           │   ├── context-detector.ts
│           │   ├── sidebar/
│           │   │   ├── Sidebar.tsx
│           │   │   ├── DecisionPanel.tsx
│           │   │   ├── PersonPanel.tsx
│           │   │   └── QueryPanel.tsx
│           │   └── highlight/
│           │       ├── highlighter.ts
│           │       └── tooltip.tsx
│           ├── popup/
│           │   ├── index.html
│           │   ├── Popup.tsx
│           │   └── QuickSearch.tsx
│           └── shared/
│               ├── api-client.ts
│               ├── auth.ts
│               └── storage.ts
│
├── infrastructure/
│   ├── terraform/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── providers.tf                      # GCP provider
│   │   ├── backend.tf                        # GCS remote state
│   │   ├── modules/
│   │   │   ├── networking/                   # VPC, subnets, firewall
│   │   │   ├── neo4j/                        # Neo4j Aura or GCE
│   │   │   ├── cloud-run/                    # API + workers
│   │   │   ├── pubsub/                       # Topics & subscriptions
│   │   │   ├── redis/                        # Memorystore
│   │   │   ├── storage/                      # GCS buckets
│   │   │   └── iam/                          # Service accounts + roles
│   │   └── environments/
│   │       ├── dev/
│   │       └── prod/
│   └── docker-compose.yml                    # Local dev: Neo4j + Redis
│
├── scripts/
│   ├── mock-data/
│   │   ├── generate.ts                       # Master data generator
│   │   ├── seed-neo4j.ts                     # Seed script
│   │   ├── techflow/
│   │   │   ├── employees.ts                  # 50 employees
│   │   │   ├── decisions.ts                  # 8 key decision stories
│   │   │   ├── systems.ts                    # 20 systems
│   │   │   ├── events.ts                     # Timeline events
│   │   │   ├── documents.ts                  # Fake docs/RFCs
│   │   │   ├── meetings.ts                   # Meeting records
│   │   │   ├── threads.ts                    # Slack/email threads
│   │   │   └── acquisition.ts               # M&A scenario
│   │   └── fixtures/
│   │       ├── slack-messages.json
│   │       ├── github-prs.json
│   │       ├── confluence-pages.json
│   │       └── jira-issues.json
│   ├── local-dev/
│   │   ├── setup.sh                          # First-time setup
│   │   ├── start-services.sh
│   │   └── reset-db.sh
│   └── build/
│       ├── build-extension.sh
│       └── release.sh
│
└── docs/
    ├── architecture.md
    ├── api-reference.md
    ├── graph-schema.md
    └── deployment.md
```

---

# SECTION 8: THE HACKATHON DEMO PLAN

## 8.1 Mock Company: TechFlow Inc.

**Profile**: B2B SaaS for supply chain visibility. Founded Feb 2022. Series B ($45M). 50 employees. Acquired "LogiSense" (8-person ML startup) Oct 2024.

### Pre-loaded Dataset

- **50 Person nodes** across Engineering (24), Product (6), Sales/CS (7), Ops (4), Executive (5), Acquired (4)
- **20 System nodes** including legacy monolith (deprecated), microservices, acquired ML model
- **8 richly detailed Decision stories** (see Section 8.2)
- **~30 Document nodes** (mock Confluence pages, GitHub PRs, Slack threads)
- **~20 Event nodes** (incidents, launches, acquisitions, departures)
- **~80+ edges** connecting everything

### 8 Key Decision Stories (Pre-loaded with Full Rationale)

1. **Microservices Migration** (Mar 2023) — Triggered by 3 P1 incidents in one month. Priya Nair decided. David Park dissented ("we don't have the ops maturity"). Result: deploy time from 4 hours to 12 minutes.

2. **PostgreSQL over MongoDB** (May 2023) — Relational user model, ACID for billing, team expertise (3 PostgreSQL vs 1 MongoDB), enterprise security review requirements. Elena Vasquez decided.

3. **Elasticsearch for Search** (Aug 2023) — 65ms vs 4200ms POC results. Jordan Kim initially opposed ("keep stack simple"), reversed after seeing data. Anika Patel drove it.

4. **React Native for Mobile** (Jan 2024) — 70% of team knew JS/TS. Flutter would require hiring Dart experts (4-month delay). Accepted known performance tradeoffs.

5. **LogiSense Acquisition Integration** (Nov 2024) — Rewrite API layer, keep model weights. Flask app failed 7 security criteria. Board demo deadline forced timeline.

6. **AWS to GCP Migration** (Apr 2024) — $67K/month AWS bill. GKE Autopilot eliminated over-provisioning. $250K GCP credits. Strategic Google partnership for enterprise sales.

7. **Monolith Deprecation** (Sep 2024) — Only 3 customers on free plans. Python 3.8 EOL, 34 CVEs, $4,200/month to keep running. **Key demo**: David Park (departed) built undocumented order normalization logic.

8. **Data Governance Post-Acquisition** (Feb 2025) — 23% training data overlap with customer data. GDPR/CCPA implications. Up to $2.8M breach fines. Marcus Webb (CEO) decided.

## 8.2 Demo Script (5 minutes)

**Minute 0-1: The Problem Setup**
> "David Park was TechFlow's senior engineer for 2 years. He built critical parts of the order normalization system. In July 2024, he left for a competitor. Three months later, during the monolith shutdown, the team discovered undocumented edge cases in his code. Without CHRONOS, they spent 3 weeks reverse-engineering from source. With CHRONOS:"

**Minute 1-2: The "Why" Query**
- Type: *"Why did we choose PostgreSQL over MongoDB?"*
- CHRONOS returns: the rationale (relational model, ACID, team expertise, compliance), the stakeholders (Elena decided, Carlos influenced, Arjun flagged compliance), the rejected alternatives with reasons, and links to the original PR and Confluence ADR.

**Minute 2-3: The "Who Knows" Query**
- Type: *"Who knows about the order normalization logic?"*
- CHRONOS returns: David Park (departed, expertise score 8.7), Elena Vasquez (reverse-engineered it, score 5.2), Carlos Torres (reviewed his PRs, score 3.1). Shows David's old PR comments and Slack messages explaining his design choices.

**Minute 3-4: The Knowledge Graph + Timeline**
- Navigate the interactive graph. Click "Microservices Migration" → see the chain: 3 P1 incidents → migration decision → billing-service extraction → deploy time improvement → infra cost spike → GCP migration decision. Show how one decision cascades.
- Timeline view: 2022-2025, all major decisions, causal arrows.

**Minute 4-5: The Chrome Extension + M&A Value**
- Open a GitHub PR page → side panel auto-shows relevant architectural decisions
- Show the "New Employee Onboarding" view: here's what Marcus Chen (joined Jan 2025) needs to know about the codebase, prioritized by relevance to his team
- Flash the M&A scenario: merging LogiSense's knowledge graph with TechFlow's, showing knowledge gaps

## 8.3 Demo Queries That Must Work

1. "Why did we move away from the monolith?"
2. "Who made the decision to use PostgreSQL and why not MongoDB?"
3. "What led to us using Elasticsearch?"
4. "Who built the order normalization logic in the old system?"
5. "Why did we switch from AWS to GCP?"
6. "What were the reasons behind the LogiSense integration strategy?"
7. "Who knows the most about the search architecture?"
8. "What decisions did David Park influence before he left?"
9. "Show me all decisions made in 2024"
10. "Why did we choose React Native for mobile?"
11. "What are the data privacy implications of the LogiSense integration?"
12. "What did Jordan Kim decide and why did he originally oppose the search service?"

---

# SECTION 9: 24-HOUR BUILD SPRINT PLAN

## Pre-Hackathon (Before Event)

**MUST BE DONE BEFORE THE 24 HOURS START:**
- [ ] Mock TechFlow dataset generated and saved as JSON fixtures
- [ ] Neo4j seed script tested locally
- [ ] All API keys provisioned (Gemini, Neo4j Aura, GCP project)
- [ ] Monorepo scaffold created with `pnpm init` + `turbo.json`
- [ ] Docker Compose file for Neo4j + Redis tested

## Team Allocation (4 people)

| Person | Role | Ownership |
|---|---|---|
| **P1** | Frontend | Next.js app, graph viz, search UI, timeline |
| **P2** | Backend/Infra | Express API, Cypher queries, Neo4j, Cloud Run |
| **P3** | ML/Python | Ingestion workers, Gemini prompts, extraction pipeline |
| **P4** | Full-stack Flex | Chrome extension, mock data, integration testing, demo prep |

## Hour-by-Hour Sprint

| Hours | P1 (Frontend) | P2 (Backend) | P3 (ML/Python) | P4 (Flex) |
|---|---|---|---|---|
| **0-1** | Next.js scaffold, Tailwind + shadcn setup, route structure | Express scaffold, Neo4j driver, route stubs | Python project setup, Gemini client wrapper | Docker Compose up, seed Neo4j with mock data |
| **1-3** | Dashboard shell, Sidebar, Decision card component | `GET /decisions/:id`, `GET /graph/neighborhood/:id`, `POST /search` (stub) | Manually craft 8 decision stories as pre-extracted JSON | Seed script: 50 Person, 30 Decision, 15 System, 20 Doc, 80+ edges. Verify in Neo4j Browser |
| **3-5** | Decision detail page: rationale card, stakeholders, alternatives | Implement search: Gemini query understanding → Cypher → synthesis | Implement Gemini query parser prompt, test with 10 sample queries | Integration testing: frontend ↔ backend. Fix CORS, types |
| **5-7** | Knowledge graph (Sigma.js): force layout, click-to-navigate | Graph neighborhood API: n-hop subgraph as JSON | Decision detection classifier (rule-based + Flash) | Begin Chrome extension: MV3 scaffold, service worker, popup |
| **7-9** | Search UI: SearchBar, SearchResults, ReasoningExplainer | WHO KNOWS Cypher query + expertise scoring | Full extraction prompt on 3-5 fixture docs, write to Neo4j | Extension: content script for GitHub PR detection, side panel |
| **9-11** | Timeline view: horizontal scroll, event markers, causal arrows | Timeline Cypher query, Person expertise API | Consequence tracking prompt, contradiction detection | Extension: API integration, show related decisions in panel |
| **11-13** | Person knowledge profile page | Polish search: add caching (Redis 5-min TTL), error handling | Stakeholder attribution pipeline refinement | End-to-end demo walkthrough #1. Bug list. |
| **13-15** | Loading states, error states, empty states, responsive fixes | Neo4j indexes: Decision.orgId+decisionDate, Person.email, System.name | Improve extraction quality based on demo walkthrough | Fix bugs from walkthrough. Seed additional decisions if needed. |
| **15-17** | Onboarding page: "30-60-90 day knowledge guide" | Ingestion trigger API: POST /ingestion/trigger | Slack fixture connector: read from local JSON, run full pipeline | Polish Chrome extension UI |
| **17-19** | Graph explorer filters, mini-map, legend | Privacy settings API endpoints | Privacy filter: PII detection with Presidio on fixture data | Settings page: integration toggles, privacy controls |
| **19-21** | Final UI polish, animations, dark mode toggle | Load testing: verify 20 demo queries return correct results | Fine-tune extraction prompts based on result quality | End-to-end demo walkthrough #2. Fix remaining bugs. |
| **21-23** | **Stretch**: contradiction alert widget, export decision as PDF | **Stretch**: Slack bot `/chronos` command | **Stretch**: Live Gemini extraction from real GitHub repo | **Stretch**: M&A dashboard with side-by-side graphs |
| **23-24** | Screenshot backup of all pages | README with setup instructions | Final prompt quality verification | Demo script rehearsal. Record 2-min backup video. |

## MVP Must-Work Checklist

- [ ] Pre-loaded TechFlow mock data in Neo4j (50 people, 30 decisions, 20 systems, 80+ edges)
- [ ] Natural language search returning sourced answers for 12 demo queries
- [ ] Decision detail page with rationale, stakeholders, alternatives, consequence chain
- [ ] Knowledge graph visualization (2-hop neighborhood, click to navigate)
- [ ] WHO KNOWS feature ("Who knows about the search architecture?")
- [ ] Timeline view for 2022-2025
- [ ] Person knowledge profile for at least 3 key people

## Stretch Goals (Priority Order)

1. Chrome extension showing context on GitHub PRs
2. Live Gemini extraction from fixture documents
3. Contradiction detection alerts
4. Onboarding mode for new employees
5. Slack bot integration
6. M&A knowledge gap dashboard

---

# SECTION 10: PRODUCTION & SCALE

## 10.1 Deployment Architecture

```
                    ┌─── Cloudflare CDN ───┐
                    │                       │
                    ▼                       ▼
           ┌──────────────┐      ┌──────────────────┐
           │ Next.js Web  │      │ Chrome Extension  │
           │ (Vercel/     │      │ (Chrome Web Store)│
           │  Cloud Run)  │      └────────┬─────────┘
           └──────┬───────┘               │
                  │                        │
                  ▼                        ▼
         ┌────────────────────────────────────┐
         │     Cloud Run — API Service        │
         │     (Node.js, autoscaling 0-100)   │
         └──────────┬──────────┬──────────────┘
                    │          │
         ┌──────────▼──┐  ┌───▼──────────────┐
         │  Neo4j Aura │  │  Redis            │
         │  (Managed)  │  │  (Memorystore)    │
         └─────────────┘  └──────────────────┘
                    │
         ┌──────────▼─────────────────────────┐
         │   Cloud Run — Worker Services      │
         │   (Python, autoscaling 0-50)       │
         │   ┌──────────┐ ┌──────────┐        │
         │   │ Ingestor │ │ Extractor│        │
         │   └────┬─────┘ └────┬─────┘        │
         │        │            │               │
         │   ┌────▼────────────▼────┐          │
         │   │     Cloud Pub/Sub    │          │
         │   └──────────────────────┘          │
         └─────────────────────────────────────┘
```

## 10.2 Scaling Strategy

| Component | Scaling Mechanism | Target |
|---|---|---|
| API Service | Cloud Run autoscaling (0-100 instances, CPU-based) | <200ms p95 latency |
| Ingestion Workers | Cloud Run Jobs, Pub/Sub-triggered, concurrent workers | Process 100K docs/day |
| Neo4j | Aura Professional → Business Critical (3-zone cluster) | 99.95% SLA |
| Redis | Memorystore Standard (replica) | Sub-ms cache hits |
| Vector Search | Vertex AI Matching Engine (auto-scaling) | <50ms ANN queries |
| Frontend | Vercel Edge Network or Cloud CDN | Global <100ms TTFB |

## 10.3 Enterprise Requirements

- **SOC 2 Type II**: 8-15 month certification timeline. Start from Day 1: audit logging, access controls, encryption at rest + transit, incident response procedures.
- **On-premise deployment**: Offer Kubernetes Helm chart for sensitive industries (finance, defense, healthcare). Neo4j Enterprise on-prem + self-hosted Gemini via Vertex AI on GKE.
- **SSO**: SAML 2.0 and OIDC support via NextAuth.js enterprise providers (Okta, Azure AD, Google Workspace).
- **Data residency**: GCP region selection per customer. EU customers → `europe-west1`. Configurable per-tenant.
- **Audit trail**: Every graph read/write logged to BigQuery with timestamp, user, query, results count. 7-year retention.

---

# SECTION 11: TECHNICAL RISKS & MITIGATIONS

| Risk | Severity | Likelihood | Mitigation |
|---|---|---|---|
| **Gemini hallucinating decision rationales** | Critical | Medium | Confidence scoring + verbatim evidence quotes required. Low-confidence extractions queued for human review. Never display without source attribution. |
| **Data source access complexity** (Slack Enterprise Grid, GitHub Enterprise require admin approval) | High | High | Start with standard tiers. Offer "manual upload" mode for restrictive orgs (CSV/JSON import). Build admin onboarding wizard. |
| **Knowledge graph consistency** (duplicate entities, broken links) | Medium | High | Entity resolution pipeline with multi-signal matching. Weekly consistency audit job. Admin merge/split UI. |
| **Privacy/legal concerns about employee monitoring** | Critical | High | Strong opt-out. Only shared channels/inboxes. Never personal DMs. SOC 2 certification. DPA with every customer. Transparent data handling docs. |
| **Gemini rate limits during bulk ingestion** | Medium | Medium | Exponential backoff. Queue-based processing. Context caching for repeated org contexts. Budget Flash for classification, Pro only for extraction. |
| **Neo4j performance at scale** (>10M nodes) | Medium | Low | Query profiling from day 1. Composite indexes. Consider BigQuery Graph for analytics-heavy queries. Shard by orgId at extreme scale. |
| **Chrome Extension MV3 service worker limitations** | Low | Medium | All state in Chrome Storage API. Robust message-passing. No reliance on persistent background state. |
| **Slack API rate limit changes** (2025-2026 restrictions on non-Marketplace apps) | Medium | High | Apply for Slack Marketplace listing. Internal/customer-built apps get 50+ RPM (not affected by changes). Webhook-first architecture reduces API calls. |
| **Stale knowledge graph** (decisions extracted months after they were made) | Low | Medium | Real-time webhook processing for new content. Monthly re-extraction sweep for missed decisions. "Last updated" badges in UI. |

---

# SECTION 12: BUSINESS MODEL

## 12.1 Pricing

| Tier | Price | Features |
|---|---|---|
| **Starter** | $25/seat/month | 3 data sources, 10K documents, basic search, 5 users min |
| **Professional** | $50/seat/month | All data sources, unlimited documents, graph explorer, Chrome extension, API access, 25 users min |
| **Enterprise** | Custom ($75-100/seat) | On-premise option, SSO/SAML, custom retention, dedicated support, SLA, M&A module |

## 12.2 Revenue Streams

1. **Core SaaS** ($50/seat/month) — Primary revenue. Target: 100 companies × 200 seats × $50 = $1M MRR in Year 2.
2. **Professional Services** ($250/hr) — Implementation, data source integration, custom connectors, knowledge graph tuning.
3. **M&A Knowledge Audit** ($50K-$200K per engagement) — Pre-acquisition due diligence: map the target company's institutional knowledge. Post-acquisition: merge knowledge graphs, identify gaps.
4. **Talent Retention ROI Calculator** (Free tool, lead generation) — Upload your turnover data, we estimate your knowledge loss cost. Leads to sales conversations.

## 12.3 Unit Economics

| Metric | Value |
|---|---|
| **COGS per seat** (Gemini API + infrastructure) | ~$10/seat/month |
| **Gross margin** | ~80% |
| **Target CAC** | $5,000 (enterprise sales motion) |
| **Target ACV** | $120,000 (200 seats × $50 × 12 months) |
| **LTV/CAC ratio** | >10x (enterprise contracts are 3+ year with expansion) |
| **Payback period** | <6 months |

## 12.4 Go-to-Market

**Phase 1 (Months 1-6)**: Target 50-200 person engineering teams experiencing knowledge loss from attrition. Offer free pilot (30 days, 3 data sources). Vertical: B2B SaaS companies with >30% annual engineering turnover.

**Phase 2 (Months 6-12)**: Expand to M&A-active companies. Partner with M&A advisory firms (Deloitte, McKinsey tech practices) as a due diligence tool.

**Phase 3 (Year 2+)**: Enterprise tier launch. Regulated industries (finance, healthcare) with on-premise option. Government contracting (FedRAMP pathway).

---

# SECTION 13: JUDGING CRITERIA + PITCH STRUCTURES

## 13.1 Hackathon Judging Criteria Alignment

| Criteria | CHRONOS Strength | Demo Proof |
|---|---|---|
| **Innovation** | First system to extract decision *reasoning* (not just records) using LLM analysis across multi-source corporate data | Show the rationale extraction: "why we chose PostgreSQL" with full reasoning chain, alternatives, and dissent |
| **Technical Complexity** | Knowledge graph + LLM extraction + multi-source ingestion + NL query engine + Chrome extension | Live graph visualization, multi-hop query traversal, real-time context surfacing |
| **Practical Impact** | $47B problem. Every company with >50 employees has this pain. | "David Park left and took 2 years of context. CHRONOS preserved it." |
| **Google API Usage** | Gemini 1.5 Pro (core engine), Vertex AI Embeddings, Speech-to-Text, Document AI, Workspace APIs, BigQuery, Cloud Run, Pub/Sub, GCS | API integration map showing 10+ Google APIs working in concert |
| **Completeness** | Full working product: search, graph, timeline, person profiles, Chrome extension | End-to-end demo: ask question → get sourced answer → explore graph → find expert |
| **Polish** | Professional UI, loading states, error handling, responsive design | Clean dashboard with shadcn/ui components, smooth graph animations |

## 13.2 Pitch Structures

### 60-Second Elevator Pitch
> "Every year, companies lose $47 billion when employees leave and take their knowledge with them. Notion and Confluence store WHAT was decided — but never WHY. CHRONOS changes that. We ingest your Slack, GitHub, Confluence, and meeting transcripts, and use Gemini's 2-million-token context window to extract the reasoning behind every decision your company has ever made. The result is a living knowledge graph you can query in natural language: 'Why did we choose PostgreSQL?' 'Who knows about the payment system?' 'What led to this architecture?' When your best engineer leaves tomorrow, their knowledge stays forever."

### 3-Minute Technical Pitch
1. **Problem** (30s): The David Park story — senior engineer leaves, team spends 3 weeks reverse-engineering his code
2. **Insight** (15s): Existing tools capture records, not reasoning. The WHY is trapped in Slack threads and meeting transcripts.
3. **Solution** (30s): CHRONOS ingests multi-source data → Gemini extracts decision reasoning → builds a knowledge graph → queryable in natural language
4. **Demo** (90s): Live demo — search query → answer with sources → graph visualization → Chrome extension
5. **Google APIs** (15s): Powered by 10+ Google APIs — Gemini 1.5 Pro's 2M context window is the key technical advantage
6. **Market** (15s): $50/seat/month. TAM: every company with >50 employees. $47B problem.

### One-Slide Summary
```
CHRONOS — Institutional Knowledge Immortality Engine

PROBLEM: $47B/year lost when employees leave with undocumented reasoning
INSIGHT: Tools capture WHAT was decided. CHRONOS captures WHY.
HOW:     Slack + GitHub + Confluence + Meetings
         → Gemini 1.5 Pro reasoning extraction
         → Knowledge graph (Decision → Person → System → Outcome)
         → Natural language query engine
DEMO:    "Why did we choose PostgreSQL?" → Full rationale + stakeholders + alternatives
MARKET:  $50/seat/month · Every company >50 employees · $47B TAM
GOOGLE:  Gemini 1.5 Pro · Vertex AI · Speech-to-Text · Document AI · Cloud Run
```

---

# SECTION 14: APPENDICES

## Appendix A: Chrome Extension Manifest

```json
{
  "manifest_version": 3,
  "name": "CHRONOS — Institutional Knowledge",
  "version": "1.0.0",
  "description": "Surface institutional knowledge on any page you visit",
  "permissions": ["storage", "activeTab", "sidePanel"],
  "host_permissions": [
    "https://*.github.com/*",
    "https://*.atlassian.net/*",
    "https://mail.google.com/*",
    "https://docs.google.com/*"
  ],
  "background": {
    "service_worker": "background/service-worker.js",
    "type": "module"
  },
  "content_scripts": [
    {
      "matches": ["https://*.github.com/*", "https://*.atlassian.net/*"],
      "js": ["content/index.js"],
      "css": ["content/styles.css"]
    }
  ],
  "side_panel": {
    "default_path": "sidepanel/index.html"
  },
  "action": {
    "default_popup": "popup/index.html",
    "default_icon": {
      "16": "icon-16.png",
      "48": "icon-48.png",
      "128": "icon-128.png"
    }
  },
  "icons": {
    "16": "icon-16.png",
    "48": "icon-48.png",
    "128": "icon-128.png"
  }
}
```

## Appendix B: Sample Query → Answer Flow

**User query**: *"Why did we choose Elasticsearch instead of just using PostgreSQL full-text search?"*

**Phase 1 — Query Understanding** (Gemini):
```json
{
  "queryType": "RATIONALE_DEEP_DIVE",
  "entities": [
    {"name": "Elasticsearch", "type": "SYSTEM"},
    {"name": "PostgreSQL full-text search", "type": "CONCEPT"}
  ],
  "relationshipFocus": ["was_because_of", "led_to", "made_by"],
  "depth": 2
}
```

**Phase 2 — Entity Resolution**: Matches "Elasticsearch" → System node `search-service`. Matches "PostgreSQL full-text search" → Alternative node "PG full-text search" linked to Decision "Elasticsearch for Search".

**Phase 3 — Cypher**:
```cypher
MATCH (d:Decision {id: "DECISION-003"})
MATCH (d)-[:made_by]->(p:Person)
MATCH (d)-[:was_because_of]->(cause)
OPTIONAL MATCH (d)<-[:proposed]-(alt:Alternative)
OPTIONAL MATCH (d)-[:led_to]->(outcome:Outcome)
OPTIONAL MATCH (diss:Person)-[dis:dissented_from]->(d)
RETURN d, collect(p), collect(cause), collect(alt), collect(outcome), collect({person: diss, dissent: dis})
```

**Phase 5 — Synthesized Answer**:
> **TechFlow chose Elasticsearch over PostgreSQL full-text search in August 2023** primarily because of a conclusive performance proof-of-concept by Anika Patel that showed **65ms query response time vs 4,200ms** for the same search queries on PostgreSQL.
>
> The decision was driven by product requirements: Ben Nakamura (Senior PM) was receiving constant feedback about slow search (>4 seconds) and poor result quality. The product roadmap also included multi-language support and fuzzy search for supplier names, where typos are common in logistics data — capabilities that PostgreSQL's `tsvector` couldn't match.
>
> **Jordan Kim (VP Engineering) initially opposed this**, pushing to keep PostgreSQL full-text search to maintain stack simplicity. He reversed his position after seeing the POC numbers, writing in Slack: *"OK I was wrong, those numbers are not even close. Ship it."*
>
> Other alternatives considered: **Algolia** (rejected at $8,400/month projected cost — TechFlow was pre-Series A) and **Typesense** (rejected due to lack of team experience; Anika had used Elasticsearch at her previous job).
>
> **Outcome**: Search satisfaction scores improved from 2.8/5 to 4.4/5 in NPS surveys.
>
> *Sources: Slack #eng-general Aug 14 2023, GitHub PR #891*

## Appendix C: Neo4j Schema Initialization

```cypher
// Constraints
CREATE CONSTRAINT decision_id IF NOT EXISTS FOR (d:Decision) REQUIRE d.id IS UNIQUE;
CREATE CONSTRAINT person_id IF NOT EXISTS FOR (p:Person) REQUIRE p.id IS UNIQUE;
CREATE CONSTRAINT system_id IF NOT EXISTS FOR (s:System) REQUIRE s.id IS UNIQUE;
CREATE CONSTRAINT concept_id IF NOT EXISTS FOR (c:Concept) REQUIRE c.id IS UNIQUE;
CREATE CONSTRAINT event_id IF NOT EXISTS FOR (e:Event) REQUIRE e.id IS UNIQUE;
CREATE CONSTRAINT document_id IF NOT EXISTS FOR (d:Document) REQUIRE d.id IS UNIQUE;

// Indexes for query performance
CREATE INDEX decision_org_date IF NOT EXISTS FOR (d:Decision) ON (d.orgId, d.decisionDate);
CREATE INDEX person_email IF NOT EXISTS FOR (p:Person) ON (p.email);
CREATE INDEX person_org IF NOT EXISTS FOR (p:Person) ON (p.orgId);
CREATE INDEX system_name IF NOT EXISTS FOR (s:System) ON (s.name, s.orgId);
CREATE INDEX document_source IF NOT EXISTS FOR (d:Document) ON (d.sourceSystem, d.sourceId);
CREATE INDEX document_hash IF NOT EXISTS FOR (d:Document) ON (d.contentHash);

// Full-text indexes for search
CREATE FULLTEXT INDEX decision_search IF NOT EXISTS
  FOR (d:Decision) ON EACH [d.title, d.summary, d.rationale];
CREATE FULLTEXT INDEX person_search IF NOT EXISTS
  FOR (p:Person) ON EACH [p.fullName, p.displayName];
CREATE FULLTEXT INDEX system_search IF NOT EXISTS
  FOR (s:System) ON EACH [s.name, s.displayName, s.description];
CREATE FULLTEXT INDEX concept_search IF NOT EXISTS
  FOR (c:Concept) ON EACH [c.name, c.description];
```

## Appendix D: Verification Plan

### How to Test End-to-End

1. **Local setup**: `docker compose up` (Neo4j + Redis) → `pnpm run seed` → `pnpm run dev`
2. **Verify graph**: Open Neo4j Browser (`localhost:7474`), run `MATCH (n) RETURN count(n)` — expect 150+ nodes
3. **Verify API**: `curl localhost:3001/api/v1/health` → `{"status": "ok"}`
4. **Verify search**: `curl -X POST localhost:3001/api/v1/search -d '{"query": "Why PostgreSQL?"}'` → returns Decision-002 with rationale
5. **Verify frontend**: Open `localhost:3000`, navigate to search, type "Why did we choose PostgreSQL?" — see synthesized answer with source cards
6. **Verify graph viz**: Navigate to `/graph`, click any Decision node — see 2-hop neighborhood with colored nodes and labeled edges
7. **Verify timeline**: Navigate to `/timeline`, filter to 2024 — see 6+ events with causal arrows
8. **Verify Chrome extension**: Load unpacked extension, open any GitHub PR → side panel shows related decisions
9. **Run all 12 demo queries**: Each must return a relevant, sourced answer within 5 seconds

---

# CRITICAL FILES FOR IMPLEMENTATION

| File | Why It Matters |
|---|---|
| `packages/shared-types/src/graph/nodes.ts` | Core type definitions for all 11 node types; every package depends on these |
| `apps/workers/src/extraction/prompts/decision_detect.py` | The Gemini prompt engineering is the intellectual core — master extraction prompt + classifier |
| `apps/api/src/services/search.service.ts` | Orchestrates the 5-phase NL→Cypher pipeline |
| `apps/api/src/cypher/decisions.cypher.ts` | Parameterized Cypher templates including multi-hop and WHO KNOWS |
| `scripts/mock-data/techflow/decisions.ts` | Pre-seeded TechFlow decision stories — the demo's backbone |
| `apps/web/src/components/graph/KnowledgeGraph.tsx` | Sigma.js graph visualization — the visual "wow factor" |
| `apps/extension/manifest.json` | Chrome Extension MV3 configuration |
| `apps/workers/src/pipeline/orchestrator.py` | 7-stage pipeline orchestration |
