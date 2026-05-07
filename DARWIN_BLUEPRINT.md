# DARWIN: AI-Powered Research Validation Platform
## Complete Technical & Strategic Master Blueprint

**Tagline:** Upload any dataset, hypothesis, or research question. DARWIN designs the experiment, simulates it 10,000 times, stress-tests every assumption, finds p-hacking traps, identifies confounders, and delivers a peer-review-ready result — before you run a single real-world trial.

---

## SECTION 1: PROBLEM DEFINITION & VISION

### 1.1 The Reproducibility Crisis — Quantified

- **$28 billion/year** wasted on irreproducible US preclinical research alone (Freedman, Cockburn & Simcoe, PLOS Biology 2015)
- **>50%** of preclinical studies are irreproducible (cumulative estimate across disciplines)
- **Only 36%** of 100 psychology studies replicated successfully (Open Science Collaboration, Science 2015)
- Replication effect sizes average **50% of originals** — even "successful" replications show inflated original claims
- **97%** of original studies reported significant results; only **36%** of replications did
- Global estimate: **$800B+ cumulatively** wasted across all scientific disciplines when accounting for downstream R&D built on irreproducible foundations

### 1.2 How P-Hacking, HARKing, and Publication Bias Corrupt Science

- **P-hacking:** Exploiting researcher degrees of freedom (outlier exclusion, covariate selection, subgroup analysis, stopping rules) to achieve p < 0.05. Studies show p-value distributions cluster suspiciously just below 0.05
- **HARKing (Hypothesizing After Results Known):** Presenting post-hoc findings as a priori predictions. Turns exploratory analysis into confirmatory claims without proper correction
- **Publication bias:** Journals preferentially publish significant results. File-drawer effect means the published literature systematically overstates effect sizes
- **Garden of forking paths** (Gelman & Loken, 2013): Even honest researchers make dozens of contingent analytical decisions that inflate false positive rates far beyond the nominal 5%

### 1.3 The Confounder Problem

- Most observational studies fail to account for unmeasured confounders
- Simpson's paradox: aggregate trends can reverse when confounders are controlled
- Without causal reasoning (DAGs, do-calculus), correlational findings are routinely misinterpreted as causal
- Domain expertise is required to enumerate plausible confounders — this is where AI reasoning excels

### 1.4 Existing Software Gap

| Tool | Strengths | What It Cannot Do |
|---|---|---|
| SPSS | GUI-friendly, clinical standard | No simulation, no causal inference, no AI reasoning |
| R/RStudio | Powerful, extensible | Requires deep expertise, no automated confounder detection |
| Stata | Econometrics, panel data | No synthetic data generation, no AI interpretation |
| Python (scipy/statsmodels) | Flexible, free | Requires coding, no automated experimental design |
| **None of the above** | — | Automated hypothesis parsing, adversarial simulation, AI-powered peer review |

**The gap:** No tool combines natural language hypothesis input → automated experimental design → large-scale simulation → confounder detection → p-hacking analysis → replication prediction → peer-review-ready reporting.

### 1.5 Why AI Reasoning + Synthetic Data + Parallel Simulation Is the Breakthrough

- **LLM reasoning** (Gemini 1.5 Pro): Parses natural language hypotheses into formal statistical models, reasons about domain-specific confounders using world knowledge, generates human-readable peer review
- **Synthetic data generation**: Bootstrap, parametric, and Bayesian posterior sampling create 10,000 plausible datasets preserving statistical properties
- **Parallel simulation** (Cloud Run / Vertex AI): Runs 10,000 experiments in minutes, not months
- Combined: what would take a statistician weeks of manual analysis, DARWIN delivers in under 5 minutes

### 1.6 Three Primary Users

1. **Academic researchers**: Pre-register studies, validate designs before IRB submission, anticipate reviewer objections
2. **Pharma R&D teams**: De-risk clinical trial designs, estimate required sample sizes, identify confounders before spending millions on enrollment
3. **Industry data scientists**: Validate A/B test designs, detect p-hacking in experimental pipelines, ensure causal (not correlational) conclusions

### 1.7 North Star

DARWIN as the world's first AI peer reviewer that runs *before* submission — catching flawed methodology, underpowered designs, and hidden confounders before they waste resources or corrupt the scientific record.

---

## SECTION 2: CORE TECHNICAL ARCHITECTURE

### 2.1 System Pipeline

```
[Upload Data + Hypothesis]
        │
        ▼
[Document AI / Parser] ─── Parse CSV/Excel/SPSS/PDF
        │
        ▼
[Hypothesis Engine] ─── Gemini extracts formal hypothesis from natural language
        │                  ├── Null hypothesis (H₀)
        │                  ├── Alternative hypothesis (H₁)
        │                  ├── Implicit assumptions
        │                  └── Required statistical test
        ▼
[Experimental Design] ─── Power analysis, sample size, test selection
        │
        ▼
[Confounder Detection] ─── DAG generation, domain reasoning
        │
        ▼
[Simulation Engine] ─── 10,000 parallel runs via Cloud Run
        │                  ├── Bootstrap resampling
        │                  ├── Parametric simulation
        │                  ├── Bayesian posterior sampling
        │                  └── Adversarial datasets
        ▼
[Analysis Layer]
        ├── P-hacking watchdog (multiple comparisons, forking paths)
        ├── Robustness scoring
        ├── Replication probability
        └── Sensitivity analysis
        │
        ▼
[Report Generator] ─── PDF peer-review report, visualizations, honest abstract
```

### 2.2 Hypothesis Parsing Engine

**Input:** Natural language like "I think coffee consumption causes increased longevity in adults over 50"

**Gemini Processing Pipeline:**
1. **Extract variables:** Independent (coffee consumption), Dependent (longevity/mortality), Population (adults >50)
2. **Formalize hypotheses:**
   - H₀: Coffee consumption has no effect on mortality rates in adults over 50
   - H₁: Coffee consumption is associated with reduced mortality rates in adults over 50
