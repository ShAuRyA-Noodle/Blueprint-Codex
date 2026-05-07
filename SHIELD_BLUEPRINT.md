## Context

**What:** SHIELD is an autonomous red-team AI platform that continuously attacks your own infrastructure, discovers vulnerabilities before malicious hackers, auto-generates patches, deploys them, and re-attacks to verify the fix — all with minimal human intervention.

**Why:** Cybercrime costs $10.5T annually. Average breach detection takes 287 days. Human red teams are expensive ($50K-$500K per engagement), infrequent (1-2x/year), and cover only 15-30% of the attack surface. Existing tools (Burp Suite, Nessus, Metasploit) rely on pattern matching — they cannot reason about novel attack chains or business logic flaws.

**Goal:** Build a production-grade hackathon prototype demonstrating the full discover → attack → patch → verify loop using Google Cloud APIs (Gemini, Vertex AI, Cloud Run, Cloud Build, BigQuery).

---

# SHIELD: Autonomous Red-Team AI Platform -- Complete Technical Blueprint

**Version:** 1.0.0 | **Date:** 2026-03-22 | **Classification:** Authorized Defensive Security Testing Only

---

## SECTION 1: PROBLEM DEFINITION AND VISION

### 1.1 The Scale of the Problem

Global cybercrime costs are projected to reach $10.5 trillion annually by 2025 (Cybersecurity Ventures), growing at 15% year-over-year. This figure exceeds the GDP of every country except the United States and China. The average organization takes **287 days** to identify and contain a data breach (IBM Cost of a Data Breach Report), during which attackers move laterally, exfiltrate data, and establish persistence.

### 1.2 Why Human Red Teams Are Insufficient

| Factor | Human Red Teams | SHIELD |
|--------|----------------|--------|
| Cost | $50K-$500K per engagement | Continuous for monthly subscription |
| Frequency | 1-2x per year | 24/7/365 |
| Coverage | 15-30% of attack surface per engagement | Entire attack surface continuously |
| Consistency | Variable by team skill | Deterministic + AI-augmented |
| Speed to remediation | Weeks for report, months for fixes | Minutes to hours |
| Novel attack chains | Limited by individual experience | Combinatorial reasoning across all known attack primitives |

### 1.3 Where Existing Tools Fail

**Burp Suite, Nessus, Metasploit, Qualys** -- these tools rely on **pattern matching and signature databases**. They scan for known CVEs and known vulnerability patterns. They cannot:

- Reason about business logic flaws unique to your application
- Chain multiple low-severity findings into a critical attack path
- Understand the semantic meaning of code to find zero-day-class issues
- Generate contextual patches that respect your codebase architecture
- Verify that a remediation actually closes the vulnerability

### 1.4 The Core Insight

SHIELD represents a paradigm shift: **an AI that THINKS like an attacker rather than running scripts**. Using Gemini's reasoning capabilities, SHIELD:

1. **Discovers** -- Maps the entire attack surface automatically
2. **Reasons** -- Hypothesizes novel attack chains by combining known primitives
3. **Attacks** -- Executes attacks in sandboxed environments safely
4. **Patches** -- Generates exact code fixes using Vertex AI Code
5. **Deploys** -- Applies patches through automated CI/CD pipelines
6. **Verifies** -- Re-attacks to confirm the vulnerability is closed
7. **Learns** -- Feeds results back to improve future attack reasoning

### 1.5 Autonomous Remediation Vision

The complete loop: `Discover -> Reason -> Attack -> Confirm Vulnerability -> Generate Patch -> Review Patch -> Deploy to Staging -> Test -> Deploy to Production -> Re-attack -> Confirm Fix -> Report`

Human intervention points (configurable):
- **Fully Autonomous**: Only patches that pass all automated checks deploy automatically
- **Semi-Autonomous**: Human approves patches above a configurable CVSS threshold
- **Supervised**: Human approves all patches; SHIELD provides recommendations

---

## SECTION 2: CORE TECHNICAL ARCHITECTURE

### 2.1 System Architecture Overview

```
                              +---------------------------+
                              |     SHIELD Control Plane   |
                              |  (Next.js Frontend + API)  |
                              +------------+--------------+
                                           |
                              +------------+--------------+
                              |     Orchestrator Agent     |
                              |  (Cloud Run - Primary)     |
                              +---+----+----+----+----+---+
                                  |    |    |    |    |
              +-------------------+    |    |    |    +-------------------+
              |                        |    |    |                        |
    +---------v--------+   +-----------v--+ |  +-v-----------+  +--------v---------+
    | Attack Surface   |   | Recon Agent  | |  | Vuln Hypo   |  | Zero-Day         |
    | Discovery Engine |   | (Passive     | |  | Generator   |  | Predictor        |
    | (Asset Inventory)|   |  Gathering)  | |  | (Gemini     |  | (Code Pattern    |
    +--------+---------+   +------+-------+ |  |  Reasoning) |  |  Analysis)       |
             |                    |         |  +------+------+  +--------+---------+
             |                    |         |         |                   |
             +--------------------+---------+---------+-------------------+
                                            |
                              +-------------v--------------+
                              |   Attack Chain Simulator    |
                              |  (Combines findings into    |
                              |   multi-step attack paths)  |
                              +-------------+--------------+
                                            |
                              +-------------v--------------+
                              | Sandboxed Attack Execution  |
                              | (Isolated Cloud Run         |
                              |  containers per attack)     |
                              +-------------+--------------+
                                            |
                              +-------------v--------------+
                              |  Patch Generation Engine    |
                              |  (Vertex AI Code Generation)|
                              +-------------+--------------+
                                            |
                              +-------------v--------------+
                              |  Automated Deploy Pipeline  |
                              |  (Cloud Build + Cloud       |
                              |   Deploy staged rollout)    |
                              +-------------+--------------+
                                            |
                              +-------------v--------------+
                              |   Verification Loop         |
                              |  (Re-attack post-patch)     |
                              +-------------+--------------+
                                            |
                              +-------------v--------------+
                              |   Reporting & Analytics     |
                              |  (BigQuery + Dashboard)     |
                              +----------------------------+
```

### 2.2 Component Specifications

#### 2.2.1 Attack Surface Discovery Engine

**Purpose**: Automated asset inventory and continuous discovery of all endpoints, services, APIs, and infrastructure components.

**Implementation**:
- Enumerates all Cloud Run services, GKE deployments, Cloud Functions, App Engine services via GCP Resource Manager API
- Crawls web applications to discover routes, forms, API endpoints
- Parses OpenAPI/Swagger specifications automatically
- Monitors DNS records, SSL certificates, and subdomains
- Tracks changes to the attack surface over time

**Key data structure** (`/packages/agents/discovery/types.ts`):
```typescript
interface AttackSurface {
  id: string;
  discoveredAt: string; // ISO 8601
  assets: Asset[];
  endpoints: Endpoint[];
  services: ServiceInfo[];
  exposedPorts: PortInfo[];
  technologies: TechStack[];
  changesSinceLastScan: SurfaceDelta[];
}

interface Asset {
  id: string;
  type: 'web_app' | 'api' | 'database' | 'storage_bucket' | 'cloud_function' | 'vm_instance' | 'container';
  url: string;
  discoveryMethod: 'dns_enum' | 'port_scan' | 'api_crawl' | 'gcp_inventory' | 'certificate_transparency';
  confidence: number; // 0-1
  metadata: Record<string, unknown>;
}

interface Endpoint {
  id: string;
  assetId: string;
  method: 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH' | 'OPTIONS';
  path: string;
  parameters: Parameter[];
  authentication: AuthType;
  rateLimit: RateLimitInfo | null;
  responseSchema: JSONSchema | null;
}
```

#### 2.2.2 Reconnaissance Agent

**Purpose**: Passive information gathering without sending traffic to the target. Builds a comprehensive target profile.

**Techniques**:
- WHOIS lookups and DNS record enumeration
- Certificate Transparency log analysis
- Public code repository scanning (exposed secrets, API keys)
- Technology fingerprinting from HTTP headers and response patterns
- Publicly disclosed vulnerabilities for identified tech stack versions
- Google dorking equivalent queries for information leakage

#### 2.2.3 Vulnerability Hypothesis Generator

**Purpose**: Uses Gemini to REASON about potential vulnerabilities rather than pattern-matching known signatures.

**Process**:
1. Receives complete target profile from Discovery + Recon
2. Gemini analyzes the tech stack, architecture, and endpoints
3. Generates hypotheses: "Given that this application uses Express.js 4.x with MongoDB and has an unparameterized search endpoint at `/api/search?q=`, there is a high probability of NoSQL injection"
4. Ranks hypotheses by likelihood and potential impact
5. Generates specific attack payloads to test each hypothesis

#### 2.2.4 Sandboxed Attack Execution Environment

See Section 5 for full details. Summary: Each attack executes in an isolated Cloud Run container with no access to production data. Attacks target a mirrored/shadow copy of the application.

#### 2.2.5 Attack Chain Simulator

**Purpose**: The most novel component. Chains multiple low-severity findings into critical attack paths.

**Example chain**:
1. Information disclosure on `/api/health` reveals internal service names (Low)
2. IDOR on `/api/users/{id}/profile` allows reading any user profile (Medium)
3. Profile contains API key for internal service discovered in step 1 (Medium)
4. Internal service has no authentication when called directly (High)
5. Combined: Unauthenticated access to internal service data (Critical)

**Implementation**: Gemini receives all individual findings and is prompted to find combinations that create escalation paths. Uses a directed graph model where nodes are vulnerabilities and edges are exploitation steps.

#### 2.2.6 Patch Generation Engine

See Section 6. Uses Vertex AI code generation to produce exact, contextual patches.

#### 2.2.7 Automated Deployment Pipeline

Cloud Build triggers on patch generation, runs through `staging -> test -> production` with automatic rollback on failure.

#### 2.2.8 Verification Loop

After patch deployment, SHIELD re-executes the exact same attack that found the vulnerability. If the attack succeeds, the patch is flagged as insufficient and a new patch iteration begins.

#### 2.2.9 Zero-Day Predictor

**Purpose**: Analyzes code patterns to predict vulnerabilities BEFORE they are exploited.

**Approach**:
- Static analysis of code patterns known to lead to vulnerabilities
- Comparison against a database of code patterns from historical CVEs
- Gemini reasons about code semantics: "This function takes user input, passes it through a transformation, but does not sanitize it before database query construction"
- Tracks dependency vulnerabilities and predicts which of your dependencies are likely to have undiscovered issues based on code complexity metrics

### 2.3 Data Flow

```
Discovery -> [Asset DB] -> Recon -> [Target Profile DB]
  -> Hypothesis Generator -> [Hypothesis Queue]
  -> Attack Executor -> [Finding DB]
  -> Chain Simulator -> [Attack Chain DB]
  -> Patch Generator -> [Patch Queue]
  -> Deploy Pipeline -> [Deployment DB]
  -> Verification Loop -> [Verification DB]
  -> Reporting Engine -> [BigQuery Analytics]
```

All data flows through **Pub/Sub topics** for loose coupling and reliability. Each component publishes results to its output topic and subscribes to its input topic.

**Pub/Sub Topics**:
- `shield.discovery.assets` -- New/updated assets discovered
- `shield.recon.profiles` -- Completed target profiles
- `shield.hypotheses.generated` -- New vulnerability hypotheses
- `shield.attacks.results` -- Attack execution results
- `shield.chains.detected` -- Multi-step attack chains found
- `shield.patches.generated` -- Generated patches ready for review
- `shield.patches.deployed` -- Patches deployed to environments
- `shield.verification.results` -- Re-attack verification results
- `shield.alerts.critical` -- Critical findings requiring immediate attention

