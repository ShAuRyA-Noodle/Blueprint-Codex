---

# ORACLE: Omniscient Repository of Assembled Collective Literary Expertise

## A Complete Production-Grade Implementation Blueprint

### Context

**Build context**: Multi-week production build (not hackathon-constrained). Solo or small team. Full provider flexibility — not locked to Google Cloud. The blueprint should be treated as a reference architecture document. The hackathon sections (8, 9) are retained as optional accelerated-demo strategies but the primary focus is production-grade quality.

**Provider flexibility note**: Throughout this document, Google-specific services (Vertex AI, Gemini, Firebase) are used as reference implementations. All components have provider-agnostic alternatives:
- **LLM**: Gemini 1.5 Pro → also OpenAI GPT-4o (128K), Claude Opus/Sonnet (200K), or any model with large context
- **Embeddings**: Gemini text-embedding-004 → also OpenAI text-embedding-3-large, Cohere embed-v3, or open-source (e5-mistral)
- **Vector DB**: Vertex AI Vector Search → also Pinecone, Weaviate, Qdrant, Milvus, pgvector
- **Object Storage**: Cloud Storage → also S3, R2, MinIO
- **Auth/User Data**: Firebase → also Supabase, Auth0 + Postgres, Clerk
- **Compute**: Cloud Run → also AWS Lambda, Fly.io, Railway, self-hosted
- **Analytics DB**: BigQuery → also Postgres, DuckDB, ClickHouse

---

## SECTION 1: PROBLEM DEFINITION AND VISION

### 1.1 The Knowledge Access Problem

Humanity has produced an estimated 130 million unique books since the invention of the printing press. The average adult reads approximately 12 books per year. At that rate, a person with a 60-year reading life will consume roughly 720 books -- 0.00055% of all published human knowledge. Even the most voracious reader in history, someone reading a book per day for 80 years, would reach only 29,200 books -- still 0.022%. The problem is not that knowledge does not exist. The problem is that no human being can access more than a negligible fraction of it within a lifetime.

### 1.2 The Synthesis Problem

Knowledge does not exist in isolation. The most powerful insights come from synthesis across domains: a military strategist reading economic theory, a physician reading philosophy, a CEO reading behavioral psychology. Yet the current tools for knowledge access are single-domain, single-source, and linear. You read one book at a time, by one author, from one perspective. Cross-pollination happens only by accident or extraordinary effort.

### 1.3 Why Existing Tools Fall Short

- **Google Books / Google Scholar**: Search-based, returning individual passages. No synthesis, no multi-perspective reasoning, no awareness of contradictions between sources.
- **Kindle / Apple Books**: Consumption-oriented. Highlights are siloed per book. No cross-book reasoning.
- **ChatGPT / Perplexity**: Trained on web data, not deeply grounded in specific texts. Cannot cite specific passages from specific editions. Prone to hallucinated attributions.
- **Wikipedia**: Summary-oriented, consensus-driven. Eliminates the very disagreements and tensions that produce insight.

### 1.4 The "Consulting Greatest Minds" Vision

ORACLE transforms the act of seeking knowledge from "reading a book" to "consulting a panel." When you ask ORACLE a question, you are not searching -- you are convening. Imagine sitting at a table with Sun Tzu, Adam Smith, Marie Curie, and Frantz Fanon, each responding to your question from their documented worldview, grounded in their actual writings, with contradictions surfaced rather than suppressed.

### 1.5 Five Primary Use Cases

1. **Strategic Decisions**: A startup founder asks "Should I pursue rapid growth or sustainable profitability?" ORACLE surfaces perspectives from Peter Thiel's "Zero to One," Adam Smith's "Wealth of Nations," Nassim Taleb's "Antifragile," and Sun Tzu's "Art of War" -- each grounded in cited passages, contradictions mapped.

2. **Philosophical Dilemmas**: A student asks "Is free will real?" ORACLE convenes Spinoza, Kant, Sartre, Libet (neuroscience), and Buddhist philosophy -- not a summary, but a structured debate with primary-source citations.

3. **Legal Research**: A lawyer asks about the evolution of privacy rights. ORACLE traces the concept from Brandeis's 1890 "Right to Privacy" through Griswold v. Connecticut to GDPR scholarship, surfacing the philosophical and legal tensions.

4. **Medical Decisions**: A patient asks about treatment trade-offs. ORACLE surfaces relevant medical literature, ethical frameworks (autonomy vs. beneficence), and historical context -- always with citations, never with medical advice framing.

5. **Creative Inspiration**: A novelist asks "How have authors depicted the experience of exile?" ORACLE pulls from Ovid, Dante, Nabokov, Said, and Adichie -- not summaries but specific passages and techniques.

### 1.6 Why Large Context Windows Are the Unlock (Gemini 2M / Claude 200K / GPT-4o 128K)

Previous LLMs were limited to 4K-32K context windows. This forced RAG systems to retrieve tiny snippets, losing the surrounding argument structure. Modern large-context models (Gemini 1.5 Pro at 2M tokens, Claude at 200K, GPT-4o at 128K) mean:

- **Full-chapter retrieval**: Instead of 200-word snippets, retrieve entire chapters that preserve argumentative structure.
- **Multi-source synthesis in a single pass**: Load 150-200 relevant passages (each 1-3 pages) into a single prompt, enabling the model to find connections, contradictions, and synthesis points that snippet-based RAG misses.
- **Context caching**: With Gemini's context caching, the base corpus context costs drop by 90% on subsequent queries, making the 2M window economically viable for production.
- **Persona grounding**: Load an author's entire relevant chapter (not just a sentence) so the model can faithfully represent their reasoning style and conclusions.

---

## SECTION 2: CORE TECHNICAL ARCHITECTURE

### 2.1 Full Pipeline Overview

```
[Corpus Sources] --> [Document AI Parser] --> [Chunker + Metadata Extractor]
        |                                              |
        v                                              v
[Cloud Storage (raw)]                    [Cloud Storage (processed JSON)]
                                                       |
                                                       v
                                          [Gemini Embedding API (batch)]
                                                       |
                                                       v
                                          [Vertex AI Vector Search Index]
                                                       |
        User Query --> [Query Reformulator] --> [Hybrid Retriever]
                                                       |
                                                       v
                                          [Passage Ranker + Diversity Filter]
                                                       |
                                                       v
                                          [Pantheon Synthesis Engine (Gemini 1.5 Pro 2M)]
                                                       |
                                                       v
                                          [Structured Response + Citations]
                                                       |
                                                       v
                                          [Frontend Consultation Interface]
```

### 2.2 Corpus Ingestion Pipeline

**Sources (prioritized for hackathon):**

| Source | Count | Format | License | Priority |
|--------|-------|--------|---------|----------|
| Project Gutenberg | 70,000+ | TXT/HTML/EPUB | Public Domain | P0 - Hackathon |
| arXiv (open access) | 2M+ papers | PDF/LaTeX | CC/Open | P1 |
| PubMed Central (open access) | 8M+ | XML/PDF | Open Access | P1 |
| Internet Archive (open library) | 3M+ | Various | Public Domain/Lending | P1 |
| OpenStax (textbooks) | 50+ | HTML | CC | P0 - Hackathon |
| PhilPapers (open access subset) | 50K+ | PDF | Various | P2 |
| Legal: CourtListener | 4M+ opinions | HTML/PDF | Public Domain | P2 |
| Religious texts | 100+ | TXT | Public Domain | P0 - Hackathon |

**Parsing Pipeline (per document):**

```python
# /Users/shauryapunj/Desktop/009/services/ingestion/parser.py

async def parse_document(gcs_uri: str, doc_type: str) -> ParsedDocument:
    """
    Routes to appropriate parser based on document type.
    Returns structured ParsedDocument with chapters, metadata.
    """
    if doc_type in ("pdf", "scanned_pdf"):
        # Use Document AI OCR + Layout Parser
        result = await document_ai_client.process_document(
            name=PROCESSOR_NAME,
            raw_document={"gcs_uri": gcs_uri, "mime_type": "application/pdf"},
        )
        return extract_layout(result)
    elif doc_type == "epub":
        # Use ebooklib to extract chapters, then clean HTML
        return parse_epub(gcs_uri)
    elif doc_type in ("txt", "utf8"):
        # Direct text with heuristic chapter detection
        return parse_plaintext(gcs_uri)
    elif doc_type == "html":
        # BeautifulSoup extraction with boilerplate removal
        return parse_html(gcs_uri)
```

**Chunking Strategy:**

The chunking strategy is hierarchical, producing three levels of granularity stored simultaneously:

1. **Book-level summary** (1 chunk per book): A 500-token summary capturing the book's thesis, major arguments, and historical context. Generated by Gemini 1.5 Flash for cost efficiency.

2. **Chapter-level chunks** (1 chunk per chapter, ~2,000-5,000 tokens): Preserves the argumentative structure of a full chapter. Used for deep retrieval when a user wants to understand an author's full reasoning.

3. **Passage-level chunks** (target 512 tokens, overlap 64 tokens): The primary retrieval unit. Split at paragraph boundaries where possible, with sentence-boundary fallback. Each passage retains a pointer to its parent chapter and book.

```python
# /Users/shauryapunj/Desktop/009/services/ingestion/chunker.py

@dataclass
class ChunkConfig:
    passage_target_tokens: int = 512
    passage_overlap_tokens: int = 64
    chapter_max_tokens: int = 5000
    split_boundaries: list = field(default_factory=lambda: [
        "\n\n",      # Double newline (paragraph break) - preferred
        "\n",        # Single newline
        ". ",        # Sentence boundary
        "? ",        # Question mark
        "! ",        # Exclamation
    ])

def chunk_document(parsed: ParsedDocument, config: ChunkConfig) -> list[Chunk]:
    chunks = []
    
    # Level 1: Book summary (generated later by LLM)
    chunks.append(Chunk(
        level="book",
        book_id=parsed.book_id,
        text=None,  # Filled by summary generation step
        metadata=parsed.metadata,
    ))
    
    # Level 2: Chapter chunks
    for chapter in parsed.chapters:
        chunks.append(Chunk(
            level="chapter",
            book_id=parsed.book_id,
            chapter_idx=chapter.index,
            chapter_title=chapter.title,
            text=chapter.full_text[:config.chapter_max_tokens * 4],  # approx
            metadata=parsed.metadata,
        ))
        
        # Level 3: Passage chunks with sliding window
        passages = split_with_overlap(
            text=chapter.full_text,
            target_tokens=config.passage_target_tokens,
            overlap_tokens=config.passage_overlap_tokens,
            boundaries=config.split_boundaries,
        )
        for i, passage in enumerate(passages):
            chunks.append(Chunk(
                level="passage",
                book_id=parsed.book_id,
                chapter_idx=chapter.index,
                passage_idx=i,
                text=passage,
                metadata=parsed.metadata,
            ))
    
    return chunks
```

**Metadata Extraction Schema:**

```python
# /Users/shauryapunj/Desktop/009/services/ingestion/metadata.py

@dataclass
class BookMetadata:
    book_id: str                    # UUID
    title: str
    authors: list[str]
    publication_year: int | None
    language: str                   # ISO 639-1
    domains: list[str]              # ["philosophy", "ethics", "western"]
    source: str                     # "gutenberg", "arxiv", "pubmed"
    source_id: str                  # e.g., Gutenberg ID "1342"
    license: str                    # "public_domain", "cc_by", etc.
    word_count: int
    chapter_count: int
    era: str                        # "ancient", "medieval", "early_modern", "modern", "contemporary"
    tradition: str | None           # "stoic", "existentialist", "keynesian", etc.
    geographic_origin: str | None   # "greece", "china", "usa", etc.
    content_type: str               # "treatise", "dialogue", "novel", "paper", "legal_opinion"
    quality_score: float            # 0.0-1.0, computed from source reputation + text quality
```

**Quality Filtering:**

```python
# /Users/shauryapunj/Desktop/009/services/ingestion/quality.py

def compute_quality_score(parsed: ParsedDocument, metadata: BookMetadata) -> float:
    score = 0.0
    
    # Source reputation (0-0.3)
    source_scores = {
        "gutenberg": 0.25,    # Curated public domain
        "arxiv": 0.20,        # Preprints, variable quality
        "pubmed": 0.30,       # Peer-reviewed
        "court_listener": 0.28,
        "internet_archive": 0.15,
    }
    score += source_scores.get(metadata.source, 0.10)
    
    # Text quality heuristics (0-0.3)
    avg_sentence_length = compute_avg_sentence_length(parsed.full_text)
    if 10 < avg_sentence_length < 35:
        score += 0.15
    vocabulary_richness = compute_type_token_ratio(parsed.full_text)
    if vocabulary_richness > 0.4:
        score += 0.15
    
    # Historical significance (0-0.4)
    # Cross-reference against curated "canonical works" list
    if metadata.book_id in CANONICAL_WORKS:
        score += 0.40
    elif metadata.publication_year and metadata.publication_year < 1900:
        score += 0.20  # Survived the test of time
    
    return min(score, 1.0)
```