3. **Identify implicit assumptions:**
   - Linear dose-response relationship assumed
   - Self-reported consumption assumed accurate
   - No interaction with medication use assumed
   - Survivorship bias: people who live longer drink more coffee because they're alive to drink it
4. **Select statistical framework:** Cox proportional hazards model (survival analysis) with coffee consumption as continuous predictor
5. **Generate testable model specification:**
   ```
   h(t|X) = h₀(t) · exp(β₁·coffee + β₂·age + β₃·smoking + β₄·exercise)
   ```

**Prompt Engineering Pattern:**
```
System: You are a biostatistician. Given a research hypothesis in natural language:
1. Extract IV, DV, population, and study design
2. State H₀ and H₁ formally
3. List ALL implicit assumptions (minimum 5)
4. Recommend the appropriate statistical test with justification
5. Identify the top 5 potential confounders from domain knowledge
6. Rate the hypothesis testability on a 1-10 scale

Output as structured JSON.
```

### 2.3 Synthetic Data Generation Pipeline

Three methods, automatically selected based on data characteristics:

#### A. Bootstrap Resampling (Default)
- **When:** Distribution-free, sufficient sample size (n > 30), no parametric assumptions
- **How:** Draw n samples with replacement from original data, repeat 10,000 times
- **Library:** `scipy.stats.bootstrap` with `method='BCa'` (bias-corrected accelerated)
- **Preserves:** Empirical distribution, correlations between variables, heteroscedasticity

#### B. Parametric Simulation
- **When:** Known generating process, need to extrapolate beyond observed range
- **How:** Fit parametric distribution (normal, gamma, Poisson, etc.) via MLE, generate from fitted parameters
- **Library:** `scipy.stats` distribution fitting + `numpy.random` generation
- **Validation:** Kolmogorov-Smirnov test to verify fit quality before proceeding

#### C. Bayesian Posterior Sampling
- **When:** Small N (< 30), prior knowledge available, hierarchical/multi-site designs
- **How:** Define generative model with priors, MCMC sampling via NUTS, posterior predictive checks
- **Library:** `PyMC v5.x` with `sample_posterior_predictive(samples=10000)`
- **Output:** 10,000 synthetic datasets, each drawn from a different posterior parameter sample

#### D. Adversarial Dataset Generation
- **Purpose:** Actively try to BREAK the hypothesis
- **Methods:**
  - Inject unmeasured confounders with varying effect sizes
  - Simulate measurement error in key variables
  - Introduce selection bias (non-random missingness)
  - Generate datasets where the null hypothesis is true but spurious significance emerges from multiple testing

**Selection Logic:**
```python
def select_simulation_method(data, metadata):
    if metadata.get('prior_knowledge') and len(data) < 30:
        return 'bayesian'
    elif metadata.get('known_distribution'):
        return 'parametric'
    else:
        return 'bootstrap'  # safe default
```

### 2.4 Confounder Detection Engine

**Two-layer approach:**

**Layer 1: AI Reasoning (Gemini)**
- Given the hypothesis, variables, and domain, Gemini generates a ranked list of potential confounders
- Uses world knowledge (e.g., for coffee → longevity: smoking status, socioeconomic status, exercise, genetics, diet)
- Each confounder rated by: plausibility (high/medium/low), measurability, expected direction of bias

**Layer 2: Statistical Testing (DoWhy + EconML)**
- Generate causal DAG from Gemini's confounder list
- Apply backdoor criterion to identify adjustment sets
- Estimate causal effect with and without each confounder
- Sensitivity analysis: how strong would an unmeasured confounder need to be to nullify the result?

```python
# DoWhy integration
from dowhy import CausalModel

model = CausalModel(
    data=df,
    treatment='coffee_consumption',
    outcome='mortality',
    graph=gemini_generated_dag  # DOT format from Gemini
)
estimand = model.identify_effect()
estimate = model.estimate_effect(estimand, method_name="backdoor.linear_regression")

# Refutation: random common cause
refute = model.refute_estimate(estimand, estimate, method_name="random_common_cause")
# Refutation: placebo treatment
refute2 = model.refute_estimate(estimand, estimate, method_name="placebo_treatment_refuter")
```

**Output:** "Alternative explanations ranked by plausibility":
1. Socioeconomic status (high plausibility) — wealthier people drink more coffee AND have better healthcare
2. Healthy user bias (high) — health-conscious people choose coffee over soda
3. Physical activity (medium) — coffee drinkers may be more active
4. Smoking (medium) — historical correlation between coffee and smoking
5. Genetics (low-medium) — CYP1A2 polymorphisms affect both caffeine metabolism and cardiovascular risk

### 2.5 P-Hacking Watchdog

**Four-component detection system:**

1. **Multiple comparisons correction:**
   - Auto-detect number of tests being performed
   - Apply Bonferroni (conservative), Holm (step-down), and Benjamini-Hochberg (FDR) corrections
   - Report: "Your result is significant at p=0.03, but after Bonferroni correction for 12 comparisons, adjusted p=0.36"
   - Library: `statsmodels.stats.multitest.multipletests`

2. **Researcher degrees of freedom analysis:**
   - Enumerate all reasonable analytical choices (outlier criteria, covariate sets, subgroup definitions)
   - Run the analysis under each combination
   - Report: "Of 48 reasonable analytical specifications, 31 (65%) produce p < 0.05 — suggesting the result is specification-dependent"

3. **Garden of forking paths enumeration:**
   - For each decision point in the analysis, branch and compute outcomes
   - Visualize as a decision tree showing which paths lead to significance
   - Compute the "forking-adjusted false positive rate"

4. **Sensitivity analysis:**
   - Vary key parameters (alpha level, outlier threshold, covariate inclusion) and plot how p-value changes
   - Flag results that are "fragile" — significant only under narrow parameter choices

