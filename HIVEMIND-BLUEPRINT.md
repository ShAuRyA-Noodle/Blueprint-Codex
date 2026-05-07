# HIVEMIND: Master Technical & Strategic Blueprint

> **Transforms any group of humans — a company, a city, a government — into a superintelligent decision-making organism.**

---

## SECTION 1: PROBLEM DEFINITION & VISION

### The $9.8 Trillion Problem
- **Corporate decision failure cost:** McKinsey estimates organizations waste $9.8T/year on poor strategic decisions. 60% of executives report that bad decision-making is as frequent as good decision-making.
- **The HiPPO Problem:** "Highest Paid Person's Opinion" dominates 70%+ of corporate decisions. Research (Kahneman, Sibony & Sunstein, *Noise*, 2021) shows expert judgment is riddled with noise — the same expert gives different answers on different days.
- **The Survey Failure:** 5-point Likert scales flatten nuance into meaningless averages. A "3.7 satisfaction score" tells you nothing. Response bias, social desirability bias, and anchoring effects corrupt data. SurveyMonkey collects data but extracts zero collective intelligence.
- **The Meeting Failure:** In meetings, the loudest voices win. Studies show 3 people dominate 70% of speaking time. Groupthink (Janis, 1972) causes information cascading — early opinions anchor the group. Introverts, remote participants, and junior employees self-censor.
- **The Tyranny of the Majority:** Simple majority voting buries minority wisdom. The best idea in the room often belongs to the person who didn't speak.

### The Academic Foundation
- **Condorcet's Jury Theorem (1785):** If each individual is >50% likely to be correct, the group's majority decision approaches 100% accuracy as group size increases. BUT — this breaks down when opinions are correlated (groupthink), when the question isn't binary, or when competence varies wildly.
- **Surowiecki's Wisdom of Crowds (2004):** Groups outperform individuals when four conditions hold: diversity of opinion, independence of judgment, decentralization, and proper aggregation. Current tools (meetings, surveys, polls) violate ALL FOUR.
- **Swarm Intelligence:** Ant colonies, bee swarms, and fish schools make collectively intelligent decisions through simple local interactions. Unanimous.ai demonstrated that human "swarms" outperform polls by 2-3x on prediction accuracy.
- **Pol.is (2014):** Proved that opinion clustering on a 2D landscape reveals hidden consensus structures. Used by Taiwan's vTaiwan platform for national policy deliberation. Limitation: requires binary agree/disagree on specific statements — can't handle open-ended text.

### Where Existing Tools Fail

| Tool | What It Does | Where It Fails |
|------|-------------|----------------|
| SurveyMonkey | Collects structured responses | Zero intelligence extraction. Averages flatten signal. |
| Slido | Live Q&A + polls | Upvoting creates popularity bias, not wisdom extraction. |
| Mentimeter | Real-time word clouds | Word clouds are noise visualization, not signal extraction. |
| Pol.is | 2D opinion clustering | Binary agree/disagree only. No open-ended text. No synthesis. |
| Loomio | Deliberative decision-making | Slow, text-heavy. No AI. Doesn't scale past 50 people. |
| Unanimous.ai | Swarm voting | Single-question focus. No semantic analysis. No text processing. |

### The HIVEMIND Insight
**AI as a group intelligence amplifier.** Instead of asking humans to compress their thinking into 5-point scales or binary votes, let them express full, nuanced thoughts in natural language. Then use AI to:
1. Understand the semantic meaning of every response
2. Cluster responses by meaning (not keyword)
3. Identify the signal — high-specificity, high-insight contributions
4. Surface dissent that the majority would bury
5. Synthesize thousands of voices into actionable collective wisdom
6. Visualize the group's "belief landscape" as a navigable map

---

## SECTION 2: CORE TECHNICAL ARCHITECTURE

### 2.1 System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     INPUT COLLECTION LAYER                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────────────┐  │
│  │  Web UI   │ │  Google  │ │  Voice   │ │  Meeting Transcript│  │
│  │(Realtime) │ │  Forms   │ │(STT API) │ │  (Post-meeting)   │  │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────────┬──────────┘  │
│       │             │            │                 │             │
│       └─────────────┴────────┬───┴─────────────────┘             │
│                              ▼                                   │
│              ┌──────────────────────────┐                        │
│              │   Cloud Translation API   │                        │
│              │   (if non-English input)  │                        │
│              └────────────┬─────────────┘                        │
└───────────────────────────┼──────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                   NLP PROCESSING PIPELINE                        │
│                    (FastAPI Backend)                              │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Gemini Flash: Sentiment · Specificity · Argument Extract  │  │
│  │  Gemini Embeddings: 3072-dim semantic vector per response  │  │
│  └────────────────────────┬───────────────────────────────────┘  │
│                           ▼                                      │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Manipulation Detection: timing, similarity, IP analysis   │  │
│  └────────────────────────┬───────────────────────────────────┘  │
└───────────────────────────┼──────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                  SEMANTIC CLUSTERING ENGINE                       │
│  ┌───────────────┐  ┌─────────────┐  ┌───────────────────────┐  │
│  │ pgvector store │→│ UMAP reduce │→│ HDBSCAN cluster        │  │
│  │ (3072-dim)    │  │ (3072→2D)   │  │ (auto cluster count)  │  │
│  └───────────────┘  └─────────────┘  └───────────┬───────────┘  │
│                                                   ▼              │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Gemini Flash: cluster labeling, cross-cluster tensions    │  │
│  │  Dissent cluster ID · Hidden agreement detection           │  │
│  └────────────────────────┬───────────────────────────────────┘  │
└───────────────────────────┼──────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     SYNTHESIS ENGINE                             │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Gemini Pro: 3-5 actionable insights + confidence scores   │  │
│  │  Hidden agreements · Blocking minorities · Evidence quotes  │  │
│  └────────────────────────┬───────────────────────────────────┘  │
└───────────────────────────┼──────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                  VISUALIZATION & OUTPUT                           │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐    │
│  │ Belief Land- │ │  Insight     │ │  PDF/Slide Export    │    │
│  │ scape (Sigma)│ │  Dashboard   │ │  (Playwright)        │    │
│  └──────────────┘ └──────────────┘ └──────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Input Collection Layer

**Multi-modal input types supported:**

| Input Type | Implementation | Use Case |
|-----------|---------------|----------|
| Free text | Custom textarea (mobile-first, auto-resize) | Primary — open-ended thoughts |
| Voice | Cloud Speech-to-Text V2 → text | Accessibility, mobile convenience |
| Rating scale | Custom slider (1-100, not 1-5) | Forced quantification when needed |
| Forced choice | A/B/C selection cards | Binary/ternary decisions |
| Ranked preferences | Drag-and-drop ranking UI | Priority ordering |
| Google Forms | Forms API + Pub/Sub watch → Firestore | Pre-session structured surveys |