### 2.3 Embedding Architecture

**Model**: Gemini Embedding (text-embedding-004 or gemini-embedding-001, 768 dimensions for cost/performance balance, upgradable to 3072 for production)

**Embedding Schema (per chunk):**

```json
{
    "id": "chunk_uuid_here",
    "embedding": [0.0123, -0.0456, ...],   // 768-dim float32
    "restricts": [
        {"namespace": "domain", "allow_list": ["philosophy", "ethics"]},
        {"namespace": "era", "allow_list": ["ancient"]},
        {"namespace": "level", "allow_list": ["passage"]},
        {"namespace": "language", "allow_list": ["en"]}
    ],
    "numeric_restricts": [
        {"namespace": "quality_score", "value_float": 0.85},
        {"namespace": "pub_year", "value_int": -380}
    ],
    "crowding_tag": "book_plato_republic"  // Ensures diversity across books
}
```

**Batch Embedding Pipeline:**

```python
# /Users/shauryapunj/Desktop/009/services/embedding/batch_embed.py

from google.cloud import aiplatform
from vertexai.language_models import TextEmbeddingModel

async def batch_embed_chunks(chunks: list[Chunk], batch_size: int = 250) -> list[EmbeddedChunk]:
    """
    Embeds chunks in batches of 250 (API limit).
    Uses text-embedding-004 with task_type for better retrieval quality.
    
    Cost estimate for 100K books:
    - ~100M passage chunks * 512 tokens avg = 51.2B tokens
    - At $0.10/1M tokens (batch) = ~$5,120 total embedding cost
    """
    model = TextEmbeddingModel.from_pretrained("text-embedding-004")
    results = []
    
    for i in range(0, len(chunks), batch_size):
        batch = chunks[i:i + batch_size]
        texts = [chunk.text for chunk in batch]
        
        embeddings = model.get_embeddings(
            texts=texts,
            task_type="RETRIEVAL_DOCUMENT",
            output_dimensionality=768,
        )
        
        for chunk, emb in zip(batch, embeddings):
            results.append(EmbeddedChunk(
                chunk=chunk,
                embedding=emb.values,
            ))
    
    return results
```

**Vertex AI Vector Search Index Configuration:**

```python
# /Users/shauryapunj/Desktop/009/services/embedding/index_manager.py

def create_vector_index():
    """
    Creates the Vertex AI Vector Search index.
    Uses ScaNN algorithm for billion-scale approximate nearest neighbor.
    """
    index = aiplatform.MatchingEngineIndex.create_tree_ah_index(
        display_name="oracle-corpus-index",
        description="ORACLE 100K book corpus embeddings",
        dimensions=768,
        approximate_neighbors_count=200,  # Return top 200 candidates
        distance_measure_type="COSINE_DISTANCE",
        leaf_node_embedding_count=1000,
        leaf_nodes_to_search_percent=10,  # Search 10% of leaves
        shard_size="SHARD_SIZE_MEDIUM",   # For 10M-100M embeddings
        feature_norm_type="UNIT_L2_NORM",
    )
    
    # Deploy to endpoint for online queries
    index_endpoint = aiplatform.MatchingEngineIndexEndpoint.create(
        display_name="oracle-endpoint",
        public_endpoint_enabled=True,
    )
    
    index_endpoint.deploy_index(
        index=index,
        deployed_index_id="oracle_deployed",
        min_replica_count=1,
        max_replica_count=4,  # Auto-scale based on QPS
        machine_type="e2-standard-16",
    )
```

### 2.4 Retrieval System

**Multi-Query Reformulation:**

```python
# /Users/shauryapunj/Desktop/009/services/retrieval/query_reformulator.py

REFORMULATION_PROMPT = """You are a search query optimizer for a library of 100,000 books 
spanning philosophy, science, strategy, law, economics, literature, and religion.

Given the user's question, generate exactly 5 search queries that will retrieve 
diverse, relevant passages. Each query should target a DIFFERENT angle:

1. DIRECT: Restate the core question in precise academic language
2. HISTORICAL: Frame as a historical question about how thinkers have addressed this
3. CONTRARIAN: Search for the strongest opposing viewpoint
4. ADJACENT: Search for a related concept from a different domain
5. SPECIFIC: Search for a specific thinker/work most likely to address this

User question: {user_query}

Return as JSON array of 5 strings. No explanation."""

async def reformulate_query(user_query: str) -> list[str]:
    response = await gemini_flash.generate_content_async(
        REFORMULATION_PROMPT.format(user_query=user_query),
        generation_config={"response_mime_type": "application/json"},
    )
    queries = json.loads(response.text)
    return [user_query] + queries  # Include original + 5 reformulations = 6 total
```

**Hybrid Retrieval (Semantic + Keyword + Metadata):**

```python
# /Users/shauryapunj/Desktop/009/services/retrieval/hybrid_retriever.py

async def hybrid_retrieve(
    queries: list[str],
    domain_filter: list[str] | None = None,
    era_filter: list[str] | None = None,
    target_passages: int = 200,
) -> list[ScoredPassage]:
    """
    Three-signal retrieval:
    1. Vector search (semantic similarity via Vertex AI)
    2. Keyword search (BM25 via BigQuery full-text search)
    3. Metadata boost (domain relevance, quality score, canonical status)
    
    Returns top 200 passages with fused scores.
    """
    
    # Signal 1: Vector search across all reformulated queries
    vector_results = []
    query_embeddings = await batch_embed_queries(queries, task_type="RETRIEVAL_QUERY")
    
    for qemb in query_embeddings:
        restricts = []
        if domain_filter:
            restricts.append(Namespace(name="domain", allow_tokens=domain_filter))
        if era_filter:
            restricts.append(Namespace(name="era", allow_tokens=era_filter))
        restricts.append(Namespace(name="level", allow_tokens=["passage"]))
        
        neighbors = await index_endpoint.find_neighbors_async(
            deployed_index_id="oracle_deployed",
            queries=[qemb],
            num_neighbors=100,
            per_crowding_attribute_neighbor_count=3,  # Max 3 passages per book
            restricts=restricts,
        )
        vector_results.extend(neighbors[0])
    
    # Signal 2: BM25 keyword search via BigQuery
    keyword_results = await bigquery_bm25_search(
        queries=queries,
        limit=200,
        domain_filter=domain_filter,
    )
    
    # Signal 3: Metadata boosting
    # Canonical works get a 1.5x boost, high-quality works get quality_score boost
    
    # Reciprocal Rank Fusion (RRF) to combine signals
    fused = reciprocal_rank_fusion(
        rankings=[vector_results, keyword_results],
        weights=[0.6, 0.3],  # Vector gets more weight
        k=60,  # RRF constant
    )
    
    # Apply metadata boost
    for passage in fused:
        metadata_boost = passage.metadata.quality_score * 0.1
        if passage.metadata.book_id in CANONICAL_WORKS:
            metadata_boost += 0.05
        passage.fused_score += metadata_boost
    
    # Diversity enforcement: max 5 passages per author, spread across domains
    diversified = enforce_diversity(
        passages=fused,
        max_per_author=5,
        max_per_book=3,
        min_domains=3,
        target=target_passages,
    )
    
    return diversified[:target_passages]
```

### 2.5 Pantheon Synthesis Engine

This is the core differentiator. The synthesis engine takes the 200 retrieved passages, selects relevant "minds" (author personas), and produces a structured multi-perspective response.

```python
# /Users/shauryapunj/Desktop/009/services/synthesis/pantheon_engine.py

SYNTHESIS_SYSTEM_PROMPT = """You are ORACLE, a synthesis engine that channels the 
perspectives of history's greatest thinkers. You have been given a user's question 
and a set of source passages from various books and papers.

YOUR TASK:
1. Identify 3-6 distinct perspectives on the question from the provided sources
2. For each perspective, ground it in SPECIFIC passages (cite by [Book Title, Author, Chapter/Page])
3. Map contradictions between perspectives explicitly
4. Synthesize a meta-insight that emerges from the tension between perspectives
5. NEVER attribute a claim to an author unless it is directly supported by a provided passage
6. If a thinker's view on a modern topic must be INFERRED, explicitly mark it as [INFERENCE] 
   and explain the reasoning chain from their documented positions

RESPONSE FORMAT:
{
  "perspectives": [
    {
      "thinker": "Author Name",
      "era": "Date range",
      "tradition": "School of thought",
      "position": "Their position on this question in 2-3 sentences",
      "key_passages": [
        {
          "text": "Direct quote from source",
          "source": "Book Title, Chapter X",
          "confidence": 0.95  // How directly this passage supports the attributed position
        }
      ],
      "reasoning": "How this thinker arrives at their position based on their broader framework",
      "limitations": "What this perspective misses or assumes"
    }
  ],
  "contradictions": [
    {
      "between": ["Thinker A", "Thinker B"],
      "nature": "Description of the core disagreement",
      "resolution_attempts": "How others have tried to resolve this tension"
    }
  ],
  "synthesis": {
    "meta_insight": "What emerges from considering all perspectives together",
    "practical_implications": "What this means for the user's actual decision/question",
    "further_reading": ["Book recommendations for deeper exploration"]
  },
  "confidence_assessment": {
    "retrieval_coverage": 0.85,  // How well the sources cover the question
    "attribution_confidence": 0.90,  // How confident we are in the attributions
    "synthesis_novelty": 0.70  // How much the synthesis goes beyond individual sources
  }
}"""

async def synthesize(
    query: str,
    passages: list[ScoredPassage],
    selected_personas: list[str] | None = None,
    mode: str = "consultation",  # "consultation" | "debate" | "deep_dive"
) -> OracleResponse:
    
    # Construct the context block from retrieved passages
    context_block = format_passages_for_context(passages)  # ~150K-500K tokens
    
    # Select personas if not specified by user
    if not selected_personas:
        persona_prompt = PERSONA_SELECTION_PROMPT.format(
            query=query,
            available_authors=extract_unique_authors(passages),
        )
        persona_response = await gemini_flash.generate_content_async(persona_prompt)
        selected_personas = json.loads(persona_response.text)
    
    # Build the full prompt
    if mode == "debate":
        system = DEBATE_SYSTEM_PROMPT
    elif mode == "deep_dive":
        system = DEEP_DIVE_SYSTEM_PROMPT
    else:
        system = SYNTHESIS_SYSTEM_PROMPT
    
    full_prompt = f"""SOURCE PASSAGES:
{context_block}

SELECTED PERSPECTIVES TO REPRESENT: {json.dumps(selected_personas)}

USER QUESTION: {query}

Respond in the JSON format specified in your instructions."""
    
    # Use Gemini 1.5 Pro with 2M context
    response = await gemini_pro.generate_content_async(
        contents=[full_prompt],
        generation_config={
            "response_mime_type": "application/json",
            "temperature": 0.3,  # Low for factual grounding
            "max_output_tokens": 8192,
        },
        system_instruction=system,
    )
    
    return OracleResponse.from_json(response.text)
```

### 2.6 NotebookLM Integration (Per-Session Working Memory)

```python
# /Users/shauryapunj/Desktop/009/services/notebooklm/session_manager.py

from google.cloud import notebooklm_v1alpha  # Enterprise API

class NotebookLMSessionManager:
    """
    Creates a NotebookLM notebook per user session.
    Uploads the retrieved passages as sources so users can 
    deep-dive with NotebookLM's native capabilities (audio overview, 
    source-grounded Q&A, study guides).
    """
    
    async def create_session_notebook(
        self,
        user_id: str,
        query: str,
        passages: list[ScoredPassage],
        oracle_response: OracleResponse,
    ) -> str:
        """Returns a URL to the NotebookLM notebook for deep-dive."""
        
        # Create notebook
        client = notebooklm_v1alpha.NotebookServiceClient()
        notebook = await client.create_notebook(
            parent=f"projects/{PROJECT_ID}/locations/us",
            notebook=notebooklm_v1alpha.Notebook(
                display_name=f"ORACLE Session: {query[:50]}",
            ),
        )
        
        # Upload top passages as individual sources (max 50 sources per notebook)
        top_passages = passages[:50]
        for passage in top_passages:
            source_text = f"# {passage.metadata.title} by {passage.metadata.authors[0]}\n"
            source_text += f"## {passage.chapter_title}\n\n"
            source_text += passage.text
            
            await client.create_source(
                parent=notebook.name,
                source=notebooklm_v1alpha.Source(
                    display_name=f"{passage.metadata.title} - {passage.chapter_title}",
                    inline_content=notebooklm_v1alpha.InlineContent(
                        content=source_text,
                        mime_type="text/plain",
                    ),
                ),
            )
        
        # Upload ORACLE's synthesis as an additional source
        await client.create_source(
            parent=notebook.name,
            source=notebooklm_v1alpha.Source(
                display_name="ORACLE Synthesis",
                inline_content=notebooklm_v1alpha.InlineContent(
                    content=oracle_response.to_markdown(),
                    mime_type="text/plain",
                ),
            ),
        )
        
        return f"https://notebooklm.google.com/notebook/{notebook.uid}"
```