**Null simulation for calibration:**
```python
# Run analysis 10,000 times under H₀ (no real effect)
null_p_values = []
for _ in range(10000):
    null_data = generate_null_dataset(original_data)
    p = run_analysis(null_data, analytical_choices)
    null_p_values.append(p)

observed_false_positive_rate = np.mean(np.array(null_p_values) < 0.05)
# Expected: 0.05; if >> 0.05, the design has inflated Type I error
```

### 2.6 Replication Predictor

**Features used to predict replication probability:**
- Effect size (Cohen's d, η², odds ratio)
- Sample size per group
- p-value (closer to 0.05 = less likely to replicate)
- Number of tested hypotheses (more = less likely)
- Study design (RCT > quasi-experimental > observational)
- Statistical power achieved
- Robustness score from DARWIN's simulation
- Domain (psychology historically worst, physics best)

**Calibration:** Train on Open Science Foundation Reproducibility Project data (100 psychology replications), Many Labs projects, and Camerer et al. economics replications.

**Model:** Logistic regression (interpretable) with the above features. Output: "Estimated replication probability: 73% (95% CI: 61-84%)"

**Key insight from literature:** The single best predictor of replication success is the original p-value — studies with p < 0.001 replicate at ~70%, while p = 0.04 replicates at ~20%.

### 2.7 Literature Integration via NotebookLM API

- Ingest domain-specific papers to inform confounder hypotheses
- Researcher uploads 5-10 key papers in their field
- NotebookLM extracts: known confounders, established effect sizes, methodological debates, common criticisms
- This context feeds into Gemini's confounder reasoning for domain-specific accuracy

### 2.8 Vertex AI Pipelines for Orchestration

- Pipeline DAG: Parse → Design → Simulate (fan-out 100 parallel batches × 100 sims each) → Aggregate (fan-in) → Analyze → Report
- Each simulation batch runs as a containerized KFP component
- Auto-scaling based on simulation complexity

### 2.9 BigQuery for Results Storage

Schema:
```sql
CREATE TABLE simulation_results.runs (
  simulation_run_id STRING,
  study_id STRING,
  iteration INT64,
  p_value FLOAT64,
  effect_size_d FLOAT64,
  ci_low FLOAT64,
  ci_high FLOAT64,
  test_method STRING,
  analytical_specification STRING,
  is_null_simulation BOOL,
  timestamp TIMESTAMP
);
```

---

## SECTION 3: GOOGLE API INTEGRATION PLAN

| API | Purpose | Usage in DARWIN |
|---|---|---|
| **Gemini 1.5 Pro** | Hypothesis parsing, confounder reasoning, peer review generation, statistical interpretation | Core reasoning engine — every analysis request |
| **NotebookLM API** | Domain literature corpus ingestion | Optional — when researcher uploads reference papers |
| **Vertex AI** | ML model hosting (replication predictor) | Hosts the trained logistic regression model |
| **Vertex AI Pipelines** | Orchestrating parallel simulation DAG | Every simulation run (10K sims split into 100 parallel batches) |
| **BigQuery** | Simulation result storage and aggregation | Stores all 10K results per run, SQL aggregation for reporting |
| **Document AI** | Parsing uploaded PDFs, extracting tables/statistics | When researcher uploads existing papers or protocols |
| **Gemini Embeddings** | Semantic matching to published studies | Finds similar published studies for context |
| **Cloud Run** | Stateless simulation worker execution | 100 parallel jobs per simulation run |
| **Cloud Storage** | Dataset and report file storage | All uploads and generated PDF reports |
| **Firebase** | Auth, Firestore (session/progress), Hosting | User management, real-time progress tracking |

### API Call Flow for One Analysis:

1. **Upload** → Cloud Storage (dataset) + Document AI (if PDF)
2. **Parse** → Gemini 1.5 Pro (hypothesis → formal model)
3. **Design** → Gemini (confounder list) + statsmodels (power analysis)
4. **Simulate** → Cloud Run Jobs (100 parallel × 100 sims) → BigQuery (results)
5. **Analyze** → BigQuery SQL (aggregation) + Python (p-hacking detection)
6. **Report** → Gemini (interpretation + honest abstract) → PDF generation
7. **Track** → Firestore (real-time progress) → Firebase Auth (user sessions)

---

## SECTION 4: SIMULATION ENGINE DETAIL

### 4.1 Parallelization Architecture

```
Client triggers simulation
        │
        ▼
Next.js API route → creates Firestore job document
        │
        ▼
Cloud Run Job launched (100 tasks, parallelism=100)
    ├── Task 0: sims 0-99
    ├── Task 1: sims 100-199
    ├── ...
    └── Task 99: sims 9900-9999
        │ (each writes batch results to BigQuery)
        │ (each updates Firestore shard counter)
        ▼
Aggregator Cloud Function (triggered on job completion)
    → BigQuery aggregation queries
    → Writes final results to Firestore
    → Triggers report generation
```

### 4.2 Supported Input Formats

| Format | Parser | Library |
|---|---|---|
| CSV | Node.js (direct) | `papaparse ^5.4.x` |
| Excel (.xlsx/.xls) | Node.js (direct) | `xlsx (SheetJS) ^0.18.x` |
| SPSS (.sav) | Python microservice | `pyreadstat ^1.2.x` |
| R (.rds/.rdata) | Python microservice | `pyreadr ^0.5.x` |
| PDF (papers) | Google API | Document AI Layout Parser |
| JSON | Node.js (direct) | Built-in `JSON.parse` |

### 4.3 Statistical Test Coverage

**Phase 1 (Hackathon MVP):**
- Independent samples t-test
- Paired samples t-test
- One-way ANOVA
- Pearson/Spearman correlation
- Linear regression (OLS)
- Chi-square test of independence

**Phase 2 (Post-hackathon):**
- Logistic regression
- Cox proportional hazards (survival analysis)
- Mixed-effects models
- Bayesian regression
- Mann-Whitney U, Kruskal-Wallis (non-parametric)
- Meta-analysis (forest plots)

### 4.4 Results Aggregation

From 10,000 simulation runs, compute:
- **P-value distribution:** Histogram + proportion below each threshold (0.05, 0.01, 0.001)
- **Effect size distribution:** Mean, median, SD, 95% CI of Cohen's d (or equivalent)
- **Confidence interval coverage:** What % of simulated CIs contain the "true" effect?
- **Power achieved:** What % of simulations detected a true effect (when one exists)?
- **False positive rate:** What % of null simulations produced p < 0.05?

### 4.5 Robustness Score

A single 0-100 number capturing overall result reliability:

```
Robustness = w₁·PowerScore + w₂·ConsistencyScore + w₃·SpecificationScore + w₄·ConfounderScore

Where:
  PowerScore (0-25)        = 25 × (achieved_power / 0.80), capped at 25
  ConsistencyScore (0-25)  = 25 × (% of simulations agreeing with conclusion)
  SpecificationScore (0-25)= 25 × (% of analytical specs giving same result)
  ConfounderScore (0-25)   = 25 × (1 - sensitivity_to_unmeasured_confounders)
```

- **80-100:** Highly robust — likely to replicate
- **60-79:** Moderately robust — some concerns
- **40-59:** Fragile — significant risk of non-replication
- **0-39:** Unreliable — redesign recommended

### 4.6 Honest Abstract Generator

Gemini rewrites the researcher's conclusion with appropriate caveats:

**Input:** "Our study demonstrates that coffee consumption significantly reduces mortality (p=0.03, n=200)"

**DARWIN's Honest Abstract:** "Coffee consumption showed a statistically significant association with reduced mortality in this sample (p=0.03, Cohen's d=0.31, n=200). However, DARWIN simulation analysis indicates: (1) achieved statistical power was 52%, below the recommended 80%; (2) after adjusting for 3 identified confounders, the association becomes non-significant (p=0.14); (3) the result is specification-dependent — only 65% of reasonable analytical choices produce significance; (4) estimated replication probability: 41%. The observed association may reflect socioeconomic confounding rather than a causal effect of coffee consumption."