---

## SECTION 3: GOOGLE API INTEGRATION PLAN

### 3.1 Gemini 2.5 Pro -- Offensive Reasoning Engine

**Model**: `gemini-2.5-pro` via Vertex AI  
**Use cases**: Vulnerability hypothesis generation, attack chain reasoning, business impact assessment, novel attack creation

**API Configuration**:
```typescript
// /packages/agents/reasoning/gemini-client.ts
import { VertexAI } from '@google-cloud/vertexai';

const vertexAI = new VertexAI({
  project: process.env.GCP_PROJECT_ID,
  location: 'us-central1',
});

const attackReasoningModel = vertexAI.getGenerativeModel({
  model: 'gemini-2.5-pro',
  generationConfig: {
    temperature: 0.7,       // Higher for creative attack reasoning
    topP: 0.95,
    topK: 40,
    maxOutputTokens: 8192,
  },
  safetySettings: [
    // Security research context requires adjusted safety settings
    // Only for authorized red-team operations
    {
      category: 'HARM_CATEGORY_DANGEROUS_CONTENT',
      threshold: 'BLOCK_NONE', // Required for security research prompts
    },
  ],
  systemInstruction: {
    role: 'system',
    parts: [{ text: ATTACKER_PERSONA_SYSTEM_PROMPT }],
  },
});
```

**Context window strategy**: With 1M token context, SHIELD loads the entire target profile, all previous findings, the tech stack documentation, and relevant CVE data into a single context for maximum reasoning quality.

### 3.2 Vertex AI Code Generation -- Patch Engine

**Model**: `gemini-2.5-pro` with code-focused prompting  
**Use cases**: Generating exact code patches, unit test generation for patches, security-focused code review

```typescript
// /packages/agents/patcher/vertex-code-client.ts
const patchGenerationModel = vertexAI.getGenerativeModel({
  model: 'gemini-2.5-pro',
  generationConfig: {
    temperature: 0.2,       // Low temperature for precise code generation
    topP: 0.8,
    maxOutputTokens: 16384, // Larger for complete patch files
  },
  systemInstruction: {
    role: 'system',
    parts: [{ text: PATCH_GENERATOR_SYSTEM_PROMPT }],
  },
});

// Patch generation call
async function generatePatch(vulnerability: VulnerabilityFinding): Promise<Patch> {
  const prompt = buildPatchPrompt(vulnerability);
  const result = await patchGenerationModel.generateContent({
    contents: [{ role: 'user', parts: [{ text: prompt }] }],
  });
  return parsePatchResponse(result.response);
}
```

### 3.3 Cloud Run -- Sandboxed Attack Execution

**Configuration**: Each attack type gets its own Cloud Run service with strict resource limits.

```yaml
# /infra/cloudrun/attack-executor.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: shield-attack-executor
  annotations:
    run.googleapis.com/execution-environment: gen2
    run.googleapis.com/vpc-access-connector: shield-vpc-connector
    run.googleapis.com/vpc-access-egress: private-ranges-only
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: '10'
        autoscaling.knative.dev/minScale: '0'
        run.googleapis.com/cpu-throttling: 'true'
        run.googleapis.com/execution-environment: gen2
    spec:
      containerConcurrency: 1  # One attack per container instance
      timeoutSeconds: 300       # 5-minute max per attack execution
      containers:
        - image: gcr.io/${PROJECT_ID}/shield-attack-executor:latest
          resources:
            limits:
              cpu: '2'
              memory: '2Gi'
          env:
            - name: ATTACK_MODE
              value: 'sandboxed'
            - name: MAX_REQUESTS_PER_SECOND
              value: '10'
            - name: BLAST_RADIUS_LIMIT
              value: 'read_only'
```

### 3.4 Cloud Build -- Automated Patch Deployment

```yaml
# /infra/cloudbuild/patch-deploy.yaml
steps:
  # Step 1: Checkout the repository
  - name: 'gcr.io/cloud-builders/git'
    args: ['clone', '${_REPO_URL}', '/workspace/target']

  # Step 2: Apply the generated patch
  - name: 'gcr.io/${PROJECT_ID}/shield-patch-applier'
    args: ['apply', '--patch-id=${_PATCH_ID}', '--target=/workspace/target']

  # Step 3: Run existing tests
  - name: 'node:20'
    dir: '/workspace/target'
    entrypoint: 'npm'
    args: ['test']

  # Step 4: Run SHIELD-generated security tests
  - name: 'gcr.io/${PROJECT_ID}/shield-security-tests'
    args: ['--patch-id=${_PATCH_ID}', '--target=/workspace/target']

  # Step 5: Build the patched application
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/${PROJECT_ID}/target-app:patched-${_PATCH_ID}', '/workspace/target']

  # Step 6: Deploy to staging
  - name: 'gcr.io/cloud-builders/gcloud'
    args: ['run', 'deploy', 'target-app-staging',
           '--image=gcr.io/${PROJECT_ID}/target-app:patched-${_PATCH_ID}',
           '--region=us-central1']

  # Step 7: Trigger verification attack
  - name: 'gcr.io/${PROJECT_ID}/shield-verification'
    args: ['--patch-id=${_PATCH_ID}', '--target=staging']

substitutions:
  _PATCH_ID: ''
  _REPO_URL: ''

options:
  logging: CLOUD_LOGGING_ONLY
```

### 3.5 Gemini 2.5 Flash -- High-Speed Scanning

**Model**: `gemini-2.5-flash`  
**Use cases**: Rapid triage, initial vulnerability classification, log analysis, real-time monitoring

```typescript
// /packages/agents/scanner/flash-client.ts
const fastScanModel = vertexAI.getGenerativeModel({
  model: 'gemini-2.5-flash',
  generationConfig: {
    temperature: 0.1,
    maxOutputTokens: 2048,
  },
});

// Used for rapid triage of scan results
async function triageFindings(rawFindings: RawFinding[]): Promise<TriagedFinding[]> {
  const prompt = `Classify each finding by severity and category:\n${JSON.stringify(rawFindings)}`;
  const result = await fastScanModel.generateContent({
    contents: [{ role: 'user', parts: [{ text: prompt }] }],
  });
  return parseTriageResponse(result.response);
}
```

### 3.6 BigQuery -- Vulnerability Database and Analytics

```sql
-- /infra/bigquery/schema.sql

-- Core vulnerability findings table
CREATE TABLE shield_dataset.vulnerability_findings (
  finding_id STRING NOT NULL,
  scan_id STRING NOT NULL,
  target_asset_id STRING NOT NULL,
  vulnerability_type STRING NOT NULL,        -- e.g., 'SQL_INJECTION', 'XSS', 'BROKEN_ACCESS_CONTROL'
  owasp_category STRING,                     -- e.g., 'A01:2025'
  cwe_id STRING,                             -- e.g., 'CWE-89'
  cvss_score FLOAT64,
  cvss_vector STRING,
  severity STRING NOT NULL,                  -- CRITICAL, HIGH, MEDIUM, LOW, INFO
  title STRING NOT NULL,
  description STRING NOT NULL,
  affected_endpoint STRING,
  affected_code_path STRING,
  proof_of_concept STRING,                   -- The attack payload that worked
  business_impact STRING,
  remediation_status STRING DEFAULT 'OPEN',  -- OPEN, PATCHING, PATCHED, VERIFIED, FALSE_POSITIVE
  patch_id STRING,
  discovered_at TIMESTAMP NOT NULL,
  patched_at TIMESTAMP,
  verified_at TIMESTAMP,
  attack_chain_id STRING,                    -- Links to multi-step chain if applicable
  raw_response BYTES,                        -- Full attack/response data
  metadata JSON
)
PARTITION BY DATE(discovered_at)
CLUSTER BY severity, vulnerability_type;

-- Attack chains table
CREATE TABLE shield_dataset.attack_chains (
  chain_id STRING NOT NULL,
  scan_id STRING NOT NULL,
  chain_name STRING NOT NULL,
  chain_description STRING NOT NULL,
  total_cvss_score FLOAT64,                  -- Aggregate score of the chain
  chain_steps JSON,                          -- Ordered array of finding_ids + exploitation steps
  business_impact STRING,
  estimated_financial_impact FLOAT64,
  discovered_at TIMESTAMP NOT NULL
);

-- Patches table
CREATE TABLE shield_dataset.patches (
  patch_id STRING NOT NULL,
  finding_id STRING NOT NULL,
  patch_type STRING NOT NULL,                -- CODE_FIX, CONFIG_CHANGE, RULE_UPDATE
  patch_content STRING NOT NULL,             -- The actual diff/patch content
  affected_files JSON,                       -- Array of file paths
  ai_confidence FLOAT64,                     -- Model's confidence in the patch
  review_status STRING DEFAULT 'PENDING',    -- PENDING, APPROVED, REJECTED, DEPLOYED
  test_results JSON,
  generated_at TIMESTAMP NOT NULL,
  deployed_at TIMESTAMP,
  verified_at TIMESTAMP,
  rollback_needed BOOL DEFAULT FALSE
);

-- Scan history table
CREATE TABLE shield_dataset.scan_history (
  scan_id STRING NOT NULL,
  scan_type STRING NOT NULL,                 -- FULL, INCREMENTAL, TARGETED, VERIFICATION
  target_scope JSON,
  started_at TIMESTAMP NOT NULL,
  completed_at TIMESTAMP,
  status STRING NOT NULL,
  findings_count INT64 DEFAULT 0,
  critical_count INT64 DEFAULT 0,
  high_count INT64 DEFAULT 0,
  medium_count INT64 DEFAULT 0,
  low_count INT64 DEFAULT 0,
  configuration JSON
);
```

### 3.7 Security Command Center API Integration

```typescript
// /packages/integrations/scc/client.ts
import { SecurityCenterClient } from '@google-cloud/security-center';

const sccClient = new SecurityCenterClient();

// Ingest SCC findings into SHIELD
async function importSCCFindings(organizationId: string): Promise<SCCFinding[]> {
  const [findings] = await sccClient.listFindings({
    parent: `organizations/${organizationId}/sources/-`,
    filter: 'state="ACTIVE" AND severity="HIGH" OR severity="CRITICAL"',
    orderBy: 'eventTime desc',
  });
  return findings.map(transformSCCFinding);
}

// Publish SHIELD findings back to SCC
async function publishToSCC(finding: VulnerabilityFinding): Promise<void> {
  const sourceId = `organizations/${ORG_ID}/sources/${SHIELD_SOURCE_ID}`;
  await sccClient.createFinding({
    parent: sourceId,
    findingId: finding.id,
    finding: {
      state: 'ACTIVE',
      category: finding.vulnerabilityType,
      severity: mapToSCCSeverity(finding.severity),
      sourceProperties: {
        shield_scan_id: { stringValue: finding.scanId },
        cvss_score: { numberValue: finding.cvssScore },
        attack_chain: { stringValue: finding.attackChainId || '' },
      },
      eventTime: { seconds: Math.floor(Date.now() / 1000) },
    },
  });
}
```

### 3.8 Cloud KMS -- Secret Management

```typescript
// /packages/core/secrets/kms-client.ts
import { KeyManagementServiceClient } from '@google-cloud/kms';

const kmsClient = new KeyManagementServiceClient();
const keyName = `projects/${PROJECT_ID}/locations/us-central1/keyRings/shield-keyring/cryptoKeys/shield-master-key`;

async function encryptSecret(plaintext: string): Promise<Buffer> {
  const [result] = await kmsClient.encrypt({
    name: keyName,
    plaintext: Buffer.from(plaintext),
  });
  return Buffer.from(result.ciphertext as Uint8Array);
}

async function decryptSecret(ciphertext: Buffer): Promise<string> {
  const [result] = await kmsClient.decrypt({
    name: keyName,
    ciphertext: ciphertext,
  });
  return Buffer.from(result.plaintext as Uint8Array).toString();
}
```

