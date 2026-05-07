# OMNISCIENT — Master Technical Blueprint

> **Upload ANY scientific paper, dataset, patent, or report. OMNISCIENT uses NotebookLM + Gemini to run live simulations, generate counter-hypotheses, find hidden connections across 10,000 papers, and tell you what experiment to run next — before any other lab on Earth does it.**

---

## SECTION 1: PROBLEM DEFINITION & VISION

### 1.1 The $2.3T R&D Problem

Global R&D spending reached $2.3 trillion in 2024. Yet:

- **23 days/year** of duplicated research waste per researcher (Nature, 2023 survey). Researchers re-derive results already published in adjacent fields they don't follow.
- **85% of clinical trials fail** — many due to prior evidence that existed but wasn't found.
- **The average researcher reads 250 papers/year**. PubMed alone adds **1.5M papers/year**. arXiv adds **20,000/month**. The human reading bottleneck is the fundamental constraint.

### 1.2 The Hidden Connection Problem

Breakthrough discoveries sit unclaimed because no single human has read both papers:

- **Don Swanson (1986)** proved that undiscovered connections between fish oil and Raynaud's disease existed across published literature for years before anyone linked them.
- **Drug repurposing**: Thalidomide's anti-cancer properties were implicit in literature for a decade before clinical exploration.
- These "Swanson links" are everywhere — but finding them requires reading across domain boundaries that no human can cross at scale.

### 1.3 Where Current Tools Fail

| Tool | What It Does | Where It Fails |
|------|-------------|----------------|
| **PubMed** | Keyword search over 36M biomedical abstracts | No semantic understanding. No cross-domain. No hypothesis generation. Boolean queries miss conceptual links. |
| **Semantic Scholar** | Citation graph + AI summaries (SPECTER embeddings) | Citation-graph only — can't find connections between papers that don't cite each other. No contradiction detection. |
| **Elsevier ScienceDirect / Scopus AI** | Publisher-locked corpus with AI Q&A | Closed ecosystem (Elsevier journals only). No cross-publisher synthesis. No experiment simulation. |
| **Google Scholar** | Broad search + citation tracking | No AI reasoning. No knowledge graph. No hypothesis generation. Keyword-bound. |
| **Consensus.app** | AI search over 200M papers | Answer extraction only. No graph. No cross-paper synthesis. No simulation. |
| **NotebookLM (standalone)** | Upload 50 sources, chat with them | Manual upload only. No automated ingestion. No knowledge graph persistence. No cross-session memory. |

### 1.4 The Technological Unlock

**Gemini 2.5 Pro (1M tokens) + Gemini 2.0 Pro Experimental (2M tokens)** can hold ~500 full papers in a single context window. Combined with:

- **Vertex AI RAG Engine** for corpus-scale retrieval
- **Neo4j knowledge graph** for structural reasoning across millions of papers
- **GROBID** for industrial-scale PDF parsing (10+ PDFs/second)
- **SPECTER2 embeddings** for semantic similarity tuned on scientific literature

This stack enables what was previously impossible: an AI that has structurally "read" every paper in a domain and can reason across them.

### 1.5 The Vision

**OMNISCIENT is an AI perpetual post-doctoral research partner that has read everything.** It:

1. Ingests every paper in your domain in real-time
2. Builds a living knowledge graph of claims, methods, findings, and contradictions
3. Finds hidden connections across domains no human would discover
4. Generates novel, testable hypotheses ranked by impact and feasibility
5. Simulates experiment outcomes before you spend a dollar in the lab
6. Tells you what experiment to run next — before any other lab on Earth

---

## SECTION 2: CORE TECHNICAL ARCHITECTURE

### 2.1 Full System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                              │
│  arXiv API  │  PubMed API  │  CrossRef API  │  User Uploads     │
└──────┬──────┴──────┬───────┴───────┬────────┴────────┬──────────┘
       │             │               │                  │
       ▼             ▼               ▼                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                   INGESTION PIPELINE                             │
│  PDF Download → GROBID Parsing → Structured XML → Chunking      │
│  → Metadata Extraction → Deduplication → Quality Scoring        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐
│  EMBEDDING   │ │  KNOWLEDGE   │ │  DOCUMENT STORE      │
│  GENERATION  │ │  GRAPH       │ │  (Cloud Storage)     │
│  SPECTER2 +  │ │  EXTRACTION  │ │  Raw PDFs + parsed   │
│  Gemini Emb  │ │  Claims,     │ │  XML + metadata      │
│      │       │ │  entities,   │ └──────────────────────┘
│      ▼       │ │  relations   │
│  Vector DB   │ │      │       │
│  (Weaviate)  │ │      ▼       │
└──────────────┘ │   Neo4j KG   │
                 └──────┬───────┘
                        │
       ┌────────────────┼────────────────┐
       ▼                ▼                ▼
┌─────────────┐ ┌──────────────┐ ┌──────────────────┐
│ CROSS-PAPER │ │  HYPOTHESIS  │ │  EXPERIMENT       │
│ CONNECTION   │ │  GENERATION  │ │  SIMULATION       │
│ ENGINE       │ │  ENGINE      │ │  LAYER            │
│ - Semantic   │ │ - Gap detect │ │ - Outcome predict │
│   proximity  │ │ - Analogy    │ │ - Power calc      │
│ - Contradict │ │ - Counter-   │ │ - Resource est    │
│   detection  │ │   hypothesis │ │ - Protocol gen    │
│ - Missing    │ │ - Novelty    │ │                   │
│   experiment │ │   scoring    │ │                   │
└──────┬──────┘ └──────┬───────┘ └────────┬──────────┘
       │               │                   │
       ▼               ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                    GEMINI REASONING LAYER                        │
│  Gemini 2.5 Pro — synthesis, hypothesis elaboration, simulation │
│  Gemini 2.0 Flash — bulk processing, summarization, extraction  │
│  NotebookLM Enterprise — per-session research pods              │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     FRONTEND (Next.js 14)                        │
│  Research Dashboard │ KG Visualizer │ Hypothesis Explorer        │
│  Contradiction Map  │ Gap Map       │ Voice Query Interface      │
│  Experiment Protocol│ Collaboration │ Paper Upload               │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Paper Ingestion Pipeline

**Step 1: Source Acquisition**

```python
# arXiv — real-time feed (hourly polling)
import arxiv
client = arxiv.Client()
search = arxiv.Search(
    query="cat:cs.AI OR cat:q-bio OR cat:cond-mat",
    max_results=100,
    sort_by=arxiv.SortCriterion.SubmittedDate
)
for paper in client.results(search):
    paper.download_pdf(dirpath="./papers/arxiv/")

# PubMed — biomedical literature
from Bio import Entrez
Entrez.email = "omniscient@example.com"
Entrez.api_key = "YOUR_NCBI_API_KEY"  # 10 req/sec with key
handle = Entrez.esearch(db="pubmed", term="CRISPR AND 2024[pdat]", retmax=100)
ids = Entrez.read(handle)["IdList"]
records = Entrez.efetch(db="pubmed", id=ids, rettype="xml")

# CrossRef — DOI resolution + metadata enrichment
import requests
resp = requests.get(
    "https://api.crossref.org/works",
    params={"query": "machine learning drug discovery", "rows": 20,
            "mailto": "omniscient@example.com"}  # polite pool
)
```

**Step 2: PDF Parsing with GROBID**

```bash
# Deploy GROBID (10.6 PDFs/sec on 16-CPU)
docker run --rm --init --ulimit core=0 -p 8070:8070 grobid/grobid:0.8.1
```

```python
from grobid_client.grobid_client import GrobidClient
client = GrobidClient(config_path="./grobid_config.json")
# Batch process — outputs structured TEI XML per paper
client.process("processFulltextDocument", "./papers/arxiv/", output="./parsed/", n=10)
```

GROBID extracts: title, authors, affiliations, abstract, section headers, paragraphs, inline citations, reference list, tables, figure captions, equations (visual).

**Step 3: Embedding Generation**

```python
# SPECTER2 for scientific paper embeddings (citation-graph trained)
from transformers import AutoTokenizer, AutoModel
tokenizer = AutoTokenizer.from_pretrained("allenai/specter2_base")
model = AutoModel.from_pretrained("allenai/specter2_base")

# For each paper: embed title + abstract (primary) + section chunks (secondary)
inputs = tokenizer(f"{paper.title} [SEP] {paper.abstract}",
                   padding=True, truncation=True, return_tensors="pt", max_length=512)
embeddings = model(**inputs).last_hidden_state[:, 0, :]  # [CLS] token

# Gemini Embedding 2 for multimodal chunks (figures + text)
import google.generativeai as genai
genai.configure(api_key="GEMINI_API_KEY")
result = genai.embed_content(
    model="models/gemini-embedding-002",
    content=chunk_text,
    task_type="RETRIEVAL_DOCUMENT"
)
```

**Step 4: Knowledge Graph Node Creation**

For every paper, extract and store:
- **Paper node**: title, abstract, year, journal, DOI, arXiv ID, categories
- **Author nodes**: name, ORCID, affiliation, h-index
- **Concept nodes**: key terms, MeSH headings, extracted entities
- **Claim nodes**: formal assertions extracted from text with provenance
- **Method nodes**: experimental methods, algorithms, datasets used
- **Finding nodes**: results with statistical measures

### 2.3 Cross-Paper Connection Engine

**Semantic Proximity vs Citation Proximity**

Citation graphs only capture *acknowledged* connections. OMNISCIENT finds *implicit* connections:

```
Citation Proximity:  Paper A cites Paper B → direct link
Semantic Proximity:  Paper A and Paper C use similar methods on similar
                     systems but never cite each other → HIDDEN CONNECTION
```

Implementation:

```python
def find_semantic_connections(paper_id: str, threshold: float = 0.82):
    """Find papers semantically similar but not citation-linked."""
    # Get paper embedding
    paper_embedding = vector_db.get_embedding(paper_id)

    # Find semantically similar papers
    similar = vector_db.query(paper_embedding, top_k=50, threshold=threshold)

    # Filter out citation-linked papers
    cited = neo4j.query("""
        MATCH (p:Paper {id: $id})-[:CITES|:CITED_BY*1..2]-(cited:Paper)
        RETURN cited.id
    """, id=paper_id)
    cited_ids = {r["cited.id"] for r in cited}

    # Hidden connections = semantically similar BUT not citation-linked
    hidden = [p for p in similar if p.id not in cited_ids]
    return hidden
```