### 4.7 Auto-Generated Visualizations

1. **P-value distribution histogram** — across 10,000 simulations with significance threshold line
2. **Effect size forest plot** — point estimates and CIs across simulation batches
3. **Robustness gauge** — circular meter showing 0-100 score
4. **Replication probability dial** — percentage with confidence band
5. **Causal DAG network** — nodes = variables, edges = causal relationships, red = confounders
6. **Specification curve** — all analytical specifications ranked by p-value
7. **Power curve** — power vs. sample size with current study marked
8. **Sensitivity tornado plot** — how each parameter change affects the conclusion

---

## SECTION 5: FRONTEND ARCHITECTURE

### 5.1 Tech Stack

```json
{
  "next": "^14.2.x",
  "react": "^18.3.x",
  "typescript": "^5.4.x",
  "tailwindcss": "^3.4.x",
  "firebase": "^10.12.x",
  "recharts": "^2.12.x",
  "d3": "^7.9.x",
  "d3-dag": "^1.1.x",
  "xlsx": "^0.18.5",
  "papaparse": "^5.4.1",
  "@react-pdf/renderer": "^3.4.x",
  "react-dropzone": "^14.2.x",
  "framer-motion": "^11.x"
}
```

### 5.2 Page Structure

| Route | Component | Description |
|---|---|---|
| `/` | Landing page | Hero, demo video, key stats about reproducibility crisis |
| `/login` | Firebase Google Auth | Single sign-on |
| `/dashboard` | Main dashboard | List of past analyses, quick stats |
| `/dashboard/upload` | Upload wizard | Drag-drop data + hypothesis input + context fields |
| `/dashboard/simulations/[id]` | Live simulation | Progress bar, live stats, streaming results |
| `/dashboard/reports/[id]` | Results & report | Full visualization suite, PDF download, peer review |

### 5.3 Upload Interface

- **Step 1:** Drag-and-drop zone (react-dropzone) — accepts CSV, Excel, SPSS, R, PDF
- **Step 2:** Data preview table — first 10 rows, column types auto-detected
- **Step 3:** Hypothesis input — free-text field: "What do you think this data shows?"
- **Step 4:** Context fields — domain (dropdown), study type (RCT/observational/etc.), sample size
- **Step 5:** Review & launch simulation

### 5.4 Analysis Dashboard

- **Progress bar:** Animated, shows X/10,000 completed, estimated time remaining
- **Live statistics:** Running p-value mean, current effect size distribution (updating chart)
- **Status indicators:** Green/yellow/red for power, false positive rate, robustness
- **SSE streaming** from `/api/simulations/[id]/stream` route handler

### 5.5 Results Visualization

All charts rendered client-side:
- **Recharts:** P-value histogram, power curve, specification curve
- **D3 + d3-dag:** Causal DAG network graph (Sugiyama layout)
- **Recharts RadialBarChart:** Robustness gauge, replication probability dial
- **Custom D3:** Forest plot (no library has this built-in)

### 5.6 Peer Review Mode

Displays simulated reviewer objections:
- "Reviewer 1 would ask: Did you control for socioeconomic status?"
- "Reviewer 2 would note: Your sample size provides only 52% power"
- "Reviewer 3 would request: Sensitivity analysis for unmeasured confounders"
- Each objection linked to DARWIN's analysis that supports it

### 5.7 Report Export

- `@react-pdf/renderer` generates academic-format A4 PDF
- Sections: Abstract, Methods, Results, Robustness Analysis, Limitations, Recommendations
- Charts embedded as pre-rendered PNG images
- Downloadable + shareable via unique URL