### 2.7 Anti-Hallucination System

Every claim in ORACLE's response must be traceable. The system enforces this through:

1. **Citation Verification**: A post-generation pass checks that every quoted passage actually appears in the retrieved context. Any "quote" not found verbatim (with fuzzy matching at 90% threshold) is flagged.

2. **Attribution Confidence Scoring**: Each attribution gets a score from 0.0 to 1.0:
   - 0.9-1.0: Direct quote from the author's work
   - 0.7-0.89: Close paraphrase of a documented position
   - 0.5-0.69: Reasonable inference from documented positions (marked [INFERENCE])
   - Below 0.5: Rejected, not included in response

3. **Anachronism Detection**: If an author is attributed a position on a topic that post-dates their death, the system flags it and reframes as inference.

```python
# /Users/shauryapunj/Desktop/009/services/synthesis/verification.py

async def verify_response(
    response: OracleResponse,
    passages: list[ScoredPassage],
) -> VerifiedResponse:
    issues = []
    
    for perspective in response.perspectives:
        for citation in perspective.key_passages:
            # Check 1: Does the quoted text exist in our retrieved passages?
            match = find_best_match(citation.text, passages)
            if match.similarity < 0.90:
                issues.append(VerificationIssue(
                    type="unverified_quote",
                    thinker=perspective.thinker,
                    quote=citation.text,
                    best_match_similarity=match.similarity,
                ))
                citation.confidence = min(citation.confidence, 0.5)
            
            # Check 2: Anachronism detection
            author_death_year = get_author_death_year(perspective.thinker)
            topic_earliest_year = estimate_topic_origin_year(response.query)
            if author_death_year and topic_earliest_year:
                if author_death_year < topic_earliest_year:
                    citation.is_inference = True
                    issues.append(VerificationIssue(
                        type="anachronism_inference",
                        thinker=perspective.thinker,
                        detail=f"{perspective.thinker} (d. {author_death_year}) "
                               f"on topic originating ~{topic_earliest_year}",
                    ))
    
    return VerifiedResponse(response=response, issues=issues)
```

---

## SECTION 3: API INTEGRATION SPECIFICATIONS (Google Reference Implementation — Provider-Swappable)

### 3.1 Gemini 1.5 Pro (2M Context)

```python
# /Users/shauryapunj/Desktop/009/config/gemini_config.py

import vertexai
from vertexai.generative_models import GenerativeModel, GenerationConfig

vertexai.init(project="oracle-prod", location="us-central1")

# Primary synthesis model - 2M context
gemini_pro_2m = GenerativeModel(
    model_name="gemini-1.5-pro-002",
    generation_config=GenerationConfig(
        max_output_tokens=8192,
        temperature=0.3,
        top_p=0.85,
        top_k=40,
        response_mime_type="application/json",
    ),
)

# Fast model for query reformulation, persona selection, summaries
gemini_flash = GenerativeModel(
    model_name="gemini-2.0-flash",
    generation_config=GenerationConfig(
        max_output_tokens=2048,
        temperature=0.2,
    ),
)

# Context caching for frequently-used corpus segments
from vertexai.generative_models import caching

CANONICAL_CACHE = caching.CachedContent.create(
    model_name="gemini-1.5-pro-002",
    display_name="oracle-canonical-works",
    contents=[load_canonical_corpus()],  # Top 100 most-referenced works
    ttl=datetime.timedelta(hours=24),
)
```

**Cost Estimates (per query):**
- Input: ~500K tokens average context = ~$0.63 (standard) or ~$0.06 (cached)
- Output: ~4K tokens = ~$0.02
- With 60% cache hit rate: ~$0.27 per query average

### 3.2 NotebookLM Enterprise API

```python
# /Users/shauryapunj/Desktop/009/config/notebooklm_config.py

# NotebookLM Enterprise API (alpha, September 2025+)
# Requires Gemini Enterprise license or Workspace add-on
NOTEBOOKLM_CONFIG = {
    "project": "oracle-prod",
    "location": "us",
    "max_sources_per_notebook": 50,
    "max_source_size_bytes": 500_000,  # ~500KB per source
    "supported_source_types": ["text/plain", "application/pdf", "text/html"],
    "api_endpoint": "notebooklm.googleapis.com",
}
```

### 3.3 Vertex AI Vector Search

```python
# /Users/shauryapunj/Desktop/009/config/vector_search_config.py

VECTOR_SEARCH_CONFIG = {
    "index_display_name": "oracle-corpus-v1",
    "dimensions": 768,
    "algorithm": "TREE_AH",  # ScaNN-based for billion-scale
    "distance_measure": "COSINE_DISTANCE",
    "shard_size": "SHARD_SIZE_MEDIUM",  # 10M-100M vectors
    "approximate_neighbors": 200,
    "leaf_node_embedding_count": 1000,
    "leaf_nodes_to_search_percent": 10,
    
    # Endpoint config
    "min_replicas": 1,
    "max_replicas": 4,
    "machine_type": "e2-standard-16",
    
    # Filtering namespaces
    "restricts": ["domain", "era", "level", "language", "source"],
    "numeric_restricts": ["quality_score", "pub_year"],
    "crowding_attribute": "book_id",  # Diversity per book
}
```

**Cost Estimates:**
- Index build: ~$3.00/GiB. For 100M embeddings at 768 dims (float32) = ~300 GiB = ~$900 one-time
- Serving: $0.38/node-hour. 1 node for low traffic = ~$274/month
- Storage: $0.02/GB-month for standard = ~$6/month

### 3.4 Gemini Embeddings

```python
# /Users/shauryapunj/Desktop/009/config/embedding_config.py

EMBEDDING_CONFIG = {
    "model": "text-embedding-004",  # or "gemini-embedding-001"
    "dimensions": 768,              # MRL allows 768/1536/3072
    "task_types": {
        "document": "RETRIEVAL_DOCUMENT",
        "query": "RETRIEVAL_QUERY",
        "summary": "SEMANTIC_SIMILARITY",
    },
    "batch_size": 250,              # Max per API call
    "rate_limit_rpm": 1500,         # Requests per minute
    "cost_per_million_tokens": 0.10,  # Batch pricing
}
```

### 3.5 Document AI

```python
# /Users/shauryapunj/Desktop/009/config/document_ai_config.py

DOCUMENT_AI_CONFIG = {
    "processors": {
        "ocr": {
            "type": "OCR_PROCESSOR",
            "display_name": "oracle-ocr",
            "version": "pretrained-ocr-v2.1-2024-08-07",
        },
        "layout": {
            "type": "LAYOUT_PARSER_PROCESSOR",
            "display_name": "oracle-layout",
            "version": "pretrained-layout-parser-v1.0-2024-08-07",
        },
    },
    "processing_config": {
        "ocr_config": {
            "enable_native_pdf_parsing": True,
            "language_hints": ["en", "la", "el", "de", "fr"],
        },
    },
    # Pricing: $1.50 per 1000 pages (OCR), $10 per 1000 pages (Layout)
    # For 100K books * avg 300 pages = 30M pages
    # OCR: $45,000 (one-time), Layout: $300,000 (one-time)
    # OPTIMIZATION: Only use Layout for complex PDFs. Use direct text extraction for Gutenberg TXT.
}
```

### 3.6 Cloud Storage

```python
# /Users/shauryapunj/Desktop/009/config/storage_config.py

STORAGE_CONFIG = {
    "buckets": {
        "raw_corpus": {
            "name": "oracle-raw-corpus",
            "location": "US",
            "storage_class": "STANDARD",
            "lifecycle": {"age_days": 90, "action": "SET_STORAGE_CLASS", "target": "NEARLINE"},
        },
        "processed_chunks": {
            "name": "oracle-processed-chunks",
            "location": "US",
            "storage_class": "STANDARD",
        },
        "embeddings_staging": {
            "name": "oracle-embeddings-staging",
            "location": "US",
            "storage_class": "STANDARD",
        },
    },
    # Estimated storage:
    # Raw corpus: ~500GB (100K books, avg 5MB each)
    # Processed chunks: ~200GB (JSON with metadata)
    # Embeddings staging: ~300GB (before index build)
    # Monthly cost: ~$20-30
}
```

### 3.7 BigQuery

```python
# /Users/shauryapunj/Desktop/009/config/bigquery_config.py

BIGQUERY_SCHEMA = {
    "dataset": "oracle_corpus",
    "tables": {
        "books": {
            "schema": [
                {"name": "book_id", "type": "STRING", "mode": "REQUIRED"},
                {"name": "title", "type": "STRING"},
                {"name": "authors", "type": "STRING", "mode": "REPEATED"},
                {"name": "publication_year", "type": "INTEGER"},
                {"name": "domains", "type": "STRING", "mode": "REPEATED"},
                {"name": "source", "type": "STRING"},
                {"name": "source_id", "type": "STRING"},
                {"name": "license", "type": "STRING"},
                {"name": "word_count", "type": "INTEGER"},
                {"name": "quality_score", "type": "FLOAT"},
                {"name": "era", "type": "STRING"},
                {"name": "tradition", "type": "STRING"},
                {"name": "geographic_origin", "type": "STRING"},
                {"name": "content_type", "type": "STRING"},
                {"name": "created_at", "type": "TIMESTAMP"},
            ],
        },
        "chunks": {
            "schema": [
                {"name": "chunk_id", "type": "STRING", "mode": "REQUIRED"},
                {"name": "book_id", "type": "STRING", "mode": "REQUIRED"},
                {"name": "level", "type": "STRING"},  # book, chapter, passage
                {"name": "chapter_idx", "type": "INTEGER"},
                {"name": "passage_idx", "type": "INTEGER"},
                {"name": "text", "type": "STRING"},
                {"name": "token_count", "type": "INTEGER"},
                {"name": "chapter_title", "type": "STRING"},
            ],
            # Enable BigQuery full-text search for BM25 keyword retrieval
            "search_index": {
                "name": "chunks_fts_idx",
                "columns": ["text"],
                "analyzer": "LOG_ANALYZER",
            },
        },
        "queries": {
            "schema": [
                {"name": "query_id", "type": "STRING"},
                {"name": "user_id", "type": "STRING"},
                {"name": "query_text", "type": "STRING"},
                {"name": "retrieved_chunk_ids", "type": "STRING", "mode": "REPEATED"},
                {"name": "response_json", "type": "JSON"},
                {"name": "latency_ms", "type": "INTEGER"},
                {"name": "created_at", "type": "TIMESTAMP"},
            ],
        },
    },
}
```

### 3.8 Cloud Run

```python
# /Users/shauryapunj/Desktop/009/config/cloudrun_config.py

CLOUD_RUN_SERVICES = {
    "api": {
        "name": "oracle-api",
        "image": "gcr.io/oracle-prod/oracle-api:latest",
        "cpu": "4",
        "memory": "8Gi",
        "max_instances": 10,
        "min_instances": 1,
        "timeout_seconds": 300,  # 5 min for full synthesis
        "concurrency": 20,
        "env_vars": {
            "PROJECT_ID": "oracle-prod",
            "VECTOR_SEARCH_ENDPOINT": "projects/oracle-prod/locations/us-central1/indexEndpoints/xxx",
            "BIGQUERY_DATASET": "oracle_corpus",
        },
    },
    "ingestion": {
        "name": "oracle-ingestion",
        "image": "gcr.io/oracle-prod/oracle-ingestion:latest",
        "cpu": "8",
        "memory": "16Gi",
        "max_instances": 50,  # High parallelism for batch ingestion
        "timeout_seconds": 900,
        "concurrency": 1,  # One document per instance
    },
}
```

### 3.9 Firebase

```python
# /Users/shauryapunj/Desktop/009/config/firebase_config.py

FIREBASE_CONFIG = {
    "auth": {
        "providers": ["google.com", "password"],
        "session_cookie_duration": 14 * 24 * 60 * 60,  # 14 days
    },
    "firestore": {
        "collections": {
            "users": {
                "fields": ["uid", "email", "display_name", "tier", "query_count", "created_at"],
            },
            "sessions": {
                "fields": ["session_id", "user_id", "query", "response", "notebook_url", 
                          "passages_used", "created_at"],
            },
            "panels": {
                "fields": ["panel_id", "user_id", "name", "thinkers", "description"],
            },
        },
    },
    "hosting": {
        "framework": "nextjs",
        "rewrites": [
            {"source": "/api/**", "run": {"serviceId": "oracle-api", "region": "us-central1"}},
            {"source": "**", "destination": "/index.html"},
        ],
    },
}
```