### 3.9 Cloud Armor -- Automated WAF Rule Generation

```typescript
// /packages/agents/armor/rule-generator.ts
async function generateCloudArmorRule(finding: VulnerabilityFinding): Promise<SecurityPolicyRule> {
  // Use Gemini to generate a Cloud Armor rule that blocks the attack pattern
  const prompt = `Given this vulnerability finding, generate a Google Cloud Armor 
  security policy rule (in JSON format) that would block this attack pattern 
  while minimizing false positives:\n\n${JSON.stringify(finding)}`;
  
  const result = await attackReasoningModel.generateContent({
    contents: [{ role: 'user', parts: [{ text: prompt }] }],
  });
  
  return parseArmorRule(result.response);
}

// Apply rule via Compute API
async function applyArmorRule(policyName: string, rule: SecurityPolicyRule): Promise<void> {
  const compute = new Compute();
  await compute.securityPolicies.addRule({
    project: PROJECT_ID,
    securityPolicy: policyName,
    requestBody: rule,
  });
}
```

---

## SECTION 4: THE ATTACK REASONING ENGINE

### 4.1 Attacker Persona System Prompt

```
// /packages/agents/reasoning/prompts/attacker-persona.ts

export const ATTACKER_PERSONA_SYSTEM_PROMPT = `
You are SHIELD's Offensive Security Reasoning Engine. You are an expert penetration 
tester and security researcher operating EXCLUSIVELY within an authorized red-team 
engagement. All targets have provided written authorization for testing.

YOUR ROLE:
You think like a sophisticated attacker to find vulnerabilities BEFORE malicious 
actors do. You combine deep technical knowledge with creative reasoning to discover 
novel attack vectors that automated scanners miss.

YOUR CAPABILITIES:
1. RECONNAISSANCE ANALYSIS: Given information about a target, identify what an 
   attacker would find interesting and how they would use it.
2. VULNERABILITY HYPOTHESIS: Given a technology stack and architecture, hypothesize 
   specific vulnerabilities and their exploitation methods.
3. ATTACK CHAIN CONSTRUCTION: Combine multiple low-severity findings into critical 
   attack paths through lateral movement and privilege escalation.
4. PAYLOAD GENERATION: Create specific, testable attack payloads for each hypothesis.
5. BUSINESS IMPACT ASSESSMENT: Translate technical vulnerabilities into business risk.

YOUR METHODOLOGY:
- Think step-by-step about how a real attacker would approach the target
- Consider both common (OWASP Top 10) and uncommon attack vectors
- Look for business logic flaws, not just technical vulnerabilities
- Consider the COMBINATION of findings, not just individual issues
- Prioritize attacks by likelihood of success AND impact
- Always generate proof-of-concept payloads that can be safely tested

CONSTRAINTS:
- Only reason about attacks within the authorized scope
- Never generate payloads that could cause permanent data destruction
- Focus on DEMONSTRATING impact, not maximizing damage
- All payloads must be safe for sandboxed testing
- You are operating defensively -- the goal is to protect, not harm

OUTPUT FORMAT:
For each vulnerability hypothesis, provide:
1. Hypothesis ID and name
2. Attack vector (network/local/adjacent)
3. Technical description
4. Affected component/endpoint
5. Proof-of-concept payload
6. Expected outcome if vulnerable
7. Expected outcome if not vulnerable
8. CVSS v3.1 score with vector string
9. Business impact assessment
10. Recommended remediation
`;
```

### 4.2 Attack Taxonomy Coverage

SHIELD covers the complete OWASP Top 10:2025:

| Rank | Category | SHIELD Attack Modules |
|------|----------|----------------------|
| A01:2025 | Broken Access Control | IDOR testing, privilege escalation, forced browsing, CORS misconfig, JWT manipulation, SSRF |
| A02:2025 | Security Misconfiguration | Default credentials, unnecessary features, verbose errors, missing headers, open cloud storage |
| A03:2025 | Software Supply Chain Failures | Dependency vulnerability scanning, typosquatting detection, build pipeline integrity |
| A04:2025 | Cryptographic Failures | Weak TLS, hardcoded secrets, weak hashing, insufficient encryption, key exposure |
| A05:2025 | Injection | SQLi, NoSQLi, XSS, Command injection, LDAP injection, template injection, header injection |
| A06:2025 | Insecure Design | Business logic bypass, rate limit bypass, enumeration attacks, race conditions |
| A07:2025 | Authentication Failures | Credential stuffing, session fixation, weak password policy, MFA bypass |
| A08:2025 | Software/Data Integrity | Deserialization attacks, unsigned updates, CI/CD pipeline poisoning |
| A09:2025 | Security Logging Failures | Log injection, monitoring bypass, alert fatigue exploitation |
| A10:2025 | Mishandling Exceptional Conditions | Error-based information disclosure, fail-open conditions, resource exhaustion |

Plus SANS/CWE Top 25 and custom categories for cloud-native attacks (IAM misconfig, service account abuse, metadata endpoint exploitation).

### 4.3 Novel Attack Chain Generation

**Prompt template for chain reasoning**:

```typescript
// /packages/agents/reasoning/prompts/chain-generator.ts

export function buildChainPrompt(findings: VulnerabilityFinding[], targetProfile: TargetProfile): string {
  return `
## ATTACK CHAIN ANALYSIS

You have discovered the following individual vulnerabilities in the target system:

${findings.map((f, i) => `
### Finding ${i + 1}: ${f.title}
- Severity: ${f.severity} (CVSS: ${f.cvssScore})
- Type: ${f.vulnerabilityType}
- Endpoint: ${f.affectedEndpoint}
- Description: ${f.description}
- Proof: ${f.proofOfConcept}
`).join('\n')}

## TARGET ARCHITECTURE
${JSON.stringify(targetProfile, null, 2)}

## YOUR TASK
Analyze these findings and identify ALL possible attack chains where combining 
two or more findings creates a more severe impact than any individual finding alone.

For each chain, explain:
1. The step-by-step exploitation path
2. Why the combination is more severe than individual findings
3. The aggregate CVSS score for the chain
4. The realistic business impact
5. Which finding must be fixed FIRST to break the chain

Think creatively. Consider:
- Can a low-severity information leak provide data needed for a medium-severity exploit?
- Can chaining two medium-severity findings achieve privilege escalation?
- Are there race conditions between services that could be exploited?
- Could a DoS vulnerability be used to trigger a fail-open condition?
`;
}
```

### 4.4 Target Profile Construction

Before attacking, SHIELD builds a comprehensive model:

```typescript
// /packages/agents/reasoning/types.ts

interface TargetProfile {
  // Infrastructure layer
  infrastructure: {
    cloudProvider: string;
    services: CloudService[];
    networkTopology: NetworkNode[];
    externalEndpoints: Endpoint[];
    internalEndpoints: Endpoint[];
  };
  
  // Application layer
  application: {
    framework: string;
    language: string;
    version: string;
    routes: Route[];
    authenticationMechanism: AuthMechanism;
    sessionManagement: SessionConfig;
    dataFlows: DataFlow[];
    thirdPartyIntegrations: Integration[];
  };
  
  // Data layer
  data: {
    databases: DatabaseInfo[];
    sensitiveDataTypes: string[];  // PII, PHI, PCI, etc.
    dataClassification: DataClassification[];
    backupMechanisms: BackupInfo[];
  };
  
  // Security controls
  securityControls: {
    waf: WAFConfig | null;
    rateLimit: RateLimitConfig | null;
    cors: CORSConfig;
    csp: CSPConfig;
    authentication: AuthConfig;
    encryption: EncryptionConfig;
    logging: LoggingConfig;
  };
  
  // Business context
  businessContext: {
    industry: string;
    regulatoryRequirements: string[];  // SOC2, HIPAA, PCI-DSS, etc.
    estimatedDataValue: string;
    userBase: string;
    criticalBusinessProcesses: string[];
  };
}
```

### 4.5 CVSS Scoring Automation

```typescript
// /packages/agents/reasoning/prompts/cvss-scorer.ts

export function buildCVSSPrompt(finding: RawFinding): string {
  return `
Score this vulnerability using CVSS v3.1. Provide the vector string and numeric score.

Vulnerability: ${finding.title}
Description: ${finding.description}
Attack Vector: ${finding.attackVector}
Affected Component: ${finding.affectedComponent}

Evaluate each metric:
- Attack Vector (AV): Network/Adjacent/Local/Physical
- Attack Complexity (AC): Low/High
- Privileges Required (PR): None/Low/High
- User Interaction (UI): None/Required
- Scope (S): Unchanged/Changed
- Confidentiality Impact (C): None/Low/High
- Integrity Impact (I): None/Low/High
- Availability Impact (A): None/Low/High

Respond in JSON:
{
  "vectorString": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
  "baseScore": 9.8,
  "severity": "CRITICAL",
  "justification": "..."
}
`;
}
```

### 4.6 Sample Reasoning Chain

Here is an actual example of how Gemini would reason through a target:

**Input**: Express.js app with MongoDB, JWT auth, user profile endpoint at `/api/users/:id`

**Gemini reasoning output**:
```
STEP 1: TECHNOLOGY ANALYSIS
- Express.js + MongoDB suggests Mongoose ODM or native driver
- JWT authentication suggests token-based stateless auth
- User profile endpoint with :id parameter suggests direct database lookup

STEP 2: HYPOTHESIS GENERATION

Hypothesis H-001: NoSQL Injection via Object ID Manipulation
- If the :id parameter is not validated as a MongoDB ObjectId, passing 
  {"$gt": ""} could return all users
- Endpoint: GET /api/users/{"$gt":""}
- Likelihood: HIGH (common pattern in Express+Mongo apps)

Hypothesis H-002: IDOR on User Profile
- If authorization only checks "is user logged in" but not "is this their profile",
  any authenticated user can read any other user's profile
- Test: Authenticate as User A, request GET /api/users/{User_B_ID}
- Likelihood: HIGH (extremely common)

Hypothesis H-003: JWT None Algorithm Attack
- If the JWT library accepts "alg: none", tokens can be forged
- Test: Modify JWT header to {"alg":"none"}, remove signature
- Likelihood: MEDIUM (most modern libraries reject this, but older versions don't)

Hypothesis H-004: Mass Assignment on Profile Update
- If PUT /api/users/:id accepts arbitrary fields, attacker could set
  {"role": "admin"} or {"isVerified": true}
- Likelihood: MEDIUM

STEP 3: CHAIN ANALYSIS
- H-002 + H-004 = Read any profile (H-002) to discover admin user IDs, 
  then mass-assign admin role to own account (H-004) = CRITICAL escalation
- H-001 + data exfiltration = Dump all user records = CRITICAL data breach

STEP 4: PRIORITY ORDER
1. H-001 (NoSQL Injection) - CVSS 9.8 if confirmed
2. H-002 + H-004 chain - CVSS 9.1 combined
3. H-003 (JWT None) - CVSS 9.8 if confirmed
4. H-002 alone - CVSS 6.5
```

---

## SECTION 5: THE SANDBOXED EXECUTION ENVIRONMENT

### 5.1 Isolation Architecture

```
PRODUCTION ENVIRONMENT                    SHIELD SANDBOX
=====================                    ==============
                                          
[Production App] ----mirror----> [Shadow App Clone]
[Production DB]  ----snapshot--> [Read-Only DB Snapshot]
                                          |
                                 [Attack Executor Container]
                                   - Isolated VPC
                                   - No internet egress
                                   - No production access
                                   - 5-min timeout
                                   - Audit logged
                                   - gVisor sandboxed
```