---

## SECTION 6: COMPLETE FILE & FOLDER STRUCTURE

```
darwin/
├── README.md
├── package.json
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── .env.local                          # Firebase + GCP keys
├── .env.example
├── .gitignore
│
├── public/
│   ├── favicon.ico
│   └── images/
│       └── logo.svg
│
├── src/
│   ├── app/
│   │   ├── layout.tsx                  # Root layout (fonts, metadata)
│   │   ├── page.tsx                    # Landing page
│   │   ├── login/
│   │   │   └── page.tsx                # Firebase Auth login
│   │   ├── dashboard/
│   │   │   ├── layout.tsx              # Dashboard shell (sidebar, nav)
│   │   │   ├── page.tsx                # Dashboard home (analysis list)
│   │   │   ├── upload/
│   │   │   │   └── page.tsx            # Upload wizard
│   │   │   ├── simulations/
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx        # Live simulation view
│   │   │   └── reports/
│   │   │       └── [id]/
│   │   │           └── page.tsx        # Results & report view
│   │   └── api/
│   │       ├── upload/
│   │       │   └── route.ts            # File upload handler
│   │       ├── simulations/
│   │       │   ├── start/
│   │       │   │   └── route.ts        # POST: trigger simulation
│   │       │   └── [id]/
│   │       │       └── stream/
│   │       │           └── route.ts    # GET: SSE progress stream
│   │       ├── hypothesis/
│   │       │   └── route.ts            # POST: Gemini hypothesis parsing
│   │       └── report/
│   │           └── [id]/
│   │               └── route.ts        # GET: generate PDF
│   │
│   ├── components/
│   │   ├── ui/                         # Shared UI primitives
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── ProgressBar.tsx
│   │   │   ├── Badge.tsx
│   │   │   └── Modal.tsx
│   │   ├── upload/
│   │   │   ├── FileDropzone.tsx        # Drag-and-drop upload
│   │   │   ├── DataPreview.tsx         # Table preview of uploaded data
│   │   │   └── HypothesisInput.tsx     # Natural language hypothesis form
│   │   ├── simulation/
│   │   │   ├── ProgressPanel.tsx       # Real-time progress (SSE client)
│   │   │   ├── LiveStats.tsx           # Running statistics display
│   │   │   └── SimulationCard.tsx      # Summary card for dashboard list
│   │   ├── charts/
│   │   │   ├── PValueHistogram.tsx     # Recharts histogram
│   │   │   ├── CausalDAG.tsx           # D3 + d3-dag network
│   │   │   ├── RobustnessGauge.tsx     # Recharts radial bar
│   │   │   ├── ForestPlot.tsx          # Custom D3 forest plot
│   │   │   ├── PowerCurve.tsx          # Recharts line chart
│   │   │   ├── SpecificationCurve.tsx  # Recharts dot plot
│   │   │   └── ReplicationDial.tsx     # Gauge chart
│   │   ├── report/
│   │   │   ├── PeerReviewPanel.tsx     # AI-generated reviewer objections
│   │   │   ├── HonestAbstract.tsx      # Rewritten abstract display
│   │   │   └── DarwinReport.tsx        # @react-pdf/renderer template
│   │   └── layout/
│   │       ├── Sidebar.tsx
│   │       ├── Navbar.tsx
│   │       └── Footer.tsx
│   │
│   ├── lib/
│   │   ├── firebase.ts                 # Client-side Firebase init
│   │   ├── firebase-admin.ts           # Server-side Firebase Admin
│   │   ├── gemini.ts                   # Gemini API client wrapper
│   │   ├── parser.ts                   # File parsing utilities (CSV, Excel)
│   │   ├── bigquery.ts                 # BigQuery client helpers
│   │   ├── storage.ts                  # Cloud Storage upload/download
│   │   └── constants.ts                # App-wide constants
│   │
│   ├── hooks/
│   │   ├── useSimulation.ts            # SSE hook for simulation progress
│   │   ├── useAuth.ts                  # Firebase auth state hook
│   │   └── useFileUpload.ts            # Upload state management
│   │
│   └── types/
│       ├── simulation.ts               # Simulation types/interfaces
│       ├── hypothesis.ts               # Hypothesis parsing types
│       └── report.ts                   # Report generation types
│
├── services/
│   └── simulation-engine/              # Python simulation backend
│       ├── Dockerfile
│       ├── requirements.txt
│       ├── main.py                     # FastAPI entry point
│       ├── engine/
│       │   ├── __init__.py
│       │   ├── bootstrap.py            # Bootstrap resampling
│       │   ├── parametric.py           # Parametric simulation
│       │   ├── bayesian.py             # PyMC posterior sampling
│       │   ├── adversarial.py          # Adversarial dataset gen
│       │   ├── statistical_tests.py    # All supported tests
│       │   ├── power_analysis.py       # Power computations
│       │   ├── p_hacking.py            # P-hacking detection
│       │   ├── confounder.py           # DoWhy integration
│       │   ├── robustness.py           # Robustness score calc
│       │   └── replication.py          # Replication predictor
│       ├── parsers/
│       │   ├── __init__.py
│       │   ├── spss_parser.py          # pyreadstat
│       │   └── r_parser.py             # pyreadr
│       └── worker.py                   # Cloud Run Job worker entry point
│
├── fixtures/
│   ├── demo-coffee-longevity.csv       # Demo dataset
│   ├── demo-drug-efficacy.csv          # Demo dataset
│   ├── demo-education-income.csv       # Demo dataset
│   └── mock-simulation-results.json    # Pre-generated results for demo
│
└── docs/
    └── api-spec.md                     # Internal API documentation
```

---

## SECTION 7: STATISTICAL METHODOLOGY

### 7.1 Synthetic Data Generation — Formal Methods