---

## SECTION 4: KNOWLEDGE CORPUS DESIGN

### 4.1 Selection Criteria

The 100,000-book corpus is curated for:

1. **Domain coverage**: Every major intellectual tradition must be represented
2. **Temporal depth**: From ancient texts (~3000 BCE) to contemporary works
3. **Ideological diversity**: Conflicting viewpoints are features, not bugs
4. **Quality threshold**: Minimum quality score of 0.4 (see quality scoring above)
5. **Language**: English-first for hackathon, with metadata for 20+ languages for production expansion

### 4.2 Domain Categories and Target Counts

| Domain | Subcategories | Target Books | Primary Sources |
|--------|--------------|-------------|-----------------|
| Philosophy | Western (Plato-present), Eastern (Confucius, Lao Tzu, Buddhist canon), African (Fanon, Mbiti), Indigenous, Islamic (Ibn Rushd, Al-Ghazali) | 12,000 | Gutenberg, Internet Archive, PhilPapers |
| Science & Technology | Physics, Biology, Chemistry, Computer Science, Mathematics | 15,000 | arXiv, Gutenberg, OpenStax |
| Strategy & Military | Sun Tzu through modern strategic theory | 3,000 | Gutenberg, Internet Archive |
| Economics | Classical, Keynesian, Austrian, Marxist, Behavioral, Development | 8,000 | Gutenberg, Internet Archive |
| Law | Constitutional law, international law, jurisprudence, landmark opinions | 10,000 | CourtListener, Gutenberg |
| Literature | Novels, poetry, drama across cultures and periods | 20,000 | Gutenberg, Internet Archive |
| Religion & Theology | Christianity, Islam, Judaism, Hinduism, Buddhism, Indigenous | 5,000 | Gutenberg, sacred-texts.com |
| History | World history, regional histories, historiography | 10,000 | Gutenberg, Internet Archive |
| Psychology & Sociology | Freud through modern behavioral science | 7,000 | arXiv, PubMed, Gutenberg |
| Medicine & Health | Medical literature, bioethics, public health | 5,000 | PubMed Central |
| Political Science | Political theory, governance, international relations | 5,000 | Gutenberg, Internet Archive |

### 4.3 Public Domain Strategy (Hackathon Core)

For the hackathon, the corpus is built entirely from public domain and open-access sources:

1. **Project Gutenberg**: 70,000+ books. Bulk download via mirror. Focus on:
   - All philosophy (Plato, Aristotle, Kant, Hegel, Mill, Nietzsche, etc.)
   - All classic literature (Shakespeare, Dostoevsky, Austen, Dickens, etc.)
   - All science classics (Darwin, Newton, Einstein popular works, etc.)
   - All strategy (Sun Tzu, Clausewitz, Machiavelli)
   - All economics (Smith, Marx, Ricardo, Malthus)
   - All religious texts (Bible, Quran, Bhagavad Gita, Buddhist sutras)

2. **arXiv Open Access**: Recent papers in physics, CS, math, economics. Use arXiv API with `metadata_format=arXiv` for structured access.

3. **PubMed Central Open Access Subset**: ~4M articles with full text available under open licenses.

### 4.4 Licensed Content Strategy (Production)

For production beyond public domain:
- **Publisher partnerships**: University press partnerships for academic titles
- **Creative Commons**: CC-licensed textbooks (OpenStax, MIT OCW)
- **Author opt-in program**: Living authors can opt-in their works for inclusion
- **Fair use analysis**: Scholarly citation of excerpts (consult legal counsel)

### 4.5 Temporal Coverage

| Era | Period | Target Coverage |
|-----|--------|----------------|
| Ancient | Pre-500 CE | 3,000 works |
| Medieval | 500-1500 CE | 4,000 works |
| Early Modern | 1500-1800 | 15,000 works |
| Modern | 1800-1950 | 35,000 works |
| Contemporary | 1950-present | 43,000 works |

### 4.6 Update Strategy

- **Quarterly**: New arXiv/PubMed papers in tracked categories
- **Annually**: New public domain works (copyright expiration)
- **On-demand**: User-contributed sources (verified, quality-checked)

---

## SECTION 5: PERSONA SYSTEM

### 5.1 Author Persona Construction

Personas are NOT impersonations. They are perspective-grounding frameworks that help the synthesis engine represent an author's documented worldview faithfully.

```python
# /Users/shauryapunj/Desktop/009/services/personas/persona_schema.py

@dataclass
class AuthorPersona:
    author_id: str
    name: str
    birth_year: int | None
    death_year: int | None
    primary_domain: str
    traditions: list[str]           # ["stoicism", "virtue_ethics"]
    key_works: list[str]            # Book IDs in corpus
    core_positions: list[CorePosition]
    reasoning_style: str            # "dialectical", "empirical", "aphoristic", etc.
    known_influences: list[str]     # Other author_ids
    known_opponents: list[str]      # Authors they explicitly argued against
    epistemology: str               # How they believe knowledge is obtained
    temporal_context: str           # Brief description of their historical moment
    
@dataclass
class CorePosition:
    topic: str                      # "justice", "free_will", "markets", etc.
    position_summary: str           # 2-3 sentence summary
    supporting_works: list[str]     # Book IDs
    confidence: float               # How clearly documented this position is
    evolution: str | None           # Did their view change over time?
```

### 5.2 Persona Knowledge Extraction Pipeline

```python
# /Users/shauryapunj/Desktop/009/services/personas/extraction.py

PERSONA_EXTRACTION_PROMPT = """Analyze the following complete works by {author_name} and extract 
a structured persona profile. This is NOT about impersonating the author -- it is about 
accurately documenting their intellectual positions and reasoning style.

WORKS PROVIDED:
{works_text}

Extract:
1. CORE POSITIONS: For each major topic they address, what is their documented position?
   Only include positions directly supported by the text. Rate confidence 0.0-1.0.
2. REASONING STYLE: How do they construct arguments? (dialectical, empirical, narrative, etc.)
3. EPISTEMOLOGY: What do they consider valid sources of knowledge?
4. KEY CONCEPTS: What are their signature ideas/terms?
5. KNOWN DISAGREEMENTS: Who do they explicitly argue against in these texts?
6. BLIND SPOTS: What topics relevant to their domain do they NOT address?

Return as structured JSON matching the AuthorPersona schema."""

async def extract_persona(author_id: str, works: list[Book]) -> AuthorPersona:
    """
    Uses Gemini 1.5 Pro 2M context to analyze an author's complete works
    and extract a structured persona. For authors with <2M tokens of works,
    we can process everything in a single pass.
    """
    works_text = "\n\n---\n\n".join([
        f"# {work.title}\n{work.full_text}" for work in works
    ])
    
    # Check if within context window
    token_count = estimate_tokens(works_text)
    
    if token_count < 1_800_000:  # Leave room for prompt + output
        response = await gemini_pro_2m.generate_content_async(
            PERSONA_EXTRACTION_PROMPT.format(
                author_name=works[0].metadata.authors[0],
                works_text=works_text,
            ),
            generation_config={"response_mime_type": "application/json"},
        )
        return AuthorPersona.from_json(response.text)
    else:
        # Multi-pass extraction for prolific authors
        return await extract_persona_multipass(author_id, works)
```

### 5.3 Anachronism Guard

```python
# /Users/shauryapunj/Desktop/009/services/personas/anachronism_guard.py

ANACHRONISM_GUARD_PROMPT = """You are checking for anachronistic attributions in an ORACLE response.

The response attributes the following position to {author_name} (lived {birth_year}-{death_year}):
"{attributed_position}"

The user's question involves: "{topic}"

EVALUATE:
1. Could {author_name} have had a position on this specific topic during their lifetime?
2. If the topic post-dates their death, is the attribution clearly marked as an inference?
3. Does the attributed position use concepts or terminology that did not exist in their era?

Return:
{{
  "is_anachronistic": true/false,
  "severity": "none" | "mild" | "severe",
  "explanation": "...",
  "suggested_reframing": "How to express this as a valid inference if applicable"
}}"""
```

### 5.4 Debate Engine

```python
# /Users/shauryapunj/Desktop/009/services/synthesis/debate_engine.py

DEBATE_SYSTEM_PROMPT = """You are orchestrating a structured intellectual debate between 
historical thinkers. Each thinker's position MUST be grounded in their actual documented 
works provided in the source passages.

DEBATE STRUCTURE:
1. Opening statements (each thinker's position, with citations)
2. Direct responses to each other (where documented disagreements exist)
3. Points of unexpected agreement
4. Unresolved tensions
5. What each thinker would need to concede to the other

RULES:
- Every claim must cite a specific source passage
- Mark inferences explicitly with [INFERENCE]
- Do not put modern concepts in pre-modern thinkers' mouths
- Surface genuine intellectual tensions, do not manufacture false disagreements
- Each thinker gets equal representation

THINKERS IN THIS DEBATE: {thinkers}
TOPIC: {topic}

SOURCE PASSAGES:
{passages}"""

async def run_debate(
    topic: str,
    thinkers: list[str],  # e.g., ["Plato", "Nietzsche"]
    passages: list[ScoredPassage],
) -> DebateResponse:
    """
    Orchestrates a multi-round debate between 2-4 thinkers.
    """
    context = format_passages_for_context(passages)
    
    response = await gemini_pro_2m.generate_content_async(
        DEBATE_SYSTEM_PROMPT.format(
            thinkers=json.dumps(thinkers),
            topic=topic,
            passages=context,
        ),
        generation_config={
            "response_mime_type": "application/json",
            "temperature": 0.4,  # Slightly higher for creative debate
            "max_output_tokens": 8192,
        },
    )
    
    return DebateResponse.from_json(response.text)
```

### 5.5 User-Defined Panels

Users can create custom panels of thinkers for recurring consultation:

```python
# /Users/shauryapunj/Desktop/009/services/personas/panels.py

PRESET_PANELS = {
    "strategic_council": {
        "name": "Strategic Council",
        "description": "For leadership and strategic decisions",
        "thinkers": ["Sun Tzu", "Machiavelli", "Clausewitz", "Peter Drucker", "Miyamoto Musashi"],
    },
    "philosophical_roundtable": {
        "name": "Philosophical Roundtable",
        "description": "For deep philosophical questions",
        "thinkers": ["Plato", "Aristotle", "Kant", "Nietzsche", "Simone de Beauvoir", "Confucius"],
    },
    "scientific_minds": {
        "name": "Scientific Minds",
        "description": "For scientific and empirical questions",
        "thinkers": ["Darwin", "Einstein", "Curie", "Feynman", "Rosalind Franklin"],
    },
    "economic_thinkers": {
        "name": "Economic Thinkers",
        "description": "For economic and market questions",
        "thinkers": ["Adam Smith", "Karl Marx", "Keynes", "Hayek", "Amartya Sen"],
    },
    "justice_league": {
        "name": "Justice & Ethics",
        "description": "For questions of justice, rights, and ethics",
        "thinkers": ["John Rawls", "Aristotle", "Martin Luther King Jr.", "Simone Weil", "Frantz Fanon"],
    },
}
```

---

## SECTION 6: FRONTEND ARCHITECTURE

### 6.1 Tech Stack

- **Framework**: Next.js 14+ (App Router) with TypeScript
- **Styling**: Tailwind CSS + shadcn/ui components
- **State Management**: React Context + TanStack Query for server state
- **Authentication**: Firebase Auth (Google sign-in + email/password)
- **Deployment**: Firebase App Hosting (auto-deploys to Cloud Run)
- **Real-time**: Firebase Firestore for session persistence

### 6.2 Consultation Interface

The primary interface is a "consultation room" metaphor:

```
+------------------------------------------------------------------+
|  ORACLE                                    [User Menu] [Settings] |
+------------------------------------------------------------------+
|                                                                    |
|  [Panel Selector: Strategic Council v]   [Mode: Consultation v]   |
|                                                                    |
|  +--------------------------------------------------------------+ |
|  |  Ask the Oracle...                                    [Send] | |
|  +--------------------------------------------------------------+ |
|                                                                    |
|  +--------------------------------------------------------------+ |
|  |  ORACLE RESPONSE                                              | |
|  |                                                                | |
|  |  == Perspectives ==                                           | |
|  |                                                                | |
|  |  [Avatar] Sun Tzu (544-496 BCE)                               | |
|  |  "The supreme art of war is to subdue the enemy..."           | |
|  |  Source: The Art of War, Chapter 3 [confidence: 0.95]         | |
|  |  > Position: [2-3 sentence grounded position]                 | |
|  |                                                                | |
|  |  [Avatar] Adam Smith (1723-1790)                              | |
|  |  "It is not from the benevolence of the butcher..."           | |
|  |  Source: Wealth of Nations, Book I, Ch 2 [confidence: 0.98]   | |
|  |  > Position: [2-3 sentence grounded position]                 | |
|  |                                                                | |
|  |  == Contradictions ==                                         | |
|  |  Sun Tzu vs Smith: [description of tension]                   | |
|  |                                                                | |
|  |  == Synthesis ==                                              | |
|  |  [Meta-insight paragraph]                                     | |
|  |                                                                | |
|  |  [Open in NotebookLM] [Export] [Share] [Deep Dive]           | |
|  +--------------------------------------------------------------+ |
|                                                                    |
|  == Follow-Up Questions ==                                        |
|  [Suggested Q1] [Suggested Q2] [Suggested Q3]                    |
|                                                                    |
+------------------------------------------------------------------+
```

### 6.3 Key Frontend Components

```
/Users/shauryapunj/Desktop/009/frontend/src/
  app/
    layout.tsx              -- Root layout with Firebase auth provider
    page.tsx                -- Landing/marketing page
    consultation/
      page.tsx              -- Main consultation interface
      loading.tsx           -- Streaming response skeleton
    library/
      page.tsx              -- User's saved sessions and custom panels
    explore/
      page.tsx              -- Browse corpus by domain/author/era
  components/
    consultation/
      QueryInput.tsx        -- Main query input with suggestions
      PanelSelector.tsx     -- Dropdown to select thinker panels
      ModeSelector.tsx      -- Consultation / Debate / Deep Dive toggle
      OracleResponse.tsx    -- Full response renderer
      PerspectiveCard.tsx   -- Individual thinker perspective block
      ContradictionMap.tsx  -- Visual contradiction display
      SynthesisBlock.tsx    -- Meta-synthesis section
      CitationLink.tsx      -- Clickable citation with source preview
      ConfidenceBadge.tsx   -- Visual confidence indicator (0.0-1.0)
      FollowUpSuggestions.tsx
    debate/
      DebateView.tsx        -- Debate mode renderer (back-and-forth format)
      DebateRound.tsx       -- Single round of debate
    pantheon/
      PantheonVisual.tsx    -- Circular arrangement of thinker avatars
      ThinkerAvatar.tsx     -- Individual thinker with name/dates/domain
      ThinkerModal.tsx      -- Detailed thinker info on click
    library/
      SessionList.tsx       -- List of past consultation sessions
      PanelManager.tsx      -- Create/edit custom panels
      BookmarkList.tsx      -- Saved individual responses
    shared/
      Header.tsx
      Footer.tsx
      LoadingSpinner.tsx
      ErrorBoundary.tsx
      AuthGuard.tsx
  lib/
    firebase.ts             -- Firebase client initialization
    api.ts                  -- API client for Cloud Run backend
    types.ts                -- TypeScript types matching backend schemas
    hooks/
      useOracle.ts          -- Main query hook with streaming
      useAuth.ts            -- Firebase auth hook
      usePanels.ts          -- Panel management hook
      useLibrary.ts         -- Session history hook
```

### 6.4 Response Streaming

```typescript
// /Users/shauryapunj/Desktop/009/frontend/src/lib/hooks/useOracle.ts

export function useOracle() {
  const [response, setResponse] = useState<OracleResponse | null>(null);
  const [isStreaming, setIsStreaming] = useState(false);
  const [streamPhase, setStreamPhase] = useState<
    "reformulating" | "retrieving" | "synthesizing" | "verifying" | "complete"
  >("reformulating");

  const query = async (input: QueryInput) => {
    setIsStreaming(true);
    setStreamPhase("reformulating");
    
    const eventSource = new EventSource(
      `/api/oracle/query?q=${encodeURIComponent(input.query)}&panel=${input.panelId}&mode=${input.mode}`
    );
    
    eventSource.addEventListener("phase", (e) => {
      setStreamPhase(JSON.parse(e.data).phase);
    });
    
    eventSource.addEventListener("perspective", (e) => {
      const perspective = JSON.parse(e.data);
      setResponse(prev => ({
        ...prev,
        perspectives: [...(prev?.perspectives || []), perspective],
      }));
    });
    
    eventSource.addEventListener("synthesis", (e) => {
      setResponse(prev => ({
        ...prev,
        synthesis: JSON.parse(e.data),
      }));
    });
    
    eventSource.addEventListener("complete", (e) => {
      setIsStreaming(false);
      setStreamPhase("complete");
      eventSource.close();
    });
  };

  return { query, response, isStreaming, streamPhase };
}
```

### 6.5 Deep Dive Mode

When a user clicks "Deep Dive," the system:
1. Creates a NotebookLM session with the relevant passages
2. Opens NotebookLM in a side panel or new tab
3. The user can ask follow-up questions within NotebookLM, generate audio overviews, etc.

### 6.6 Export and Sharing

- **PDF Export**: Formatted consultation report with citations
- **Markdown Export**: For note-taking apps
- **Share Link**: Generates a read-only public URL for a consultation
- **Citation Export**: BibTeX/APA formatted references for the sources used

---

## SECTION 7: FILE AND FOLDER STRUCTURE

```
/Users/shauryapunj/Desktop/009/
|
|-- README.md                           # Project overview, setup instructions
|-- LICENSE                             # MIT or Apache 2.0
|-- .gitignore                          # Node, Python, GCP ignores
|-- .env.example                        # Template for environment variables
|-- docker-compose.yml                  # Local development environment
|-- Makefile                            # Common commands (build, deploy, ingest)
|
|-- infrastructure/
|   |-- terraform/
|   |   |-- main.tf                     # GCP project, APIs enablement
|   |   |-- storage.tf                  # Cloud Storage buckets
|   |   |-- bigquery.tf                 # BigQuery dataset and tables
|   |   |-- vector_search.tf            # Vertex AI Vector Search index + endpoint
|   |   |-- cloud_run.tf               # Cloud Run service definitions
|   |   |-- firebase.tf                 # Firebase project config
|   |   |-- iam.tf                      # Service account permissions
|   |   |-- variables.tf               # Configurable variables
|   |   |-- outputs.tf                 # Output endpoints and URLs
|   |-- cloudbuild.yaml                # CI/CD pipeline
|   |-- Dockerfile.api                 # API service container
|   |-- Dockerfile.ingestion           # Ingestion worker container
|   |-- Dockerfile.frontend            # Frontend Next.js container
|
|-- services/
|   |-- api/
|   |   |-- main.py                     # FastAPI application entry point
|   |   |-- requirements.txt            # Python dependencies
|   |   |-- routers/
|   |   |   |-- query.py                # POST /api/oracle/query
|   |   |   |-- stream.py              # GET /api/oracle/stream (SSE)
|   |   |   |-- panels.py              # CRUD for user panels
|   |   |   |-- library.py             # User session history
|   |   |   |-- explore.py             # Corpus browsing endpoints
|   |   |   |-- health.py              # Health check
|   |   |-- middleware/
|   |   |   |-- auth.py                 # Firebase token verification
|   |   |   |-- rate_limit.py           # Per-user rate limiting
|   |   |   |-- cors.py                # CORS configuration
|   |   |-- dependencies.py            # FastAPI dependency injection
|   |
|   |-- ingestion/
|   |   |-- main.py                     # Ingestion pipeline orchestrator
|   |   |-- requirements.txt
|   |   |-- parser.py                   # Document parsing (PDF/EPUB/TXT/HTML)
|   |   |-- chunker.py                 # Hierarchical chunking logic
|   |   |-- metadata.py                # Metadata extraction and schema
|   |   |-- quality.py                 # Quality scoring
|   |   |-- gutenberg_loader.py        # Project Gutenberg bulk loader
|   |   |-- arxiv_loader.py            # arXiv API loader
|   |   |-- pubmed_loader.py           # PubMed Central loader
|   |   |-- summary_generator.py       # Book-level summary generation
|   |
|   |-- embedding/
|   |   |-- main.py                     # Embedding pipeline orchestrator
|   |   |-- requirements.txt
|   |   |-- batch_embed.py             # Batch embedding with Gemini
|   |   |-- index_manager.py           # Vertex AI Vector Search index ops
|   |   |-- index_updater.py           # Incremental index updates
|   |
|   |-- retrieval/
|   |   |-- __init__.py
|   |   |-- query_reformulator.py      # Multi-query generation
|   |   |-- hybrid_retriever.py        # Vector + BM25 + metadata fusion
|   |   |-- diversity_enforcer.py      # Cross-domain/author diversity
|   |   |-- passage_ranker.py          # Re-ranking with cross-encoder
|   |
|   |-- synthesis/
|   |   |-- __init__.py
|   |   |-- pantheon_engine.py         # Core multi-perspective synthesis
|   |   |-- debate_engine.py           # Debate mode orchestration
|   |   |-- deep_dive_engine.py        # Single-author deep analysis
|   |   |-- verification.py           # Citation verification + anachronism check
|   |   |-- response_formatter.py     # JSON to markdown/HTML formatting
|   |
|   |-- personas/
|   |   |-- __init__.py
|   |   |-- persona_schema.py          # AuthorPersona dataclass
|   |   |-- extraction.py             # Persona extraction from works
|   |   |-- anachronism_guard.py      # Temporal validity checking
|   |   |-- panels.py                 # Preset and custom panel definitions
|   |   |-- data/
|   |   |   |-- canonical_authors.json  # Pre-built persona profiles for top 200 authors
|   |   |   |-- canonical_works.json    # List of canonical work IDs
|   |
|   |-- notebooklm/
|   |   |-- __init__.py
|   |   |-- session_manager.py         # NotebookLM session creation
|   |   |-- source_uploader.py        # Upload passages as NotebookLM sources
|
|-- config/
|   |-- gemini_config.py               # Gemini model configuration
|   |-- embedding_config.py            # Embedding model settings
|   |-- vector_search_config.py        # Vector Search index settings
|   |-- document_ai_config.py          # Document AI processor settings
|   |-- storage_config.py             # Cloud Storage bucket settings
|   |-- bigquery_config.py            # BigQuery schema definitions
|   |-- cloudrun_config.py            # Cloud Run service settings
|   |-- firebase_config.py            # Firebase project settings
|   |-- notebooklm_config.py          # NotebookLM API settings
|   |-- constants.py                   # Global constants, thresholds
|
|-- frontend/
|   |-- package.json                   # Next.js dependencies
|   |-- tsconfig.json                  # TypeScript configuration
|   |-- tailwind.config.ts            # Tailwind CSS configuration
|   |-- next.config.js                # Next.js configuration
|   |-- firebase.json                 # Firebase Hosting config
|   |-- .firebaserc                   # Firebase project reference
|   |-- public/
|   |   |-- favicon.ico
|   |   |-- oracle-logo.svg
|   |   |-- thinker-avatars/           # Pre-generated avatar images
|   |       |-- plato.webp
|   |       |-- aristotle.webp
|   |       |-- (200+ thinker avatars)
|   |-- src/
|   |   |-- app/
|   |   |   |-- layout.tsx             # Root layout
|   |   |   |-- page.tsx              # Landing page
|   |   |   |-- globals.css           # Global styles
|   |   |   |-- consultation/
|   |   |   |   |-- page.tsx          # Main consultation page
|   |   |   |   |-- loading.tsx       # Loading state
|   |   |   |-- library/
|   |   |   |   |-- page.tsx          # User library
|   |   |   |-- explore/
|   |   |   |   |-- page.tsx          # Corpus explorer
|   |   |   |-- debate/
|   |   |   |   |-- page.tsx          # Dedicated debate mode
|   |   |-- components/
|   |   |   |-- consultation/
|   |   |   |   |-- QueryInput.tsx
|   |   |   |   |-- PanelSelector.tsx
|   |   |   |   |-- ModeSelector.tsx
|   |   |   |   |-- OracleResponse.tsx
|   |   |   |   |-- PerspectiveCard.tsx
|   |   |   |   |-- ContradictionMap.tsx
|   |   |   |   |-- SynthesisBlock.tsx
|   |   |   |   |-- CitationLink.tsx
|   |   |   |   |-- ConfidenceBadge.tsx
|   |   |   |   |-- FollowUpSuggestions.tsx
|   |   |   |-- debate/
|   |   |   |   |-- DebateView.tsx
|   |   |   |   |-- DebateRound.tsx
|   |   |   |-- pantheon/
|   |   |   |   |-- PantheonVisual.tsx
|   |   |   |   |-- ThinkerAvatar.tsx
|   |   |   |   |-- ThinkerModal.tsx
|   |   |   |-- library/
|   |   |   |   |-- SessionList.tsx
|   |   |   |   |-- PanelManager.tsx
|   |   |   |   |-- BookmarkList.tsx
|   |   |   |-- shared/
|   |   |   |   |-- Header.tsx
|   |   |   |   |-- Footer.tsx
|   |   |   |   |-- LoadingSpinner.tsx
|   |   |   |   |-- ErrorBoundary.tsx
|   |   |   |   |-- AuthGuard.tsx
|   |   |-- lib/
|   |   |   |-- firebase.ts           # Firebase client init
|   |   |   |-- api.ts               # Backend API client
|   |   |   |-- types.ts             # TypeScript interfaces
|   |   |   |-- hooks/
|   |   |   |   |-- useOracle.ts      # Main query hook
|   |   |   |   |-- useAuth.ts       # Auth state hook
|   |   |   |   |-- usePanels.ts     # Panel management
|   |   |   |   |-- useLibrary.ts    # Session history
|   |   |   |-- utils/
|   |   |   |   |-- formatters.ts    # Response formatting
|   |   |   |   |-- exporters.ts     # PDF/Markdown export
|
|-- scripts/
|   |-- setup_gcp.sh                   # One-time GCP project setup
|   |-- download_gutenberg.py          # Bulk download from Gutenberg
|   |-- ingest_corpus.py              # Run full ingestion pipeline
|   |-- build_embeddings.py           # Run embedding pipeline
|   |-- deploy_index.py              # Deploy Vector Search index
|   |-- seed_personas.py             # Generate canonical author personas
|   |-- demo_setup.sh                # Quick setup for hackathon demo
|
|-- data/
|   |-- canonical_works_list.csv       # Curated list of 100K target works
|   |-- demo_corpus_1000.txt          # IDs for 1000-book demo corpus
|   |-- domain_taxonomy.json          # Full domain/subdomain hierarchy
|   |-- sample_queries.json           # Demo questions for judging
|
|-- tests/
|   |-- test_parser.py
|   |-- test_chunker.py
|   |-- test_retrieval.py
|   |-- test_synthesis.py
|   |-- test_verification.py
|   |-- test_api.py
|   |-- test_frontend/                 # Cypress or Playwright tests
|       |-- consultation.spec.ts
|
|-- docs/
|   |-- architecture.md               # Architecture overview
|   |-- api_reference.md             # API endpoint documentation
|   |-- deployment_guide.md          # Step-by-step deployment
|   |-- corpus_guide.md             # How to add new sources
|   |-- persona_guide.md            # How to create/edit personas
```