**Contradiction Detector**

```python
CONTRADICTION_PROMPT = """
Given these two claims from different papers:

Claim 1 (from {paper1}): "{claim1}"
Claim 2 (from {paper2}): "{claim2}"

Are these claims contradictory? Analyze:
1. Do they address the same phenomenon?
2. Do they make incompatible assertions?
3. Could methodological differences explain the discrepancy?
4. Rate contradiction severity: DIRECT | PARTIAL | METHODOLOGICAL | NONE

Output JSON: {"contradicts": bool, "severity": str, "explanation": str}
"""

def detect_contradictions(domain_claims: list[Claim]):
    """Find contradictory claims across papers."""
    # Group claims by concept/topic using embeddings
    clusters = cluster_claims_by_topic(domain_claims)
    contradictions = []
    for cluster in clusters:
        # Pairwise comparison within topic cluster
        for c1, c2 in itertools.combinations(cluster, 2):
            if c1.paper_id == c2.paper_id:
                continue
            result = gemini_flash.generate(
                CONTRADICTION_PROMPT.format(
                    paper1=c1.paper_title, claim1=c1.text,
                    paper2=c2.paper_title, claim2=c2.text
                )
            )
            if result.contradicts:
                contradictions.append(Contradiction(c1, c2, result))
    return contradictions
```

**Missing Experiment Identifier**

```python
def find_missing_experiments(concept_a: str, concept_b: str):
    """Identify experimental gaps — things that should have been tested but weren't."""
    result = neo4j.query("""
        // Find methods used to study concept_a
        MATCH (ca:Concept {name: $a})<-[:STUDIES]-(e1:Experiment)-[:USES]->(m:Method)
        WITH collect(DISTINCT m.name) AS methods_for_a

        // Find methods used to study concept_b
        MATCH (cb:Concept {name: $b})<-[:STUDIES]-(e2:Experiment)-[:USES]->(m2:Method)
        WITH methods_for_a, collect(DISTINCT m2.name) AS methods_for_b

        // Methods used for A but never applied to B
        RETURN [m IN methods_for_a WHERE NOT m IN methods_for_b] AS gaps
    """, a=concept_a, b=concept_b)
    return result["gaps"]
```

### 2.4 Hypothesis Generation Engine

**Pipeline**: Gap Detection → Candidate Generation → Novelty Check → Plausibility Score → Impact Estimate → Rank

```python
class HypothesisEngine:
    def generate(self, domain: str, max_hypotheses: int = 50) -> list[Hypothesis]:
        # Step 1: Find structural gaps in the knowledge graph
        gaps = self.gap_detector.find_gaps(domain)

        # Step 2: For each gap, generate candidate hypotheses via Gemini
        candidates = []
        for gap in gaps:
            context = self.build_context(gap)  # Relevant papers, claims, methods
            response = gemini_pro.generate(
                HYPOTHESIS_GENERATION_PROMPT.format(
                    gap_description=gap.description,
                    supporting_evidence=context.evidence,
                    methods_available=context.methods,
                    related_findings=context.findings
                )
            )
            candidates.extend(parse_hypotheses(response))

        # Step 3: Score each candidate
        scored = []
        for h in candidates:
            h.novelty_score = self.novelty_scorer.score(h)      # 0-1
            h.plausibility_score = self.plausibility_scorer.score(h)  # 0-1
            h.impact_score = self.impact_estimator.score(h)      # 0-1
            h.composite_score = (
                0.3 * h.novelty_score +
                0.4 * h.plausibility_score +
                0.3 * h.impact_score
            )
            scored.append(h)

        return sorted(scored, key=lambda h: h.composite_score, reverse=True)[:max_hypotheses]
```

**Novelty Scorer**: Embeds the hypothesis statement, searches the full corpus for semantic matches. If top-k similarity > 0.92, the hypothesis is likely already published → low novelty.

**Plausibility Scorer**: Checks if the hypothesis is consistent with established findings in the knowledge graph. Uses Gemini to evaluate mechanistic soundness.

**Impact Estimator**: Uses citation counts, journal impact factors, and funding data for related work to estimate how impactful a confirmed finding would be.

### 2.5 Experiment Simulation Layer

```python
class ExperimentSimulator:
    def simulate(self, hypothesis: Hypothesis) -> SimulationResult:
        # Find analogous experiments in the knowledge graph
        analogous = neo4j.query("""
            MATCH (h:Hypothesis)-[:TESTS]->(c:Concept)<-[:STUDIES]-(e:Experiment)
            WHERE c.name IN $concepts
            RETURN e, e.outcome, e.effect_size, e.sample_size, e.method
            ORDER BY e.year DESC LIMIT 20
        """, concepts=hypothesis.concepts)

        # Gemini predicts likely outcome based on analogous results
        prediction = gemini_pro.generate(f"""
            Based on these analogous experiments: {analogous}
            Predict the likely outcome of testing: {hypothesis.statement}
            Include: expected effect size, confidence interval, likely confounds.
            Output as JSON.
        """)

        # Statistical power calculation
        power = self.calculate_power(
            effect_size=prediction.effect_size,
            alpha=0.05,
            desired_power=0.80
        )

        # Resource estimation
        resources = self.estimate_resources(
            method=hypothesis.proposed_method,
            sample_size=power.required_n,
            duration_weeks=prediction.estimated_duration
        )

        return SimulationResult(
            predicted_outcome=prediction,
            statistical_power=power,
            resources=resources,
            confidence=prediction.confidence,
            analogous_experiments=analogous
        )
```

### 2.6 NotebookLM Integration Architecture

```
┌─────────────────────────────────────────┐
│          RESEARCH POD MANAGER           │
│  Orchestrates multiple NotebookLM pods  │
└──────────────────┬──────────────────────┘
                   │
    ┌──────────────┼──────────────┐
    ▼              ▼              ▼
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Pod:     │  │ Pod:     │  │ Pod:     │
│ Cancer   │  │ Drug     │  │ Genomics │
│ Biology  │  │ Discovery│  │          │
│ 50 papers│  │ 50 papers│  │ 50 papers│
└─────────┘  └─────────┘  └─────────┘
```