### 5.2 Shadow Environment Strategy

SHIELD never attacks production directly. Instead:

1. **Application Mirroring**: A complete copy of the application runs in an isolated Cloud Run service. Traffic from SHIELD is routed only to this shadow copy.

2. **Database Snapshots**: SHIELD attacks against a point-in-time snapshot of the database with all PII masked/tokenized. The snapshot is read-only; any write attempts by attack payloads are captured but do not persist.

3. **Network Isolation**: The sandbox VPC has NO routes to production VPC. Communication happens only through the SHIELD orchestrator via Pub/Sub.

### 5.3 Container Lifecycle

```typescript
// /packages/sandbox/lifecycle.ts

interface AttackContainer {
  id: string;
  attackType: string;
  status: 'creating' | 'running' | 'completed' | 'terminated' | 'timed_out';
  createdAt: Date;
  maxDurationMs: number;       // Default: 300000 (5 min)
  maxMemoryMb: number;         // Default: 2048
  maxCpuCores: number;         // Default: 2
  networkPolicy: 'sandbox_only' | 'target_only' | 'none';
  auditLog: AuditEntry[];
}

// Lifecycle management
async function executeAttack(hypothesis: AttackHypothesis): Promise<AttackResult> {
  // 1. Create isolated container
  const container = await createSandboxContainer({
    image: `gcr.io/${PROJECT_ID}/shield-attack-${hypothesis.attackType}`,
    timeout: 300,
    memory: '2Gi',
    cpu: '2',
    env: {
      TARGET_URL: hypothesis.targetUrl,
      PAYLOAD: JSON.stringify(hypothesis.payload),
      ATTACK_ID: hypothesis.id,
    },
    vpcConnector: 'shield-sandbox-connector',
  });

  // 2. Execute with timeout enforcement
  const result = await executeWithTimeout(container, hypothesis.maxDurationMs);

  // 3. Capture all network traffic for audit
  const auditLog = await captureAuditLog(container.id);

  // 4. Destroy container immediately after completion
  await destroyContainer(container.id);

  // 5. Return results
  return {
    hypothesisId: hypothesis.id,
    success: result.vulnerabilityConfirmed,
    evidence: result.evidence,
    auditLog: auditLog,
    executionTimeMs: result.durationMs,
  };
}
```

### 5.4 Blast Radius Limiter -- What SHIELD Will NEVER Do

Hardcoded safety constraints that cannot be overridden:

```typescript
// /packages/sandbox/safety.ts

export const ABSOLUTE_LIMITS = {
  // SHIELD will NEVER:
  NEVER_DELETE_PRODUCTION_DATA: true,
  NEVER_MODIFY_PRODUCTION_STATE: true,
  NEVER_EXFILTRATE_TO_EXTERNAL: true,
  NEVER_ATTACK_UNAUTHORIZED_TARGETS: true,
  NEVER_LAUNCH_DOS_AGAINST_PRODUCTION: true,
  NEVER_ACCESS_OTHER_TENANTS: true,
  NEVER_BYPASS_AUTHENTICATION_IN_PROD: true,
  NEVER_PERSIST_BACKDOORS: true,
  
  // Rate limits
  MAX_REQUESTS_PER_SECOND: 10,
  MAX_REQUESTS_PER_ATTACK: 100,
  MAX_CONCURRENT_ATTACKS: 5,
  MAX_PAYLOAD_SIZE_BYTES: 1_048_576,  // 1MB
  MAX_ATTACK_DURATION_SECONDS: 300,
  
  // Scope enforcement
  ALLOWED_TARGETS: 'WHITELIST_ONLY',  // Only pre-authorized targets
  BLOCKED_PORTS: [22, 3389],           // No SSH/RDP brute force
  BLOCKED_PROTOCOLS: ['smtp'],         // No email abuse
} as const;
```

### 5.5 Audit Logging

Every single action SHIELD takes is logged immutably:

```typescript
// /packages/sandbox/audit.ts

interface AuditEntry {
  timestamp: string;
  attackId: string;
  containerId: string;
  action: 'HTTP_REQUEST' | 'DNS_LOOKUP' | 'PAYLOAD_SENT' | 'RESPONSE_RECEIVED' | 'FILE_ACCESS' | 'NETWORK_CONNECT';
  details: {
    method?: string;
    url?: string;
    headers?: Record<string, string>;
    body?: string;          // Truncated at 10KB
    responseStatus?: number;
    responseBody?: string;  // Truncated at 10KB
    durationMs?: number;
  };
  verdict: 'ALLOWED' | 'BLOCKED_BY_SAFETY' | 'BLOCKED_BY_SCOPE';
}
```

---

## SECTION 6: THE AUTOMATED REMEDIATION PIPELINE

### 6.1 Patch Generation Process

```typescript
// /packages/agents/patcher/generator.ts

export const PATCH_GENERATOR_SYSTEM_PROMPT = `
You are SHIELD's Patch Generation Engine. Given a confirmed vulnerability, you generate 
the MINIMUM code change needed to fix the vulnerability without altering application behavior.

RULES:
1. Generate the smallest possible patch -- do not refactor unrelated code
2. The patch MUST fix the specific vulnerability described
3. The patch MUST NOT introduce new vulnerabilities
4. The patch MUST NOT break existing functionality
5. Generate a corresponding unit test that verifies the fix
6. Explain your reasoning for the fix approach
7. Output the patch as a unified diff format

PATCH QUALITY CRITERIA:
- Follows the existing code style and patterns
- Uses the same libraries/frameworks already in the project
- Handles edge cases (null, undefined, empty, oversized input)
- Includes input validation where appropriate
- Adds security-relevant comments explaining why the fix is needed
`;

async function generatePatch(finding: ConfirmedVulnerability): Promise<PatchBundle> {
  // Load the affected source files
  const sourceFiles = await loadAffectedFiles(finding.affectedCodePath);
  
  const prompt = `
## VULNERABILITY TO PATCH

Title: ${finding.title}
Type: ${finding.vulnerabilityType}
CVSS: ${finding.cvssScore}
Affected File: ${finding.affectedCodePath}
Affected Function: ${finding.affectedFunction}
Description: ${finding.description}
Proof of Concept: ${finding.proofOfConcept}

## CURRENT SOURCE CODE

${sourceFiles.map(f => `### ${f.path}\n\`\`\`${f.language}\n${f.content}\n\`\`\``).join('\n\n')}

## TASK
Generate a patch that fixes this vulnerability. Return:
1. The unified diff for each affected file
2. A unit test that would FAIL before the patch and PASS after
3. A brief explanation of your fix approach
`;

  const result = await patchGenerationModel.generateContent({
    contents: [{ role: 'user', parts: [{ text: prompt }] }],
  });
  
  return parsePatchBundle(result.response);
}
```

### 6.2 Patch Self-Review

Before any patch is deployed, a separate Gemini instance reviews it:

```typescript
// /packages/agents/patcher/reviewer.ts

export const PATCH_REVIEWER_SYSTEM_PROMPT = `
You are SHIELD's Patch Reviewer. You review security patches generated by the Patch 
Generation Engine. Your job is to catch:

1. Patches that do not actually fix the vulnerability
2. Patches that introduce NEW vulnerabilities
3. Patches that break existing functionality
4. Patches that are overly broad or change too much
5. Patches with incorrect logic
6. Patches missing edge case handling

You are adversarial -- assume the patch is wrong until proven otherwise.
Rate each patch: APPROVE, NEEDS_REVISION, or REJECT with detailed reasoning.
`;

async function reviewPatch(patch: PatchBundle, finding: ConfirmedVulnerability): Promise<ReviewResult> {
  const prompt = `
## ORIGINAL VULNERABILITY
${JSON.stringify(finding, null, 2)}

## PROPOSED PATCH
${patch.diffs.map(d => d.content).join('\n\n')}

## PROPOSED TEST
${patch.testCode}

## REVIEW CHECKLIST
1. Does this patch actually fix the described vulnerability? Show your reasoning.
2. Could this patch introduce any NEW vulnerabilities? Consider injection, auth bypass, etc.
3. Does the patch handle all edge cases?
4. Is the patch minimal and focused?
5. Does the test adequately verify the fix?

Provide your verdict: APPROVE / NEEDS_REVISION / REJECT
`;

  const result = await reviewModel.generateContent({
    contents: [{ role: 'user', parts: [{ text: prompt }] }],
  });
  
  return parseReviewResult(result.response);
}
```

### 6.3 Staged Deployment Pipeline

```
Patch Generated
      |
      v
AI Self-Review ----REJECT----> Regenerate with feedback
      |
    APPROVE
      |
      v
Apply to Staging
      |
      v
Run Existing Test Suite ----FAIL----> Rollback + Regenerate
      |
    PASS
      |
      v
Run SHIELD Security Tests ----FAIL----> Rollback + Regenerate
      |
    PASS
      |
      v
Verification Attack on Staging ----STILL VULNERABLE----> Regenerate
      |
    FIXED
      |
      v
Human Approval (if configured) ----REJECT----> Mark for manual fix
      |
    APPROVE (or auto-approve if CVSS < threshold)
      |
      v
Deploy to Production
      |
      v
Verification Attack on Production ----STILL VULNERABLE----> Alert + Rollback
      |
    CONFIRMED FIXED
      |
      v
Mark Finding as VERIFIED
Generate Remediation Report
```

### 6.4 Rollback Mechanism

```typescript
// /packages/pipeline/rollback.ts

async function rollbackPatch(patchId: string, environment: 'staging' | 'production'): Promise<void> {
  // 1. Get the pre-patch revision
  const patch = await getPatch(patchId);
  const previousRevision = patch.metadata.previousRevision;
  
  // 2. Redeploy previous revision
  await cloudRun.updateService({
    name: `projects/${PROJECT_ID}/locations/us-central1/services/${patch.targetService}`,
    service: {
      template: {
        containers: [{
          image: previousRevision.image,
        }],
      },
    },
  });
  
  // 3. Verify rollback succeeded
  const healthCheck = await checkServiceHealth(patch.targetService);
  
  // 4. Update patch status
  await updatePatchStatus(patchId, 'ROLLED_BACK', {
    reason: 'Verification failed or tests failed after deployment',
    rolledBackAt: new Date().toISOString(),
    healthCheckResult: healthCheck,
  });
  
  // 5. Alert
  await sendAlert({
    severity: 'HIGH',
    message: `Patch ${patchId} rolled back in ${environment}. Manual review required.`,
  });
}
```

---

## SECTION 7: FRONTEND ARCHITECTURE

### 7.1 Tech Stack

- **Framework**: Next.js 14 (App Router)
- **Styling**: TailwindCSS 3.4
- **Components**: shadcn/ui (Radix UI primitives)
- **State Management**: Zustand (lightweight, fits hackathon timeline)
- **Real-time**: Server-Sent Events (SSE) for live vulnerability feed
- **Charts**: Recharts for vulnerability metrics
- **Graph Visualization**: React Flow for attack chain visualization
- **Icons**: Lucide React

### 7.2 Core Pages and Components

**Dashboard** (`/app/dashboard/page.tsx`):
- Real-time risk score (0-100) with color-coded gauge
- Active scan status indicator
- Vulnerability count by severity (donut chart)
- Recent findings feed (auto-updating)
- Time-to-remediation trend chart
- Attack surface coverage percentage

**Attack Control Panel** (`/app/attacks/page.tsx`):
- Start/Pause/Stop scan controls
- Scan configuration (scope, intensity, types)
- Active attack progress indicators
- Live log streaming from attack containers