**Bootstrap (Efron, 1979):**
- Non-parametric: resample with replacement from empirical distribution F̂
- BCa (bias-corrected accelerated) for confidence intervals
- Preserves all higher-order moments and inter-variable correlations

**Parametric simulation:**
- Fit via MLE: θ̂ = argmax L(θ|data)
- Validate fit: KS test, Q-Q plot, AIC/BIC comparison
- Generate: X_sim ~ F(θ̂)
- For multivariate: fit copula to capture dependency structure separately from marginals

**Bayesian posterior predictive:**
- Prior: π(θ) — informative or weakly informative
- Posterior: π(θ|data) ∝ L(data|θ) × π(θ)
- Posterior predictive: p(x_new|data) = ∫ p(x_new|θ) π(θ|data) dθ
- Sampled via MCMC (NUTS in PyMC)

### 7.2 Causal Inference Framework

- **Pearl's do-calculus** (Pearl, 2009): Three rules for reducing interventional queries to observational estimates
- **Backdoor adjustment:** P(Y|do(X)) = Σ_z P(Y|X,Z) P(Z) when Z satisfies backdoor criterion
- **Front-door criterion:** For cases where backdoor is blocked by unmeasured confounders
- **Instrumental variables:** When neither backdoor nor front-door applies
- **Implementation:** DoWhy library handles identification + estimation + refutation

### 7.3 Power Analysis

- Computed via `statsmodels.stats.power` for all supported test types
- DARWIN recommends minimum sample size for 80% power at α=0.05
- Shows power curve: power vs. n for the observed effect size
- Flags underpowered studies (power < 0.80) prominently

### 7.4 Pre-Registration Output

DARWIN generates an OSF-compatible pre-registration document:
- Hypotheses (H₀, H₁)
- Study design and variables
- Statistical analysis plan
- Sample size justification (power analysis)
- Exclusion criteria
- Multiple testing correction plan
- Exportable as PDF or JSON (OSF format)

### 7.5 Integration with R/Python

- Python: scipy, statsmodels, PyMC, DoWhy, EconML — all used natively in the simulation engine
- R: Not directly integrated in MVP; future phase could call R via `rpy2` for specialized packages (lme4, brms, metafor)

### 7.6 Honest Limitations

DARWIN **cannot**:
- Replace domain expertise in choosing the right model
- Detect confounders that are completely unknown to science
- Fix fundamentally flawed study designs (it can only flag them)
- Handle qualitative research methods
- Guarantee replication (it predicts probability, not certainty)
- Replace institutional IRB review or clinical trial protocols
- Analyze proprietary data formats without parser support

---

## SECTION 8: HACKATHON DEMO PLAN

### 8.1 Pre-Run: Three Famous Flawed Studies

Pre-compute DARWIN results for:

1. **Daryl Bem's "Feeling the Future" (2011)** — Claimed precognition was real (p < 0.05). DARWIN would show: specification-dependent, low robustness, p-hacking signatures in the p-value distribution. Replication probability: 12%.

2. **Power Posing (Carney, Cuddy & Yap, 2010)** — "Standing like Superman raises testosterone." Failed to replicate in 2015. DARWIN would flag: small n (42), effect size sensitivity to outlier exclusion, underpowered design (power ~35%).

3. **Chocolate accelerates weight loss (Bohannon, 2015)** — Deliberately p-hacked study published to expose flawed peer review. DARWIN would flag: 18 measured outcomes, no multiple comparisons correction → expected 0.9 false positives.

### 8.2 Live Demo Script (5 minutes)

1. **[0:00-0:30]** "Science has a $28 billion problem." Show the crisis stats.
2. **[0:30-1:30]** Open DARWIN. Upload `demo-coffee-longevity.csv`. Type: "Coffee consumption increases lifespan."
3. **[1:30-3:00]** Watch the simulation run — progress bar filling, live p-value histogram updating, robustness score changing in real-time.
4. **[3:00-4:00]** Results: Show the causal DAG (socioeconomic confounders highlighted), the honest abstract, the peer review objections.
5. **[4:00-5:00]** "DARWIN just did what takes a statistician two weeks — in 30 seconds. And it's honest about what the data actually shows."

### 8.3 Emotional Arc

- **Hook:** "What if we could stop bad science before it starts?"
- **Tension:** Watch a "significant" p=0.03 result dissolve as confounders are added
- **Resolution:** DARWIN's recommendations: "Increase sample to n=300, control for SES, pre-register your analysis plan"
- **Inspiration:** "One corrected study at a time"

### 8.4 Constructive Ending

Show how DARWIN's recommendations would have:
- Saved Vioxx ($4.85B settlement) by flagging cardiovascular confounders
- Prevented the Wakefield vaccine paper (exposed p-hacking + confounder issues)
- Caught the Cornell Food Lab fabrications (specification curve would show cherry-picking)

---

## SECTION 9: 24-HOUR BUILD SPRINT

### Hour-by-Hour Plan

| Hours | Deliverable | Owner | Risk Level |
|---|---|---|---|
| 0-1 | Project scaffolding: Next.js + Tailwind + Firebase + TypeScript | Frontend | Low |
| 1-2 | Firebase Auth (Google Sign-In) + protected routes | Frontend | Low |
| 2-3 | Python simulation engine: FastAPI scaffolding + bootstrap.py | Backend | Low |
| 3-5 | File upload: dropzone UI + CSV/Excel parsing + data preview | Frontend | Low |
| 5-7 | Hypothesis input UI + Gemini API integration (hypothesis → formal model) | Full-stack | Medium |
| 7-9 | Simulation engine: t-test + ANOVA + regression across bootstrap samples | Backend | Medium |
| 9-11 | SSE streaming: Cloud Run worker → Firestore → SSE → Progress UI | Full-stack | High |
| 11-13 | P-value histogram + robustness gauge + replication dial (Recharts) | Frontend | Medium |
| 13-15 | Causal DAG visualization (D3 + d3-dag) — hardcoded for demo datasets | Frontend | Medium |
| 15-17 | P-hacking detection: multiple comparisons + specification analysis | Backend | Medium |
| 17-19 | Report generation: honest abstract (Gemini) + peer review mode + PDF | Full-stack | Medium |
| 19-21 | End-to-end integration testing + bug fixes | Full-stack | High |
| 21-23 | UI polish: animations, loading states, error handling, responsive | Frontend | Low |
| 23-24 | Demo rehearsal, backup recording, slide deck | All | Low |