---

## SECTION 8: HACKATHON DEMO PLAN

### 8.1 Pre-Loaded Demo Corpus (1,000 Books)

The demo corpus is curated for maximum impact with judges:

**Tier 1 -- The "Greatest Hits" (100 books, pre-embedded, pre-persona'd):**
- Plato: Republic, Symposium, Apology
- Aristotle: Nicomachean Ethics, Politics, Poetics
- Sun Tzu: The Art of War
- Machiavelli: The Prince, Discourses
- Adam Smith: Wealth of Nations, Theory of Moral Sentiments
- Karl Marx: Capital (Vol 1), Communist Manifesto
- Darwin: Origin of Species
- Nietzsche: Beyond Good and Evil, Thus Spoke Zarathustra, Genealogy of Morals
- Kant: Critique of Pure Reason, Groundwork of Metaphysics of Morals
- Shakespeare: Hamlet, King Lear, Macbeth, The Tempest
- Dostoevsky: Crime and Punishment, Brothers Karamazov, Notes from Underground
- Marcus Aurelius: Meditations
- Clausewitz: On War
- Thoreau: Walden, Civil Disobedience
- Lao Tzu: Tao Te Ching
- Confucius: Analects
- Bhagavad Gita
- Bible (KJV)
- Quran (English translation)
- Buddha: Dhammapada
- Mary Shelley: Frankenstein
- Frederick Douglass: Narrative of the Life
- Mary Wollstonecraft: Vindication of the Rights of Woman
- John Stuart Mill: On Liberty, Utilitarianism
- Thomas Hobbes: Leviathan
- John Locke: Two Treatises of Government
- Jean-Jacques Rousseau: Social Contract
- Voltaire: Candide
- Montesquieu: Spirit of the Laws
- Hannah Arendt: Origins of Totalitarianism (if public domain portion available)
- And 70 more canonical works

**Tier 2 (400 books):** Broader philosophy, science, literature classics from Gutenberg
**Tier 3 (500 books):** Diverse domain coverage -- law, medicine, economics, technology

### 8.2 Demo Question Selection (5 Prepared Demos)

**Demo 1 -- "The CEO Question" (Strategic):**
> "I'm deciding whether to pursue aggressive market expansion or consolidate my position. What would history's greatest strategic and economic minds advise?"

Expected output: Sun Tzu on timing and terrain, Machiavelli on fortune and virtu, Adam Smith on market dynamics, Clausewitz on concentration of force, with contradictions mapped between offensive and defensive strategies.

**Demo 2 -- "The Trolley Problem Goes to Court" (Philosophy + Law):**
> "A self-driving car must choose between hitting one pedestrian or swerving into a group of five. How should we think about this?"

Expected output: Bentham (utilitarian calculus), Kant (categorical imperative -- cannot use person as means), Rawls (veil of ignorance), Asimov (robotics laws), plus legal analysis of liability frameworks.

**Demo 3 -- "Oracle Debates Itself" (The Showstopper):**
> Run ORACLE in debate mode: "Plato vs. Nietzsche on the purpose of human life."

This is the demo moment. Two of history's most opposing thinkers, with ORACLE generating a structured debate grounded in their actual texts -- Plato's Forms and the Good versus Nietzsche's will to power and eternal recurrence. Every claim cited. Contradictions made vivid.

**Demo 4 -- "The Recursive Question":**
> "What would the greatest thinkers in history think about an AI system that lets you consult the greatest thinkers in history?"

ORACLE consults Plato (on the nature of mediated knowledge -- the allegory of the cave), McLuhan (the medium is the message), Socrates (on writing as inferior to dialogue -- from Phaedrus), and Borges (the Library of Babel). This demonstrates self-awareness and meta-reasoning.

**Demo 5 -- "Live Judge Question":**
Let judges ask anything. The 1,000-book corpus is broad enough to handle most intellectual questions. Have backup domains pre-loaded for common judge interests (business strategy, AI ethics, creativity).

### 8.3 Demo Flow (5-Minute Pitch)

```
00:00-00:30  HOOK: "130 million books exist. You'll read 720 in your lifetime. 
             What if you could consult all of them at once?"
00:30-01:30  PROBLEM: Show the knowledge gap visually. Show why search isn't enough.
01:30-02:00  SOLUTION: "ORACLE -- consult history's greatest minds simultaneously."
02:00-03:00  LIVE DEMO: Run Demo 1 (CEO question). Show response streaming in.
             Highlight: citations, contradictions, synthesis.
03:00-04:00  WOW MOMENT: Run Demo 3 (Plato vs Nietzsche debate). 
             Show debate format with real citations from real texts.
04:00-04:30  ARCHITECTURE: 30-second flash of the technical architecture.
             "Gemini 1.5 Pro 2M context, 100K books embedded, Vertex AI Vector Search,
             NotebookLM for deep dives."
04:30-05:00  CLOSE: "Every great decision in history was informed by accumulated wisdom.
             ORACLE makes that wisdom accessible to everyone."
```

---

## SECTION 9: BUILD PLAN (Multi-Week Production Build)

> **Note**: The hour-by-hour plan below was originally designed for a 24-hour hackathon sprint. For a multi-week build, treat each "phase" as a **week-long milestone** instead. This gives time for proper testing, code review, and iteration at each stage.

| Week | Focus | Corresponds to Sprint Phase |
|------|-------|-----------------------------|
| Week 1 | Infrastructure + Ingestion Pipeline (GCP/AWS setup, parsers, chunking) | Phase 1 + Phase 2 (Hours 0-10) |
| Week 2 | Embedding + Retrieval (batch embedding, vector index, hybrid search) | Phase 2 continued |
| Week 3 | Synthesis Engine + Persona System (Pantheon engine, verification, debate) | Phase 2 + Phase 4 |
| Week 4 | Frontend (Next.js app, consultation UI, streaming, all components) | Phase 3 |
| Week 5 | Integration + Polish (end-to-end, NotebookLM, export, error handling) | Phase 4 + Phase 5 |
| Week 6 | Testing + Optimization (load testing, caching, latency, deployment) | Phase 5 + Phase 6 |

### Original 24-Hour Sprint Reference (retained for accelerated builds)

**PHASE 1: Foundation (Hours 0-4)**

| Hour | Task | Deliverable | Owner | Fallback |
|------|------|-------------|-------|----------|
| 0-1 | GCP setup: enable APIs, create buckets, create BigQuery dataset, set up Firebase project | All infrastructure running | Infra | Use terraform apply for speed |
| 1-2 | Download Gutenberg Tier 1 (100 books as plain text), parse and chunk | 100 books chunked as JSON in Cloud Storage | Backend | Pre-download before hackathon starts |
| 2-3 | Embed Tier 1 chunks with Gemini Embeddings batch API | ~200K embeddings in staging bucket | Backend | Reduce to 50 books if API rate-limited |
| 3-4 | Create and deploy Vertex AI Vector Search index with Tier 1 embeddings | Working vector search endpoint | Backend | Use in-memory FAISS as fallback |

**PHASE 2: Core Engine (Hours 4-10)**

| Hour | Task | Deliverable | Owner | Fallback |
|------|------|-------------|-------|----------|
| 4-5 | Build FastAPI skeleton: health, query endpoint, Firebase auth middleware | API responds to health check | Backend | Skip auth, add later |
| 5-6 | Implement query reformulator + vector retrieval | API returns raw passages for a query | Backend | Single query (no reformulation) |
| 6-7 | Implement BM25 search via BigQuery + hybrid fusion | Hybrid retrieval working | Backend | Vector-only (skip BM25) |
| 7-8 | Implement Pantheon synthesis engine with Gemini 1.5 Pro | Full synthesis response for a query | Backend | Simpler single-prompt, no structured JSON |
| 8-9 | Implement citation verification + anachronism guard | Verified responses with confidence scores | Backend | Skip verification, add confidence disclaimer |
| 9-10 | Deploy API to Cloud Run, test end-to-end | Deployed API returning full Oracle responses | Backend | Local only, demo from laptop |

**PHASE 3: Frontend (Hours 4-12, parallel with backend)**

| Hour | Task | Deliverable | Owner | Fallback |
|------|------|-------------|-------|----------|
| 4-6 | Next.js scaffold: layout, consultation page, Firebase auth | App skeleton with auth working | Frontend | Skip auth, hardcode user |
| 6-8 | QueryInput + OracleResponse components with streaming | Can submit query and see response stream in | Frontend | No streaming, show after complete |
| 8-10 | PerspectiveCard, ContradictionMap, SynthesisBlock, CitationLink | Full response rendering | Frontend | Plain markdown rendering |
| 10-12 | PantheonVisual (thinker avatars in circle), DebateView, ModeSelector | Visual polish, debate mode | Frontend | Static avatar list, no animation |

**PHASE 4: Integration and Polish (Hours 10-16)**

| Hour | Task | Deliverable | Owner | Fallback |
|------|------|-------------|-------|----------|
| 10-11 | Integrate frontend with deployed Cloud Run API | End-to-end working | Full team | Local API fallback |
| 11-12 | NotebookLM integration: session creation, source upload | "Deep Dive" button creates NotebookLM session | Backend | Link to NotebookLM without pre-populated sources |
| 12-13 | Debate mode end-to-end | Debate view fully working | Full team | Debate as text-only in consultation view |
| 13-14 | Ingest Tier 2 (400 books) in background while polishing | Broader corpus | Backend | Stick with 100 books |
| 14-15 | Persona extraction for top 20 authors | Pre-built personas for key thinkers | Backend | Dynamic persona inference only |
| 15-16 | Follow-up questions, session history in Firestore | Conversational continuity | Full team | Single-turn only |

**PHASE 5: Demo Prep (Hours 16-20)**

| Hour | Task | Deliverable | Owner | Fallback |
|------|------|-------------|-------|----------|
| 16-17 | Pre-run all 5 demo queries, cache responses | Demo runs in <10 seconds | Full team | Pre-recorded video backup |
| 17-18 | UI polish: loading animations, error states, responsive design | Professional-looking UI | Frontend | Functional but plain |
| 18-19 | Ingest Tier 3 (500 books) if time permits | 1,000 book corpus | Backend | 500 books is fine |
| 19-20 | Deploy to production Firebase Hosting URL | oracle-demo.web.app live | Full team | Demo from localhost |

**PHASE 6: Final Prep (Hours 20-24)**

| Hour | Task | Deliverable | Owner | Fallback |
|------|------|-------------|-------|----------|
| 20-21 | Load testing: verify latency under concurrent queries | Performance numbers for pitch | Backend | Single-user demo only |
| 21-22 | Pitch deck preparation (slides as backup to live demo) | 5-slide deck | Full team | N/A |
| 22-23 | Full rehearsal: run through all 5 demos, time the pitch | Polished 5-minute delivery | Full team | N/A |
| 23-24 | Buffer: fix last-minute issues, sleep if possible | Ready to present | Full team | N/A |

### Critical Path

The critical path is: **GCP Setup -> Gutenberg Download -> Chunking -> Embedding -> Index Deploy -> Retrieval -> Synthesis -> API Deploy -> Frontend Integration**

If any step fails:
- **Index deploy fails**: Fall back to in-memory FAISS with 100 books
- **Cloud Run deploy fails**: Demo from localhost with ngrok tunnel
- **NotebookLM API unavailable**: Show manual NotebookLM with pre-uploaded sources
- **Gemini 1.5 Pro rate limited**: Queue queries, show pre-cached results for demos

---

## SECTION 10: PRODUCTION AND SCALE

### 10.1 Cost Estimates (Monthly, at Scale)

| Component | Specification | Monthly Cost |
|-----------|--------------|-------------|
| Cloud Storage | 1TB raw corpus + 500GB processed | $30 |
| BigQuery | 500GB stored, 10TB scanned/month | $55 |
| Vertex AI Vector Search | 1 node (e2-standard-16), 100M vectors | $280 |
| Gemini 1.5 Pro (synthesis) | 10K queries/day, ~500K tokens/query avg | $4,500 (with caching) |
| Gemini Flash (reformulation) | 10K queries/day, ~2K tokens/query | $60 |
| Gemini Embeddings | Incremental only after initial build | $50 |
| Cloud Run (API) | 2 instances avg | $150 |
| Cloud Run (ingestion) | Burst during updates | $100 |
| Firebase (auth + hosting + Firestore) | 10K MAU | $50 |
| NotebookLM Enterprise | Per-session creation | $200 |
| Document AI | Incremental processing | $50 |
| **Total** | | **~$5,525/month** |

### 10.2 Latency Optimization

Target: Full Oracle response in under 15 seconds.

| Stage | Target Latency | Optimization |
|-------|---------------|-------------|
| Query reformulation | 500ms | Gemini Flash, parallel |
| Embedding (query) | 200ms | Pre-warmed connection |
| Vector search | 300ms | Vertex AI endpoint, low-latency config |
| BM25 search | 500ms | BigQuery with search index |
| Rank fusion + diversity | 100ms | In-memory Python |
| Synthesis (Gemini 1.5 Pro) | 10-12s | Stream output, context caching |
| Verification | 500ms | Parallel with streaming |
| **Total** | **~13s** | **Stream perspectives as they generate** |

### 10.3 Caching Strategy

1. **Gemini Context Cache**: Cache the top 100 most-referenced works as a persistent context. 90% cost reduction on cached tokens. Refresh every 24 hours.

2. **Response Cache (Firestore)**: Cache complete responses for identical queries. TTL: 7 days. Hit rate estimate: 15-20% for common questions.

3. **Embedding Cache (Redis on Memorystore)**: Cache query embeddings. Most repeated queries will hit this cache. TTL: 1 hour.

4. **CDN**: Firebase Hosting CDN for all static assets. Edge caching for the frontend.

### 10.4 Scaling Strategy

- **Vector Search**: Auto-scale from 1 to 4 replicas based on QPS
- **Cloud Run API**: Auto-scale from 1 to 10 instances, with min 1 for cold-start avoidance
- **BigQuery**: Serverless, auto-scales. Use reservations if cost predictability needed.
- **Gemini API**: Request quota increase to 1000 RPM for production

---

## SECTION 11: TECHNICAL RISKS AND MITIGATIONS

### 11.1 Copyright Risk

| Risk | Severity | Mitigation |
|------|----------|------------|
| Including copyrighted works without license | HIGH | Hackathon: public domain only (Gutenberg, pre-1928). Production: publisher partnerships, CC-only, author opt-in program. |
| Fair use for excerpts | MEDIUM | Display only short passages (< 500 words) with full attribution. Never display full chapters in responses. |
| Regional copyright variation | MEDIUM | Geo-fence corpus by jurisdiction. US copyright for US users, EU for EU. |

### 11.2 Hallucinated Attributions

| Risk | Severity | Mitigation |
|------|----------|------------|
| LLM attributes a position an author never held | HIGH | Citation verification system (Section 2.7). Every quote checked against retrieved passages. Confidence scores displayed. |
| LLM generates fake quotes | HIGH | Fuzzy-match verification. Any quote not found at 90% similarity in sources is flagged and removed. |
| Anachronistic attributions | MEDIUM | Anachronism guard checks author death year against topic origin year. Marks as [INFERENCE] with reasoning chain. |
| Over-confident synthesis | MEDIUM | Confidence scores on every claim. Retrieval coverage score indicates how well sources cover the question. |

### 11.3 Retrieval Quality

| Risk | Severity | Mitigation |
|------|----------|------------|
| Vector search misses relevant passages | HIGH | Multi-query reformulation (5 angles). Hybrid search (vector + BM25). Quality-weighted scoring. |
| Over-representation of popular works | MEDIUM | Crowding tag limits 3 passages per book in vector results. Diversity enforcer ensures multi-domain coverage. |
| Embedding quality degrades for non-English or archaic text | MEDIUM | Language-specific embedding models for production. For hackathon, use English translations only. |

### 11.4 Latency

| Risk | Severity | Mitigation |
|------|----------|------------|
| Gemini 1.5 Pro 2M context is slow (>30s) | HIGH | Stream responses via SSE. Show perspectives as they generate. Context caching for repeated corpus segments. |
| Vector Search cold start | MEDIUM | Keep minimum 1 replica always running. Pre-warm with periodic health checks. |
| BigQuery cold start for BM25 | LOW | BigQuery reservations for predictable latency. Fall back to vector-only if BM25 times out. |

### 11.5 API Availability

| Risk | Severity | Mitigation |
|------|----------|------------|
| NotebookLM Enterprise API is alpha, may be unreliable | MEDIUM | Graceful degradation: if API fails, link to manual NotebookLM. Do not block the core flow on it. |
| Gemini API rate limits during hackathon | HIGH | Pre-cache demo responses. Request quota increase before the event. Have a recorded video backup. |
| Vertex AI Vector Search index deployment takes hours | HIGH | Start index build as the first task. Have a FAISS fallback with 100 books for demo. |

---

## SECTION 12: BUSINESS MODEL

### 12.1 Subscription Tiers

| Tier | Price | Queries/Month | Features |
|------|-------|--------------|----------|
| **Free** | $0 | 10 | Basic consultation, 3 perspectives, no deep dive |
| **Scholar** | $19/mo | 100 | Full consultation + debate mode, 6 perspectives, NotebookLM deep dive, export |
| **Sage** | $49/mo | 500 | Everything in Scholar + custom panels, priority latency, API access |
| **Institution** | Custom | Unlimited | SSO, dedicated corpus, custom domain, analytics dashboard |

### 12.2 Revenue Projections (Year 1)

- **Target**: 50,000 users by end of Year 1
- **Conversion**: 5% to Scholar ($19), 1% to Sage ($49), 10 institutional ($500/mo avg)
  - Free: 47,000 users, $0
  - Scholar: 2,500 users * $19 = $47,500/mo
  - Sage: 500 users * $49 = $24,500/mo
  - Institution: 10 * $500 = $5,000/mo
  - **Total MRR: ~$77,000** (~$924K ARR)
- **COGS**: ~$5,500/mo infrastructure + ~$0.30/query variable = ~$20K/mo at scale
- **Gross margin**: ~74%

### 12.3 Institutional Licensing

- **Universities**: $5,000-50,000/year based on enrollment. Integrate with LMS (Canvas, Blackboard). Students use ORACLE for research papers, philosophy courses, literature analysis.
- **Law firms**: $10,000-100,000/year. Custom legal corpus with case law, statutes, legal commentary.
- **Consulting firms**: $25,000-200,000/year. Strategy corpus, proprietary research integration.
- **Libraries**: $2,000-20,000/year. Public access terminals, integration with existing catalogs.

### 12.4 Education Market

- **AP / IB courses**: Pre-built panels for Philosophy, History, Literature
- **Writing assistance**: ORACLE helps students find sources and understand multiple perspectives (not write their papers)
- **Research tool**: Graduate students use deep dive mode for literature reviews

### 12.5 Enterprise

- **Custom corpus**: Companies upload their internal knowledge base alongside the public corpus
- **Private deployment**: On-prem or VPC deployment for sensitive industries
- **API access**: Integrate Oracle consultation into existing applications

---

## SECTION 13: JUDGING CRITERIA AND PITCH STRATEGY

### 13.1 Typical Hackathon Judging Criteria

| Criterion | Weight | How ORACLE Wins |
|-----------|--------|-----------------|
| **Innovation** | 25% | No product does multi-perspective, citation-grounded synthesis across 100K books. The "consulting a panel" metaphor is novel and visceral. |
| **Technical Complexity** | 25% | Full RAG pipeline with hybrid retrieval, 2M context synthesis, citation verification, persona grounding, NotebookLM integration. This is not a wrapper around an API. |
| **Impact / Usefulness** | 25% | Universal use case: anyone who has ever needed to make a decision informed by accumulated human wisdom. Education, legal, medical, strategic. |
| **Completeness / Polish** | 15% | Working end-to-end demo with streaming UI, multiple modes (consultation, debate), real citations from real books. |
| **Presentation** | 10% | The "Plato vs. Nietzsche" debate moment is memorable. The recursive "what would thinkers think about AI consulting thinkers" question is a crowd-pleaser. |

### 13.2 Pitch Strategy

**Open with the gap**: "130 million books. You'll read 720. What's in the other 129,999,280?"

**Show, don't tell**: Go live within 90 seconds. Let judges see the response streaming in with real citations from real books.

**The "wow" moment**: The Plato vs. Nietzsche debate. Judges remember the emotional moment, not the architecture slide.

**Technical credibility in 30 seconds**: One architecture diagram. Name the stack: "Gemini 1.5 Pro 2M context, 100K books, Vertex AI Vector Search, Document AI, NotebookLM." Show you know what you built.

**Close with the vision**: "Every great decision in history was informed by accumulated wisdom. We just made that wisdom accessible to everyone."

### 13.3 Handling Judge Questions

**"How do you prevent hallucination?"** -- "Every quote is verified against source text with fuzzy matching. Every attribution has a confidence score. Anachronistic claims are automatically flagged. We show our work."

**"What about copyright?"** -- "Our hackathon corpus is 100% public domain from Project Gutenberg. For production, we have a three-tier strategy: public domain, Creative Commons, and publisher partnerships."

**"How does this scale?"** -- "Vertex AI Vector Search handles billion-scale embeddings. Gemini context caching cuts per-query costs by 90%. Our architecture is designed for 100K books today, millions tomorrow."

**"What's the moat?"** -- "Three things: the curated corpus with quality scoring, the persona extraction pipeline that faithfully represents each thinker's documented worldview, and the synthesis engine that maps contradictions rather than hiding them."

---

## SECTION 14: APPENDICES

### 14.1 Full Tech Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| LLM (Synthesis) | Gemini 1.5 Pro (2M context) | Multi-perspective synthesis with full context |
| LLM (Fast tasks) | Gemini 2.0 Flash | Query reformulation, persona selection, summaries |
| Embeddings | Gemini text-embedding-004 (768 dim) | Document and query embedding |
| Vector DB | Vertex AI Vector Search (ScaNN) | Approximate nearest neighbor at 100M scale |
| Keyword Search | BigQuery Full-Text Search | BM25 keyword retrieval |
| Document Parsing | Document AI (OCR + Layout Parser) | PDF/scanned document extraction |
| Object Storage | Cloud Storage | Raw and processed corpus |
| Analytics DB | BigQuery | Corpus metadata, query logs, analytics |
| Compute | Cloud Run | Serverless API and ingestion workers |
| Auth | Firebase Authentication | User identity with Google sign-in |
| User Data | Cloud Firestore | Sessions, panels, user preferences |
| Hosting | Firebase App Hosting | Next.js frontend with CDN |
| Deep Dive | NotebookLM Enterprise API | Per-session deep exploration |
| Frontend | Next.js 14 + TypeScript + Tailwind + shadcn/ui | Consultation interface |
| IaC | Terraform | Infrastructure as code |
| CI/CD | Cloud Build | Automated builds and deploys |

### 14.2 Corpus Source URLs

| Source | URL | Access Method |
|--------|-----|---------------|
| Project Gutenberg | https://www.gutenberg.org/ | Gutendex API (https://gutendex.com/) or mirror rsync |
| arXiv | https://arxiv.org/ | arXiv API (https://arxiv.org/help/api) |
| PubMed Central | https://www.ncbi.nlm.nih.gov/pmc/ | PMC OAI-PMH service |
| Internet Archive | https://archive.org/ | Open Library API |
| OpenStax | https://openstax.org/ | Direct download (CC-BY) |
| CourtListener | https://www.courtlistener.com/ | CourtListener API |
| Sacred Texts | https://www.sacred-texts.com/ | Web scraping (public domain) |

### 14.3 Embedding Schema (Vertex AI Vector Search JSON-L Format)

```json
{"id": "chunk_abc123", "embedding": [0.012, -0.034, ...768 floats...], "restricts": [{"namespace": "domain", "allow_list": ["philosophy", "ethics"]}, {"namespace": "era", "allow_list": ["ancient"]}, {"namespace": "level", "allow_list": ["passage"]}, {"namespace": "language", "allow_list": ["en"]}, {"namespace": "source", "allow_list": ["gutenberg"]}], "numeric_restricts": [{"namespace": "quality_score", "value_float": 0.85}, {"namespace": "pub_year", "value_int": -380}], "crowding_tag": "book_plato_republic"}
```

### 14.4 Sample Prompt Templates

**Query Reformulation Prompt:**
```
You are a search query optimizer for a library of 100,000 books spanning philosophy, science, 
strategy, law, economics, literature, and religion.

Given the user's question, generate exactly 5 search queries that will retrieve diverse, 
relevant passages. Each query should target a DIFFERENT angle:

1. DIRECT: Restate the core question in precise academic language
2. HISTORICAL: Frame as a historical question about how thinkers have addressed this
3. CONTRARIAN: Search for the strongest opposing viewpoint
4. ADJACENT: Search for a related concept from a different domain
5. SPECIFIC: Search for a specific thinker/work most likely to address this

User question: {user_query}

Return as JSON array of 5 strings. No explanation.
```

**Persona Selection Prompt:**
```
Given the following user question and list of authors represented in the retrieved passages, 
select 3-6 authors whose perspectives would create the most insightful and diverse response.

CRITERIA:
- Select at least one author who would AGREE and one who would DISAGREE
- Include perspectives from different eras and traditions
- Prefer authors whose specific works are well-represented in the passages
- Do not select more than 6

User question: {query}
Available authors (with passage counts): {authors_with_counts}

Return as JSON array of author names.
```

**Synthesis System Prompt:**
(Full version in Section 2.5 above)

**Debate System Prompt:**
(Full version in Section 5.4 above)

### 14.5 Sample Response Format

**Query**: "Is it ethical to lie to protect someone's feelings?"

```json
{
  "query": "Is it ethical to lie to protect someone's feelings?",
  "mode": "consultation",
  "perspectives": [
    {
      "thinker": "Immanuel Kant",
      "era": "1724-1804",
      "tradition": "Deontological Ethics",
      "position": "Lying is never morally permissible, regardless of consequences. The categorical imperative demands that we act only according to maxims we could will to be universal laws. A universal law of lying is self-contradictory, as it would destroy the very institution of truth-telling upon which lying depends.",
      "key_passages": [
        {
          "text": "By a lie a man throws away and, as it were, annihilates his dignity as a man.",
          "source": "Groundwork of the Metaphysics of Morals, Section II",
          "confidence": 0.97
        },
        {
          "text": "To be truthful (honest) in all declarations is therefore a sacred unconditional command of reason, and not to be limited by any expediency.",
          "source": "On a Supposed Right to Lie from Philanthropic Concerns",
          "confidence": 0.99
        }
      ],
      "reasoning": "Kant grounds his absolute prohibition on lying in the structure of rational agency itself. To lie is to treat the other person as a mere means -- manipulating their beliefs for your purposes rather than respecting their rational autonomy.",
      "limitations": "Kant's absolute prohibition leads to well-known counterexamples (lying to a murderer about a victim's location) that many find morally counterintuitive."
    },
    {
      "thinker": "John Stuart Mill",
      "era": "1806-1873",
      "tradition": "Utilitarianism",
      "position": "The morality of lying depends entirely on its consequences. If a lie produces greater overall happiness than the truth -- including protecting someone from unnecessary suffering -- it may be not only permissible but obligatory. However, the long-term utility of truthfulness as a social norm must be weighed against short-term benefits of individual lies.",
      "key_passages": [
        {
          "text": "The utilitarian morality does recognize in human beings the power of sacrificing their own greatest good for the good of others.",
          "source": "Utilitarianism, Chapter 2",
          "confidence": 0.88
        }
      ],
      "reasoning": "Mill's consequentialism evaluates actions by their outcomes. A lie that prevents genuine harm and causes no downstream damage produces net positive utility. But Mill also acknowledges the rule-level utility of honesty as a social practice.",
      "limitations": "Utilitarian calculations are difficult in practice. Predicting the full consequences of a lie is often impossible, and the habit of lying may erode character over time."
    },
    {
      "thinker": "Confucius",
      "era": "551-479 BCE",
      "tradition": "Confucian Ethics",
      "position": "Truth-telling must be understood within the context of relationships and social harmony (li). A person of virtue (junzi) exercises wisdom in knowing when directness serves the relationship and when it causes unnecessary harm. The virtue of ren (benevolence/humaneness) may sometimes require tempering truth with compassion. [INFERENCE: Confucius does not directly address 'white lies,' but his emphasis on contextual virtue and relational harmony suggests a nuanced rather than absolutist position.]",
      "key_passages": [
        {
          "text": "The Master said, 'Is it not a pleasure to learn and to apply what one has learned? Is it not a joy to have friends come from afar?'",
          "source": "Analects, Book I",
          "confidence": 0.60
        },
        {
          "text": "The Master said, 'A man of virtue is not an instrument.'",
          "source": "Analects, Book II",
          "confidence": 0.65
        }
      ],
      "reasoning": "Confucian ethics is relational and contextual rather than rule-based. The emphasis on ren (benevolence) and li (propriety) suggests that the ethical actor considers the relationship, the context, and the impact on social harmony when deciding how to communicate difficult truths.",
      "limitations": "Confucian ethics can be criticized for potentially prioritizing social harmony over individual autonomy and honest communication."
    }
  ],
  "contradictions": [
    {
      "between": ["Immanuel Kant", "John Stuart Mill"],
      "nature": "Fundamental disagreement about whether moral rules admit exceptions. Kant holds that truthfulness is an unconditional duty; Mill holds that all moral rules are subordinate to the principle of utility. This is the classic deontology vs. consequentialism divide.",
      "resolution_attempts": "W.D. Ross's prima facie duties attempt a middle ground: truthfulness is a genuine obligation, but it can be overridden by competing obligations (like preventing harm) in specific situations."
    },
    {
      "between": ["Immanuel Kant", "Confucius"],
      "nature": "Kant's universal, context-independent moral law contrasts with Confucius's relational, context-dependent virtue ethics. For Kant, the rightness of an action is determined by its conformity to rational principle regardless of relationship; for Confucius, the nature of the relationship is morally relevant.",
      "resolution_attempts": "Care ethics (Noddings, Gilligan) bridges these by arguing that moral reasoning is inherently relational while maintaining that some universal principles can emerge from relational thinking."
    }
  ],
  "synthesis": {
    "meta_insight": "The question reveals a fundamental tension in moral philosophy between rule-based and context-based ethics. Kant offers moral clarity at the cost of rigidity; Mill offers flexibility at the cost of predictability; Confucius offers relational wisdom at the cost of universal applicability. The most practically useful insight may be that the question itself is under-specified: the ethics of a specific lie depend on the relationship, the stakes, the alternatives to lying, and whether the 'protection' is genuine or paternalistic.",
    "practical_implications": "Before lying to protect someone's feelings, consider: (1) Are you protecting them or yourself? (2) Would they want to be protected from this truth? (3) Is there a way to communicate the truth with compassion rather than avoiding it? (4) What are the long-term consequences of this lie for the relationship and your character?",
    "further_reading": [
      "Sissela Bok - 'Lying: Moral Choice in Public and Private Life'",
      "Bernard Williams - 'Truth and Truthfulness'",
      "W.D. Ross - 'The Right and the Good'"
    ]
  },
  "confidence_assessment": {
    "retrieval_coverage": 0.82,
    "attribution_confidence": 0.88,
    "synthesis_novelty": 0.65
  }
}
```

### 14.6 Environment Variables

```bash
# /Users/shauryapunj/Desktop/009/.env.example

# Google Cloud
GCP_PROJECT_ID=oracle-prod
GCP_REGION=us-central1

# Gemini
GEMINI_PRO_MODEL=gemini-1.5-pro-002
GEMINI_FLASH_MODEL=gemini-2.0-flash
GEMINI_EMBEDDING_MODEL=text-embedding-004
EMBEDDING_DIMENSIONS=768

# Vertex AI Vector Search
VECTOR_SEARCH_INDEX_ENDPOINT=projects/oracle-prod/locations/us-central1/indexEndpoints/XXXX
VECTOR_SEARCH_DEPLOYED_INDEX_ID=oracle_deployed

# BigQuery
BIGQUERY_DATASET=oracle_corpus

# Cloud Storage
BUCKET_RAW_CORPUS=oracle-raw-corpus
BUCKET_PROCESSED=oracle-processed-chunks
BUCKET_EMBEDDINGS=oracle-embeddings-staging

# Firebase
FIREBASE_PROJECT_ID=oracle-prod
FIREBASE_API_KEY=XXXXX
FIREBASE_AUTH_DOMAIN=oracle-prod.firebaseapp.com

# NotebookLM
NOTEBOOKLM_API_ENDPOINT=notebooklm.googleapis.com

# Document AI
DOCUMENT_AI_PROCESSOR_ID=XXXXX

# API Config
API_PORT=8080
API_MAX_QUERY_LENGTH=2000
API_RATE_LIMIT_PER_MINUTE=10
API_RESPONSE_TIMEOUT_SECONDS=300
```

### 14.7 Key Dependencies

**Backend (Python 3.11+):**
```
# /Users/shauryapunj/Desktop/009/services/api/requirements.txt
fastapi==0.115.0
uvicorn[standard]==0.30.0
google-cloud-aiplatform==1.70.0
google-cloud-bigquery==3.25.0
google-cloud-storage==2.18.0
google-cloud-documentai==2.30.0
google-cloud-firestore==2.19.0
firebase-admin==6.5.0
vertexai==1.70.0
pydantic==2.9.0
python-multipart==0.0.12
sse-starlette==2.1.0
httpx==0.27.0
numpy==1.26.0
```

**Frontend (Node.js 20+):**
```json
// /Users/shauryapunj/Desktop/009/frontend/package.json (dependencies section)
{
  "dependencies": {
    "next": "^14.2.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "firebase": "^11.0.0",
    "@tanstack/react-query": "^5.50.0",
    "tailwindcss": "^3.4.0",
    "class-variance-authority": "^0.7.0",
    "lucide-react": "^0.400.0",
    "framer-motion": "^11.0.0"
  }
}
```

---

### Critical Files for Implementation

These are the 5 files whose design and implementation will determine whether ORACLE succeeds:

- `/Users/shauryapunj/Desktop/009/services/synthesis/pantheon_engine.py` - The core synthesis engine that takes retrieved passages and produces multi-perspective, citation-grounded responses. This is ORACLE's primary differentiator and the file where the prompt engineering must be most precise.
- `/Users/shauryapunj/Desktop/009/services/retrieval/hybrid_retriever.py` - The hybrid retrieval system combining vector search, BM25, and metadata boosting. If retrieval fails, synthesis quality collapses regardless of how good the LLM prompt is.
- `/Users/shauryapunj/Desktop/009/services/ingestion/chunker.py` - The hierarchical chunking strategy (book/chapter/passage) determines embedding quality and retrieval granularity. Bad chunks mean bad embeddings mean bad retrieval.
- `/Users/shauryapunj/Desktop/009/services/api/routers/query.py` - The main API endpoint that orchestrates the full pipeline: reformulate, retrieve, synthesize, verify, stream. This is the integration point where all subsystems connect.
- `/Users/shauryapunj/Desktop/009/frontend/src/components/consultation/OracleResponse.tsx` - The response renderer that transforms structured JSON into the visual consultation experience. This is what judges and users see; it must make citations, contradictions, and confidence scores intuitive and compelling.