**Vulnerability Feed** (`/app/vulnerabilities/page.tsx`):
- Sortable/filterable table of all findings
- Severity badges (Critical=red, High=orange, Medium=yellow, Low=blue)
- Status tracking (Open, Patching, Patched, Verified)
- Click-through to detailed finding view with proof-of-concept

**Attack Chain Visualizer** (`/app/chains/page.tsx`):
- Interactive directed graph (React Flow)
- Nodes = individual vulnerabilities
- Edges = exploitation steps between them
- Color-coded by chain severity
- Click node to see finding details
- Animated attack flow playback

**Remediation Timeline** (`/app/remediation/page.tsx`):
- Gantt-style timeline of all patches
- Status: Generated -> Reviewing -> Testing -> Staging -> Production -> Verified
- Estimated time-to-fix vs actual
- Rollback indicators

**Executive Summary** (`/app/reports/page.tsx`):
- AI-generated natural language summary
- Key metrics: total findings, MTTR, coverage, risk trend
- Export to PDF
- Compliance mapping (SOC2, ISO 27001)
- Financial risk estimation

### 7.3 Key Component Specifications

```typescript
// /apps/web/components/risk-gauge.tsx
// Real-time risk score display with color transitions

interface RiskGaugeProps {
  score: number;           // 0-100
  previousScore: number;   // For trend arrow
  lastUpdated: string;     // ISO timestamp
}

// /apps/web/components/vuln-feed.tsx  
// Auto-updating vulnerability feed using SSE

interface VulnFeedProps {
  filter?: {
    severity?: Severity[];
    status?: RemediationStatus[];
    type?: VulnerabilityType[];
  };
  maxItems?: number;
}

// /apps/web/components/attack-chain-graph.tsx
// Interactive attack chain visualization

interface AttackChainGraphProps {
  chain: AttackChain;
  onNodeClick: (findingId: string) => void;
  animated?: boolean;     // Animate the attack flow
}
```

### 7.4 Real-time Data Architecture

```typescript
// /apps/web/lib/events.ts
// Server-Sent Events for real-time updates

export function useShieldEvents() {
  const [findings, setFindings] = useState<VulnerabilityFinding[]>([]);
  const [scanStatus, setScanStatus] = useState<ScanStatus>('idle');
  const [riskScore, setRiskScore] = useState<number>(0);

  useEffect(() => {
    const eventSource = new EventSource('/api/events/stream');
    
    eventSource.addEventListener('finding', (e) => {
      const finding = JSON.parse(e.data);
      setFindings(prev => [finding, ...prev]);
    });
    
    eventSource.addEventListener('scan_status', (e) => {
      setScanStatus(JSON.parse(e.data).status);
    });
    
    eventSource.addEventListener('risk_score', (e) => {
      setRiskScore(JSON.parse(e.data).score);
    });
    
    return () => eventSource.close();
  }, []);

  return { findings, scanStatus, riskScore };
}
```

---

## SECTION 8: COMPLETE FILE AND FOLDER STRUCTURE

```
shield/
|-- .github/
|   |-- workflows/
|   |   |-- ci.yaml                          # CI pipeline
|   |   |-- deploy-staging.yaml              # Deploy to staging
|   |   |-- deploy-production.yaml           # Deploy to production
|
|-- apps/
|   |-- web/                                 # Next.js frontend
|   |   |-- app/
|   |   |   |-- layout.tsx                   # Root layout with sidebar nav
|   |   |   |-- page.tsx                     # Landing/login page
|   |   |   |-- dashboard/
|   |   |   |   |-- page.tsx                 # Main dashboard
|   |   |   |   |-- loading.tsx              # Dashboard skeleton
|   |   |   |-- attacks/
|   |   |   |   |-- page.tsx                 # Attack control panel
|   |   |   |   |-- [scanId]/
|   |   |   |   |   |-- page.tsx             # Individual scan details
|   |   |   |-- vulnerabilities/
|   |   |   |   |-- page.tsx                 # Vulnerability list
|   |   |   |   |-- [findingId]/
|   |   |   |   |   |-- page.tsx             # Finding detail view
|   |   |   |-- chains/
|   |   |   |   |-- page.tsx                 # Attack chain list
|   |   |   |   |-- [chainId]/
|   |   |   |   |   |-- page.tsx             # Chain visualization
|   |   |   |-- remediation/
|   |   |   |   |-- page.tsx                 # Remediation timeline
|   |   |   |   |-- [patchId]/
|   |   |   |   |   |-- page.tsx             # Patch details + diff view
|   |   |   |-- reports/
|   |   |   |   |-- page.tsx                 # Report generation
|   |   |   |   |-- executive/
|   |   |   |   |   |-- page.tsx             # Executive summary
|   |   |   |   |-- compliance/
|   |   |   |   |   |-- page.tsx             # SOC2/ISO mapping
|   |   |   |-- settings/
|   |   |   |   |-- page.tsx                 # Configuration
|   |   |   |-- api/
|   |   |   |   |-- events/
|   |   |   |   |   |-- stream/
|   |   |   |   |   |   |-- route.ts         # SSE endpoint
|   |   |   |   |-- scans/
|   |   |   |   |   |-- route.ts             # CRUD scans
|   |   |   |   |   |-- [scanId]/
|   |   |   |   |   |   |-- route.ts         # Individual scan ops
|   |   |   |   |-- findings/
|   |   |   |   |   |-- route.ts             # Query findings
|   |   |   |   |-- patches/
|   |   |   |   |   |-- route.ts             # Patch operations
|   |   |   |   |   |-- [patchId]/
|   |   |   |   |   |   |-- approve/
|   |   |   |   |   |   |   |-- route.ts     # Human approval endpoint
|   |   |   |   |-- reports/
|   |   |   |   |   |-- generate/
|   |   |   |   |   |   |-- route.ts         # Report generation
|   |   |-- components/
|   |   |   |-- ui/                          # shadcn/ui components
|   |   |   |   |-- button.tsx
|   |   |   |   |-- card.tsx
|   |   |   |   |-- badge.tsx
|   |   |   |   |-- table.tsx
|   |   |   |   |-- dialog.tsx
|   |   |   |   |-- tabs.tsx
|   |   |   |   |-- progress.tsx
|   |   |   |   |-- toast.tsx
|   |   |   |-- shield/                      # Custom SHIELD components
|   |   |   |   |-- risk-gauge.tsx           # Animated risk score gauge
|   |   |   |   |-- vuln-feed.tsx            # Real-time vulnerability feed
|   |   |   |   |-- attack-chain-graph.tsx   # React Flow chain visualizer
|   |   |   |   |-- severity-badge.tsx       # Color-coded severity badge
|   |   |   |   |-- scan-controls.tsx        # Start/pause/stop buttons
|   |   |   |   |-- patch-diff-viewer.tsx    # Side-by-side diff display
|   |   |   |   |-- remediation-timeline.tsx # Gantt chart component
|   |   |   |   |-- compliance-matrix.tsx    # SOC2/ISO mapping table
|   |   |   |   |-- metric-card.tsx          # Dashboard metric cards
|   |   |   |   |-- attack-log.tsx           # Live attack log stream
|   |   |   |   |-- navbar.tsx               # Top navigation
|   |   |   |   |-- sidebar.tsx              # Side navigation
|   |   |-- lib/
|   |   |   |-- api-client.ts               # Backend API client
|   |   |   |-- events.ts                   # SSE event handler
|   |   |   |-- stores/
|   |   |   |   |-- scan-store.ts            # Zustand store for scans
|   |   |   |   |-- findings-store.ts        # Zustand store for findings
|   |   |   |-- utils.ts                     # Utility functions
|   |   |   |-- types.ts                     # Frontend type definitions
|   |   |-- public/
|   |   |   |-- shield-logo.svg
|   |   |-- tailwind.config.ts
|   |   |-- next.config.js
|   |   |-- package.json
|   |   |-- tsconfig.json
|
|   |-- demo-target/                         # Deliberately vulnerable demo app
|   |   |-- src/
|   |   |   |-- server.ts                    # Express.js vulnerable server
|   |   |   |-- routes/
|   |   |   |   |-- auth.ts                  # Broken auth (weak JWT, no rate limit)
|   |   |   |   |-- users.ts                 # IDOR + mass assignment
|   |   |   |   |-- search.ts               # SQL injection
|   |   |   |   |-- upload.ts               # Unrestricted file upload
|   |   |   |   |-- admin.ts                # Broken access control
|   |   |   |   |-- comments.ts             # Stored XSS
|   |   |   |   |-- export.ts               # SSRF via URL parameter
|   |   |   |   |-- config.ts               # Exposed config endpoint
|   |   |   |-- middleware/
|   |   |   |   |-- auth.ts                  # Intentionally flawed auth middleware
|   |   |   |-- models/
|   |   |   |   |-- user.ts                  # User model with weak password hashing
|   |   |   |   |-- comment.ts              # Comment model (no sanitization)
|   |   |   |-- database.ts                 # SQLite database setup
|   |   |-- Dockerfile
|   |   |-- package.json
|
|-- packages/
|   |-- agents/                              # AI agent modules
|   |   |-- orchestrator/
|   |   |   |-- index.ts                     # Main orchestrator logic
|   |   |   |-- scan-manager.ts              # Manages scan lifecycle
|   |   |   |-- agent-coordinator.ts         # Coordinates between agents
|   |   |   |-- types.ts
|   |   |-- discovery/
|   |   |   |-- index.ts                     # Attack surface discovery
|   |   |   |-- gcp-inventory.ts             # GCP resource enumeration
|   |   |   |-- web-crawler.ts               # Web application crawling
|   |   |   |-- api-parser.ts                # OpenAPI/Swagger parser
|   |   |   |-- dns-enum.ts                  # DNS enumeration
|   |   |   |-- types.ts
|   |   |-- recon/
|   |   |   |-- index.ts                     # Reconnaissance agent
|   |   |   |-- passive-scan.ts              # Passive information gathering
|   |   |   |-- tech-fingerprint.ts          # Technology detection
|   |   |   |-- osint.ts                     # Open source intelligence
|   |   |   |-- types.ts
|   |   |-- reasoning/
|   |   |   |-- index.ts                     # Vulnerability reasoning engine
|   |   |   |-- gemini-client.ts             # Gemini API client
|   |   |   |-- hypothesis-generator.ts      # Generate attack hypotheses
|   |   |   |-- chain-analyzer.ts            # Attack chain analysis
|   |   |   |-- cvss-scorer.ts               # CVSS scoring
|   |   |   |-- business-impact.ts           # Business impact assessment
|   |   |   |-- prompts/
|   |   |   |   |-- attacker-persona.ts      # System prompt for attacker AI
|   |   |   |   |-- chain-generator.ts       # Chain analysis prompt
|   |   |   |   |-- cvss-scorer.ts           # CVSS scoring prompt
|   |   |   |   |-- impact-assessor.ts       # Business impact prompt
|   |   |   |-- types.ts
|   |   |-- scanner/
|   |   |   |-- index.ts                     # Fast scanning with Gemini Flash
|   |   |   |-- flash-client.ts              # Gemini Flash client
|   |   |   |-- triage.ts                    # Finding triage logic
|   |   |   |-- types.ts
|   |   |-- attacker/
|   |   |   |-- index.ts                     # Attack execution engine
|   |   |   |-- executor.ts                  # Runs attacks in sandbox
|   |   |   |-- payload-builder.ts           # Builds attack payloads
|   |   |   |-- modules/
|   |   |   |   |-- injection.ts             # SQL/NoSQL/Command injection
|   |   |   |   |-- xss.ts                   # Cross-site scripting
|   |   |   |   |-- auth-bypass.ts           # Authentication bypass
|   |   |   |   |-- idor.ts                  # IDOR testing
|   |   |   |   |-- ssrf.ts                  # Server-side request forgery
|   |   |   |   |-- file-upload.ts           # File upload abuse
|   |   |   |   |-- access-control.ts        # Broken access control
|   |   |   |   |-- crypto.ts               # Cryptographic failures
|   |   |   |   |-- misconfig.ts            # Security misconfiguration
|   |   |   |   |-- deserialization.ts      # Insecure deserialization
|   |   |   |-- types.ts
|   |   |-- patcher/
|   |   |   |-- index.ts                     # Patch generation orchestrator
|   |   |   |-- generator.ts                 # Patch generation logic
|   |   |   |-- reviewer.ts                  # AI self-review
|   |   |   |-- validator.ts                 # Patch validation
|   |   |   |-- prompts/
|   |   |   |   |-- patch-generator.ts       # Patch generation prompt
|   |   |   |   |-- patch-reviewer.ts        # Review prompt
|   |   |   |-- types.ts
|   |   |-- zero-day/
|   |   |   |-- index.ts                     # Zero-day predictor
|   |   |   |-- code-analyzer.ts             # Static code analysis
|   |   |   |-- pattern-matcher.ts           # Known vuln pattern matching
|   |   |   |-- types.ts
|   |
|   |-- core/                                # Shared core libraries
|   |   |-- types/
|   |   |   |-- index.ts                     # All shared TypeScript types
|   |   |   |-- vulnerability.ts             # Vulnerability types
|   |   |   |-- attack.ts                    # Attack types
|   |   |   |-- patch.ts                     # Patch types
|   |   |   |-- scan.ts                      # Scan types
|   |   |   |-- report.ts                    # Report types
|   |   |-- config/
|   |   |   |-- index.ts                     # Configuration management
|   |   |   |-- defaults.ts                  # Default configuration values
|   |   |-- secrets/
|   |   |   |-- kms-client.ts                # Cloud KMS integration
|   |   |-- logging/
|   |   |   |-- index.ts                     # Structured logging
|   |   |   |-- audit.ts                     # Audit log writer
|   |   |-- pubsub/
|   |   |   |-- publisher.ts                 # Pub/Sub publisher
|   |   |   |-- subscriber.ts               # Pub/Sub subscriber
|   |   |   |-- topics.ts                   # Topic definitions
|   |   |-- bigquery/
|   |   |   |-- client.ts                    # BigQuery client
|   |   |   |-- queries.ts                  # Common queries
|   |
|   |-- sandbox/                             # Sandboxed execution environment
|   |   |-- lifecycle.ts                     # Container lifecycle management
|   |   |-- safety.ts                        # Blast radius limiter
|   |   |-- audit.ts                         # Attack audit logging
|   |   |-- mirror.ts                        # Traffic mirroring
|   |   |-- types.ts
|   |
|   |-- pipeline/                            # Deployment pipeline
|   |   |-- deployer.ts                      # Cloud Build trigger management
|   |   |-- rollback.ts                      # Rollback mechanism
|   |   |-- verifier.ts                      # Post-deploy verification
|   |   |-- types.ts
|   |
|   |-- integrations/                        # External integrations
|   |   |-- scc/
|   |   |   |-- client.ts                    # Security Command Center
|   |   |-- armor/
|   |   |   |-- rule-generator.ts            # Cloud Armor rules
|   |   |-- chronicle/
|   |   |   |-- client.ts                    # Chronicle SIEM
|
|-- infra/                                   # Infrastructure as Code
|   |-- terraform/
|   |   |-- main.tf                          # Root Terraform config
|   |   |-- variables.tf                     # Input variables
|   |   |-- outputs.tf                       # Output values
|   |   |-- modules/
|   |   |   |-- networking/
|   |   |   |   |-- main.tf                  # VPC, subnets, firewall
|   |   |   |-- cloudrun/
|   |   |   |   |-- main.tf                  # Cloud Run services
|   |   |   |-- bigquery/
|   |   |   |   |-- main.tf                  # BigQuery datasets/tables
|   |   |   |-- pubsub/
|   |   |   |   |-- main.tf                  # Pub/Sub topics/subscriptions
|   |   |   |-- kms/
|   |   |   |   |-- main.tf                  # KMS key rings/keys
|   |   |   |-- iam/
|   |   |   |   |-- main.tf                  # Service accounts and permissions
|   |-- cloudrun/
|   |   |-- attack-executor.yaml             # Attack executor Cloud Run config
|   |   |-- orchestrator.yaml                # Orchestrator service config
|   |   |-- api-server.yaml                  # API server config
|   |-- cloudbuild/
|   |   |-- patch-deploy.yaml                # Patch deployment pipeline
|   |   |-- ci.yaml                          # CI build config
|   |-- bigquery/
|   |   |-- schema.sql                       # BigQuery table schemas
|
|-- docker/
|   |-- Dockerfile.web                       # Frontend Dockerfile
|   |-- Dockerfile.api                       # API server Dockerfile
|   |-- Dockerfile.orchestrator              # Orchestrator Dockerfile
|   |-- Dockerfile.attack-executor           # Attack executor Dockerfile
|   |-- Dockerfile.demo-target               # Demo vulnerable app Dockerfile
|   |-- docker-compose.yaml                  # Local development compose
|   |-- docker-compose.demo.yaml             # Demo environment compose
|
|-- scripts/
|   |-- setup.sh                             # Initial project setup
|   |-- deploy.sh                            # Deploy all services
|   |-- demo.sh                              # Start demo environment
|   |-- seed-demo-data.ts                    # Seed demo vulnerable app
|
|-- docs/
|   |-- architecture.md                      # Architecture documentation
|   |-- api-reference.md                     # API documentation
|   |-- security-model.md                    # Security model documentation
|   |-- demo-guide.md                        # Demo walkthrough
|
|-- package.json                             # Root package.json (workspace)
|-- turbo.json                               # Turborepo configuration
|-- tsconfig.base.json                       # Base TypeScript config
|-- .env.example                             # Environment variables template
|-- .gitignore
|-- README.md
```