**Anonymous vs. Attributed Mode:**
- Session creator chooses mode at creation time
- Anonymous: no names shown, responses shuffled, no IP logging
- Attributed: names attached, enables "idea attribution" tracking
- Research shows anonymous mode increases honesty by 40% (meta-analysis, Lelkes et al., 2012) but attributed mode increases accountability and quality by 25%

**Real-Time Collection Architecture:**
```
Participant Device → Supabase Realtime (WebSocket) → responses table INSERT
                                                    → triggers Postgres function
                                                    → calls FastAPI /process-response via pg_net
```

### 2.3 NLP Processing Pipeline (FastAPI)

**Service:** `hivemind-nlp` — Python/FastAPI on Railway

**Core endpoint:** `POST /api/v1/process`

```python
# Input
{
  "response_id": "uuid",
  "session_id": "uuid",
  "text": "We should implement a caching layer using Redis to reduce p99 latency from 800ms to under 100ms",
  "language": "en",  # auto-detected or specified
  "anonymous": true
}

# Output (enriched response)
{
  "response_id": "uuid",
  "original_text": "...",
  "translated_text": "...",  # if non-English
  "embedding": [0.023, -0.891, ...],  # 3072-dim float32 array
  "sentiment": {
    "primary": "analytical",  # joy, anger, fear, sadness, surprise, analytical
    "confidence": 0.87,
    "valence": 0.6  # -1 to 1
  },
  "specificity_score": 0.92,  # 0-1, measures concreteness
  "argument_structure": {
    "has_premise": true,
    "has_conclusion": true,
    "has_evidence": true,
    "premises": ["p99 latency is currently 800ms"],
    "conclusion": "We should implement Redis caching",
    "evidence": ["Redis reduces p99 latency to under 100ms"]
  },
  "authenticity_score": 0.95,  # manipulation detection composite
  "word_count": 22,
  "processed_at": "2026-03-22T10:30:00Z"
}
```

**Gemini Flash Prompt for Enrichment:**
```
Analyze this response from a group decision-making session.
Return JSON with:
- sentiment: {primary: one of [joy, anger, fear, sadness, surprise, analytical, neutral], confidence: 0-1, valence: -1 to 1}
- specificity_score: 0-1 (0 = vague platitude like "we should improve", 1 = concrete actionable like "add Redis caching to reduce p99 from 800ms to 100ms")
- argument_structure: {has_premise: bool, has_conclusion: bool, has_evidence: bool, premises: [], conclusion: string, evidence: []}

Response: "{text}"
```

**Batch Processing Strategy:**
- Use asyncio + httpx for concurrent Gemini API calls
- Gemini Flash supports 300 RPM on paid tier → 5 requests/sec
- For bursts of 1000+ responses: queue via Redis (or in-memory asyncio.Queue), process in batches of 50 with 10 concurrent workers
- Embedding generation: batch endpoint supports up to 100 texts per call

### 2.4 Semantic Clustering Engine

**Step 1: Embedding Storage (pgvector)**
```sql
-- Embeddings stored alongside responses
ALTER TABLE responses ADD COLUMN embedding vector(3072);
CREATE INDEX ON responses USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

**Step 2: Dimensionality Reduction (UMAP)**
```python
import umap

# For visualization (2D projection)
reducer_2d = umap.UMAP(n_components=2, metric='cosine', n_neighbors=15, min_dist=0.1)
coords_2d = reducer_2d.fit_transform(embeddings)  # Shape: (N, 2)

# For clustering (higher-dim preserves more structure)
reducer_50d = umap.UMAP(n_components=50, metric='cosine', n_neighbors=15, min_dist=0.0)
coords_50d = reducer_50d.fit_transform(embeddings)  # Shape: (N, 50)
```

**Step 3: Clustering (HDBSCAN)**
```python
import hdbscan