### Critical Path & Fallback Cuts

**If behind by hour 12 (no streaming working):**
- **Cut:** Real Cloud Run parallelism → Use mock simulation with `setTimeout` delays
- **Cut:** SPSS/R parsing → CSV-only
- **Keep:** Gemini hypothesis parsing (core differentiator)
- **Keep:** At least one chart (p-value histogram)

**If behind by hour 18:**
- **Cut:** Causal DAG → Replace with text list of confounders
- **Cut:** Forest plot → Simple table
- **Cut:** PDF report → On-screen results only
- **Keep:** Upload → hypothesis → simulation progress → results flow

**Nuclear fallback (only upload + results):**
- Pre-computed results for 3 demo datasets
- User uploads CSV, selects hypothesis from dropdown
- Results display immediately from pre-computed JSON
- Still demonstrates the concept convincingly

---

## SECTION 10: PRODUCTION & SCALE

### Cost Per Analysis

| Component | Cost per 10K simulations | Notes |
|---|---|---|
| Cloud Run (100 tasks × 2 vCPU × ~2 min) | ~$0.30 | Pay-per-use |
| BigQuery (10K rows write + queries) | ~$0.05 | First 1TB/month free |
| Gemini 1.5 Pro (3-4 calls) | ~$0.15 | Input + output tokens |
| Cloud Storage (dataset + report) | ~$0.01 | Negligible |
| **Total per analysis** | **~$0.50** | At scale |

### Pricing Model

- **Free tier:** 5 analyses/month (researcher adoption)
- **Academic:** $29/month unlimited (university email required)
- **Team:** $99/month per seat (pharma R&D, labs)
- **Enterprise:** Custom pricing (journal integrations, API access)

### Institutional Integration

- **University IRB systems:** Generate pre-registration documents compatible with IRB submission requirements
- **Journal submission APIs:** Future integration with ScholarOne, Editorial Manager for pre-submission validation
- **OSF integration:** Direct export of pre-registration documents to Open Science Framework

---

## SECTION 11: TECHNICAL RISKS & MITIGATIONS

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Simulation cost overrun (user submits huge dataset) | Medium | High | Cap dataset size at 100K rows; sample for simulation |
| Gemini hallucinating statistical interpretations | Medium | Critical | Validate all Gemini outputs against computed statistics; never let Gemini invent numbers |
| Replication predictor overfitting (small training set) | High | Medium | Use simple logistic regression, cross-validate, report wide confidence intervals |
| Non-standard data formats | Medium | Low | Start CSV-only, add formats incrementally |
| Cloud Run cold start latency | Low | Medium | Keep minimum instances warm; show "preparing" state |
| Firebase Firestore write limits (10K rapid writes) | Medium | Medium | Use sharded counters (10 shards), batch writes |
| User uploads sensitive/private data | High | Critical | All data encrypted at rest, auto-delete after 30 days, HIPAA-compliant option for pharma |

---

## SECTION 12: BUSINESS MODEL

### Revenue Streams

1. **SaaS subscriptions** (primary): Academic, Team, Enterprise tiers
2. **Journal partnerships:** Per-submission validation fee ($5-10/paper), paid by journal or author
3. **Pharma R&D contracts:** Custom integration + dedicated support + compliance features
4. **Grant funding:** NIH/NSF grants for improving research reproducibility

### Market Size

- 8 million researchers worldwide
- ~2.5 million papers published per year
- Even 1% adoption = 25,000 analyses/year = $750K ARR at academic pricing
- Pharma preclinical market: $56B/year spend, even 0.01% efficiency gain = $5.6M value

### Competitive Moat

- **Network effects:** More analyses → better replication predictor calibration
- **Domain knowledge accumulation:** Each analysis improves confounder detection
- **Pre-registration standard:** If DARWIN becomes the standard pre-registration tool, switching costs are high

---

## SECTION 13: JUDGING CRITERIA ALIGNMENT

### For Google Gemini API Hackathon

| Criterion | DARWIN's Strength |
|---|---|
| **Gemini API usage** | Core to hypothesis parsing, confounder reasoning, report generation, peer review |
| **Innovation** | First AI system that simulates experiments before they run — entirely new category |
| **Impact** | Addresses $28B/year problem, improves scientific integrity globally |
| **Technical complexity** | Parallel simulation, causal inference, Bayesian methods, real-time streaming |
| **Completeness** | End-to-end: upload → parse → simulate → analyze → report → export |
| **Demo quality** | Emotional arc, live simulation, debunking famous studies |

### Pitch Structure (3 minutes)

1. **[0:00-0:30] The Problem:** "$28 billion wasted annually on irreproducible research. 64% of studies don't replicate."
2. **[0:30-1:00] The Insight:** "What if you could run your experiment 10,000 times before running it once?"
3. **[1:00-2:00] The Demo:** Live upload → simulation → results dissolving under scrutiny
4. **[2:00-2:30] The Technology:** "Gemini reasons about confounders. Cloud Run simulates in parallel. DARWIN catches what peer reviewers miss."
5. **[2:30-3:00] The Vision:** "One corrected study at a time, DARWIN prevents the next reproducibility crisis."

---

## SECTION 14: APPENDICES

### A. Complete Tech Stack

**Frontend:** Next.js 14, React 18, TypeScript 5, Tailwind CSS 3.4, Recharts 2.12, D3.js 7.9, d3-dag 1.1, Framer Motion 11, react-dropzone 14.2, @react-pdf/renderer 3.4