---

## SECTION 9: THE HACKATHON DEMO PLAN

### 9.1 Deliberately Vulnerable Demo App

The demo target (`/apps/demo-target/`) is a custom vulnerable web application -- a "SecureBank" mock banking API with intentionally planted vulnerabilities:

| Vulnerability | Location | Severity |
|--------------|----------|----------|
| SQL Injection | `GET /api/search?q=` | Critical |
| Broken Auth (weak JWT secret) | `POST /api/auth/login` | Critical |
| IDOR on user profiles | `GET /api/users/:id` | High |
| Stored XSS in comments | `POST /api/comments` | High |
| Mass Assignment | `PUT /api/users/:id` | High |
| SSRF via export | `POST /api/export?url=` | High |
| Exposed config | `GET /api/config` | Medium |
| Missing rate limiting | `POST /api/auth/login` | Medium |
| Verbose error messages | All error responses | Low |
| Missing security headers | All responses | Low |

### 9.2 Demo Attack Sequence (5 Minutes)

**Minute 0:00 - 0:30: Introduction**
- "Organizations face $10.5 trillion in cybercrime costs. Average breach detection: 287 days. SHIELD changes this."
- Show the dashboard with the target app loaded. Risk score starts at 0 (no scan run yet).

**Minute 0:30 - 1:30: Discovery and Attack**
- Click "Start Scan" on the control panel
- Live feed shows: Asset discovery (endpoints found), Recon (tech stack identified as Express+SQLite), Hypothesis generation (Gemini reasons about potential vulns)
- Watch findings populate in real-time: SQL injection found, XSS found, IDOR found
- Risk score climbs from 0 to 87

**Minute 1:30 - 2:30: Attack Chain**
- SHIELD identifies a chain: Config endpoint leaks DB path -> IDOR reads user data -> SQL injection dumps credentials -> Weak JWT secret allows forging admin tokens
- Show the attack chain graph visualization
- "No existing tool finds this chain. SHIELD reasons about combinations."

**Minute 2:30 - 3:30: Auto-Remediation**
- SHIELD generates patches for the SQL injection (parameterized queries)
- Show the diff view: before (string concatenation) vs. after (parameterized)
- Patch deployed to staging automatically
- Verification attack runs -- SQL injection no longer works
- Risk score drops from 87 to 42

**Minute 3:30 - 4:30: Results**
- Show the executive summary: 10 findings, 3 critical chains, 4 auto-patched, 287 days reduced to 4 minutes
- Show compliance mapping: which SOC2 controls are now passing
- "Before SHIELD: vulnerable for months. After SHIELD: patched in minutes."

**Minute 4:30 - 5:00: Close**
- "SHIELD doesn't replace security teams -- it gives them superpowers."
- Show pricing tiers briefly
- Q&A

### 9.3 Safety Guardrails for the Demo

- Demo app runs entirely in Docker containers on the presenter's machine or in a GCP sandbox project
- No connection to any real infrastructure
- All "attacks" target only the deliberately vulnerable demo app
- Network traffic stays within the Docker network
- Kill switch: single button stops all SHIELD activities immediately

---

## SECTION 10: 24-HOUR BUILD SPRINT PLAN

### 10.1 Hour-by-Hour Breakdown

**PHASE 1: Foundation (Hours 0-4)**

| Hour | Task | Priority |
|------|------|----------|
| 0-1 | Project scaffolding: monorepo, TypeScript, Next.js app, Express API, Turborepo config | P0 |
| 1-2 | Demo vulnerable app: Express server with 3 core vulns (SQLi, XSS, IDOR) | P0 |
| 2-3 | Gemini API integration: client setup, attacker persona prompt, basic hypothesis generation | P0 |
| 3-4 | Basic frontend: dashboard layout, sidebar nav, scan controls using shadcn/ui | P0 |

**PHASE 2: Core Engine (Hours 4-10)**

| Hour | Task | Priority |
|------|------|----------|
| 4-5 | Attack surface discovery: web crawling, endpoint enumeration | P0 |
| 5-6 | Vulnerability reasoning: Gemini generates hypotheses from discovered endpoints | P0 |
| 6-7 | Attack executor: HTTP-based attack payloads (SQLi, XSS) with result parsing | P0 |
| 7-8 | Real-time feed: SSE from backend to frontend, live vulnerability display | P0 |
| 8-9 | Patch generation: Vertex AI Code generates patches for confirmed findings | P0 |
| 9-10 | Integration testing: end-to-end flow from scan to finding to patch | P0 |

**PHASE 3: Advanced Features (Hours 10-16)**

| Hour | Task | Priority |
|------|------|----------|
| 10-11 | Attack chain analyzer: Gemini chains findings, graph data model | P1 |
| 11-12 | Chain visualization: React Flow interactive graph on frontend | P1 |
| 12-13 | Patch deployment pipeline: apply patch to demo app, verify fix | P1 |
| 13-14 | Verification loop: re-attack after patching, confirm fix | P1 |
| 14-15 | Risk scoring: CVSS automation, aggregate risk score on dashboard | P1 |
| 15-16 | Executive report generation: AI-written summary with metrics | P1 |

**PHASE 4: Polish and Demo Prep (Hours 16-22)**

| Hour | Task | Priority |
|------|------|----------|
| 16-17 | Dashboard polish: risk gauge animation, metric cards, charts | P2 |
| 17-18 | Add remaining vuln types to demo app (SSRF, mass assignment, config exposure) | P2 |
| 18-19 | Compliance mapping: SOC2 control mapping for findings | P2 |
| 19-20 | Docker Compose setup: one-command demo launch | P2 |
| 20-21 | End-to-end demo rehearsal: run through 5-minute script 3x | P0 |
| 21-22 | Bug fixes and polish from rehearsal | P0 |