clusterer = hdbscan.HDBSCAN(
    min_cluster_size=max(5, len(responses) // 20),  # At least 5% of responses per cluster
    min_samples=3,
    metric='euclidean',  # On UMAP-reduced space
    cluster_selection_method='eom'  # Excess of Mass — better for varying density
)
labels = clusterer.fit_predict(coords_50d)
# labels: array of cluster IDs, -1 = noise/outlier
```

**Step 4: Cluster Analysis**
```python
for cluster_id in set(labels):
    if cluster_id == -1: continue
    cluster_responses = [r for r, l in zip(responses, labels) if l == cluster_id]
    cluster_centroid = np.mean([r.embedding for r in cluster_responses], axis=0)

    # Gemini Flash generates cluster label
    label = gemini_flash.generate(f"""
        These {len(cluster_responses)} responses form a coherent opinion cluster.
        Summarize their shared position in ONE sentence (max 15 words).
        Responses: {[r.text for r in cluster_responses[:20]]}
    """)

    # Metrics
    avg_specificity = np.mean([r.specificity_score for r in cluster_responses])
    avg_sentiment = np.mean([r.sentiment.valence for r in cluster_responses])
    internal_coherence = np.mean(cosine_similarity(cluster_embeddings))
```

**Step 5: Dissent Cluster Identification**
```python
# The dissent cluster = smallest cluster with highest average specificity
# (small but insightful minority view)
clusters_by_insight = sorted(clusters, key=lambda c: c.avg_specificity / log(c.size + 1), reverse=True)
dissent_cluster = clusters_by_insight[0]
```

**Step 6: Cross-Cluster Tension Mapping**
```python
# Compute cosine distance between every pair of cluster centroids
tension_matrix = cosine_distances(cluster_centroids)
# High distance (>0.7) = opposing views
# Low distance (<0.3) = complementary/allied views (potential "hidden agreement")
```

### 2.5 Manipulation Detection System

| Signal | Method | Threshold |
|--------|--------|-----------|
| Copy-paste | Levenshtein distance between all response pairs | Flag if similarity > 90% |
| Timing anomaly | Kolmogorov-Smirnov test on inter-response intervals | p < 0.01 vs expected distribution |
| IP clustering | Group responses by /24 subnet | Flag if >5 responses from same subnet in <60s |
| Bot detection | Response time < 3 seconds for >100 word responses | Implausible typing speed |
| Template responses | TF-IDF unusually high overlap with a small set of phrases | Cosine similarity > 0.95 on TF-IDF vectors |

**Authentic Voice Score:** `authenticity = 0.3 * uniqueness + 0.3 * specificity + 0.2 * timing_score + 0.2 * length_naturalness`

Flagged responses are not deleted — they're downweighted in clustering and synthesis. The facilitator sees a "manipulation alert" panel.

### 2.6 Synthesis Engine

**Gemini 2.5 Pro Synthesis Prompt:**
```
You are synthesizing the collective intelligence of {n} participants on the question:
"{session_question}"

## Cluster Summary
{for each cluster: label, size, avg_specificity, avg_sentiment, top 3 responses by specificity}

## Dissent Cluster (highest insight-density minority view)
{dissent_cluster.label}: {top 5 dissent responses}

## Cross-Cluster Tensions
{pairs with distance > 0.7: "Cluster A vs Cluster B"}

## Hidden Agreements
{pairs with distance < 0.3: "Cluster A and Cluster B share underlying values despite different language"}

---

Produce exactly this JSON output:
{
  "insights": [
    {
      "title": "string (max 10 words)",
      "description": "string (2-3 sentences)",
      "confidence": 0.0-1.0,
      "supporting_clusters": ["cluster_ids"],
      "evidence_quotes": ["3 direct quotes from responses"],
      "dissent_acknowledged": "string (what the minority view says)",
      "action_recommendation": "string (specific next step)"
    }
  ],  // 3-5 insights
  "hidden_agreements": [...],
  "blocking_minority": {
    "cluster_id": "string",
    "size": int,
    "position": "string",
    "removal_consensus_increase": 0.0-1.0
  },
  "overall_consensus_score": 0.0-1.0,
  "participation_quality": {
    "avg_specificity": 0.0-1.0,
    "diversity_index": 0.0-1.0,  // Shannon entropy of cluster distribution
    "manipulation_flags": int
  }
}
```

### 2.7 Consensus Visualization Engine

**Technology:** Sigma.js + graphology (WebGL-rendered force graph)

**Node representation:**
- Each node = one response
- Node color = cluster assignment (categorical color palette)
- Node size = specificity score (higher specificity = larger node)
- Node position = 2D UMAP coordinates
- Node opacity = authenticity score

**Edge representation:**
- Edges drawn between responses with cosine similarity > 0.8
- Edge thickness = similarity strength
- Only draw top-K edges per node (K=3) to prevent visual clutter

**Cluster hulls:**
- Convex hull polygon drawn around each cluster
- Semi-transparent fill matching cluster color
- Label positioned at cluster centroid

**Interactive features:**
- Hover node → tooltip with response text + metrics
- Click cluster hull → panel showing cluster summary, top responses, sentiment distribution
- Time slider → animate the landscape formation (responses appear chronologically)
- Search → highlight responses containing keywords
- Filter → show/hide clusters, filter by specificity threshold

**The "Dissent Amplifier" UI:**
- Dissent cluster hull has a pulsing border animation
- A callout card: "Minority Insight: {dissent_cluster.label}" with the top 3 highest-specificity dissent responses
- Size is visually amplified (1.5x node size) so it's visible despite small count

**The "Hidden Agreement" Detector UI:**
- When two clusters have centroid distance < 0.3, draw a glowing bridge between them
- Tooltip: "These groups use different language but share 80%+ underlying values"
- Reveal animation: the bridge lights up during the "insight reveal" moment

### 2.8 Meeting Intelligence Module

**Approach for hackathon:** Post-meeting transcript analysis (not real-time Meet Media API)

**Architecture:**
1. User uploads Google Meet transcript (auto-generated .txt/.srt from Meet)
2. FastAPI parses transcript into speaker-segmented utterances
3. Each utterance processed through the same NLP pipeline
4. Additional meeting-specific analysis:

**Voice Equity Meter:**
```python
speaking_time = {}
for segment in transcript:
    speaking_time[segment.speaker] = speaking_time.get(segment.speaker, 0) + segment.duration
equity_score = 1 - gini_coefficient(list(speaking_time.values()))
# 1.0 = perfectly equal, 0.0 = one person dominated
```

**Idea Attribution Tracker:**
```
Gemini Flash prompt:
"Given this meeting transcript, identify instances where a speaker references,
builds upon, or responds to a previous speaker's idea. Return pairs:
[{original_speaker, original_idea, building_speaker, how_they_built}]"
```

**"Would Have Said" Detector:**
- Compare participant expertise profiles (if available) against topics discussed
- Identify silent participants whose expertise was relevant but unutilized
- "3 team members with database expertise were present but didn't speak during the 12-minute database architecture discussion"

**Post-Meeting Intelligence Report:**
- Voice equity distribution (pie chart)
- Idea flow graph (who influenced whom)
- Topic timeline (what was discussed when, for how long)
- Unresolved questions extracted from transcript
- AI-generated action items with assigned owners

---

## SECTION 3: GOOGLE API INTEGRATION PLAN

### API Configuration Matrix

| API | Model/Version | Use In Pipeline | Rate Limit | Cost Estimate/Session |
|-----|--------------|-----------------|------------|----------------------|
| Gemini 2.5 Flash | `gemini-2.5-flash` | Sentiment, specificity, argument extraction, cluster labeling | 300 RPM (paid) | ~$0.50 (1000 responses) |
| Gemini 2.5 Pro | `gemini-2.5-pro` | Final synthesis (1 call per session) | 150 RPM (paid) | ~$0.15 (1 synthesis call) |
| Gemini Embedding 2 | `gemini-embedding-001` | Semantic embedding generation | Free tier | $0.00 (free) |
| Cloud Translation | v3 | Non-English response translation | 1M chars/100s | ~$10 (1000 multilingual responses) |
| Cloud Speech-to-Text | V2 (gRPC) | Voice contribution transcription | 300 concurrent | ~$0.50 (20 min total audio) |
| Google Forms API | REST v1 | Structured pre-session surveys | Standard quota | Free |
| Cloud Pub/Sub | Standard | Event bus (Forms→processor, STT→NLP) | Millions msg/sec | ~$0.01 |
| Firestore | Native mode | Real-time session state (fallback option) | 10K writes/sec | ~$1/hr at scale |
| BigQuery | Storage Write API | Analytics warehouse | 1GB/sec | ~$0.05/session |

### Authentication Setup
```bash
# Service account with these roles:
# - Vertex AI User (Gemini API)
# - Cloud Translation API User
# - Cloud Speech-to-Text User
# - Pub/Sub Publisher/Subscriber
# - BigQuery Data Editor
# - Firestore User (if using Firestore)
gcloud iam service-accounts create hivemind-backend \
  --display-name="HIVEMIND Backend Service"
```

---

## SECTION 4: FRONTEND ARCHITECTURE

### 4.1 Session Creation Interface (`/create`)
- Question input (rich text, supports markdown)
- Session type selector: Open-ended | Forced choice | Ranked preferences | Mixed
- Privacy mode: Anonymous | Attributed
- Duration: Time-limited (30min, 1hr, 24hr, 1wk) | Open-ended
- Participant limit: Unlimited | Capped
- Share method: Link | QR code | Email invite | Google Forms embed
- Advanced: Enable voice input | Enable multilingual | Enable meeting transcript upload

### 4.2 Participant Contribution Interface (`/s/[session-code]`)
- Mobile-first, single-screen experience
- Large textarea with auto-resize (`react-textarea-autosize`)
- Character counter (suggested: 50-500 chars for optimal clustering)
- Optional voice input button (triggers browser MediaRecorder → Speech-to-Text)
- "Submit anonymously" confirmation
- After submit: see your response appear as a node on the belief landscape (if enabled)
- Multiple submissions allowed (configurable by session creator)

### 4.3 Organizer Real-Time Dashboard (`/dashboard/[session-id]`)
- **Left panel:** Live response feed (newest first, with sentiment color indicators)
- **Center:** Belief landscape visualization (Sigma.js force graph)
- **Right panel:** Cluster summaries (auto-updating as clusters form)
- **Bottom bar:** Session stats — response count, avg specificity, voice equity (if meeting mode), manipulation alerts
- **Top bar:** Session controls — close session, trigger synthesis, export

### 4.4 Insight Reveal Animation
- Motion (Framer Motion) orchestrated sequence:
  1. Belief landscape zooms to fit all clusters (0.5s ease-out)
  2. Cluster hulls pulse and labels appear (staggered 0.2s each)
  3. Dissent cluster glows with pulsing animation
  4. Hidden agreement bridges animate in (1s)
  5. Insight cards slide in from right (staggered 0.3s each)
  6. Overall consensus score counter animates from 0 to final value
- Total sequence: ~5 seconds, skippable

### 4.5 Export Suite
- **PDF Report:** Playwright renders `/report/[session-id]` route → PDF (includes landscape screenshot, insights, evidence quotes, cluster breakdown)
- **Slide Deck:** Pre-formatted 5-slide summary (Title, Question, Landscape, Insights, Action Items) → exported as PDF slides
- **CSV Data:** Raw responses with enrichment data
- **JSON API:** Full session data for programmatic access

---

## SECTION 5: MEETING INTELLIGENCE MODULE (DETAILED)

### 5.1 Architecture (Hackathon-Feasible)
```
Google Meet (recording) → Auto-generated transcript (.txt)
                              ↓
                     User uploads to HIVEMIND
                              ↓
                     FastAPI /api/v1/meeting/upload
                              ↓
                     Parse into speaker segments
                              ↓
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
        Voice Equity    Idea Attribution   NLP Pipeline
        Calculator      Tracker (Gemini)   (same as responses)
              ↓               ↓               ↓
              └───────────────┼───────────────┘
                              ↓
                    Meeting Intelligence Report
```

### 5.2 Speaker Identification
- Parse transcript format: `Speaker Name (HH:MM:SS): text`
- Assign unique speaker IDs, track cumulative speaking time
- Segment utterances into semantic units (sentence-level)

### 5.3 Voice Equity Meter
- Pie chart: % of total speaking time per participant
- Gini coefficient for inequality measurement
- Color coding: green (equitable) → red (dominated)
- Benchmark: "In this meeting, 2 of 8 participants accounted for 65% of speaking time"

### 5.4 Idea Attribution Tracker
- Gemini Flash identifies idea references across speakers
- Builds directed graph: Speaker A's idea → referenced by Speaker B → built upon by Speaker C
- Visualized as a Sankey diagram or directed graph
- Insight: "Maria's caching proposal was referenced by 4 subsequent speakers and became the team's consensus approach"

### 5.5 Post-Meeting Synthesis Report
- Auto-generated sections: Summary, Key Decisions, Action Items, Unresolved Questions
- Voice equity analysis with recommendations
- Idea attribution map
- Sentiment trajectory over meeting duration
- "Topics that deserved more time" (high-complexity topics with short discussion)

---

## SECTION 6: DATABASE SCHEMA (Supabase/PostgreSQL)

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Sessions table
CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  question TEXT NOT NULL,
  description TEXT,
  session_type TEXT NOT NULL DEFAULT 'open_ended', -- open_ended, forced_choice, ranked, mixed
  privacy_mode TEXT NOT NULL DEFAULT 'anonymous',  -- anonymous, attributed
  status TEXT NOT NULL DEFAULT 'active',           -- active, closed, synthesized
  created_by UUID REFERENCES auth.users(id),
  session_code TEXT UNIQUE NOT NULL,               -- 6-char join code
  settings JSONB DEFAULT '{}',                     -- voice_enabled, multilingual, max_responses, etc.
  created_at TIMESTAMPTZ DEFAULT now(),
  closed_at TIMESTAMPTZ,
  participant_count INT DEFAULT 0,
  response_count INT DEFAULT 0
);

-- Participants table
CREATE TABLE participants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  user_id UUID REFERENCES auth.users(id),          -- NULL if anonymous
  display_name TEXT,                                -- NULL if anonymous
  joined_at TIMESTAMPTZ DEFAULT now(),
  is_facilitator BOOLEAN DEFAULT false
);

-- Responses table (core data)
CREATE TABLE responses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  participant_id UUID REFERENCES participants(id),
  text TEXT NOT NULL,
  original_language TEXT DEFAULT 'en',
  translated_text TEXT,                             -- English translation if non-English
  input_type TEXT DEFAULT 'text',                   -- text, voice, form
  embedding vector(3072),                           -- Gemini embedding
  sentiment_primary TEXT,                           -- joy, anger, fear, sadness, surprise, analytical, neutral
  sentiment_confidence FLOAT,
  sentiment_valence FLOAT,                          -- -1 to 1
  specificity_score FLOAT,                          -- 0 to 1
  authenticity_score FLOAT,                         -- 0 to 1
  argument_structure JSONB,                         -- {has_premise, has_conclusion, has_evidence, ...}
  cluster_id INT,                                   -- assigned after clustering
  umap_x FLOAT,                                    -- 2D UMAP x-coordinate
  umap_y FLOAT,                                    -- 2D UMAP y-coordinate
  processing_status TEXT DEFAULT 'pending',         -- pending, processing, completed, failed
  created_at TIMESTAMPTZ DEFAULT now(),
  processed_at TIMESTAMPTZ
);

-- Clusters table
CREATE TABLE clusters (
  id SERIAL PRIMARY KEY,
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  cluster_index INT NOT NULL,                       -- HDBSCAN cluster label
  label TEXT,                                       -- Gemini-generated 1-sentence summary
  response_count INT DEFAULT 0,
  avg_specificity FLOAT,
  avg_sentiment_valence FLOAT,
  centroid vector(3072),                            -- Mean embedding of cluster
  centroid_x FLOAT,                                 -- 2D centroid x
  centroid_y FLOAT,                                 -- 2D centroid y
  is_dissent BOOLEAN DEFAULT false,                 -- Highest insight-density minority cluster
  hull_points JSONB,                                -- [[x,y], ...] convex hull for visualization
  created_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(session_id, cluster_index)
);

-- Cross-cluster relationships
CREATE TABLE cluster_relationships (
  id SERIAL PRIMARY KEY,
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  cluster_a INT REFERENCES clusters(id),
  cluster_b INT REFERENCES clusters(id),
  cosine_distance FLOAT,                            -- 0 (identical) to 2 (opposite)
  relationship_type TEXT,                            -- complementary, tension, hidden_agreement
  description TEXT                                   -- Gemini-generated description
);

-- Synthesis results
CREATE TABLE insights (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  confidence FLOAT,
  supporting_cluster_ids INT[],
  evidence_quotes TEXT[],
  dissent_acknowledged TEXT,
  action_recommendation TEXT,
  insight_order INT,                                 -- 1-5
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Session synthesis metadata
CREATE TABLE synthesis_results (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  overall_consensus_score FLOAT,
  diversity_index FLOAT,                             -- Shannon entropy
  avg_specificity FLOAT,
  manipulation_flags INT,
  hidden_agreements JSONB,
  blocking_minority JSONB,
  raw_gemini_output JSONB,                           -- Full Gemini Pro response
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Meeting transcripts
CREATE TABLE meeting_transcripts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  raw_transcript TEXT NOT NULL,
  speaker_segments JSONB,                            -- [{speaker, text, start_time, end_time}]
  voice_equity JSONB,                                -- {speaker: seconds, gini_coefficient}
  idea_attributions JSONB,                           -- [{original_speaker, idea, referencing_speaker}]
  meeting_duration_seconds INT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Indexes
CREATE INDEX idx_responses_session ON responses(session_id);
CREATE INDEX idx_responses_cluster ON responses(session_id, cluster_id);
CREATE INDEX idx_responses_embedding ON responses USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
CREATE INDEX idx_clusters_session ON clusters(session_id);
CREATE INDEX idx_sessions_code ON sessions(session_code);

-- Real-time subscriptions (Supabase enables these automatically)
-- Frontend subscribes to: responses (new inserts), clusters (updates), insights (new inserts)
```

---

## SECTION 7: COMPLETE FILE & FOLDER STRUCTURE

```
hivemind/
├── README.md
├── .gitignore
├── .env.example
│
├── frontend/                          # Next.js 15 application
│   ├── package.json
│   ├── next.config.ts
│   ├── tailwind.config.ts
│   ├── tsconfig.json
│   ├── .env.local                     # NEXT_PUBLIC_SUPABASE_URL, ANON_KEY, API_URL
│   │
│   ├── public/
│   │   ├── favicon.ico
│   │   └── og-image.png
│   │
│   ├── src/
│   │   ├── app/
│   │   │   ├── layout.tsx             # Root layout with providers
│   │   │   ├── page.tsx               # Landing page / hero
│   │   │   ├── globals.css            # Tailwind base + custom styles
│   │   │   │
│   │   │   ├── create/
│   │   │   │   └── page.tsx           # Session creation form
│   │   │   │
│   │   │   ├── s/
│   │   │   │   └── [code]/
│   │   │   │       └── page.tsx       # Participant contribution interface
│   │   │   │
│   │   │   ├── dashboard/
│   │   │   │   └── [sessionId]/
│   │   │   │       ├── page.tsx       # Main organizer dashboard
│   │   │   │       ├── insights/
│   │   │   │       │   └── page.tsx   # Insight reveal view
│   │   │   │       └── meeting/
│   │   │   │           └── page.tsx   # Meeting intelligence view
│   │   │   │
│   │   │   ├── report/
│   │   │   │   └── [sessionId]/
│   │   │   │       └── page.tsx       # Printable report (for PDF export)
│   │   │   │
│   │   │   └── api/
│   │   │       ├── synthesize/
│   │   │       │   └── route.ts       # Trigger synthesis (calls FastAPI)
│   │   │       ├── export-pdf/
│   │   │       │   └── route.ts       # Playwright PDF generation
│   │   │       └── meeting-upload/
│   │   │           └── route.ts       # Meeting transcript upload handler
│   │   │
│   │   ├── components/
│   │   │   ├── ui/                    # shadcn/ui components (auto-generated)
│   │   │   │   ├── button.tsx
│   │   │   │   ├── card.tsx
│   │   │   │   ├── input.tsx
│   │   │   │   ├── textarea.tsx
│   │   │   │   ├── slider.tsx
│   │   │   │   ├── dialog.tsx
│   │   │   │   ├── badge.tsx
│   │   │   │   ├── tabs.tsx
│   │   │   │   └── tooltip.tsx
│   │   │   │
│   │   │   ├── belief-landscape.tsx   # Sigma.js force-directed graph
│   │   │   ├── cluster-panel.tsx      # Cluster summary sidebar
│   │   │   ├── response-feed.tsx      # Live response stream
│   │   │   ├── insight-cards.tsx      # Synthesis insight display
│   │   │   ├── insight-reveal.tsx     # Animated insight reveal sequence
│   │   │   ├── voice-equity.tsx       # Meeting voice distribution chart
│   │   │   ├── idea-flow.tsx          # Idea attribution directed graph
│   │   │   ├── session-stats.tsx      # Response count, specificity, etc.
│   │   │   ├── contribution-form.tsx  # Participant text/voice input
│   │   │   ├── session-create-form.tsx # Session creation form
│   │   │   ├── qr-code.tsx            # QR code for session join link
│   │   │   ├── manipulation-alert.tsx # Facilitator manipulation warnings
│   │   │   ├── time-slider.tsx        # Temporal replay of landscape formation
│   │   │   └── consensus-score.tsx    # Animated consensus score display
│   │   │
│   │   ├── lib/
│   │   │   ├── supabase/
│   │   │   │   ├── client.ts          # Supabase browser client
│   │   │   │   ├── server.ts          # Supabase server client
│   │   │   │   └── types.ts           # Generated database types
│   │   │   ├── api.ts                 # FastAPI client functions
│   │   │   ├── utils.ts               # Utility functions
│   │   │   └── constants.ts           # App constants
│   │   │
│   │   ├── hooks/
│   │   │   ├── use-realtime-responses.ts  # Supabase realtime subscription
│   │   │   ├── use-session.ts             # Session data + state
│   │   │   ├── use-clusters.ts            # Cluster data subscription
│   │   │   └── use-belief-landscape.ts    # Graph data management
│   │   │
│   │   └── types/
│   │       └── index.ts               # TypeScript type definitions
│   │
│   └── supabase/
│       ├── config.toml                # Supabase local dev config
│       └── migrations/
│           └── 001_initial_schema.sql # Database schema
│
├── backend/                           # Python FastAPI service
│   ├── pyproject.toml                 # Dependencies (uv/poetry)
│   ├── Dockerfile                     # Railway deployment
│   ├── .env                           # GEMINI_API_KEY, SUPABASE_URL, etc.
│   │
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                    # FastAPI app entry point
│   │   ├── config.py                  # Settings / env vars
│   │   │
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── routes.py              # All API route definitions
│   │   │   └── dependencies.py        # Request dependencies
│   │   │
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── nlp.py                 # Gemini Flash NLP processing
│   │   │   ├── embeddings.py          # Gemini embedding generation
│   │   │   ├── clustering.py          # UMAP + HDBSCAN pipeline
│   │   │   ├── synthesis.py           # Gemini Pro synthesis
│   │   │   ├── manipulation.py        # Manipulation detection
│   │   │   ├── meeting.py             # Meeting transcript analysis
│   │   │   └── translation.py         # Cloud Translation wrapper
│   │   │
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── schemas.py             # Pydantic request/response models
│   │   │   └── database.py            # Supabase client
│   │   │
│   │   └── utils/
│   │       ├── __init__.py
│   │       └── helpers.py             # Shared utilities
│   │
│   └── tests/
│       ├── __init__.py
│       ├── test_nlp.py
│       ├── test_clustering.py
│       └── test_synthesis.py
│
├── scripts/
│   ├── seed-demo-data.py              # Generate 500+ realistic demo responses
│   └── generate-types.sh              # Supabase type generation
│
└── demo/
    ├── sample-responses.json          # 500 pre-generated responses for demo
    └── sample-transcript.txt          # Sample meeting transcript
```

---

## SECTION 8: API ENDPOINTS

### FastAPI Backend (`backend/`)

| Method | Endpoint | Description |
|--------|---------|-------------|
| POST | `/api/v1/process` | Process single response (NLP + embedding) |
| POST | `/api/v1/process/batch` | Process batch of responses |
| POST | `/api/v1/cluster/{session_id}` | Run clustering on all session responses |
| POST | `/api/v1/synthesize/{session_id}` | Run Gemini Pro synthesis |
| POST | `/api/v1/meeting/upload` | Upload + analyze meeting transcript |
| GET | `/api/v1/session/{session_id}/landscape` | Get 2D coordinates + clusters for visualization |
| GET | `/api/v1/session/{session_id}/manipulation` | Get manipulation detection report |
| GET | `/health` | Health check |

### Next.js API Routes (`frontend/src/app/api/`)

| Method | Endpoint | Description |
|--------|---------|-------------|
| POST | `/api/synthesize` | Proxy to FastAPI synthesis (adds auth) |
| POST | `/api/export-pdf` | Generate PDF report via Playwright |
| POST | `/api/meeting-upload` | Handle transcript file upload |

### Supabase Direct (client-side via supabase-js)
- `supabase.from('sessions').insert(...)` — create session
- `supabase.from('responses').insert(...)` — submit response
- `supabase.channel('session:xxx').on('INSERT', ...)` — real-time response stream
- `supabase.from('clusters').select(...)` — fetch cluster data
- `supabase.from('insights').select(...)` — fetch synthesis results

---

## SECTION 9: THE HACKATHON DEMO PLAN

### The "Judges Are The Demo" Strategy

**Setup (before presentation):**
1. Pre-load a session with 500+ realistic demo responses on the question: *"What is the biggest problem in tech today?"*
2. Responses are pre-clustered and synthesized — the belief landscape is ready to show
3. Have a LIVE session ready with a fresh question for judges

**Demo Script (8 minutes):**

**[0:00-1:30] The Problem (slides)**
- "$9.8 trillion lost to bad corporate decisions annually"
- Show a SurveyMonkey result: "Average satisfaction: 3.7/5" — "What does this MEAN?"
- Show a meeting recording: 2 people talking 70% of the time
- "What if we could hear EVERYONE?"

**[1:30-3:30] The Pre-Loaded Demo**
- Switch to HIVEMIND dashboard showing 500 pre-clustered responses
- Navigate the belief landscape: "Each dot is a person. Each color is a cluster of similar thinking."
- Click into clusters: "This cluster of 85 people believes AI regulation is the biggest issue. This cluster of 62 people says it's developer burnout."
- Show the dissent cluster: "But this small group of 12 people had the highest-quality insights — they're talking about the interaction between regulation AND burnout. The majority missed this."
- Reveal the synthesis: "HIVEMIND found 4 actionable insights from 500 voices in 30 seconds"

**[3:30-6:00] The LIVE Demo — "Let's try it on YOU"**
- Show QR code on screen: "Scan this. You have 60 seconds."
- Question for judges: *"What should every hackathon project do differently?"*
- Judges type responses on their phones (anonymous mode)
- Dashboard shows responses appearing in real-time on the feed
- After 60 seconds: trigger clustering
- Watch the belief landscape form LIVE — clusters emerge, labels appear
- Trigger synthesis: "Here's what YOUR collective wisdom says..."
- Show the insight cards appearing with animation
- "We just turned 8 judges into a collective intelligence in 90 seconds."

**[6:00-7:30] The Scale Vision**
- "Now imagine this with 10,000 employees deciding company strategy"
- "Or 100,000 citizens in participatory budgeting"
- "Or UN delegates finding hidden agreement across 193 nations"
- Show meeting intelligence: voice equity meter, idea attribution

**[7:30-8:00] Close**
- "HIVEMIND doesn't replace human judgment. It amplifies it."
- "Groups are already smarter than individuals. They just don't have the tools to prove it."

### Demo Data Generation Strategy (`scripts/seed-demo-data.py`)
- Use Gemini to generate 500 diverse, realistic responses to "What is the biggest problem in tech today?"
- Ensure responses span: AI safety, developer burnout, regulation, inequality, sustainability, misinformation, open source sustainability, privacy, quantum threats
- Vary specificity: 30% vague ("Tech needs to be better"), 40% moderate ("We need better AI regulation"), 30% highly specific ("We need mandatory algorithmic impact assessments for any AI system deployed to >1M users")
- Vary sentiment: analytical (40%), concerned (30%), optimistic (15%), angry (10%), hopeful (5%)

---

## SECTION 10: 24-HOUR BUILD SPRINT PLAN

### Hour-by-Hour Schedule

| Hours | Phase | Deliverables |
|-------|-------|-------------|
| **0-1** | **Project Setup** | Monorepo scaffold, Next.js + FastAPI initialized, Supabase project created, GitHub repo, Vercel + Railway connected |
| **1-3** | **Database + Auth** | Run migration (full schema), Supabase auth configured, session CRUD working, join-by-code flow |
| **3-5** | **Input Collection** | Contribution form (mobile-first textarea), real-time response insertion, Supabase realtime subscription showing live feed |
| **5-8** | **NLP Pipeline** | FastAPI `/process` endpoint working: Gemini Flash sentiment + specificity + argument extraction, Gemini Embedding generation, results written back to DB |
| **8-11** | **Clustering Engine** | UMAP reduction, HDBSCAN clustering, cluster labeling via Gemini Flash, dissent cluster identification, results stored in clusters table |
| **11-14** | **Belief Landscape** | Sigma.js force graph rendering 2D UMAP coordinates, cluster coloring, node sizing by specificity, cluster hulls, click interactions |
| **14-16** | **Synthesis Engine** | Gemini Pro synthesis endpoint, insight generation, hidden agreement + blocking minority detection, insight cards UI |
| **16-18** | **Meeting Intelligence** | Transcript upload, speaker parsing, voice equity calculation, idea attribution via Gemini, meeting report view |
| **18-20** | **Demo Data + Polish** | Generate 500 demo responses, pre-cluster and synthesize, insight reveal animation, dissent amplifier UI, hidden agreement bridges |
| **20-22** | **Export + Final UI** | PDF report generation, QR code for session join, landing page, responsive polish, error handling |
| **22-23** | **Demo Rehearsal** | Run through demo script 3x, fix any bugs, test on multiple devices, prepare backup screenshots |
| **23-24** | **Final Deploy + Buffer** | Production deploy, final smoke test, prepare pitch deck backup, rest before presentation |

### Critical Path Dependencies
```
Setup → DB Schema → Input Collection → NLP Pipeline → Clustering → Visualization
                                                                  ↘ Synthesis → Insight UI
                                          Meeting Intelligence (parallel track after Hour 14)
                                          Demo Data Generation (parallel track after Hour 8)
```

### Minimum Viable Demo (if behind schedule)
If falling behind, cut in this order (last = first to cut):
1. CUT: Meeting intelligence module (impressive but not core)
2. CUT: PDF export (show on screen instead)
3. CUT: Manipulation detection (nice-to-have for demo)
4. CUT: Voice input (text-only is fine for demo)
5. KEEP: Belief landscape + clustering + synthesis + live demo = the core wow factor

---

## SECTION 11: PRODUCTION & SCALE

### Pricing Model

| Tier | Price | Target | Features |
|------|-------|--------|----------|
| **Free** | $0 | Individual/small teams | 3 sessions/month, 50 responses/session, basic clustering |
| **Pro** | $10/participant/session | Companies | Unlimited responses, full synthesis, meeting intelligence, PDF export |
| **Enterprise** | Custom | Large orgs (1000+ employees) | SSO, dedicated infrastructure, custom branding, API access, SLA |
| **Government** | Per-project | City/national gov | Participatory budgeting, civic engagement, multilingual, compliance |
| **Education** | $2/student/semester | Universities | Classroom deliberation, research mode with data export |

### Scale Architecture (Post-Hackathon)
- **Kubernetes on GKE** for auto-scaling FastAPI workers
- **Redis** for job queuing (celery workers for NLP processing)
- **CDN** (Cloudflare) for static assets
- **Multi-region Supabase** for global latency reduction
- **Dedicated Gemini API quota** via Google Cloud partnership

### Market Opportunity
- Corporate decision tools: $15B TAM
- Gov-tech civic engagement: $8B TAM
- Ed-tech deliberation tools: $3B TAM
- International organizations: $500M TAM

---

## SECTION 12: TECHNICAL RISKS & MITIGATIONS

| Risk | Severity | Mitigation |
|------|----------|-----------|
| **Clustering quality with <20 responses** | High | Fallback to simple similarity grouping (no UMAP/HDBSCAN). Show individual responses instead of clusters below threshold. |
| **Gemini API rate limits during demo** | High | Pre-process demo data. Cache all NLP results. Only live-process judge responses (5-10 calls). |
| **Real-time processing latency** | Medium | Process responses asynchronously. Show "processing" indicator. Landscape updates in batches (every 5 responses). |
| **UMAP/HDBSCAN computation time** | Medium | Pre-compute for demo data. For live: incremental updates (add new points to existing projection). Fallback: PCA instead of UMAP. |
| **Sigma.js performance with 10K+ nodes** | Low | WebGL handles 100K+ nodes. Limit visible edges. Use level-of-detail rendering. |
| **Manipulation detection false positives** | Medium | Conservative thresholds. Alert facilitator, don't auto-remove. Allow facilitator override. |
| **Meeting transcript parsing failures** | Medium | Support multiple transcript formats (Google Meet, Zoom, Teams). Graceful fallback to unstructured text processing. |
| **Supabase free tier limits** | Low | Free tier: 500MB DB, 50K auth users, 2GB bandwidth. Sufficient for hackathon. Upgrade to Pro ($25/mo) if needed. |

---

## SECTION 13: BUSINESS MODEL & PITCH STRUCTURE

### Elevator Pitch (30 seconds)
"HIVEMIND turns any group into a superintelligence. Instead of surveys that flatten nuance or meetings where the loudest voice wins, HIVEMIND lets thousands of people express their full thinking, then uses AI to find the signal — clustering opinions by meaning, surfacing high-quality minority views, and synthesizing actionable collective wisdom in seconds."

### Pitch Structure (3 minutes)

1. **Hook** (15s): "Companies lose $9.8 trillion a year to bad decisions. Not because individuals are stupid — because groups are broken."
2. **Problem** (30s): HiPPO, survey flatness, meeting dominance, buried minority wisdom
3. **Solution** (30s): AI-powered collective intelligence — semantic clustering, dissent amplification, synthesis
4. **Demo** (60s): Live belief landscape forming from judge responses
5. **Market** (15s): $15B corporate + $8B gov-tech + $3B ed-tech
6. **Traction/Vision** (15s): "Today: 8 judges. Tomorrow: 10,000 employees. Next year: 193 UN member states."
7. **Ask** (15s): What you need (investment, partnership, pilot customers)

### Hackathon Judging Criteria Alignment

| Criterion | How HIVEMIND Scores |
|-----------|-------------------|
| **Innovation** | First platform to combine semantic clustering + dissent amplification + AI synthesis for collective intelligence. Novel "belief landscape" visualization. |
| **Technical Complexity** | Full NLP pipeline, real-time clustering, force-directed graph visualization, multi-modal input, meeting intelligence. |
| **Impact** | Addresses $9.8T decision failure cost. Applications: corporate strategy, civic engagement, education, international diplomacy. |
| **Completeness** | End-to-end: input collection → processing → clustering → synthesis → visualization → export. |
| **Demo Quality** | "Judges are the demo" — live collective intelligence formation on stage. |
| **Google API Usage** | Gemini Flash + Pro + Embeddings, Cloud Translation, Speech-to-Text, Forms API, Pub/Sub, BigQuery, Firestore. |

---

## SECTION 14: APPENDICES

### A. Sample Clustering Output

```json
{
  "session_id": "abc-123",
  "question": "What is the biggest problem in tech today?",
  "response_count": 523,
  "cluster_count": 7,
  "clusters": [
    {
      "id": 0,
      "label": "AI systems need mandatory safety regulation before deployment",
      "size": 112,
      "percentage": 21.4,
      "avg_specificity": 0.73,
      "avg_sentiment": "analytical",
      "is_dissent": false,
      "top_responses": [
        "We need mandatory algorithmic impact assessments for any AI system deployed to more than 1M users",
        "The EU AI Act is a good start but enforcement mechanisms are too weak",
        "Self-regulation has failed — look at social media. We need binding legislation."
      ]
    },
    {
      "id": 1,
      "label": "Developer burnout and unsustainable work culture is the root problem",
      "size": 89,
      "percentage": 17.0,
      "avg_specificity": 0.68,
      "avg_sentiment": "concerned",
      "is_dissent": false
    },
    {
      "id": 6,
      "label": "The real issue is that regulation and burnout interact — burned out developers ship unsafe AI",
      "size": 18,
      "percentage": 3.4,
      "avg_specificity": 0.91,
      "avg_sentiment": "analytical",
      "is_dissent": true
    }
  ],
  "hidden_agreements": [
    {
      "cluster_a": 0,
      "cluster_b": 3,
      "shared_value": "Both groups believe external accountability mechanisms are needed — one focuses on government regulation, the other on open-source transparency"
    }
  ]
}
```

### B. Belief Landscape Visualization Spec

```
Canvas: 1200x800px (responsive)
Background: #0a0a0f (dark)
Node colors: [#6366f1, #ec4899, #14b8a6, #f59e0b, #8b5cf6, #06b6d4, #f43f5e] (per cluster)
Node size: 4px (specificity 0) → 16px (specificity 1)
Node opacity: 0.3 (authenticity < 0.5) → 1.0 (authenticity 1.0)
Edge color: #ffffff15 (very subtle)
Cluster hull: fill opacity 0.08, stroke opacity 0.3, stroke width 2px
Dissent cluster: stroke-dasharray animated, glow filter
Hidden agreement bridge: gradient stroke between connected cluster colors, animated dash
Labels: Inter font, 13px, white, positioned at cluster centroid
Zoom: mouse wheel, pinch-to-zoom on mobile
Pan: click-and-drag
```

### C. Meeting Intelligence Report Template

```
╔══════════════════════════════════════╗
║   HIVEMIND MEETING INTELLIGENCE     ║
║   {meeting_title}                    ║
║   {date} · {duration}               ║
╚══════════════════════════════════════╝

VOICE EQUITY SCORE: {score}/100
┌──────────────────────────────────────┐
│ ████████████████░░░░  Sarah (38%)   │
│ ████████████░░░░░░░░  James (28%)   │
│ ████░░░░░░░░░░░░░░░░  Priya (12%)  │
│ ███░░░░░░░░░░░░░░░░░  Alex  (10%)  │
│ ██░░░░░░░░░░░░░░░░░░  Others (12%) │
└──────────────────────────────────────┘
⚠ 2 participants with relevant expertise did not speak.

TOP INSIGHTS:
1. {insight_1}
2. {insight_2}
3. {insight_3}

IDEA FLOW:
Sarah's caching proposal → built upon by James → adopted as team consensus
Priya's security concern → not addressed (FLAGGED for follow-up)

ACTION ITEMS:
[ ] {action_1} — Owner: {name}
[ ] {action_2} — Owner: {name}
[ ] {action_3} — Owner: {name}

UNRESOLVED QUESTIONS:
? {question_1}
? {question_2}
```

### D. Key Dependencies

**Frontend (package.json):**
```json
{
  "dependencies": {
    "next": "^15.0.0",
    "@supabase/supabase-js": "^2.45.0",
    "@supabase/ssr": "^0.5.0",
    "@sigma/react": "^1.0.0",
    "graphology": "^0.25.0",
    "graphology-layout-forceatlas2": "^0.10.0",
    "motion": "^12.0.0",
    "recharts": "^2.12.0",
    "react-textarea-autosize": "^8.5.0",
    "qrcode.react": "^4.0.0",
    "tailwindcss": "^4.0.0",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.0",
    "lucide-react": "^0.400.0"
  }
}
```

**Backend (pyproject.toml):**
```toml
[project]
dependencies = [
    "fastapi>=0.115.0",
    "uvicorn>=0.30.0",
    "google-genai>=1.0.0",
    "supabase>=2.5.0",
    "umap-learn>=0.5.6",
    "hdbscan>=0.8.38",
    "numpy>=1.26.0",
    "scikit-learn>=1.5.0",
    "python-Levenshtein>=0.25.0",
    "httpx>=0.27.0",
    "python-multipart>=0.0.9",
    "pydantic>=2.8.0",
]
```

---

## VERIFICATION PLAN

### How to Test End-to-End

1. **Input Collection:** Create a session → join via code on mobile → submit 5 test responses → verify they appear in Supabase `responses` table and on the real-time feed
2. **NLP Pipeline:** Check `responses` table for populated `sentiment_primary`, `specificity_score`, `embedding` columns after processing
3. **Clustering:** Trigger clustering via `/api/v1/cluster/{session_id}` → verify `clusters` table populated with labels, centroids, hull points
4. **Visualization:** Open dashboard → verify Sigma.js renders nodes at correct UMAP coordinates with correct cluster colors
5. **Synthesis:** Trigger synthesis → verify `insights` table populated → verify insight cards appear on dashboard
6. **Meeting Intelligence:** Upload sample transcript → verify voice equity chart and idea attribution graph render
7. **Export:** Click export → verify PDF downloads with correct content
8. **Live Demo Flow:** Full rehearsal: QR scan → contribute → watch landscape form → trigger synthesis → see insights → under 3 minutes total

### Smoke Test Commands
```bash
# Backend health
curl http://localhost:8000/health

# Process a test response
curl -X POST http://localhost:8000/api/v1/process \
  -H "Content-Type: application/json" \
  -d '{"response_id":"test","session_id":"test","text":"We need better AI regulation","language":"en"}'

# Run clustering
curl -X POST http://localhost:8000/api/v1/cluster/SESSION_ID

# Run synthesis
curl -X POST http://localhost:8000/api/v1/synthesize/SESSION_ID

# Frontend
npm run dev  # http://localhost:3000
```