**Strategy**: NotebookLM Enterprise API (Alpha) supports max 50 sources per notebook. For each research domain, create a dedicated "pod" — a NotebookLM notebook loaded with the 50 most relevant papers (selected by embedding similarity to the user's query).

```python
class ResearchPodManager:
    def create_pod(self, domain: str, focus_query: str) -> str:
        # Select top 50 papers by relevance
        papers = vector_db.query(focus_query, top_k=50, filter={"domain": domain})

        # Create NotebookLM notebook via Enterprise API
        notebook = notebooklm_client.notebooks.create(
            title=f"OMNISCIENT: {domain} - {focus_query[:50]}"
        )

        # Add papers as sources
        for paper in papers:
            notebooklm_client.sources.create(
                notebook_id=notebook.id,
                source_type="text",
                content=paper.full_text[:500000]  # API text limits
            )
        return notebook.id

    def cross_domain_query(self, query: str, domains: list[str]) -> str:
        """Query across multiple domain pods and synthesize."""
        pod_responses = []
        for domain in domains:
            pod_id = self.get_or_create_pod(domain, query)
            response = notebooklm_client.notebooks.query(
                notebook_id=pod_id, query=query
            )
            pod_responses.append({"domain": domain, "response": response})

        # Gemini synthesizes cross-domain responses
        synthesis = gemini_pro.generate(f"""
            Synthesize these research findings from different domains:
            {json.dumps(pod_responses)}
            Identify cross-domain connections and contradictions.
        """)
        return synthesis
```

**Fallback (if NotebookLM API unavailable)**: Build equivalent with Vertex AI RAG Engine + Gemini 2.5 Pro. This is actually more flexible and production-ready.

### 2.7 Live arXiv Ingestion Pipeline

```python
# Runs every 30 minutes via Cloud Scheduler → Cloud Run
async def ingest_new_papers():
    # 1. Fetch new papers since last checkpoint
    new_papers = arxiv_client.search(
        query="cat:cs.AI OR cat:q-bio.BM",
        sort_by=SortCriterion.SubmittedDate,
        max_results=50
    )

    for paper in new_papers:
        # 2. Download PDF
        pdf_path = await download_pdf(paper.pdf_url)

        # 3. Parse with GROBID
        structured = await grobid_client.process(pdf_path)

        # 4. Generate embeddings
        embedding = specter2.embed(f"{paper.title} [SEP] {paper.summary}")

        # 5. Extract claims and entities
        claims = await extract_claims(structured, gemini_flash)

        # 6. Update knowledge graph
        await neo4j_client.create_paper_node(paper, structured, claims)

        # 7. Upsert to vector DB
        await vector_db.upsert(paper.entry_id, embedding, paper.metadata)

        # 8. Auto-summarize
        summary = await gemini_flash.generate(
            f"Summarize this paper in 3 sentences for a domain expert: {structured.abstract}"
        )

        # 9. Check for connections to existing papers
        connections = await find_semantic_connections(paper.entry_id)
        if connections:
            await notify_subscribers(paper, connections)
```

### 2.8 Voice Interface

```
User speaks → Speech-to-Text API → Query text
    → Intent classification (Gemini Flash)
    → Route to: KG query / RAG search / Hypothesis request / Simulation
    → Generate response
    → Text-to-Speech API → Audio response
```

Implementation: Google Cloud Speech-to-Text v2 for input, Text-to-Speech (Neural2 voices) for output. WebSocket connection for streaming.

---

## SECTION 3: GOOGLE API INTEGRATION PLAN

### 3.1 Service Matrix

| Service | Purpose | Auth | Endpoint | Pricing |
|---------|---------|------|----------|---------|
| Gemini 2.5 Pro | Synthesis, hypothesis gen | API Key / OAuth | `generativelanguage.googleapis.com` | $1.25/M input, $10/M output |
| Gemini 2.0 Flash | Bulk processing, extraction | API Key / OAuth | `generativelanguage.googleapis.com` | $0.10/M input, $0.40/M output |
| Gemini Embedding 2 | Paper chunk embeddings | API Key | `generativelanguage.googleapis.com` | $0.20/M tokens |
| NotebookLM Enterprise | Research pods | OAuth (Workspace) | `notebooklm.googleapis.com` | $9/user/month |
| Document AI | PDF parsing | Service Account | `documentai.googleapis.com` | $0.03/page |
| Vertex AI RAG Engine | Corpus retrieval | Service Account | `aiplatform.googleapis.com` | $0.00003/1K chars |
| Cloud Storage | Paper corpus | Service Account | `storage.googleapis.com` | $0.020/GB/month |
| Pub/Sub | Event pipeline | Service Account | `pubsub.googleapis.com` | $40/TB |
| Cloud Run | Compute | Service Account | N/A | $0.00002400/vCPU-sec |
| Speech-to-Text v2 | Voice input | API Key | `speech.googleapis.com` | $0.016/min |
| Text-to-Speech | Voice output | API Key | `texttospeech.googleapis.com` | $16/M chars |
| BigQuery | Analytics, metadata | Service Account | `bigquery.googleapis.com` | $6.25/TB queried |

### 3.2 Authentication Architecture

```python
# Service account for backend services
from google.cloud import storage, documentai, bigquery
from google.oauth2 import service_account

credentials = service_account.Credentials.from_service_account_file(
    "service-account.json",
    scopes=["https://www.googleapis.com/auth/cloud-platform"]
)

# API key for Gemini (simpler, for hackathon)
import google.generativeai as genai
genai.configure(api_key=os.environ["GEMINI_API_KEY"])

# NotebookLM Enterprise requires Workspace OAuth
# User must authenticate via Google Workspace domain
```

### 3.3 Data Flow

```
User uploads PDF
    → Cloud Storage (gs://omniscient-papers/{user_id}/{paper_id}.pdf)
    → Pub/Sub message → Cloud Run job:
        → Document AI processDocument() → structured JSON
        → Gemini Flash: extract claims, entities, methods
        → Gemini Embedding 2: generate embeddings
        → Write to Neo4j + Weaviate + BigQuery metadata table
        → Pub/Sub: "paper_indexed" event
    → Frontend receives SSE notification: "Paper ready"
```

---

## SECTION 4: THE KNOWLEDGE GRAPH DESIGN

### 4.1 Complete Graph Schema

**Node Types**

```cypher
// Paper — the core unit
CREATE (p:Paper {
    id: "arxiv:2401.12345",
    title: "...",
    abstract: "...",
    year: 2024,
    month: 1,
    journal: "Nature",
    doi: "10.1038/...",
    arxiv_id: "2401.12345",
    categories: ["cs.AI", "q-bio.BM"],
    citation_count: 42,
    pdf_url: "...",
    ingested_at: datetime()
})

// Author
CREATE (a:Author {
    id: "orcid:0000-0002-1234-5678",
    name: "Jane Smith",
    affiliation: "MIT",
    h_index: 34,
    fields: ["machine learning", "drug discovery"]
})

// Institution
CREATE (i:Institution {
    id: "ror:042nb2s44",
    name: "Massachusetts Institute of Technology",
    country: "US",
    type: "university"
})

// Concept — extracted entities / topics
CREATE (c:Concept {
    id: "mesh:D000077195",
    name: "CRISPR-Cas9",
    type: "technique",  // technique | molecule | disease | organism | phenomenon
    mesh_id: "D000077195",
    description: "..."
})

// Claim — formal assertion from a paper
CREATE (cl:Claim {
    id: "claim-uuid",
    text: "CRISPR-Cas9 editing efficiency exceeds 90% in human T-cells",
    type: "empirical",  // empirical | theoretical | methodological
    confidence: 0.87,
    section: "Results",
    page: 5,
    sentence_span: "342-398",
    paper_id: "arxiv:2401.12345"
})

// Experiment
CREATE (e:Experiment {
    id: "exp-uuid",
    description: "In vitro CRISPR editing of CD4+ T-cells",
    method: "CRISPR-Cas9 ribonucleoprotein delivery",
    sample_size: 150,
    outcome: "92.3% editing efficiency",
    effect_size: 0.923,
    p_value: 0.001,
    paper_id: "arxiv:2401.12345"
})

// Finding
CREATE (f:Finding {
    id: "finding-uuid",
    text: "Off-target effects reduced by 74% with high-fidelity Cas9",
    statistical_significance: true,
    effect_size: 0.74,
    paper_id: "arxiv:2401.12345"
})

// Hypothesis (OMNISCIENT-generated)
CREATE (h:Hypothesis {
    id: "hyp-uuid",
    statement: "High-fidelity Cas9 combined with AAV delivery may achieve >95% editing in vivo",
    novelty_score: 0.89,
    plausibility_score: 0.76,
    impact_score: 0.92,
    composite_score: 0.85,
    generated_at: datetime(),
    status: "proposed"  // proposed | under_review | validated | refuted
})

// Dataset
CREATE (d:Dataset {
    id: "dataset-uuid",
    name: "CRISPR editing efficiency benchmark",
    size: "15,000 sequences",
    url: "...",
    paper_id: "arxiv:2401.12345"
})

// Method
CREATE (m:Method {
    id: "method-uuid",
    name: "Ribonucleoprotein electroporation",
    category: "gene_editing",
    description: "..."
})
```

**Edge Types**

```cypher
// Paper relationships
(p1:Paper)-[:CITES {context: "supporting"}]->(p2:Paper)
(p:Paper)-[:AUTHORED_BY {position: 1}]->(a:Author)
(a:Author)-[:AFFILIATED_WITH {from: 2020, to: null}]->(i:Institution)
(p:Paper)-[:PUBLISHED_IN]->(j:Journal)

// Claim relationships
(p:Paper)-[:MAKES_CLAIM]->(cl:Claim)
(cl1:Claim)-[:CONTRADICTS {severity: "direct", explanation: "..."}]->(cl2:Claim)
(cl1:Claim)-[:SUPPORTS {strength: 0.85}]->(cl2:Claim)
(cl:Claim)-[:ABOUT]->(c:Concept)

// Experiment relationships
(p:Paper)-[:REPORTS_EXPERIMENT]->(e:Experiment)
(e:Experiment)-[:USES_METHOD]->(m:Method)
(e:Experiment)-[:STUDIES]->(c:Concept)
(e:Experiment)-[:PRODUCES_FINDING]->(f:Finding)
(f:Finding)-[:SUPPORTS_CLAIM]->(cl:Claim)

// Hypothesis relationships
(h:Hypothesis)-[:DERIVED_FROM]->(cl:Claim)
(h:Hypothesis)-[:ADDRESSES_GAP]->(g:Gap)
(h:Hypothesis)-[:INVOLVES]->(c:Concept)
(h:Hypothesis)-[:PROPOSED_METHOD]->(m:Method)

// Concept relationships
(c1:Concept)-[:RELATED_TO {strength: 0.7}]->(c2:Concept)
(c:Concept)-[:SUBTYPE_OF]->(c2:Concept)

// Cross-paper edges (OMNISCIENT-generated)
(p1:Paper)-[:HIDDEN_CONNECTION {
    type: "semantic",
    similarity: 0.87,
    explanation: "Both papers study X but from different angles"
}]->(p2:Paper)
```

### 4.2 Claim Extraction Pipeline

```python
CLAIM_EXTRACTION_PROMPT = """
Extract all scientific claims from this paper section. For each claim:
1. The exact claim text
2. Type: empirical (data-backed), theoretical (model-based), methodological (technique claim)
3. Confidence: how strongly the authors state this (0-1)
4. Key entities mentioned
5. Supporting evidence (figure/table references, p-values, effect sizes)

Section text:
{section_text}

Output as JSON array:
[{"text": "...", "type": "...", "confidence": 0.9, "entities": [...], "evidence": "..."}]
"""

async def extract_claims_from_paper(parsed_xml: dict) -> list[Claim]:
    claims = []
    for section in parsed_xml["sections"]:
        if section["type"] in ["introduction", "results", "discussion", "conclusion"]:
            response = await gemini_flash.generate(
                CLAIM_EXTRACTION_PROMPT.format(section_text=section["text"])
            )
            section_claims = json.loads(response)
            for c in section_claims:
                c["paper_id"] = parsed_xml["id"]
                c["section"] = section["title"]
            claims.extend(section_claims)
    return claims
```

### 4.3 Claim Verification System

Cross-reference each new claim against existing claims in the graph:

```cypher
// Find potentially conflicting claims about the same concept
MATCH (new_claim:Claim {id: $claim_id})-[:ABOUT]->(c:Concept)<-[:ABOUT]-(existing:Claim)
WHERE existing.id <> new_claim.id
AND existing.type = new_claim.type
RETURN existing, c
```

Then use Gemini to classify the relationship (supports / contradicts / extends / independent).

### 4.4 Temporal Dimension

```cypher
// Track how consensus evolves
CREATE (cs:ConsensusSnapshot {
    concept: "CRISPR efficiency",
    year: 2024,
    dominant_claim: "...",
    support_count: 45,
    contradiction_count: 3,
    confidence: 0.93
})

// Query consensus trajectory
MATCH (cs:ConsensusSnapshot {concept: "CRISPR efficiency"})
RETURN cs.year, cs.dominant_claim, cs.confidence
ORDER BY cs.year
```

### 4.5 Graph Database Selection: Neo4j

**Why Neo4j over alternatives:**
- Native graph storage → O(1) relationship traversal (critical for path-finding across papers)
- GDS library provides PageRank, Louvain community detection, Node2Vec, link prediction out-of-the-box
- Mature Python driver, LangChain integration, GraphRAG support
- AuraDB free tier for hackathon; Enterprise for production
- 20+ years of production usage at scale (hundreds of millions of nodes)

**Why not Spanner Graph**: Newer, thinner tooling ecosystem, no equivalent to GDS algorithms
**Why not BigQuery Graph**: OLAP-only, not suited for transactional graph operations
**Why not Neptune**: AWS-only, weaker algorithm library, RDF complexity unnecessary

### 4.6 Query Patterns for Hypothesis Generation

```cypher
// 1. Find bridging concepts (Swanson linking)
MATCH (a:Concept)<-[:ABOUT]-(c1:Claim)<-[:MAKES_CLAIM]-(p1:Paper)
MATCH (b:Concept)<-[:ABOUT]-(c2:Claim)<-[:MAKES_CLAIM]-(p2:Paper)
WHERE NOT EXISTS((p1)-[:CITES]-(p2))
AND NOT EXISTS((a)-[:RELATED_TO]-(b))
MATCH path = shortestPath((a)-[*..4]-(b))
WHERE length(path) >= 3
RETURN a.name, b.name, [n IN nodes(path) | n.name] AS bridge_path
LIMIT 100

// 2. Find contradictory claim clusters
MATCH (c1:Claim)-[:CONTRADICTS]->(c2:Claim)
MATCH (c1)-[:ABOUT]->(concept:Concept)<-[:ABOUT]-(c2)
RETURN concept.name, collect(c1.text) AS claims_for, collect(c2.text) AS claims_against

// 3. Find methods never applied to a concept
MATCH (c:Concept)<-[:STUDIES]-(e:Experiment)-[:USES_METHOD]->(m:Method)
WITH c, collect(DISTINCT m.name) AS used_methods
MATCH (m2:Method) WHERE NOT m2.name IN used_methods
AND m2.category = "gene_editing"  // domain filter
RETURN c.name, collect(m2.name) AS untried_methods
LIMIT 50

// 4. Community detection for research subfields
CALL gds.louvain.stream('paper-graph', {relationshipTypes: ['CITES']})
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).title, communityId
ORDER BY communityId

// 5. Link prediction — likely future citations
CALL gds.linkPrediction.adamicAdar.stream('paper-graph', {
    topK: 100,
    relationshipTypes: ['CITES']
})
YIELD node1, node2, score
RETURN gds.util.asNode(node1).title, gds.util.asNode(node2).title, score
ORDER BY score DESC
```

---

## SECTION 5: THE HYPOTHESIS ENGINE IN DETAIL

### 5.1 Gap Detection Algorithm

```python
class GapDetector:
    def find_gaps(self, domain: str) -> list[ResearchGap]:
        gaps = []

        # Type 1: Untested intersections
        # Concepts A and B are well-studied individually but never together
        untested = neo4j.query("""
            MATCH (a:Concept)-[:RELATED_TO*2..3]-(b:Concept)
            WHERE NOT EXISTS((a)<-[:STUDIES]-(:Experiment)-[:STUDIES]->(b))
            AND a.name <> b.name
            AND a.domain = $domain AND b.domain = $domain
            WITH a, b, size((a)<-[:ABOUT]-(:Claim)) AS a_claims,
                 size((b)<-[:ABOUT]-(:Claim)) AS b_claims
            WHERE a_claims > 5 AND b_claims > 5
            RETURN a.name, b.name, a_claims, b_claims
            ORDER BY a_claims + b_claims DESC LIMIT 50
        """, domain=domain)
        for row in untested:
            gaps.append(ResearchGap(type="untested_intersection",
                                   concepts=[row["a.name"], row["b.name"]]))

        # Type 2: Stale contradictions (unresolved for >2 years)
        contradictions = neo4j.query("""
            MATCH (c1:Claim)-[r:CONTRADICTS]->(c2:Claim)
            MATCH (c1)<-[:MAKES_CLAIM]-(p1:Paper)
            MATCH (c2)<-[:MAKES_CLAIM]-(p2:Paper)
            WHERE NOT EXISTS(
                (c1)-[:ABOUT]->(:Concept)<-[:ABOUT]-(:Claim)<-[:MAKES_CLAIM]-
                (:Paper {year: date().year})
            )
            AND p1.year < date().year - 2
            RETURN c1, c2, p1.title, p2.title
        """)
        for row in contradictions:
            gaps.append(ResearchGap(type="unresolved_contradiction",
                                   claims=[row["c1"], row["c2"]]))

        # Type 3: Method transfer opportunities
        # Method M works well in domain D1, never tried in D2
        transfers = neo4j.query("""
            MATCH (m:Method)<-[:USES_METHOD]-(e:Experiment)-[:STUDIES]->(c:Concept)
            WHERE c.domain = $domain
            WITH m, count(e) AS usage_count, avg(e.effect_size) AS avg_effect
            WHERE usage_count >= 3 AND avg_effect > 0.5
            MATCH (c2:Concept) WHERE c2.domain <> $domain
            AND NOT EXISTS((m)<-[:USES_METHOD]-(:Experiment)-[:STUDIES]->(c2))
            RETURN m.name, c2.name, c2.domain, usage_count, avg_effect
            LIMIT 30
        """, domain=domain)
        for row in transfers:
            gaps.append(ResearchGap(type="method_transfer", method=row["m.name"],
                                   target=row["c2.name"]))

        return gaps
```

### 5.2 The Analogy Engine

"This worked for X — could it work for Y?"

```python
ANALOGY_PROMPT = """
A successful experiment found: {finding}
Using method: {method}
In domain: {source_domain}

Target domain: {target_domain}
Target concept: {target_concept}

Analyze whether this approach could be transferred:
1. What structural analogies exist between source and target?
2. What would need to change in the methodology?
3. What are the key assumptions that might not transfer?
4. Rate transferability: HIGH / MEDIUM / LOW with justification.
5. Propose the specific experiment to test this.

Output JSON with fields: transferability, rationale, proposed_experiment, assumptions, risks
"""

class AnalogyEngine:
    def find_analogies(self, finding: Finding, target_domain: str) -> list[Analogy]:
        # Find structurally similar concepts in the target domain
        source_embedding = vector_db.get_embedding(finding.concept_id)
        similar_concepts = vector_db.query(
            source_embedding, top_k=10,
            filter={"domain": target_domain}
        )

        analogies = []
        for concept in similar_concepts:
            result = gemini_pro.generate(ANALOGY_PROMPT.format(
                finding=finding.text,
                method=finding.method,
                source_domain=finding.domain,
                target_domain=target_domain,
                target_concept=concept.name
            ))
            if result.transferability in ["HIGH", "MEDIUM"]:
                analogies.append(Analogy(source=finding, target=concept, analysis=result))
        return analogies
```

### 5.3 Counter-Hypothesis Generator

```python
COUNTER_HYPOTHESIS_PROMPT = """
The current consensus in {domain} is:
"{consensus_claim}"

Supported by {support_count} papers. Key supporting evidence:
{evidence_summary}

Generate systematic counter-hypotheses:
1. What if the observed effect is due to a confounding variable?
2. What if the causal direction is reversed?
3. What if the effect is real but the mechanism is different?
4. What if the effect only holds under specific conditions not yet tested?

For each counter-hypothesis:
- Statement
- What evidence would support it
- What experiment would distinguish it from the consensus
- Plausibility rating (0-1)
"""
```

### 5.4 Novelty Verification

```python
async def verify_novelty(hypothesis: str) -> float:
    """Score 0 (already published) to 1 (completely novel)."""
    # Step 1: Embed the hypothesis
    h_embedding = await embed(hypothesis)

    # Step 2: Search corpus for semantic matches
    matches = await vector_db.query(h_embedding, top_k=20)

    # Step 3: For high-similarity matches, ask Gemini to compare
    novelty_scores = []
    for match in matches:
        if match.similarity > 0.80:
            result = await gemini_flash.generate(f"""
                Hypothesis: {hypothesis}
                Existing paper: {match.title} — {match.abstract}

                Is this hypothesis already addressed by the existing paper?
                Rate: NOVEL (not addressed), PARTIAL (tangentially related),
                      DUPLICATE (substantially the same idea)
            """)
            if result == "DUPLICATE":
                return 0.1
            elif result == "PARTIAL":
                novelty_scores.append(0.5)
            else:
                novelty_scores.append(0.9)

    return sum(novelty_scores) / len(novelty_scores) if novelty_scores else 0.95
```

### 5.5 Hypothesis Ranking System

```
Composite Score = 0.3 × Novelty + 0.4 × Plausibility + 0.3 × Impact

Where:
- Novelty (0-1): From novelty verification (5.4)
- Plausibility (0-1): Mechanistic soundness + consistency with known evidence
- Impact (0-1): Citation potential × field importance × clinical/commercial relevance
```

### 5.6 Experiment Protocol Generator

```python
PROTOCOL_PROMPT = """
Generate a detailed experiment protocol to test this hypothesis:
"{hypothesis}"

Based on analogous successful experiments:
{analogous_experiments}

Include:
1. Objective and primary endpoints
2. Materials and equipment
3. Step-by-step methodology
4. Sample size (from power calculation: n={required_n})
5. Controls (positive, negative, vehicle)
6. Statistical analysis plan
7. Expected timeline
8. Estimated budget
9. Potential pitfalls and contingency plans
10. Ethical considerations (if applicable)

Format as a structured protocol document.
"""
```

---

## SECTION 6: FRONTEND ARCHITECTURE

### 6.1 Tech Stack

- **Framework**: Next.js 14+ (App Router)
- **Styling**: Tailwind CSS + shadcn/ui components
- **Graph Visualization**: react-force-graph-3d (WebGL) + d3.js for 2D fallback
- **State Management**: Zustand
- **Real-time**: Server-Sent Events (SSE) for paper ingestion updates
- **Voice**: Web Speech API (browser-native) + Google Speech-to-Text fallback
- **Charts**: Recharts for analytics, Plotly.js for scientific plots

### 6.2 Page Architecture

```
/                           → Landing page / marketing
/dashboard                  → Main research dashboard
/dashboard/upload           → Paper upload interface
/dashboard/graph            → Knowledge graph visualizer (full-screen)
/dashboard/hypotheses       → Hypothesis explorer with filters
/dashboard/contradictions   → Contradiction map
/dashboard/gaps             → Research gap visualization
/dashboard/simulate         → Experiment simulation interface
/dashboard/protocols        → Generated experiment protocols
/dashboard/voice            → Voice query interface
/dashboard/pods             → NotebookLM research pod manager
/dashboard/settings         → User preferences, API keys, domains
/collaborate/:pod_id        → Shared research pod view
```

### 6.3 Key Components

**Knowledge Graph Visualizer** (the hero feature):
```
┌────────────────────────────────────────────────────┐
│  [Search: ___________]  [Filter: Domain ▼] [3D/2D]│
│                                                     │
│         ○ Paper A                                   │
│        / \          ○ Paper D                       │
│       /   \        / \                              │
│   ○ Concept X    ○ Method Y                        │
│       \   /        \                                │
│        \ /          ○ Claim Z                       │
│     ◉ Paper B -------- Paper E                      │
│     (selected)    (hidden connection!)               │
│                                                     │
│  ──────────────────────────────────────────         │
│  Panel: Paper B details | Claims | Connections      │
└────────────────────────────────────────────────────┘
```

- Nodes colored by type (paper=blue, concept=green, claim=orange, hypothesis=purple)
- Edges colored by relationship (cites=gray, contradicts=red, supports=green, hidden=gold-dashed)
- Click node to expand its neighborhood
- Right-click for "Generate hypotheses from this node"

**Hypothesis Explorer**:
```
┌────────────────────────────────────────────────────┐
│  Hypotheses (47 generated)    [Sort: Impact ▼]     │
│  ──────────────────────────────────────────         │
│  ◉ 0.92  "High-fidelity Cas9 + AAV delivery..."   │
│    Novelty: ████████░░ 0.89                        │
│    Plausibility: ███████░░░ 0.76                   │
│    Impact: █████████░ 0.92                         │
│    [View Evidence Chain] [Simulate] [Protocol]     │
│  ──────────────────────────────────────────         │
│  ○ 0.87  "Combining X with Y may produce..."      │
│  ○ 0.81  "The observed effect in Z could..."       │
│  ○ 0.79  ...                                       │
└────────────────────────────────────────────────────┘
```

**Contradiction Map** (heatmap of conflicting claims by concept):
```
┌────────────────────────────────────────────────────┐
│        Concept A  Concept B  Concept C  Concept D  │
│ Paper1    ✓          ✗          ✓          -       │
│ Paper2    ✗          ✓          ✓          ✓       │
│ Paper3    ✓          ✗          -          ✗       │
│                                                     │
│ ✓ = supports consensus  ✗ = contradicts  - = N/A   │
│ Click cell for detailed claim comparison            │
└────────────────────────────────────────────────────┘
```

### 6.4 Collaboration Features

- Share research pods via invite link
- Real-time collaborative graph exploration (Liveblocks or Yjs)
- Annotation system — tag papers, claims, hypotheses with team notes
- Export research reports as PDF with full citation chain

---

## SECTION 7: COMPLETE FILE & FOLDER STRUCTURE

```
omniscient/
├── README.md
├── docker-compose.yml                 # Full local dev stack
├── docker-compose.prod.yml
├── .env.example
├── .gitignore
├── Makefile                           # dev shortcuts
│
├── backend/                           # Python FastAPI backend
│   ├── Dockerfile
│   ├── pyproject.toml                 # Poetry/uv project config
│   ├── requirements.txt
│   ├── alembic.ini
│   ├── alembic/                       # DB migrations
│   │   └── versions/
│   │
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                    # FastAPI app entrypoint
│   │   ├── config.py                  # Settings / env vars
│   │   ├── dependencies.py            # DI container
│   │   │
│   │   ├── api/                       # REST API routes
│   │   │   ├── __init__.py
│   │   │   ├── papers.py              # Paper CRUD + upload
│   │   │   ├── hypotheses.py          # Hypothesis endpoints
│   │   │   ├── graph.py               # KG query endpoints
│   │   │   ├── search.py              # Semantic search
│   │   │   ├── simulate.py            # Experiment simulation
│   │   │   ├── voice.py               # Voice query endpoint
│   │   │   ├── pods.py                # NotebookLM pod management
│   │   │   ├── collaborate.py         # Collaboration endpoints
│   │   │   └── health.py
│   │   │
│   │   ├── models/                    # Pydantic schemas
│   │   │   ├── __init__.py
│   │   │   ├── paper.py
│   │   │   ├── claim.py
│   │   │   ├── hypothesis.py
│   │   │   ├── experiment.py
│   │   │   ├── graph.py
│   │   │   └── simulation.py
│   │   │
│   │   ├── services/                  # Business logic
│   │   │   ├── __init__.py
│   │   │   ├── ingestion/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── arxiv_client.py    # arXiv API integration
│   │   │   │   ├── pubmed_client.py   # PubMed E-utilities
│   │   │   │   ├── crossref_client.py # CrossRef DOI resolution
│   │   │   │   ├── pdf_parser.py      # GROBID + Document AI
│   │   │   │   ├── pipeline.py        # Orchestrates full ingestion
│   │   │   │   └── scheduler.py       # Periodic arXiv polling
│   │   │   │
│   │   │   ├── knowledge_graph/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── neo4j_client.py    # Neo4j driver wrapper
│   │   │   │   ├── claim_extractor.py # LLM claim extraction
│   │   │   │   ├── entity_extractor.py
│   │   │   │   ├── graph_builder.py   # Builds nodes + edges
│   │   │   │   ├── connection_engine.py # Cross-paper connections
│   │   │   │   └── contradiction_detector.py
│   │   │   │
│   │   │   ├── hypothesis/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── gap_detector.py
│   │   │   │   ├── analogy_engine.py
│   │   │   │   ├── counter_hypothesis.py
│   │   │   │   ├── novelty_scorer.py
│   │   │   │   ├── plausibility_scorer.py
│   │   │   │   ├── impact_estimator.py
│   │   │   │   ├── ranking.py
│   │   │   │   └── protocol_generator.py
│   │   │   │
│   │   │   ├── simulation/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── experiment_simulator.py
│   │   │   │   ├── power_calculator.py
│   │   │   │   └── resource_estimator.py
│   │   │   │
│   │   │   ├── embeddings/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── specter2.py        # SPECTER2 embeddings
│   │   │   │   ├── gemini_embeddings.py
│   │   │   │   └── vector_store.py    # Weaviate/ChromaDB client
│   │   │   │
│   │   │   ├── gemini/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── client.py          # Gemini API wrapper
│   │   │   │   ├── prompts.py         # All prompt templates
│   │   │   │   └── notebooklm.py      # NotebookLM pod manager
│   │   │   │
│   │   │   └── voice/
│   │   │       ├── __init__.py
│   │   │       ├── speech_to_text.py
│   │   │       └── text_to_speech.py
│   │   │
│   │   ├── db/                        # Database configs
│   │   │   ├── __init__.py
│   │   │   ├── postgres.py            # User data, sessions
│   │   │   └── neo4j.py               # Graph DB connection
│   │   │
│   │   └── utils/
│   │       ├── __init__.py
│   │       ├── rate_limiter.py
│   │       ├── pdf_utils.py
│   │       └── auth.py
│   │
│   └── tests/
│       ├── conftest.py
│       ├── test_ingestion.py
│       ├── test_hypothesis.py
│       ├── test_graph.py
│       └── test_simulation.py
│
├── frontend/                          # Next.js 14 frontend
│   ├── Dockerfile
│   ├── package.json
│   ├── next.config.js
│   ├── tailwind.config.ts
│   ├── tsconfig.json
│   │
│   ├── public/
│   │   ├── logo.svg
│   │   └── demo-papers/              # Pre-loaded demo PDFs
│   │
│   ├── src/
│   │   ├── app/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx               # Landing page
│   │   │   ├── globals.css
│   │   │   │
│   │   │   └── dashboard/
│   │   │       ├── layout.tsx         # Dashboard shell
│   │   │       ├── page.tsx           # Main dashboard
│   │   │       ├── upload/page.tsx
│   │   │       ├── graph/page.tsx
│   │   │       ├── hypotheses/page.tsx
│   │   │       ├── contradictions/page.tsx
│   │   │       ├── gaps/page.tsx
│   │   │       ├── simulate/page.tsx
│   │   │       ├── protocols/page.tsx
│   │   │       ├── voice/page.tsx
│   │   │       ├── pods/page.tsx
│   │   │       └── settings/page.tsx
│   │   │
│   │   ├── components/
│   │   │   ├── ui/                    # shadcn/ui components
│   │   │   ├── graph/
│   │   │   │   ├── KnowledgeGraph3D.tsx   # react-force-graph-3d
│   │   │   │   ├── GraphControls.tsx
│   │   │   │   ├── NodeDetail.tsx
│   │   │   │   └── EdgeDetail.tsx
│   │   │   ├── hypothesis/
│   │   │   │   ├── HypothesisList.tsx
│   │   │   │   ├── HypothesisCard.tsx
│   │   │   │   ├── ScoreBar.tsx
│   │   │   │   └── EvidenceChain.tsx
│   │   │   ├── papers/
│   │   │   │   ├── PaperUpload.tsx
│   │   │   │   ├── PaperCard.tsx
│   │   │   │   └── PaperDetail.tsx
│   │   │   ├── contradiction/
│   │   │   │   ├── ContradictionMap.tsx
│   │   │   │   └── ClaimComparison.tsx
│   │   │   ├── simulation/
│   │   │   │   ├── SimulationRunner.tsx
│   │   │   │   └── ProtocolViewer.tsx
│   │   │   ├── voice/
│   │   │   │   └── VoiceQuery.tsx
│   │   │   └── layout/
│   │   │       ├── Sidebar.tsx
│   │   │       ├── Header.tsx
│   │   │       └── SearchBar.tsx
│   │   │
│   │   ├── lib/
│   │   │   ├── api.ts                 # Backend API client
│   │   │   ├── types.ts               # TypeScript types
│   │   │   └── utils.ts
│   │   │
│   │   ├── hooks/
│   │   │   ├── useGraph.ts
│   │   │   ├── useHypotheses.ts
│   │   │   ├── useVoice.ts
│   │   │   └── useSSE.ts             # Server-Sent Events
│   │   │
│   │   └── store/
│   │       ├── graphStore.ts          # Zustand
│   │       ├── paperStore.ts
│   │       └── sessionStore.ts
│   │
│   └── tests/
│       └── components/
│
├── infrastructure/
│   ├── terraform/                     # GCP infrastructure
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── cloud_run.tf
│   │   ├── storage.tf
│   │   ├── pubsub.tf
│   │   └── networking.tf
│   │
│   ├── kubernetes/                    # K8s manifests (prod)
│   │   ├── backend-deployment.yaml
│   │   ├── frontend-deployment.yaml
│   │   ├── neo4j-statefulset.yaml
│   │   └── ingress.yaml
│   │
│   └── grobid/
│       └── Dockerfile                 # Custom GROBID config
│
├── scripts/
│   ├── seed_papers.py                 # Pre-load demo papers
│   ├── init_neo4j.py                  # Create constraints/indexes
│   ├── bulk_ingest.py                 # One-time bulk ingestion
│   └── benchmark.py                   # Performance benchmarks
│
├── data/
│   ├── demo_papers/                   # 50-100 pre-selected PDFs
│   ├── sample_hypotheses.json
│   └── graph_schema.cypher            # Neo4j schema DDL
│
└── docs/
    ├── architecture.md
    ├── api.md
    └── deployment.md
```

---

## SECTION 8: HACKATHON DEMO PLAN

### 8.1 Demo Domain Selection: Cancer Immunotherapy

**Why cancer immunotherapy:**
- High-impact, emotionally resonant with judges
- Rapid publication rate (~15,000 papers/year) → rich connection graph
- Real unresolved contradictions (checkpoint inhibitor response rates vary wildly)
- Cross-domain potential (CS + biology + chemistry + clinical medicine)
- Well-understood by general audience ("fighting cancer with AI")

### 8.2 Pre-Loading Strategy

**Before the hackathon (1-2 weeks prior):**
- Download 847 papers on cancer immunotherapy from PubMed + arXiv
  - Top 500 most-cited papers (2020-2026) on PD-1/PD-L1, CAR-T, checkpoint inhibitors
  - 200 recent papers (last 6 months) for "live" discovery demo
  - 147 cross-domain papers (computational + clinical + materials science)
- Process all through GROBID → structured XML
- Extract claims and entities → populate Neo4j graph
- Generate embeddings → load into Weaviate/ChromaDB
- Pre-generate 50+ hypotheses with scoring
- **Total pre-processing time: ~8-12 hours on a 16-CPU machine**

### 8.3 Live Demo Script (12 minutes)

**Minute 0-2: The Problem**
"Last year, $2.3 trillion was spent on R&D. 23 days per researcher were wasted rediscovering things already published. Breakthroughs sit in papers no one has read together."

**Minute 2-4: Upload & Instant Integration**
- Drag-and-drop a new paper onto the platform
- Watch it appear in the knowledge graph in real-time
- Show the paper's claims being extracted and connected

**Minute 4-6: The Hidden Connection** (HERO MOMENT)
- "OMNISCIENT found that Paper A (2022, immune checkpoint) and Paper C (2023, materials science) both describe the same nanoparticle mechanism — but neither cites the other."
- Show the graph path connecting them through intermediate concepts
- "No human has made this connection. We verified — it's not in any published review."

**Minute 6-8: Hypothesis Generation**
- Show 10 generated hypotheses ranked by score
- Click the top hypothesis — show evidence chain, novelty score, plausibility analysis
- "This hypothesis suggests combining Method X from Paper A with Target Y from Paper C could improve CAR-T cell persistence by 40%."

**Minute 8-10: Experiment Simulation**
- Click "Simulate Experiment"
- Show predicted outcomes, statistical power, resource requirements
- "OMNISCIENT predicts this experiment would require 120 mice, 14 weeks, and $45,000 — with a 73% probability of confirming the hypothesis."

**Minute 10-11: Live arXiv Moment**
- "This paper was published on arXiv YESTERDAY."
- Show it appearing in the ingestion feed
- Show new connections forming in real-time

**Minute 11-12: Voice Query**
- Speak: "What are the most promising untested approaches for improving PD-1 inhibitor response rates?"
- OMNISCIENT responds with a synthesized answer citing 5 papers from different domains

### 8.4 Domain Expert Validator

If possible, have a cancer researcher in the audience or on video call confirm: "Yes, that connection between Papers A and C is novel — I've never seen it published."

---

## SECTION 9: 24-HOUR BUILD SPRINT PLAN

### Pre-Hackathon (MUST COMPLETE BEFORE)

| Task | Time | Priority |
|------|------|----------|
| Download & parse 847 papers through GROBID | 4 hours | CRITICAL |
| Extract claims & entities, load Neo4j | 3 hours | CRITICAL |
| Generate embeddings, load vector DB | 2 hours | CRITICAL |
| Pre-generate 50 hypotheses | 2 hours | CRITICAL |
| Set up GCP project, enable APIs, get keys | 1 hour | CRITICAL |
| Set up Neo4j AuraDB instance | 30 min | CRITICAL |
| Deploy GROBID Docker on a cloud VM | 30 min | HIGH |
| Prepare demo paper PDFs (the "wow" connections) | 2 hours | HIGH |
| **Total pre-work: ~15 hours** | | |

### Hour-by-Hour Hackathon Schedule

```
HOUR 0-1: Foundation
├── Person 1 (Backend): FastAPI scaffolding, config, Neo4j connection
├── Person 2 (Frontend): Next.js project, Tailwind, shadcn/ui, routing
├── Person 3 (AI/Data): Verify pre-loaded data, test Gemini API calls
└── Checkpoint: Backend serves /health, Frontend renders dashboard shell

HOUR 1-3: Core Backend APIs
├── P1: Paper upload endpoint, GROBID integration, ingestion pipeline
├── P2: Dashboard layout, sidebar, paper upload UI with drag-and-drop
├── P3: Claim extraction prompts, Neo4j query functions, embedding search
└── Checkpoint: Can upload a PDF and see it parsed

HOUR 3-5: Knowledge Graph Visualization
├── P1: Graph query API — nodes, edges, neighborhood expansion
├── P2: react-force-graph-3d integration, node/edge rendering
├── P3: Cross-paper connection engine, contradiction detector
└── Checkpoint: Interactive 3D knowledge graph with real data

HOUR 5-7: Hypothesis Engine
├── P1: Hypothesis generation endpoints, gap detection queries
├── P2: Hypothesis explorer UI — list, cards, score bars, evidence chains
├── P3: Novelty scorer, plausibility scorer, impact estimator prompts
└── Checkpoint: Can generate and display ranked hypotheses

HOUR 7-9: Experiment Simulation
├── P1: Simulation endpoint, protocol generator
├── P2: Simulation UI, protocol viewer
├── P3: Power calculator, resource estimator, outcome predictor
└── Checkpoint: End-to-end hypothesis → simulation → protocol

HOUR 9-11: Cross-Paper Connections (HERO FEATURE)
├── P1: Hidden connection API, semantic vs citation proximity
├── P2: Connection highlighting in graph (gold dashed lines), detail panel
├── P3: Pre-compute and cache the "wow" connections for demo
└── Checkpoint: Can show hidden connections with explanations

HOUR 11-13: Live Ingestion + Voice
├── P1: arXiv polling scheduler, real-time SSE notifications
├── P2: Live feed UI, toast notifications for new papers
├── P3: Voice query integration (Web Speech API + Gemini)
└── Checkpoint: New papers appear live, voice queries work

HOUR 13-15: Contradiction Map + Gap Map
├── P1: Contradiction query API, gap detection API
├── P2: Contradiction heatmap UI, research gap visualization
├── P3: Polish prompts, improve hypothesis quality
└── Checkpoint: All core features functional

HOUR 15-18: Polish & Integration
├── ALL: Bug fixes, UI polish, loading states, error handling
├── P2: Animations, transitions, responsive design
├── P3: Optimize Gemini calls (caching, batching)
└── Checkpoint: All features work end-to-end without errors

HOUR 18-20: Demo Preparation
├── ALL: Rehearse demo 3 times
├── P1: Ensure backend stability under demo load
├── P2: Landing page, marketing copy, screenshots
├── P3: Prepare specific demo data paths (the "wow" moments)
└── Checkpoint: Demo rehearsed, all "wow" moments verified

HOUR 20-22: Final Polish
├── ALL: Fix any remaining bugs
├── Record backup demo video
├── Prepare pitch deck (3-5 slides max)
└── Checkpoint: Ship-ready

HOUR 22-24: Buffer
├── Sleep if possible
├── Final demo rehearsal
└── Submit
```

### Critical Path

The critical path is: **Neo4j data pre-loaded → Graph visualization working → Hypothesis engine producing results → Hidden connection demo verified**. Everything else is supporting. If you're behind schedule at Hour 10, cut voice, cut collaboration, cut contradiction map — protect the graph + hypotheses + connections.

---

## SECTION 10: PRODUCTION & SCALE

### 10.1 Cost Modeling at Scale

**Corpus: 10M papers (full PubMed + arXiv)**

| Component | Calculation | Monthly Cost |
|-----------|-------------|-------------|
| **PDF Storage** | 10M papers × 2MB avg = 20TB on GCS | $400/mo |
| **Parsed XML Storage** | 10M × 500KB = 5TB on GCS | $100/mo |
| **SPECTER2 Embeddings** | 10M × 768 dims × 4 bytes = 30GB (Weaviate) | $150/mo |
| **Gemini Embeddings** | 10M papers × 5 chunks × 500 tokens = 25B tokens | $5,000 (one-time) |
| **Neo4j Enterprise** | 10M papers → ~100M nodes, ~500M edges | $2,500/mo (AuraDB Pro) |
| **Weaviate Cloud** | 50M vectors, 768 dims | $500/mo |
| **Gemini 2.0 Flash (ingestion)** | 10M × 5K tokens input = 50B tokens | $5,000 (one-time) |
| **Gemini 2.5 Pro (queries)** | 100K queries/mo × 10K tokens = 1B tokens | $1,250/mo input + $10,000/mo output |
| **Cloud Run (backend)** | 4 vCPUs, 16GB RAM, always-on | $300/mo |
| **GROBID cluster** | 3× n1-standard-16 VMs | $1,500/mo |
| **Cloud Storage egress** | 5TB/mo | $600/mo |
| **Total operational** | | **~$17,300/mo** |
| **One-time ingestion** | | **~$10,000** |

### 10.2 Query Latency Targets

| Operation | Target | Strategy |
|-----------|--------|----------|
| Paper upload → graph | < 30 seconds | Async pipeline, pre-warmed GROBID |
| Semantic search | < 200ms | Weaviate HNSW index, pre-computed embeddings |
| Graph neighborhood query | < 100ms | Neo4j native traversal, index on node IDs |
| Hypothesis generation | < 45 seconds | Gemini streaming, cached intermediate results |
| Experiment simulation | < 30 seconds | Pre-computed analogous experiments |
| Voice query → response | < 5 seconds | Streaming STT + cached graph context |

### 10.3 Scaling Strategy

- **Read replicas**: Neo4j read replicas for query scaling
- **Embedding cache**: Redis cache for frequently-queried paper embeddings
- **CDN**: Vercel Edge for frontend, CloudFlare for API caching
- **Batch processing**: Cloud Dataflow for periodic bulk re-indexing
- **Sharding**: Shard by research domain for independent scaling

---

## SECTION 11: TECHNICAL RISKS & MITIGATIONS

| Risk | Severity | Likelihood | Mitigation |
|------|----------|------------|------------|
| **arXiv rate limit (1 req/3s)** | Medium | High | Use OAI-PMH for bulk. Pre-download via S3. Cache aggressively. |
| **PDF parsing failures (equations, figures)** | High | High | GROBID + Nougat fallback for math. Gemini Vision for figures. Quality scoring to flag bad parses. |
| **Knowledge graph inconsistency at scale** | High | Medium | ACID transactions in Neo4j. Claim deduplication pipeline. Periodic consistency checks. |
| **Gemini hallucinating research claims** | Critical | High | Always ground in source text. Cite paper + section + page for every claim. Confidence thresholds. Human-in-the-loop for high-stakes hypotheses. |
| **NotebookLM session limits (50 sources)** | Medium | High | Multi-pod architecture. Fallback to Vertex AI RAG Engine. Prioritize most relevant papers per pod. |
| **NotebookLM API unavailability (Alpha)** | High | Medium | Full fallback stack: Vertex AI RAG Engine + Gemini 2.5 Pro. NotebookLM is enhancement, not dependency. |
| **Cost overrun on Gemini API** | Medium | Medium | Gemini 2.0 Flash for bulk (10x cheaper). Aggressive caching. Rate limiting per user. Budget alerts. |
| **Neo4j scaling beyond 1B edges** | Medium | Low (at launch) | Sharding by domain. Read replicas. Consider Neptune or Spanner for >1B scale. |
| **Embedding drift as models update** | Low | Medium | Version all embeddings. Re-embed incrementally on model updates. |
| **Demo failure during hackathon** | Critical | Medium | Pre-compute all demo paths. Backup video recording. Offline fallback mode with cached results. |

---

## SECTION 12: BUSINESS MODEL

### 12.1 Pricing Tiers

| Tier | Target | Price | Includes |
|------|--------|-------|----------|
| **Researcher** | Individual academics | $29/mo | 500 paper uploads, 100 hypotheses/mo, 1 domain |
| **Lab** | Research groups (5-15) | $199/mo | 5,000 papers, unlimited hypotheses, 5 domains, collaboration |
| **Institution** | Universities, hospitals | $2,999/mo | Unlimited papers, custom domains, API access, SSO, dedicated support |
| **Enterprise** | Pharma R&D | $15,000/mo | Unlimited everything, custom models, on-premise option, SLA |

### 12.2 Revenue Streams

1. **Research institution subscriptions** (primary): Universities, hospitals, national labs. Sell to library/IT procurement. Similar to Elsevier/Springer pricing model but for AI analysis.

2. **Pharma R&D licensing**: Drug discovery teams. Premium tier with binding affinity prediction, clinical trial design, patent landscape analysis. $50K-500K/year per pharmaceutical company.

3. **Grant discovery platform**: Match researchers with funding opportunities based on their research graph. Partnership with grant databases (NIH Reporter, NSF Award Search). Commission or premium feature.

4. **Academic publisher partnerships**: Offer OMNISCIENT as a value-add to journal subscriptions. Revenue share with Elsevier, Springer Nature, Wiley. They get AI differentiation; we get corpus access.

5. **API access**: Let other tools (lab management systems, ELN software) integrate OMNISCIENT's hypothesis engine. Usage-based pricing ($0.10 per hypothesis generated).

### 12.3 Market Sizing

- **TAM**: 8M researchers globally × $29/mo avg = $2.8B/yr
- **SAM**: 2M researchers in top-50 countries with institutional budgets = $700M/yr
- **SOM (Year 3)**: 50K paying researchers = $17M ARR

---

## SECTION 13: JUDGING CRITERIA + PITCH STRUCTURE

### 13.1 Google Hackathon Judging Alignment

| Criterion | Weight | How OMNISCIENT Scores |
|-----------|--------|----------------------|
| **Technical Complexity** | 25% | Multi-model pipeline (Gemini + SPECTER2), knowledge graph, real-time ingestion, hypothesis engine. Extremely deep. |
| **Use of Google Technology** | 25% | Gemini 2.5 Pro, Gemini 2.0 Flash, Gemini Embedding 2, NotebookLM, Document AI, Vertex AI RAG, Cloud Run, Pub/Sub, Speech APIs, Cloud Storage, BigQuery. **Maximum Google stack.** |
| **Impact / Usefulness** | 25% | Accelerates scientific discovery. Could literally cure diseases faster. Addresses $2.3T R&D waste. |
| **Creativity / Innovation** | 15% | No existing product does cross-paper hypothesis generation from a live knowledge graph. Novel combination of graph + LLM + embeddings. |
| **Design / UX** | 10% | 3D knowledge graph is visually stunning. Voice interface is compelling. Clean, professional dashboard. |

### 13.2 Three-Minute Pitch Structure

```
[0:00-0:30] THE HOOK
"Every 23 days, a researcher wastes an entire day rediscovering
something already published. Last year, that cost $2.3 trillion
globally. But worse — breakthroughs are sitting in papers right
now that no one will find because no human has read both papers."

[0:30-1:00] THE DEMO — UPLOAD
*Upload paper, watch it integrate into the knowledge graph*
"OMNISCIENT reads every paper, builds a living knowledge graph,
and finds connections no human would find."

[1:00-1:45] THE DEMO — HIDDEN CONNECTION + HYPOTHESIS
*Show the hidden connection between two uncited papers*
"These two papers — one from Stanford, one from Tokyo — describe
the same mechanism. Neither cites the other. OMNISCIENT found them."
*Show generated hypothesis*
"And from this connection, OMNISCIENT generated this hypothesis —
verified as novel against the entire literature."

[1:45-2:15] THE DEMO — SIMULATION
*Show experiment simulation*
"Click simulate: OMNISCIENT predicts the experiment outcome, calculates
sample size, estimates cost. You know whether to run it before
touching a pipette."

[2:15-2:45] THE TECH
"Built on Gemini 2.5 Pro with 1M context, NotebookLM research pods,
a Neo4j knowledge graph with 847 papers and 12,000 extracted claims,
SPECTER2 embeddings, and live arXiv integration."

[2:45-3:00] THE CLOSE
"OMNISCIENT is the world's first AI post-doc. It's read everything.
It never forgets. And it just found a connection that could change
cancer treatment — before any other lab on Earth."
```

### 13.3 Demo Script Fallback

If live demo fails: switch to pre-recorded video (recorded at Hour 20-22 of the sprint). Always have this ready.

---

## SECTION 14: APPENDICES

### 14.1 Full Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Frontend** | Next.js | 14.x |
| **UI Components** | shadcn/ui + Radix | latest |
| **Styling** | Tailwind CSS | 3.x |
| **Graph Viz** | react-force-graph-3d | 1.x |
| **Charts** | Recharts | 2.x |
| **State** | Zustand | 4.x |
| **Backend** | Python + FastAPI | 3.12 / 0.110+ |
| **Task Queue** | Celery + Redis | 5.x |
| **Graph DB** | Neo4j | 5.x (AuraDB) |
| **Vector DB** | Weaviate (prod) / ChromaDB (dev) | 1.25+ / 0.5+ |
| **Relational DB** | PostgreSQL | 16 |
| **PDF Parsing** | GROBID | 0.8.1 |
| **Embeddings** | SPECTER2 (allenai) + Gemini Embedding 2 | - |
| **LLM** | Gemini 2.5 Pro + 2.0 Flash | latest |
| **Object Storage** | Google Cloud Storage | - |
| **Compute** | Google Cloud Run | - |
| **Events** | Google Pub/Sub | - |
| **Voice** | Google Speech-to-Text v2 + TTS | - |
| **Container** | Docker + Docker Compose | - |
| **IaC** | Terraform | 1.x |

### 14.2 Sample Knowledge Graph Schema (Cypher DDL)

```cypher
// Constraints
CREATE CONSTRAINT paper_id IF NOT EXISTS FOR (p:Paper) REQUIRE p.id IS UNIQUE;
CREATE CONSTRAINT author_id IF NOT EXISTS FOR (a:Author) REQUIRE a.id IS UNIQUE;
CREATE CONSTRAINT concept_id IF NOT EXISTS FOR (c:Concept) REQUIRE c.id IS UNIQUE;
CREATE CONSTRAINT claim_id IF NOT EXISTS FOR (cl:Claim) REQUIRE cl.id IS UNIQUE;
CREATE CONSTRAINT hypothesis_id IF NOT EXISTS FOR (h:Hypothesis) REQUIRE h.id IS UNIQUE;
CREATE CONSTRAINT experiment_id IF NOT EXISTS FOR (e:Experiment) REQUIRE e.id IS UNIQUE;
CREATE CONSTRAINT method_id IF NOT EXISTS FOR (m:Method) REQUIRE m.id IS UNIQUE;
CREATE CONSTRAINT dataset_id IF NOT EXISTS FOR (d:Dataset) REQUIRE d.id IS UNIQUE;
CREATE CONSTRAINT institution_id IF NOT EXISTS FOR (i:Institution) REQUIRE i.id IS UNIQUE;

// Indexes for common query patterns
CREATE INDEX paper_year IF NOT EXISTS FOR (p:Paper) ON (p.year);
CREATE INDEX paper_domain IF NOT EXISTS FOR (p:Paper) ON (p.categories);
CREATE INDEX concept_name IF NOT EXISTS FOR (c:Concept) ON (c.name);
CREATE INDEX claim_type IF NOT EXISTS FOR (cl:Claim) ON (cl.type);
CREATE INDEX hypothesis_score IF NOT EXISTS FOR (h:Hypothesis) ON (h.composite_score);
CREATE INDEX author_name IF NOT EXISTS FOR (a:Author) ON (a.name);

// Full-text search indexes
CREATE FULLTEXT INDEX paper_search IF NOT EXISTS
FOR (p:Paper) ON EACH [p.title, p.abstract];
CREATE FULLTEXT INDEX claim_search IF NOT EXISTS
FOR (cl:Claim) ON EACH [cl.text];
```

### 14.3 Sample Hypothesis Output Format

```json
{
  "id": "hyp-a7f3c2d1-4e5b-4c8a-9b2e-1d3f5a7c9b0e",
  "statement": "Combining anti-PD-L1 checkpoint blockade with ferritin-based nanoparticle delivery of IL-12 may enhance tumor-infiltrating lymphocyte persistence in cold tumors, achieving response rates >40% where current monotherapy achieves <15%.",
  "domain": "cancer_immunotherapy",
  "generated_at": "2026-03-22T14:30:00Z",
  "scores": {
    "novelty": 0.89,
    "plausibility": 0.76,
    "impact": 0.92,
    "composite": 0.853
  },
  "evidence_chain": [
    {
      "paper_id": "pubmed:38571234",
      "title": "Ferritin nanoparticles as immunomodulatory carriers",
      "claim": "Ferritin cages achieve 3x longer circulation time than PEGylated liposomes",
      "relevance": "Delivery mechanism"
    },
    {
      "paper_id": "arxiv:2401.09876",
      "title": "IL-12 microenvironment remodeling in cold tumors",
      "claim": "Local IL-12 delivery converts cold tumors to hot in 72% of murine models",
      "relevance": "Therapeutic mechanism"
    },
    {
      "paper_id": "pubmed:39012345",
      "title": "Anti-PD-L1 combination strategies: a meta-analysis",
      "claim": "Cytokine co-delivery improves checkpoint inhibitor response by 2.3x",
      "relevance": "Combination rationale"
    }
  ],
  "gap_type": "untested_intersection",
  "proposed_experiment": {
    "model": "B16-F10 melanoma, murine",
    "method": "Ferritin-IL12 nanoparticle + anti-PD-L1 combination",
    "primary_endpoint": "Tumor-infiltrating CD8+ T-cell count at day 14",
    "secondary_endpoints": ["Tumor volume", "Overall survival at 60 days"],
    "estimated_sample_size": 120,
    "estimated_duration_weeks": 14,
    "estimated_cost_usd": 45000
  },
  "novelty_check": {
    "closest_existing_paper": "pubmed:38899999",
    "similarity": 0.71,
    "distinction": "Existing paper uses liposomal delivery, not ferritin. Different pharmacokinetics."
  }
}
```

### 14.4 arXiv Integration Code Sample

```python
"""Complete arXiv ingestion pipeline for OMNISCIENT."""
import arxiv
import asyncio
import aiohttp
from pathlib import Path
from datetime import datetime, timedelta

class ArxivIngestionPipeline:
    def __init__(self, grobid_url: str, neo4j_client, vector_db, gemini_client):
        self.arxiv_client = arxiv.Client(
            page_size=50,
            delay_seconds=3.0,  # Respect rate limit
            num_retries=3
        )
        self.grobid_url = grobid_url
        self.neo4j = neo4j_client
        self.vector_db = vector_db
        self.gemini = gemini_client

    async def fetch_recent_papers(self, categories: list[str],
                                   hours_back: int = 1) -> list[arxiv.Result]:
        """Fetch papers submitted in the last N hours."""
        query = " OR ".join(f"cat:{cat}" for cat in categories)
        search = arxiv.Search(
            query=query,
            max_results=100,
            sort_by=arxiv.SortCriterion.SubmittedDate,
            sort_order=arxiv.SortOrder.Descending
        )
        cutoff = datetime.utcnow() - timedelta(hours=hours_back)
        results = []
        for paper in self.arxiv_client.results(search):
            if paper.published.replace(tzinfo=None) >= cutoff:
                results.append(paper)
            else:
                break
        return results

    async def process_paper(self, paper: arxiv.Result) -> dict:
        """Full pipeline: download → parse → extract → embed → store."""
        # 1. Download PDF
        pdf_path = Path(f"/tmp/papers/{paper.get_short_id()}.pdf")
        pdf_path.parent.mkdir(parents=True, exist_ok=True)
        paper.download_pdf(dirpath=str(pdf_path.parent), filename=pdf_path.name)

        # 2. Parse with GROBID
        async with aiohttp.ClientSession() as session:
            with open(pdf_path, "rb") as f:
                resp = await session.post(
                    f"{self.grobid_url}/api/processFulltextDocument",
                    data={"input": f},
                    params={"consolidateHeader": "1", "includeRawAffiliations": "1"}
                )
                tei_xml = await resp.text()

        # 3. Extract claims via Gemini Flash
        structured = parse_tei_xml(tei_xml)
        claims = await self.gemini.extract_claims(structured)

        # 4. Generate embeddings
        embedding = self.specter2.embed(f"{paper.title} [SEP] {paper.summary}")

        # 5. Store in Neo4j
        await self.neo4j.execute("""
            MERGE (p:Paper {id: $id})
            SET p.title = $title, p.abstract = $abstract,
                p.year = $year, p.categories = $categories,
                p.pdf_url = $pdf_url, p.ingested_at = datetime()
        """, id=paper.entry_id, title=paper.title,
             abstract=paper.summary, year=paper.published.year,
             categories=[c.term for c in paper.categories],
             pdf_url=paper.pdf_url)

        # Store claims
        for claim in claims:
            await self.neo4j.execute("""
                MATCH (p:Paper {id: $paper_id})
                CREATE (c:Claim {id: $claim_id, text: $text,
                        type: $type, confidence: $conf})
                CREATE (p)-[:MAKES_CLAIM]->(c)
            """, paper_id=paper.entry_id, claim_id=claim["id"],
                 text=claim["text"], type=claim["type"], conf=claim["confidence"])

        # 6. Upsert to vector DB
        await self.vector_db.upsert(
            id=paper.entry_id,
            vector=embedding.tolist(),
            metadata={"title": paper.title, "year": paper.published.year,
                      "categories": [c.term for c in paper.categories]}
        )

        # 7. Find new connections
        connections = await self.find_connections(paper.entry_id, embedding)

        return {"paper_id": paper.entry_id, "claims": len(claims),
                "connections": len(connections)}

    async def run_hourly(self):
        """Main loop — called by Cloud Scheduler."""
        categories = ["cs.AI", "cs.LG", "q-bio.BM", "q-bio.GN",
                       "cond-mat.mtrl-sci", "physics.bio-ph"]
        papers = await self.fetch_recent_papers(categories, hours_back=1)
        results = []
        for paper in papers:
            result = await self.process_paper(paper)
            results.append(result)
        return {"processed": len(results), "results": results}
```

### 14.5 Architecture Diagram (System Context)

```
                    ┌──────────────┐
                    │  Researchers │
                    │  (Browser)   │
                    └──────┬───────┘
                           │ HTTPS
                           ▼
                    ┌──────────────┐
                    │   Vercel /   │
                    │   Cloud Run  │
                    │  (Next.js)   │
                    └──────┬───────┘
                           │ REST API
                           ▼
                    ┌──────────────┐
                    │  Cloud Run   │◄──── Cloud Scheduler (hourly)
                    │  (FastAPI)   │
                    └──┬───┬───┬───┘
                       │   │   │
          ┌────────────┘   │   └────────────┐
          ▼                ▼                ▼
   ┌────────────┐  ┌─────────────┐  ┌──────────────┐
   │   Neo4j    │  │  Weaviate   │  │  PostgreSQL  │
   │  AuraDB    │  │  (vectors)  │  │  (users,     │
   │  (graph)   │  │             │  │   sessions)  │
   └────────────┘  └─────────────┘  └──────────────┘

          ┌────────────────────────────────┐
          │        External APIs           │
          │  Gemini 2.5 Pro / 2.0 Flash   │
          │  Document AI  │  Speech APIs   │
          │  NotebookLM Enterprise (opt)   │
          │  arXiv  │  PubMed  │  CrossRef │
          └────────────────────────────────┘

          ┌────────────────────────────────┐
          │        Data Pipeline           │
          │  Pub/Sub → Cloud Run Jobs      │
          │  GROBID (Docker) → parsing     │
          │  Cloud Storage (paper PDFs)    │
          └────────────────────────────────┘
```

---

## VERIFICATION & TESTING PLAN

### End-to-End Test Sequence

1. **Paper Upload Test**: Upload a PDF → verify it appears in Neo4j (claims, entities) + Weaviate (embedding) within 30 seconds
2. **Search Test**: Search "CRISPR efficiency" → verify relevant papers returned with semantic ranking
3. **Graph Test**: Query a paper's neighborhood → verify connected claims, concepts, authors render correctly
4. **Hidden Connection Test**: Verify the pre-computed hidden connections display with explanation
5. **Hypothesis Test**: Generate hypotheses for "cancer immunotherapy" → verify scored, ranked, with evidence chains
6. **Simulation Test**: Simulate the top hypothesis → verify outcome prediction, power calculation, resource estimate
7. **Contradiction Test**: Query contradictions in a domain → verify conflicting claims displayed with sources
8. **Live Ingestion Test**: Wait for arXiv poll → verify new paper appears in feed and graph
9. **Voice Test**: Speak a query → verify transcription, processing, and audio response
10. **Demo Run**: Execute the full 12-minute demo script end-to-end without errors

### Automated Tests

```bash
# Backend
cd backend && pytest tests/ -v --cov=app --cov-report=term-missing

# Frontend
cd frontend && npm run test

# Integration (requires running services)
cd backend && pytest tests/integration/ -v --timeout=60

# Load test
locust -f tests/load/locustfile.py --host=http://localhost:8000
```