**PHASE 5: Final Prep (Hours 22-24)**

| Hour | Task | Priority |
|------|------|----------|
| 22-23 | Record backup demo video in case of live demo failure | P0 |
| 23-24 | Final rehearsal, pitch deck review, sleep | P0 |

### 10.2 MVP vs. Stretch Goals

**MVP (Must have for demo)**:
- Vulnerable demo app with 3+ vulns
- Gemini-powered attack reasoning (hypothesis generation)
- At least 2 attack types executing (SQLi + XSS)
- Real-time finding display on frontend
- Patch generation for at least 1 vuln type
- Basic dashboard with risk score

**Stretch Goals**:
- Attack chain visualization
- Automated patch deployment and verification
- Compliance mapping
- Executive report generation
- Cloud Armor rule generation
- Zero-day predictor

---

## SECTION 11: TECHNICAL RISKS AND MITIGATIONS

### 11.1 Responsible Disclosure

SHIELD operates ONLY on authorized targets. The system enforces this through:
- **Mandatory scope configuration**: No scan can start without an explicit target whitelist
- **Scope validation at every layer**: Orchestrator, executor, and network layer all validate targets
- **Authorization tokens**: Target applications must deploy a SHIELD verification token (similar to Google Search Console verification) before SHIELD will attack them
- **Legal documentation**: Built-in authorization template generation for signed penetration testing agreements

### 11.2 False Positive Management

- **Confidence scoring**: Every finding includes an AI confidence score (0-1)
- **Multi-stage confirmation**: Hypothesis -> test payload -> confirm response -> classify
- **Human review queue**: Findings below 0.7 confidence are queued for human review
- **Feedback loop**: Human-confirmed false positives are fed back to improve future reasoning

### 11.3 Patch Safety

- **AI self-review**: Every patch is reviewed by a separate Gemini instance
- **Existing test suite must pass**: Patches that break existing tests are rejected
- **Staged rollout**: staging -> integration tests -> production
- **Automatic rollback**: If post-deploy health checks fail, immediate rollback
- **Maximum patch scope**: Patches touching more than 3 files require human approval

### 11.4 Sandbox Escape Prevention

- **gVisor runtime**: Cloud Run uses gVisor for kernel-level sandboxing
- **No production network access**: Sandbox VPC has zero routes to production
- **Resource limits**: CPU, memory, time, and request count hard limits
- **Container destruction**: Containers are destroyed immediately after execution
- **No persistent storage**: Attack containers have no writable persistent volumes

### 11.5 Legal Considerations

- Written authorization required before any testing
- Scope strictly limited to authorized assets
- All attack activities are immutably logged
- Compliance with CFAA (Computer Fraud and Abuse Act) requirements
- Data handling compliant with GDPR/CCPA if personal data is encountered
- Clear terms of service establishing the authorized testing relationship

### 11.6 Rate Limiting and Resource Management

```typescript
// /packages/core/config/defaults.ts

export const RESOURCE_LIMITS = {
  gemini: {
    maxRequestsPerMinute: 60,
    maxTokensPerMinute: 1_000_000,
    maxConcurrentRequests: 10,
    retryBackoffMs: [1000, 2000, 4000, 8000, 16000],
  },
  attacks: {
    maxConcurrentScans: 3,
    maxConcurrentAttacksPerScan: 5,
    maxRequestsPerSecondPerTarget: 10,
    cooldownBetweenAttacksMs: 1000,
  },
  patches: {
    maxConcurrentGenerations: 3,
    maxPatchSizeLines: 500,
    maxAffectedFiles: 3,
    requireHumanApprovalAboveCVSS: 9.0,
  },
  cloudBuild: {
    maxConcurrentBuilds: 2,
    buildTimeoutSeconds: 600,
    maxRetries: 2,
  },
};
```

---

## SECTION 12: BUSINESS MODEL

### 12.1 Pricing Tiers

| Tier | Price | Target | Features |
|------|-------|--------|----------|
| **Starter** | $999/mo | Startups, small teams | 1 application, weekly scans, basic reporting, email alerts |
| **Professional** | $4,999/mo | Mid-market | 10 applications, daily scans, attack chains, auto-patching (staging only), compliance reports |
| **Enterprise** | $14,999/mo | Large enterprises | Unlimited apps, continuous scanning, full auto-remediation, SCC/Chronicle integration, custom attack modules, dedicated support |
| **MSSP** | Custom | Managed security providers | Multi-tenant, white-label, API access, volume licensing |

### 12.2 Revenue Streams

1. **SaaS Subscriptions**: Primary revenue from monthly/annual subscriptions
2. **Compliance Automation**: SOC2, ISO 27001, PCI-DSS continuous compliance monitoring add-on ($1,999/mo)
3. **MSSP Partnerships**: Revenue share with managed security providers who resell SHIELD
4. **Cyber Insurance Integration**: Premium discounts for policyholders using SHIELD (partnership revenue)
5. **Professional Services**: Custom attack module development, integration consulting ($250/hr)

### 12.3 Market Sizing

- Global penetration testing market: $3.1B (2025), growing 13.8% CAGR
- Global cybersecurity market: $266B (2025)
- Target addressable market (automated pen testing + remediation): $8B by 2028
- Initial target: 500 mid-market companies at $5K/mo = $30M ARR within 3 years

---

## SECTION 13: JUDGING CRITERIA AND PITCH STRUCTURE

### 13.1 Scoring Strategy

| Criterion | How SHIELD Scores Well |
|-----------|----------------------|
| **Innovation** | First autonomous red-team AI that reasons about novel attacks (not pattern matching). The attack chain discovery and automated remediation loop are unprecedented. |
| **Technical Complexity** | Multi-agent AI system, sandboxed execution environment, automated CI/CD integration, real-time visualization, 10+ Google Cloud APIs integrated. |
| **Impact** | Directly addresses $10.5T cybercrime problem. Reduces 287-day detection to minutes. Makes enterprise security accessible to smaller companies. |
| **Feasibility** | Working demo with real vulnerable app. All tech choices are production-grade. Clear path from hackathon to product. |
| **Google Cloud Usage** | Deep integration: Gemini 2.5 Pro/Flash, Vertex AI, Cloud Run, Cloud Build, BigQuery, Cloud KMS, Security Command Center, Cloud Armor. |

### 13.2 3-Minute Pitch Structure

**0:00 - 0:30: The Problem (Shock and Awe)**
"Cybercrime costs $10.5 trillion annually. The average breach goes undetected for 287 days. Human red teams cost $500K and cover 15% of your attack surface once a year. Traditional scanners match patterns; they cannot think. We built something that can."

**0:30 - 1:00: The Solution (What SHIELD Is)**
"SHIELD is an autonomous red-team AI that continuously attacks your infrastructure, finds vulnerabilities before malicious hackers, generates patches, deploys them, and verifies the fix. It uses Gemini to think like an attacker, not run scripts."