**Backend:** Python 3.11, FastAPI, scipy 1.17, statsmodels, PyMC 5.x, DoWhy, EconML, pyreadstat, pyreadr, numpy, pandas

**Infrastructure:** Firebase (Auth, Firestore, Hosting), Cloud Run, Cloud Storage, BigQuery, Vertex AI Pipelines, Document AI, Gemini 1.5 Pro API

**DevOps:** Docker, Google Cloud Build, GitHub Actions

### B. Sample Simulation Output Schema

```json
{
  "study_id": "darwin_abc123",
  "hypothesis": {
    "original": "Coffee consumption increases lifespan",
    "null": "Coffee consumption has no effect on mortality",
    "alternative": "Coffee consumption is associated with reduced mortality",
    "test": "cox_proportional_hazards",
    "variables": {
      "independent": "coffee_cups_per_day",
      "dependent": "years_survived",
      "covariates": ["age", "smoking", "exercise", "ses"]
    }
  },
  "simulation_summary": {
    "n_simulations": 10000,
    "method": "bootstrap",
    "duration_seconds": 47,
    "p_value_distribution": {
      "mean": 0.087,
      "median": 0.062,
      "below_005": 0.42,
      "below_001": 0.11
    },
    "effect_size_distribution": {
      "mean_cohens_d": 0.28,
      "sd": 0.14,
      "ci_95": [0.01, 0.55]
    }
  },
  "robustness_score": 47,
  "replication_probability": 0.38,
  "confounders_detected": [
    { "name": "Socioeconomic status", "plausibility": "high", "bias_direction": "positive" },
    { "name": "Healthy user bias", "plausibility": "high", "bias_direction": "positive" },
    { "name": "Physical activity", "plausibility": "medium", "bias_direction": "positive" }
  ],
  "p_hacking_analysis": {
    "specifications_tested": 48,
    "specifications_significant": 31,
    "forking_adjusted_fpr": 0.12,
    "multiple_comparisons": {
      "bonferroni_adjusted_p": 0.36,
      "bh_adjusted_p": 0.14
    }
  },
  "recommendations": [
    "Increase sample size to n≥300 for 80% power",
    "Control for socioeconomic status (strong confounder)",
    "Pre-register analysis plan to prevent specification flexibility",
    "Consider instrumental variable approach to address healthy user bias"
  ],
  "honest_abstract": "Coffee consumption showed a marginally significant association..."
}
```

### C. Sample DARWIN Report Structure (PDF)

1. **Title Page:** Study title, DARWIN analysis ID, date, robustness score badge
2. **Executive Summary:** 3-sentence honest assessment
3. **Hypothesis Analysis:** Formal hypotheses, assumptions identified, statistical framework
4. **Simulation Results:** P-value distribution, effect size distribution, power analysis
5. **Confounder Analysis:** Causal DAG, confounder rankings, sensitivity analysis
6. **Robustness Assessment:** Specification curve, robustness score breakdown
7. **P-Hacking Risk:** Multiple comparisons analysis, forking paths enumeration
8. **Replication Forecast:** Predicted probability with calibration context
9. **Recommendations:** Actionable steps to strengthen the study
10. **Appendix:** Full statistical details, simulation parameters, software versions

### D. Key Statistical References

- Efron, B. (1979). Bootstrap methods: another look at the jackknife. *Annals of Statistics*
- Pearl, J. (2009). *Causality: Models, Reasoning, and Inference*. Cambridge University Press
- Gelman, A. & Loken, E. (2013). The garden of forking paths. Columbia University
- Open Science Collaboration (2015). Estimating reproducibility of psychological science. *Science*
- Freedman, L.P. et al. (2015). The economics of reproducibility in preclinical research. *PLOS Biology*
- Simmons, J.P. et al. (2011). False-positive psychology. *Psychological Science*
- Benjamin, D.J. et al. (2018). Redefine statistical significance. *Nature Human Behaviour*

---

## IMPLEMENTATION PRIORITY ORDER

For the hackathon build, implement in this exact order:

1. **Next.js scaffolding + Firebase Auth** (must work first)
2. **File upload + CSV parsing + data preview** (core interaction)
3. **Gemini hypothesis parsing API** (core differentiator)
4. **Mock simulation with SSE progress streaming** (wow factor)
5. **P-value histogram visualization** (proof of statistical output)
6. **Robustness score + replication probability gauges** (visual impact)
7. **Causal DAG visualization** (scientific credibility)
8. **Honest abstract + peer review panel** (emotional impact)
9. **PDF report generation** (completeness)
10. **Real simulation engine** (if time permits — mock is fine for demo)

---

## VERIFICATION PLAN

### How to Test End-to-End

1. **Auth:** Sign in with Google, verify protected routes redirect unauthenticated users
2. **Upload:** Drop `demo-coffee-longevity.csv`, verify data preview table renders correctly
3. **Hypothesis:** Type "Coffee increases lifespan", verify Gemini returns structured JSON with H₀, H₁, assumptions
4. **Simulation:** Click "Run Analysis", verify progress bar streams from 0-100%, verify Firestore document updates
5. **Results:** Verify all 7 chart types render with data, verify robustness score computes correctly
6. **Report:** Click "Download PDF", verify PDF opens with all sections populated
7. **Pre-computed demos:** Verify all 3 famous studies load with pre-computed results

### Smoke Tests

```bash
# Frontend
npm run build          # Verify no build errors
npm run lint           # Verify no lint errors

# Backend
cd services/simulation-engine
pip install -r requirements.txt
python -m pytest       # Run unit tests for statistical functions

# Integration
curl -X POST http://localhost:3000/api/hypothesis -d '{"text":"Coffee increases lifespan"}'
# Verify 200 response with structured hypothesis JSON
```