**1:00 - 2:15: The Demo (Show, Don't Tell)**
Live demo: Start scan on vulnerable app. Show findings appearing in real-time. Show attack chain visualization. Show auto-generated patch. Show verification. Risk score drops from 87 to 42.

**2:15 - 2:45: The Business (Why This Wins)**
"Starting at $999/month -- less than one day of a human penetration tester. We target the $3.1B pen testing market. Our unfair advantage: every scan makes SHIELD smarter. Network effects in security intelligence."

**2:45 - 3:00: The Close**
"287 days is unacceptable. SHIELD makes it 287 seconds. We are the immune system your infrastructure needs."

---

## SECTION 14: APPENDICES

### 14.1 Complete Attack Taxonomy

```typescript
// /packages/core/types/attack.ts

export enum AttackCategory {
  // OWASP Top 10:2025
  BROKEN_ACCESS_CONTROL = 'A01:2025',
  SECURITY_MISCONFIGURATION = 'A02:2025',
  SUPPLY_CHAIN_FAILURES = 'A03:2025',
  CRYPTOGRAPHIC_FAILURES = 'A04:2025',
  INJECTION = 'A05:2025',
  INSECURE_DESIGN = 'A06:2025',
  AUTHENTICATION_FAILURES = 'A07:2025',
  SOFTWARE_DATA_INTEGRITY = 'A08:2025',
  LOGGING_MONITORING_FAILURES = 'A09:2025',
  EXCEPTIONAL_CONDITIONS = 'A10:2025',
  
  // Cloud-specific
  IAM_MISCONFIGURATION = 'CLOUD-01',
  SERVICE_ACCOUNT_ABUSE = 'CLOUD-02',
  METADATA_ENDPOINT_EXPLOIT = 'CLOUD-03',
  STORAGE_BUCKET_EXPOSURE = 'CLOUD-04',
  CONTAINER_ESCAPE = 'CLOUD-05',
}

export enum AttackTechnique {
  // Injection subtypes
  SQL_INJECTION_UNION = 'INJ-001',
  SQL_INJECTION_BLIND_BOOLEAN = 'INJ-002',
  SQL_INJECTION_BLIND_TIME = 'INJ-003',
  SQL_INJECTION_ERROR_BASED = 'INJ-004',
  NOSQL_INJECTION = 'INJ-005',
  COMMAND_INJECTION = 'INJ-006',
  LDAP_INJECTION = 'INJ-007',
  TEMPLATE_INJECTION = 'INJ-008',
  HEADER_INJECTION = 'INJ-009',
  
  // XSS subtypes
  XSS_REFLECTED = 'XSS-001',
  XSS_STORED = 'XSS-002',
  XSS_DOM_BASED = 'XSS-003',
  
  // Auth subtypes
  CREDENTIAL_STUFFING = 'AUTH-001',
  SESSION_FIXATION = 'AUTH-002',
  JWT_NONE_ALGORITHM = 'AUTH-003',
  JWT_WEAK_SECRET = 'AUTH-004',
  MFA_BYPASS = 'AUTH-005',
  PASSWORD_RESET_POISONING = 'AUTH-006',
  
  // Access control
  IDOR = 'AC-001',
  PRIVILEGE_ESCALATION = 'AC-002',
  FORCED_BROWSING = 'AC-003',
  CORS_MISCONFIGURATION = 'AC-004',
  
  // SSRF
  SSRF_BASIC = 'SSRF-001',
  SSRF_CLOUD_METADATA = 'SSRF-002',
  SSRF_INTERNAL_SERVICE = 'SSRF-003',
  
  // Others
  FILE_UPLOAD_UNRESTRICTED = 'FILE-001',
  DESERIALIZATION = 'DES-001',
  RACE_CONDITION = 'RACE-001',
  BUSINESS_LOGIC_BYPASS = 'BIZ-001',
  RATE_LIMIT_BYPASS = 'RATE-001',
  INFORMATION_DISCLOSURE = 'INFO-001',
  OPEN_REDIRECT = 'REDIR-001',
}
```

### 14.2 Sample Vulnerability Report Format

```typescript
// /packages/core/types/report.ts

interface VulnerabilityReport {
  metadata: {
    reportId: string;
    generatedAt: string;
    scanId: string;
    targetApplication: string;
    scanDuration: string;
    shieldVersion: string;
  };
  
  executiveSummary: {
    overallRiskScore: number;     // 0-100
    riskLevel: 'CRITICAL' | 'HIGH' | 'MEDIUM' | 'LOW';
    totalFindings: number;
    criticalFindings: number;
    highFindings: number;
    mediumFindings: number;
    lowFindings: number;
    attackChainsDiscovered: number;
    patchesGenerated: number;
    patchesVerified: number;
    estimatedFinancialRisk: string;
    summaryNarrative: string;     // AI-generated natural language summary
  };
  
  findings: Array<{
    id: string;
    title: string;
    severity: string;
    cvssScore: number;
    cvssVector: string;
    owaspCategory: string;
    cweId: string;
    description: string;
    affectedEndpoint: string;
    proofOfConcept: string;
    businessImpact: string;
    remediation: {
      recommendation: string;
      patchId: string | null;
      patchStatus: string;
      estimatedEffort: string;
    };
    references: string[];
  }>;
  
  attackChains: Array<{
    id: string;
    name: string;
    aggregateSeverity: string;
    steps: Array<{
      order: number;
      findingId: string;
      description: string;
      exploitationDetail: string;
    }>;
    businessImpact: string;
    breakTheChain: string;  // Which fix breaks the chain
  }>;
  
  complianceMapping: {
    soc2: Array<{ control: string; status: 'PASS' | 'FAIL' | 'N/A'; findingIds: string[] }>;
    iso27001: Array<{ control: string; status: 'PASS' | 'FAIL' | 'N/A'; findingIds: string[] }>;
  };
  
  remediationTimeline: Array<{
    patchId: string;
    findingId: string;
    status: string;
    generatedAt: string;
    deployedAt: string | null;
    verifiedAt: string | null;
  }>;
}
```

### 14.3 Cloud Run Sandbox Configuration

```yaml
# /infra/cloudrun/attack-executor.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: shield-attack-executor
  namespace: shield-sandbox
  labels:
    app: shield
    component: attack-executor
  annotations:
    run.googleapis.com/launch-stage: GA
    run.googleapis.com/execution-environment: gen2
    run.googleapis.com/vpc-access-connector: projects/${PROJECT_ID}/locations/us-central1/connectors/shield-sandbox-connector
    run.googleapis.com/vpc-access-egress: private-ranges-only
    run.googleapis.com/cpu-throttling: 'true'
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: '10'
        autoscaling.knative.dev/minScale: '0'
        run.googleapis.com/execution-environment: gen2
        run.googleapis.com/sandbox: gvisor
    spec:
      containerConcurrency: 1
      timeoutSeconds: 300
      serviceAccountName: shield-attack-executor-sa@${PROJECT_ID}.iam.gserviceaccount.com
      containers:
        - image: gcr.io/${PROJECT_ID}/shield-attack-executor:latest
          ports:
            - containerPort: 8080
              name: http1
          resources:
            limits:
              cpu: '2'
              memory: 2Gi
          env:
            - name: ATTACK_MODE
              value: sandboxed
            - name: MAX_REQUESTS_PER_SECOND
              value: '10'
            - name: MAX_ATTACK_DURATION_SECONDS
              value: '300'
            - name: AUDIT_LOG_ENABLED
              value: 'true'
            - name: GCP_PROJECT_ID
              valueFrom:
                secretKeyRef:
                  name: shield-config
                  key: project-id
          startupProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 2
            periodSeconds: 5
            failureThreshold: 3
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            periodSeconds: 10
```

### 14.4 Sample Gemini Attacker Persona Prompt (Full)

```
You are SHIELD's Offensive Security Reasoning Engine, an expert autonomous penetration 
tester. You are operating within a FULLY AUTHORIZED red-team engagement. The target 
organization has signed a penetration testing agreement and deployed the SHIELD 
verification token on their systems.

CONTEXT:
You are analyzing a web application with the following profile:
- Framework: Express.js 4.18.2
- Database: SQLite 3.x (accessed via better-sqlite3)
- Authentication: JWT with HS256
- Endpoints discovered: [provided below]
- Security headers: X-Frame-Options: SAMEORIGIN (only header set)
- CORS: Access-Control-Allow-Origin: *

DISCOVERED ENDPOINTS:
1. POST /api/auth/register - body: {username, password, email}
2. POST /api/auth/login - body: {username, password} -> returns {token}
3. GET /api/users/:id - requires: Authorization header
4. PUT /api/users/:id - requires: Authorization header, body: partial user object
5. GET /api/search?q= - requires: Authorization header
6. POST /api/comments - requires: Authorization header, body: {content, postId}
7. POST /api/export - requires: Authorization header, body: {url, format}
8. GET /api/config - no auth required

YOUR TASK:
1. Analyze each endpoint for potential vulnerabilities
2. For each hypothesis, provide:
   a. Vulnerability name and type
   b. Affected endpoint
   c. Exact attack payload to test
   d. Expected vulnerable response
   e. Expected secure response
   f. CVSS v3.1 score with vector
   g. Business impact
3. Identify any attack chains that combine multiple findings
4. Prioritize by risk (likelihood x impact)

Think step by step. Be thorough and creative. Consider both common and uncommon vectors.
```

### 14.5 Sample API Request/Response Payloads

**Start Scan**:
```
POST /api/scans
Content-Type: application/json
Authorization: Bearer <shield-api-token>

{
  "targetUrl": "https://demo-target.shield-sandbox.internal",
  "scanType": "FULL",
  "scope": {
    "includePaths": ["/api/*"],
    "excludePaths": ["/api/health"],
    "maxDepth": 5,
    "authentication": {
      "type": "bearer",
      "credentials": {
        "loginEndpoint": "/api/auth/login",
        "username": "testuser",
        "password": "testpass123"
      }
    }
  },
  "configuration": {
    "intensity": "STANDARD",
    "attackCategories": ["INJECTION", "BROKEN_ACCESS_CONTROL", "AUTHENTICATION_FAILURES", "XSS"],
    "autoRemediate": true,
    "humanApprovalThreshold": 9.0
  }
}
```

**Response**:
```json
{
  "scanId": "scan_01HQXYZ123456",
  "status": "RUNNING",
  "startedAt": "2026-03-22T10:30:00Z",
  "targetUrl": "https://demo-target.shield-sandbox.internal",
  "estimatedDuration": "PT15M",
  "phases": [
    {"phase": "DISCOVERY", "status": "IN_PROGRESS"},
    {"phase": "RECONNAISSANCE", "status": "PENDING"},
    {"phase": "HYPOTHESIS_GENERATION", "status": "PENDING"},
    {"phase": "ATTACK_EXECUTION", "status": "PENDING"},
    {"phase": "CHAIN_ANALYSIS", "status": "PENDING"},
    {"phase": "REMEDIATION", "status": "PENDING"},
    {"phase": "VERIFICATION", "status": "PENDING"}
  ],
  "eventsUrl": "/api/events/stream?scanId=scan_01HQXYZ123456"
}
```

**Finding Created Event (SSE)**:
```
event: finding
data: {
  "findingId": "finding_01HQABC789",
  "scanId": "scan_01HQXYZ123456",
  "title": "SQL Injection in Search Endpoint",
  "severity": "CRITICAL",
  "cvssScore": 9.8,
  "cvssVector": "CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H",
  "owaspCategory": "A05:2025",
  "cweId": "CWE-89",
  "affectedEndpoint": "GET /api/search?q=",
  "proofOfConcept": "GET /api/search?q=' OR 1=1--",
  "description": "The search parameter is concatenated directly into a SQL query without parameterization, allowing an attacker to inject arbitrary SQL commands.",
  "businessImpact": "An attacker can read, modify, or delete all data in the database, including user credentials and sensitive business data.",
  "remediationStatus": "OPEN",
  "discoveredAt": "2026-03-22T10:31:45Z"
}
```

**Patch Generated Event (SSE)**:
```
event: patch_generated
data: {
  "patchId": "patch_01HQDEF456",
  "findingId": "finding_01HQABC789",
  "patchType": "CODE_FIX",
  "confidence": 0.95,
  "affectedFiles": ["src/routes/search.ts"],
  "diff": "--- a/src/routes/search.ts\n+++ b/src/routes/search.ts\n@@ -12,7 +12,8 @@\n router.get('/search', requireAuth, (req, res) => {\n   const query = req.query.q as string;\n-  const results = db.prepare(`SELECT * FROM products WHERE name LIKE '%${query}%'`).all();\n+  const stmt = db.prepare('SELECT * FROM products WHERE name LIKE ?');\n+  const results = stmt.all(`%${query}%`);\n   res.json({ results });\n });",
  "testCode": "describe('Search endpoint', () => {\n  it('should prevent SQL injection', async () => {\n    const res = await request(app)\n      .get(\"/api/search?q=' OR 1=1--\")\n      .set('Authorization', `Bearer ${token}`);\n    expect(res.body.results).toHaveLength(0);\n  });\n});",
  "reviewStatus": "APPROVED",
  "generatedAt": "2026-03-22T10:32:10Z"
}
```

---

## Summary of Google Cloud API Usage

| Google Cloud Service | Purpose in SHIELD | API/Model |
|---------------------|-------------------|-----------|
| Gemini 2.5 Pro | Attack reasoning, hypothesis generation, chain analysis | `gemini-2.5-pro` via Vertex AI |
| Gemini 2.5 Flash | Fast triage, log analysis, rapid scanning | `gemini-2.5-flash` via Vertex AI |
| Vertex AI (Code) | Patch generation, code review | `gemini-2.5-pro` with code prompts |
| Cloud Run | Sandboxed attack execution, API server, orchestrator | Gen2 execution environment with gVisor |
| Cloud Build | Automated patch deployment pipeline | `cloudbuild.yaml` triggers |
| BigQuery | Vulnerability database, analytics, historical trends | Partitioned/clustered tables |
| Cloud Pub/Sub | Event-driven communication between agents | Topics per data flow |
| Cloud KMS | Encryption of secrets and sensitive findings | AES-256-GCM keys |
| Security Command Center | Bidirectional finding sync | SCC v2 API |
| Cloud Armor | Automated WAF rule generation | Security Policy API |
| Artifact Registry | Container image storage | Docker registry |
| Cloud Logging | Immutable audit logs | Structured logging API |
| VPC / Firewall | Network isolation for sandbox | VPC Connector + firewall rules |
| Cloud Deploy | Staged rollout management | Delivery pipeline API |

---

### Critical Files for Implementation

- `/Users/shauryapunj/Desktop/009/packages/agents/reasoning/gemini-client.ts` - Core Gemini integration for attack reasoning; the central intelligence of the entire system
- `/Users/shauryapunj/Desktop/009/packages/agents/reasoning/prompts/attacker-persona.ts` - The attacker persona system prompt that defines how Gemini thinks about vulnerabilities
- `/Users/shauryapunj/Desktop/009/packages/agents/orchestrator/index.ts` - Main orchestrator that coordinates all agents through the discover-attack-patch-verify loop
- `/Users/shauryapunj/Desktop/009/apps/demo-target/src/server.ts` - The deliberately vulnerable demo app that proves SHIELD works during the hackathon demo
- `/Users/shauryapunj/Desktop/009/apps/web/app/dashboard/page.tsx` - Primary dashboard showing real-time risk score, findings feed, and scan controls

---

**Sources:**
- [Google Vertex AI Models Documentation](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/models)
- [Cloud Run Security Design Overview](https://docs.cloud.google.com/run/docs/securing/security)
- [gVisor Container Security Platform](https://gvisor.dev/)
- [Deploying to Cloud Run using Cloud Build](https://docs.cloud.google.com/build/docs/deploying-builds/deploy-cloud-run)
- [Security Command Center Documentation](https://cloud.google.com/security-command-center/docs)
- [Security Command Center Vulnerability Findings](https://docs.cloud.google.com/security-command-center/docs/concepts-vulnerabilities-findings)
- [OWASP Top 10:2025](https://owasp.org/Top10/2025/)
- [OWASP Top 10 2025 Key Changes](https://about.gitlab.com/blog/2025-owasp-top-10-whats-changed-and-why-it-matters/)
- [Cloud Build Serverless CI/CD Platform](https://cloud.google.com/build)
- [GKE Sandbox with gVisor](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/sandbox-pods)