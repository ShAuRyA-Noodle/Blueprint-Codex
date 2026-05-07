# SYMPHONY: Creative AI Agent Swarm System — Production Master Blueprint

> **Tagline:** Describe a creative vision — a film, a game, a music album, a novel, a marketing campaign. SYMPHONY deploys a swarm of 20 specialized creative AI agents that collaborate, argue, and synthesize your vision into a complete, professional-grade creative product in one hour.

---

## SECTION 1: PROBLEM DEFINITION & VISION

### The Problem
Creative development is the most expensive, time-consuming, and gatekept phase of any creative industry. A film pitch takes 3-6 months and $50K+ in development. A game design document takes weeks. An album concept package requires hiring multiple specialists. Independent creators are locked out — not by lack of talent, but by lack of team.

Existing AI tools (ChatGPT, Midjourney, Suno) are single-agent, single-modality tools. They produce isolated outputs — a script OR images OR music — with no cross-pollination, no internal tension, no quality control. The outputs feel generated, not developed.

### The Breakthrough
SYMPHONY is the first **multi-agent creative collaboration system**. It doesn't generate — it *develops*. Twenty specialized AI agents with distinct creative perspectives argue, challenge, and synthesize a user's vision into a cohesive, professional-grade creative package. The key insight: **the best creative work comes from productive disagreement, not from a single mind.**

### Market Sizing
- **Independent filmmakers**: 500K+ globally, spending avg $2K on pitch development → $1B TAM
- **Game developers (indie)**: 300K+ studios, design document creation → $600M TAM
- **Ad agencies & creative studios**: 120K agencies globally, campaign concepting → $2.4B TAM
- **Authors & screenwriters**: 2M+ active, pitch/synopsis packages → $400M TAM
- **Total Addressable Market**: ~$4.4B in creative development services

### 3-Year Vision
- **Year 1**: Launch MVP. Film + game verticals. 10K users, $500K ARR. Win hackathons, get press.
- **Year 2**: Add music album, novel, campaign verticals. Custom agent marketplace. Studio partnerships. 100K users, $5M ARR.
- **Year 3**: Real-time collaborative mode (human + AI agents in loop). Video generation integration (Veo/Sora). Enterprise white-label. 500K users, $25M ARR.

### Existing Tool Gaps

| Tool | What It Does | What It Lacks |
|------|-------------|---------------|
| ChatGPT | Single-agent text generation | No multi-agent debate, no multimodal output, no creative tension |
| Midjourney/DALL-E | Image generation | No narrative context, no style consistency across 20+ images |
| Suno/Udio | Music generation | No sync to visual/narrative arc, no scoring concept |
| Jasper/Copy.ai | Marketing copy | No world-building, no character depth, no holistic creative package |
| **SYMPHONY** | **20-agent creative swarm** | **The whole package: narrative + visuals + music + voice + pitch deck, with built-in creative conflict** |

---

## SECTION 2: CORE TECHNICAL ARCHITECTURE

### Tech Stack
- **Frontend**: Next.js 14+ (App Router), TypeScript, Tailwind CSS, Framer Motion
- **Backend**: Python FastAPI microservice on Google Cloud Run
- **AI Models**: Gemini 1.5 Pro (director + key agents), Gemini 1.5 Flash (16 agents + conflicts)
- **Media**: Imagen 3 (storyboards), Vertex AI MusicLM/Lyria (music), Google Cloud TTS (voice acting)
- **Infrastructure**: Firebase (Auth + Firestore), Google Cloud Storage, Cloud Run
- **Real-time**: Server-Sent Events (SSE) for live dashboard

### Base Agent Interface

```python
# services/orchestrator/agents/base_agent.py
class SymphonyAgent:
    agent_id: str
    role: str
    wave: int                    # deployment wave (0-4)
    dependencies: list[str]      # agent_ids this agent needs output from
    model: str                   # "gemini-1.5-pro" or "gemini-1.5-flash"
    google_apis: list[str]       # additional Google APIs called
    temperature: float           # creativity dial
    max_output_tokens: int
    system_prompt: str

    async def execute(self, context: AgentContext) -> AgentOutput: ...
    async def critique(self, target_output: AgentOutput) -> CritiqueOutput: ...
    async def revise(self, critiques: list[CritiqueOutput]) -> AgentOutput: ...
```

### THE 20 AGENTS — Complete Specifications

---

#### Agent 1: Creative Director
| Field | Value |
|-------|-------|
| **agent_id** | `creative_director` |
| **Model** | Gemini 1.5 Pro |
| **Wave** | 0 (runs first, then again in arbitration rounds) |
| **Dependencies** | User brief only (wave 0); all agent outputs (arbitration) |
| **Google APIs** | Gemini 1.5 Pro API |
| **Temperature** | 0.7 |

**System Prompt:**
```
You are the Creative Director of SYMPHONY. You receive a raw creative brief from a human
and transform it into a structured creative vision document. You set the tone, genre
boundaries, target audience, emotional arc, and success criteria. In arbitration rounds,
you resolve conflicts between agents by weighing artistic merit, coherence, and the
original vision. You never compromise on the core emotional truth of the project. When
agents disagree, you choose the bolder option unless it breaks continuity.
```

**Input Schema:**
```typescript
interface CreativeDirectorInput {
  brief: string;
  project_type: "film" | "game" | "album" | "novel" | "campaign";
  conflict_queue?: ConflictItem[];
  agent_outputs?: Record<string, AgentOutput>;
}
```

**Output Schema:**
```typescript
interface CreativeDirectorOutput {
  vision_document: {
    title_working: string;
    logline: string;
    genre_primary: string;
    genre_secondary: string[];
    tone_keywords: string[];        // e.g. ["melancholic", "defiant", "neon"]
    target_audience: AudienceProfile;
    emotional_arc: EmotionalBeat[]; // 5-7 beats
    thematic_pillars: string[];     // 3 core themes
    success_criteria: string[];
    constraints: string[];
    reference_works: string[];
  };
  arbitration_decisions?: ArbitrationDecision[];
  creative_tension_score?: number;  // 0-100
}

interface EmotionalBeat {
  beat_index: number;
  label: string;
  emotion: string;
  intensity: number;  // 0.0-1.0
  description: string;
}
```

---

#### Agent 2: Worldbuilder
| Field | Value |
|-------|-------|
| **agent_id** | `worldbuilder` |
| **Model** | Gemini 1.5 Pro |
| **Wave** | 1 |
| **Dependencies** | `creative_director` |
| **Google APIs** | Gemini 1.5 Pro, Imagen 3 (concept maps) |
| **Temperature** | 0.85 |

**System Prompt:**
```
You are a Worldbuilder. You construct the physical, cultural, economic, and metaphysical
rules of the creative universe. Every detail must be internally consistent. You define
geography, technology level, social structures, history, and the "rules of magic"
(literal or metaphorical). You push for specificity — not "a dystopian city" but
"a vertical city built into a dried-up reservoir in Nevada, 2089, where water rights
determine social class." Challenge the Narrative Architect if the plot requires
world-breaking exceptions.
```

**Output Schema:**
```typescript
interface WorldbuilderOutput {
  world: {
    name: string;
    time_period: string;
    locations: Location[];        // 3-8 key locations
    rules: WorldRule[];
    social_structure: string;
    economy: string;
    history_timeline: HistoryEvent[];
    sensory_palette: {
      dominant_colors: string[];
      sounds: string[];
      textures: string[];
      smells: string[];
    };
    unique_terminology: Record<string, string>;
  };
  concept_image_prompts: string[];
}
```

---

#### Agent 3: Character Psychologist
| Field | Value |
|-------|-------|
| **agent_id** | `character_psychologist` |
| **Model** | Gemini 1.5 Pro |
| **Wave** | 1 |
| **Dependencies** | `creative_director` |
| **Google APIs** | Gemini 1.5 Pro |
| **Temperature** | 0.8 |

**System Prompt:**
```
You are a Character Psychologist. You create characters with genuine psychological depth —
attachment styles, defense mechanisms, core wounds, and unconscious desires that differ
from their stated goals. Every character must have an internal contradiction that drives
their arc. You use frameworks from Jungian archetypes, Enneagram, and modern trauma theory.
You will fight the Dialogue Writer if their lines don't match the character's psychological
profile. A character who experienced abandonment does NOT speak in neat, articulate
paragraphs about their feelings.
```

**Output Schema:**
```typescript
interface CharacterPsychologistOutput {
  characters: Character[];  // 3-7 characters
  relationship_map: Relationship[];
  ensemble_dynamics: string;
}

interface Character {
  name: string;
  role: "protagonist" | "antagonist" | "deuteragonist" | "mentor" | "trickster" | "support";
  age: number;
  appearance_brief: string;
  archetype: string;
  enneagram: string;
  core_wound: string;
  conscious_desire: string;
  unconscious_need: string;
  defense_mechanisms: string[];
  speech_pattern: {
    vocabulary_level: "simple" | "moderate" | "elaborate";
    sentence_structure: string;
    verbal_tics: string[];
    topics_they_avoid: string[];
  };
  arc_summary: string;
  internal_contradiction: string;
  casting_notes: string;
}
```

---

#### Agent 4: Narrative Architect
| Field | Value |
|-------|-------|
| **agent_id** | `narrative_architect` |
| **Model** | Gemini 1.5 Pro |
| **Wave** | 2 |
| **Dependencies** | `creative_director`, `worldbuilder`, `character_psychologist` |
| **Google APIs** | Gemini 1.5 Pro |
| **Temperature** | 0.75 |

**System Prompt:**
```
You are the Narrative Architect. You construct plot structure using a hybrid of the
Save The Cat beat sheet, Kishotenketsu, and Dan Harmon's Story Circle — choosing the
framework that best serves the project type. You receive the world and characters and
weave them into a tight, surprising narrative. Every scene must serve at least two
purposes (advance plot + reveal character, or build world + raise stakes). You challenge
the Worldbuilder if the world's rules don't serve the story, and the Character
Psychologist if character arcs feel predetermined rather than earned.
```

**Output Schema:**
```typescript
interface NarrativeArchitectOutput {
  structure_framework: string;
  acts: Act[];                   // 3-5 acts
  scenes: Scene[];               // 15-25 scenes
  plot_threads: PlotThread[];
  twists: Twist[];
  thematic_resolution: string;
}

interface Scene {
  scene_id: string;
  act: number;
  sequence: number;
  title: string;
  location: string;
  characters_present: string[];
  time_of_day: string;
  weather_mood: string;
  scene_goal: string;
  conflict: string;
  outcome: string;
  emotional_beat: string;
  storyboard_prompt: string;     // detailed visual description for Imagen 3
  duration_estimate: string;
  dialogue_seeds: string[];
}
```

---

#### Agent 5: Cinematographer
| Field | Value |
|-------|-------|
| **agent_id** | `cinematographer` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 2 |
| **Dependencies** | `creative_director`, `worldbuilder`, `visual_style_director` |
| **Google APIs** | Gemini 1.5 Flash, Imagen 3 |
| **Temperature** | 0.7 |

**System Prompt:**
```
You are a Cinematographer. You think in frames, lenses, and light. For every scene, you
specify camera placement, movement, lens choice (and why), lighting setup, and color
temperature. You reference real cinematographers (Roger Deakins, Hoyte van Hoytema,
Rachel Morrison) when describing your approach. You argue with the Visual Style Director
about color grading choices and with the Production Designer about set depth and practicals.
Your storyboard notes must be precise enough for Imagen 3 to generate accurate frames.
```

**Output Schema:**
```typescript
interface CinematographerOutput {
  visual_language: {
    aspect_ratio: string;
    color_palette_hex: string[];
    lighting_philosophy: string;
    camera_movement_style: string;
    lens_preferences: string;
    reference_cinematographers: string[];
  };
  scene_shots: SceneShot[];
  storyboard_prompts: StoryboardPrompt[];
}

interface StoryboardPrompt {
  scene_id: string;
  shot_number: number;
  imagen_prompt: string;
  negative_prompt: string;
  style_modifiers: string[];
  aspect_ratio: "16:9" | "2.39:1" | "4:3" | "1:1";
}
```

---

#### Agent 6: Composer
| Field | Value |
|-------|-------|
| **agent_id** | `composer` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 3 |
| **Dependencies** | `creative_director`, `narrative_architect`, `cinematographer` |
| **Google APIs** | Gemini 1.5 Flash, Vertex AI (MusicLM/Lyria) |
| **Temperature** | 0.85 |

**System Prompt:**
```
You are a Composer. You design the sonic emotional architecture of the project. You
specify instrumentation, tempo, key, and emotional intent for each musical cue,
mapped to narrative beats. You think about leitmotifs — recurring musical themes tied
to characters or ideas. You argue with the Sound Designer about sonic territory
(score vs. sound design boundaries). Generate prompts for MusicLM/Lyria that specify
genre, mood, instrumentation, tempo BPM, and duration.
```

**Output Schema:**
```typescript
interface ComposerOutput {
  musical_identity: {
    genre: string;
    subgenre: string;
    key_signature: string;
    tempo_range_bpm: [number, number];
    primary_instruments: string[];
    sonic_references: string[];
  };
  leitmotifs: Leitmotif[];
  cues: MusicalCue[];
  music_generation_prompts: MusicGenPrompt[];
}

interface MusicGenPrompt {
  cue_id: string;
  prompt: string;
  duration_seconds: number;
  genre: string;
  mood: string;
  instruments: string;
  tempo_bpm: number;
}
```

---

#### Agent 7: Sound Designer
| Field | Value |
|-------|-------|
| **agent_id** | `sound_designer` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 3 |
| **Dependencies** | `creative_director`, `worldbuilder`, `narrative_architect` |
| **Temperature** | 0.8 |

**System Prompt:**
```
You are a Sound Designer. You create the non-musical sonic world — ambiences, foley,
sound effects, and silence. Silence is your most powerful tool. You define the sonic
texture of each location, the signature sounds of key moments, and the use of
diegetic vs non-diegetic sound. You argue with the Composer about where score ends
and sound design begins. You believe sound design often carries more emotion than music.
```

---

#### Agent 8: Visual Style Director
| Field | Value |
|-------|-------|
| **agent_id** | `visual_style_director` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 1 |
| **Dependencies** | `creative_director` |
| **Google APIs** | Gemini 1.5 Flash, Imagen 3 |
| **Temperature** | 0.9 |

**System Prompt:**
```
You are the Visual Style Director. You define the complete visual language — color
palettes, typography, graphic motifs, texture philosophy, and art direction across all
deliverables. You create mood boards via Imagen 3 prompts. You reference specific
art movements, designers, and visual cultures. You fight for visual coherence across
all outputs. You argue with the Cinematographer about whether the visual language
should be consistent or deliberately disrupted at key moments.
```

**Output Schema:**
```typescript
interface VisualStyleDirectorOutput {
  style_guide: {
    color_palette: ColorPalette;
    typography: TypographySpec;
    texture_philosophy: string;
    graphic_motifs: string[];
    art_references: string[];
    mood: string[];
    do_list: string[];
    dont_list: string[];
  };
  mood_board_prompts: string[];  // 5-8 Imagen 3 prompts
  scene_style_overrides: Record<string, Partial<StyleGuide>>;
}
```

---

#### Agent 9: Dialogue Writer
| Field | Value |
|-------|-------|
| **agent_id** | `dialogue_writer` |
| **Model** | Gemini 1.5 Pro |
| **Wave** | 3 |
| **Dependencies** | `narrative_architect`, `character_psychologist`, `worldbuilder` |
| **Google APIs** | Gemini 1.5 Pro, Google Cloud TTS |
| **Temperature** | 0.85 |

**System Prompt:**
```
You are a Dialogue Writer. You write dialogue that sounds like it was overheard, not
written. People interrupt each other. They say one thing and mean another. They trail
off. They use the wrong word. You honor each character's speech pattern from the
Character Psychologist's profile. You argue with the Character Psychologist when their
profile doesn't account for how people actually speak under duress. Subtext is everything;
if the audience can understand the emotional content from the words alone, you've failed.
```

**Output Schema:**
```typescript
interface DialogueWriterOutput {
  scene_dialogues: SceneDialogue[];
  voice_casting_notes: VoiceCastingNote[];
  tts_scripts: TTSScript[];
}

interface TTSScript {
  scene_id: string;
  character: string;
  line: string;
  voice_params: {
    language_code: string;
    voice_name: string;
    speaking_rate: number;
    pitch: number;
    volume_gain_db: number;
  };
}
```

---

#### Agent 10: Continuity Agent
| Field | Value |
|-------|-------|
| **agent_id** | `continuity_agent` |
| **Model** | Gemini 1.5 Pro |
| **Wave** | 4 (QC phase) |
| **Dependencies** | ALL previous agents |
| **Temperature** | 0.2 (very precise) |

**System Prompt:**
```
You are the Continuity Agent. You are the most detail-obsessed member of the team.
You check every output against every other output for contradictions. Does the
Worldbuilder say it's always raining in the capital, but the Cinematographer's
scene 5 storyboard shows harsh sunlight? You produce a continuity error report with
severity ratings. You have veto power over outputs that break established facts.
```

**Output Schema:**
```typescript
interface ContinuityAgentOutput {
  continuity_report: {
    total_checks: number;
    errors_found: ContinuityError[];
    warnings: ContinuityWarning[];
    consistency_score: number;    // 0-100
  };
}
```

---

#### Agent 11: Marketing Agent
| Field | Value |
|-------|-------|
| **agent_id** | `marketing_agent` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 3 |
| **Dependencies** | `creative_director`, `narrative_architect`, `audience_research_agent`, `visual_style_director` |
| **Google APIs** | Gemini 1.5 Flash, Imagen 3 |
| **Temperature** | 0.8 |

**System Prompt:**
```
You are the Marketing Agent. You create the campaign strategy, taglines, poster concepts,
trailer beat sheets, and social media rollout plan. You think in hooks and shareability.
You argue with the Creative Director if the artistic vision lacks a marketable angle.
You generate Imagen 3 prompts for key art and poster concepts.
```

---

#### Agent 12: Casting Agent
| Field | Value |
|-------|-------|
| **agent_id** | `casting_agent` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 2 |
| **Dependencies** | `character_psychologist`, `creative_director` |
| **Temperature** | 0.7 |

**System Prompt:**
```
You are the Casting Agent. Based on character profiles, you suggest ideal voice types,
actor archetypes (not real actors for legal reasons), and performance styles. You define
Google TTS voice parameters for each character. You argue with the Character Psychologist
about whether a character should be cast against type for dramatic impact.
```

**Output:** `CastEntry[]` with TTS voice config per character (voice_name, speaking_rate, pitch, volume).

---

#### Agent 13: VFX Concept Agent
| Field | Value |
|-------|-------|
| **agent_id** | `vfx_concept_agent` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 3 |
| **Dependencies** | `narrative_architect`, `cinematographer`, `worldbuilder` |
| **Google APIs** | Gemini 1.5 Flash, Imagen 3 |
| **Temperature** | 0.85 |

**System Prompt:**
```
You are the VFX Concept Agent. You identify every scene that requires visual effects,
categorize the complexity, and create concept art prompts. You push for photorealism
where appropriate and stylization where it serves the story. You argue with the
Production Designer about what should be built versus rendered.
```

---

#### Agent 14: Production Designer
| Field | Value |
|-------|-------|
| **agent_id** | `production_designer` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 2 |
| **Dependencies** | `worldbuilder`, `visual_style_director`, `creative_director` |
| **Google APIs** | Gemini 1.5 Flash, Imagen 3 |
| **Temperature** | 0.8 |

**System Prompt:**
```
You are the Production Designer. You design the physical spaces — sets, props, vehicles,
costumes at a high concept level. Every object in frame tells a story. You argue with
the Worldbuilder about whether their world is buildable, and with the Cinematographer
about depth and practicals.
```

---

#### Agent 15: Distribution Strategist
| Field | Value |
|-------|-------|
| **agent_id** | `distribution_strategist` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 3 |
| **Dependencies** | `creative_director`, `audience_research_agent`, `marketing_agent` |
| **Temperature** | 0.6 |

**System Prompt:**
```
You are the Distribution Strategist. You determine the optimal release strategy —
platform, timing, territories, windowing, festival strategy, and partnership
opportunities. You argue with the Marketing Agent about whether to lead with
festival prestige or direct-to-audience.
```

---

#### Agent 16: Audience Research Agent
| Field | Value |
|-------|-------|
| **agent_id** | `audience_research_agent` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 1 |
| **Dependencies** | `creative_director` |
| **Temperature** | 0.5 |

**System Prompt:**
```
You are the Audience Research Agent. You analyze the creative vision and define the
target audience with demographic, psychographic, and behavioral precision. You argue
with the Creative Director if the vision targets an audience too niche to be commercially
viable, or too broad to have identity.
```

---

#### Agent 17: Legal/IP Clearance Agent
| Field | Value |
|-------|-------|
| **agent_id** | `legal_ip_agent` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 4 |
| **Dependencies** | ALL previous agents |
| **Temperature** | 0.2 |

**System Prompt:**
```
You are the Legal and IP Clearance Agent. You review all creative outputs for potential
intellectual property issues — character names too similar to existing IP, plot elements
that closely mirror copyrighted works. You flag risks and suggest alternatives. You are
conservative. Better to flag a false positive than miss a real issue.
```

---

#### Agent 18: Localization Agent
| Field | Value |
|-------|-------|
| **agent_id** | `localization_agent` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 4 |
| **Dependencies** | `dialogue_writer`, `marketing_agent`, `narrative_architect` |
| **Google APIs** | Gemini 1.5 Flash, Google Cloud Translation API |
| **Temperature** | 0.6 |

**System Prompt:**
```
You are the Localization Agent. You evaluate the creative product's international
viability. You identify cultural references that won't translate, humor that is
region-specific, and content that may face censorship in key markets.
```

---

#### Agent 19: Pitch Deck Agent
| Field | Value |
|-------|-------|
| **agent_id** | `pitch_deck_agent` |
| **Model** | Gemini 1.5 Flash |
| **Wave** | 4 (final assembly) |
| **Dependencies** | ALL previous agents |
| **Google APIs** | Gemini 1.5 Flash, Imagen 3 |
| **Temperature** | 0.7 |

**System Prompt:**
```
You are the Pitch Deck Agent. You synthesize all agent outputs into a compelling,
investor/studio-ready pitch deck. You know the structure: hook, logline, synopsis,
characters, visual language, comparable titles, market opportunity, team, and ask.
Every slide must earn its place. You write for executives with 3-minute attention spans.
```

**Output Schema:**
```typescript
interface PitchDeckAgentOutput {
  deck: {
    slides: PitchSlide[];
    presenter_notes: string[];
    one_sheet: string;
    executive_summary: string;
  };
  final_image_prompts: ImagePrompt[];
}
```

---

#### Agent 20: Quality Control Agent
| Field | Value |
|-------|-------|
| **agent_id** | `quality_control_agent` |
| **Model** | Gemini 1.5 Pro |
| **Wave** | 4 (final) |
| **Dependencies** | ALL previous agents |
| **Temperature** | 0.3 |

**System Prompt:**
```
You are the Quality Control Agent. You evaluate the complete creative package on
10 dimensions: originality, emotional impact, internal consistency, market viability,
visual coherence, narrative strength, character depth, technical feasibility,
audience appeal, and overall polish. You assign a final quality score and identify
the single weakest element. You trigger one targeted revision cycle if the score is below 75.
```

**Output Schema:**
```typescript
interface QualityControlAgentOutput {
  quality_report: {
    overall_score: number;         // 0-100
    dimension_scores: Record<QualityDimension, number>;
    strengths: string[];
    weaknesses: string[];
    weakest_link: {
      dimension: string;
      agent_responsible: string;
      specific_issue: string;
      revision_prompt: string;
    };
    revision_required: boolean;    // true if overall_score < 75
    final_verdict: string;
  };
}
```

---

## SECTION 3: GOOGLE API INTEGRATION PLAN

### API Map

| Google Service | Used By | Purpose |
|----------------|---------|---------|
| **Gemini 1.5 Pro** | Creative Director, Worldbuilder, Character Psychologist, Narrative Architect, Dialogue Writer, Continuity Agent, Quality Control Agent | Deep creative reasoning, long-context analysis |
| **Gemini 1.5 Flash** | All other 13 agents + conflict detection | Fast creative generation, argument rounds |
| **Imagen 3** | Cinematographer, Visual Style Director, VFX Concept Agent, Production Designer, Marketing Agent, Pitch Deck Agent | Storyboard frames, mood boards, concept art, key art, pitch deck visuals |
| **Vertex AI MusicLM/Lyria** | Composer | Musical cue generation |
| **Google Cloud TTS** | Dialogue Writer, Casting Agent | Character voice acting |
| **Cloud Translation API** | Localization Agent | Cultural adaptation assessment |
| **Cloud Run** | Orchestrator service | Pipeline execution, agent management |
| **Firebase Auth** | Frontend | Google sign-in |
| **Firestore** | All services | Project state, agent outputs, conflict transcripts |
| **Cloud Storage (GCS)** | Media services | Storyboard images, audio files, PDFs |

### Gemini API Client Implementation

```python
# services/orchestrator/integrations/gemini.py
import google.generativeai as genai

class GeminiClient:
    def __init__(self):
        genai.configure(api_key=os.environ["GEMINI_API_KEY"])
        self.pro_model = genai.GenerativeModel("gemini-1.5-pro-latest")
        self.flash_model = genai.GenerativeModel("gemini-1.5-flash-latest")

    async def generate(
        self, prompt: str, system_prompt: str,
        model: str = "flash", temperature: float = 0.7,
        max_tokens: int = 4096, response_schema: dict | None = None,
    ) -> dict:
        model_instance = self.pro_model if model == "pro" else self.flash_model
        config = GenerationConfig(
            temperature=temperature,
            max_output_tokens=max_tokens,
            response_mime_type="application/json" if response_schema else "text/plain",
            response_schema=response_schema,
        )
        response = await model_instance.generate_content_async(
            contents=[{"role": "user", "parts": [{"text": prompt}]}],
            generation_config=config,
            system_instruction=system_prompt,
        )
        return json.loads(response.text) if response_schema else response.text
```

### Imagen 3 Service

```python
# services/orchestrator/media/imagen_service.py
class ImagenService:
    def __init__(self):
        self.client = vertexai.ImageGenerationModel("imagen-3.0-generate-001")

    async def generate_storyboard(self, project_id, prompts, style_guide):
        style_prefix = self._build_style_prefix(style_guide)
        results = []
        for batch in chunk(prompts, 4):  # Rate limit: batches of 4
            batch_results = await asyncio.gather(*[
                self._generate_single(p, style_prefix) for p in batch
            ])
            results.extend(batch_results)
            await asyncio.sleep(1)
        urls = await self._upload_all(project_id, results)
        return urls
```

**Image generation budget per project:** 25-35 images, ~2-3 sec each, batched 4 at a time = ~20 seconds total.

### Music Generation Service

```python
# services/orchestrator/media/music_service.py
class MusicService:
    async def generate_cues(self, project_id, cues, musical_identity):
        for cue in cues:
            prompt = f"{cue.genre} music, {cue.mood} mood, instruments: {cue.instruments}, tempo: {cue.tempo_bpm} BPM"
            response = await self.vertex_client.predict(
                endpoint=self.music_endpoint,
                instances=[{"prompt": prompt, "duration_seconds": cue.duration_seconds}]
            )
            # Upload to GCS
```

### Google Cloud TTS Service

```python
# services/orchestrator/media/tts_service.py
from google.cloud import texttospeech

class TTSService:
    async def generate_dialogue(self, project_id, scripts: list[TTSScript]):
        for script in scripts:
            voice = texttospeech.VoiceSelectionParams(
                language_code=script.voice_params.language_code,
                name=script.voice_params.voice_name,  # e.g. "en-US-Studio-O"
            )
            audio_config = texttospeech.AudioConfig(
                audio_encoding=texttospeech.AudioEncoding.LINEAR16,
                speaking_rate=script.voice_params.speaking_rate,
                pitch=script.voice_params.pitch,
            )
            response = await self.client.synthesize_speech(...)
```

---

## SECTION 4: THE CREATIVE CONFLICT ENGINE

### Why Conflict Produces Better Output

Real creative teams argue. The director fights the DP about lighting. The writer fights the actor about motivation. The producer fights everyone about budget. These tensions are not bugs — they are the mechanism through which mediocre ideas become great ones. SYMPHONY encodes this into its architecture.

### Designed Conflict Pairs

| Agent A | Agent B | Tension Type | Core Disagreement |
|---------|---------|-------------|-------------------|
| Worldbuilder | Narrative Architect | world_vs_story | "The world says no" vs "The story needs this" |
| Character Psychologist | Dialogue Writer | psychology_vs_voice | "This person wouldn't say that" vs "It sounds better this way" |
| Cinematographer | Production Designer | beauty_vs_feasibility | "I need depth" vs "That's unbuildable" |
| Composer | Sound Designer | score_vs_silence | "Music carries emotion" vs "Silence is louder" |
| Creative Director | Audience Research Agent | art_vs_commerce | "Art first" vs "Will anyone watch this?" |
| Marketing Agent | Creative Director | hook_vs_integrity | "Lead with the hook" vs "Don't reduce the art" |
| VFX Concept Agent | Production Designer | digital_vs_practical | "We can render it" vs "Build it for real" |
| Visual Style Director | Cinematographer | consistency_vs_disruption | "Stay on brand" vs "Break the rules here" |

### Conflict Detection Algorithm

```python
# services/orchestrator/conflict/engine.py
class ConflictEngine:
    async def detect_conflicts(self, outputs: dict[str, AgentOutput]) -> list[Conflict]:
        conflicts = []
        for agent_a, agent_b, tension_type in self.CONFLICT_PAIRS:
            if agent_a in outputs and agent_b in outputs:
                conflict = await self._compare_outputs(outputs[agent_a], outputs[agent_b], tension_type)
                if conflict and conflict.severity > 0.3:
                    conflicts.append(conflict)
        return conflicts
```

### Argument Transcript System

```typescript
interface ArgumentTranscript {
  conflict_id: string;
  tension_type: string;
  rounds: ArgumentRound[];      // max 3 rounds
  resolution: Resolution;
  creative_tension_score: number;
}

interface ArgumentRound {
  round_number: number;
  speaker: string;              // agent_id
  position: string;
  evidence: string;
  concession: string | null;
  escalation: boolean;
}
```

**Argument Flow:**
1. Conflict detected between Agent A and Agent B
2. Agent A critiques Agent B's output (Round 1)
3. Agent B defends or concedes (Round 2)
4. If no resolution, Agent A responds (Round 3, max)
5. Creative Director reviews transcript and makes binding decision
6. Losing agent revises output to comply

### Director Arbitration Protocol

```python
async def director_arbitrate(self, transcript, conflict) -> Resolution:
    prompt = f"""
    You are the Creative Director arbitrating a conflict.
    Original vision: {self.vision_document}
    Conflict: {conflict.point_of_contention}
    {transcript.format_for_director()}

    Rules:
    1. The original creative vision is the tiebreaker.
    2. When in doubt, choose the BOLDER option.
    3. Never split the difference — that produces mediocrity.
    4. If both positions have merit, synthesize into something NEITHER agent proposed.
    5. Your decision is final and binding.
    """
```

### Creative Tension Score (CTS)

```
CTS = (conflict_diversity * 0.3) + (resolution_quality * 0.3) + (quality_delta * 0.4)

- conflict_diversity: how many different tension types were activated (0-1)
- resolution_quality: ratio of syntheses vs one-sided wins (higher = more synthesis = better)
- quality_delta: improvement in QC score after conflict resolution
```

A high CTS (70+) means the arguments genuinely improved the work. It's displayed on the dashboard as a key metric.

---

## SECTION 5: MULTIMODAL OUTPUT PIPELINE

### Pipeline Flow

```
Narrative Beats (Narrative Architect)
    ↓
Scene Storyboard Prompts (Cinematographer) + Style Prefix (Visual Style Director)
    ↓
Imagen 3 → 25-35 storyboard frames → GCS
    ↓
Musical Cues mapped to scenes (Composer) → Vertex AI MusicLM → 6-8 audio cues → GCS
    ↓
Dialogue per scene (Dialogue Writer) + Voice Config (Casting Agent) → Cloud TTS → voice clips → GCS
    ↓
Timeline Sync JSON (maps storyboard frames ↔ music cues ↔ voice clips by scene_id)
    ↓
Pitch Deck (Pitch Deck Agent) + Hero Images → PDF via WeasyPrint → GCS
    ↓
Final Package: ZIP archive with all assets + shareable gallery link
```

### Style Consistency Strategy for Imagen 3

1. **Style prefix injection**: Visual Style Director output → consistent prefix on every Imagen 3 prompt
2. **Character anchoring**: Fixed appearance description (from Character Psychologist) in every prompt featuring that character
3. **Negative prompt consistency**: Same negative prompt across all images from style guide's `dont_list`
4. **Post-generation QC**: Quality Control Agent flags visual inconsistencies across thumbnails

### Music-to-Visual Sync

The Composer maps each cue to a `scene_id`. The scene has a `duration_estimate`. The assembly step outputs `timeline.json` that the frontend uses to synchronize audio playback with storyboard slide transitions.

### Final Package Contents

```
project_output.zip
├── storyboard/
│   ├── frame_001_act1_scene1.png
│   ├── frame_002_act1_scene2.png
│   └── ... (25-35 frames)
├── music/
│   ├── cue_01_opening.wav
│   ├── cue_02_tension.wav
│   └── ... (6-8 cues)
├── voice/
│   ├── act1_scene3_protagonist.wav
│   └── ... (15-25 clips)
├── pitch_deck.pdf
├── one_sheet.pdf
├── timeline.json
├── quality_report.json
├── conflict_transcripts.json
└── raw_agent_outputs/
    ├── creative_director.json
    ├── worldbuilder.json
    └── ... (all 20 agents)
```

---

## SECTION 6: FRONTEND ARCHITECTURE

### Page Structure (Next.js App Router)

```
/                           → Landing page with hero + demo video
/create                     → Creative brief input wizard
/project/:id                → Live project dashboard (main experience)
/project/:id/gallery        → Output gallery
/project/:id/conflicts      → Full argument transcript viewer
/project/:id/export         → Download center
/pricing                    → Business plans
/login                      → Firebase Auth (Google sign-in)
```

### Creative Brief Input (`/create`)

**Step 1 — The Vision:** Large centered textarea with ghost prompt, project type selector (Film | Game | Album | Novel | Campaign), example prompts carousel

**Step 2 — Optional Refinements:** Tone sliders (Dark↔Light, Serious↔Playful, Grounded↔Fantastical), reference works tags, constraints textarea

**Step 3 — Launch:** Summary card, "Deploy SYMPHONY" button with dramatic animation

### Live Agent Dashboard (`/project/:id`) — THE CORE EXPERIENCE

```
┌─────────────────────────────────────────────────────────────┐
│  Progress Bar (phase label + time elapsed/remaining)         │
├──────────────────────────────┬──────────────────────────────┤
│                              │                              │
│   Agent Grid (4x5)           │   Live Activity Feed         │
│   Each agent: avatar, name,  │   - Agent outputs streaming  │
│   status badge, mini output  │   - Conflicts appearing      │
│   preview. Glow = active.    │   - Director decisions       │
│   Pulse = conflict.          │   - Quality scores           │
│                              │                              │
├──────────────────────────────┼──────────────────────────────┤
│                              │                              │
│   Conflict Theater           │   Output Preview Panel       │
│   Chat-bubble UI showing     │   Storyboard thumbnails,     │
│   agent arguments live       │   audio waveforms, text      │
│                              │   snippets as they appear    │
└──────────────────────────────┴──────────────────────────────┘
```

### Real-time SSE Integration

```typescript
// app/project/[id]/hooks/useProjectStream.ts
export function useProjectStream(projectId: string) {
  const [agents, setAgents] = useState<Record<string, AgentState>>({});
  const [conflicts, setConflicts] = useState<ConflictEvent[]>([]);
  const [phase, setPhase] = useState<string>("intake");

  useEffect(() => {
    const eventSource = new EventSource(`/api/projects/${projectId}/stream`);
    eventSource.addEventListener("agent_started", (e) => { /* update agent status */ });
    eventSource.addEventListener("agent_complete", (e) => { /* update with preview */ });
    eventSource.addEventListener("conflict_detected", (e) => { /* add conflict */ });
    eventSource.addEventListener("argument_round", (e) => { /* update conflict rounds */ });
    eventSource.addEventListener("phase_change", (e) => { /* update phase */ });
    eventSource.addEventListener("pipeline_complete", (e) => { /* mark complete */ });
    return () => eventSource.close();
  }, [projectId]);

  return { agents, conflicts, phase };
}
```

### Key UI Components
- `AgentCard` — Status-aware card with Framer Motion glow/pulse animations
- `ConflictTheater` — Chat-bubble UI showing agent arguments with Director ruling
- `StoryboardViewer` — Horizontal filmstrip with lightbox, keyboard nav
- `AudioPlayer` — Waveform player with cue labels, synced to storyboard
- `QualityScoreRadar` — 10-dimension radar chart from QC agent
- `CreativeTensionMeter` — Visual gauge showing CTS score

---

## SECTION 7: COMPLETE FILE & FOLDER STRUCTURE

```
symphony/
├── .github/workflows/
│   ├── ci.yml
│   ├── deploy-frontend.yml
│   └── deploy-services.yml
│
├── apps/web/                          # Next.js 14 frontend
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx                   # Landing
│   │   ├── globals.css
│   │   ├── providers.tsx
│   │   ├── (auth)/login/page.tsx
│   │   ├── create/
│   │   │   ├── page.tsx
│   │   │   └── components/
│   │   │       ├── VisionInput.tsx
│   │   │       ├── ProjectTypeSelector.tsx
│   │   │       ├── ToneSliders.tsx
│   │   │       ├── ReferenceTagInput.tsx
│   │   │       ├── ConstraintsInput.tsx
│   │   │       ├── BriefSummaryCard.tsx
│   │   │       ├── DeployButton.tsx
│   │   │       └── BriefWizard.tsx
│   │   ├── project/[id]/
│   │   │   ├── page.tsx               # Live dashboard
│   │   │   ├── layout.tsx
│   │   │   ├── gallery/page.tsx
│   │   │   ├── conflicts/page.tsx
│   │   │   ├── export/page.tsx
│   │   │   ├── hooks/
│   │   │   │   ├── useProjectStream.ts
│   │   │   │   ├── useProjectData.ts
│   │   │   │   └── useAgentOutput.ts
│   │   │   └── components/
│   │   │       ├── AgentGrid.tsx
│   │   │       ├── AgentCard.tsx
│   │   │       ├── AgentDetailModal.tsx
│   │   │       ├── ConflictTheater.tsx
│   │   │       ├── LiveActivityFeed.tsx
│   │   │       ├── OutputPreviewPanel.tsx
│   │   │       ├── ProgressBar.tsx
│   │   │       ├── StoryboardViewer.tsx
│   │   │       ├── AudioPlayer.tsx
│   │   │       ├── PitchDeckViewer.tsx
│   │   │       ├── CharacterProfileCard.tsx
│   │   │       ├── QualityScoreRadar.tsx
│   │   │       ├── CreativeTensionMeter.tsx
│   │   │       └── TimelineSync.tsx
│   │   ├── pricing/page.tsx
│   │   └── api/
│   │       ├── projects/
│   │       │   ├── route.ts
│   │       │   └── [id]/
│   │       │       ├── route.ts
│   │       │       ├── stream/route.ts
│   │       │       ├── agents/[agentId]/route.ts
│   │       │       ├── conflicts/route.ts
│   │       │       ├── outputs/route.ts
│   │       │       └── feedback/route.ts
│   │       └── webhooks/stripe/route.ts
│   ├── lib/
│   │   ├── firebase.ts
│   │   ├── firestore.ts
│   │   ├── api-client.ts
│   │   ├── constants.ts
│   │   └── utils.ts
│   ├── components/ui/                 # Shared UI primitives
│   ├── types/
│   │   ├── agents.ts
│   │   ├── conflicts.ts
│   │   ├── project.ts
│   │   ├── media.ts
│   │   └── api.ts
│   ├── next.config.js
│   ├── tailwind.config.ts
│   ├── tsconfig.json
│   └── package.json
│
├── services/orchestrator/             # Python Cloud Run service
│   ├── main.py                        # FastAPI entry
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── api/
│   │   ├── routes_pipeline.py
│   │   ├── routes_agents.py
│   │   ├── routes_conflicts.py
│   │   ├── routes_assembly.py
│   │   ├── routes_media.py
│   │   └── routes_health.py
│   ├── agents/
│   │   ├── base_agent.py
│   │   ├── registry.py
│   │   ├── creative_director.py
│   │   ├── worldbuilder.py
│   │   ├── character_psychologist.py
│   │   ├── narrative_architect.py
│   │   ├── cinematographer.py
│   │   ├── composer.py
│   │   ├── sound_designer.py
│   │   ├── visual_style_director.py
│   │   ├── dialogue_writer.py
│   │   ├── continuity_agent.py
│   │   ├── marketing_agent.py
│   │   ├── casting_agent.py
│   │   ├── vfx_concept_agent.py
│   │   ├── production_designer.py
│   │   ├── distribution_strategist.py
│   │   ├── audience_research_agent.py
│   │   ├── legal_ip_agent.py
│   │   ├── localization_agent.py
│   │   ├── pitch_deck_agent.py
│   │   └── quality_control_agent.py
│   ├── conflict/
│   │   ├── engine.py
│   │   ├── argument.py
│   │   ├── tension_score.py
│   │   └── pairs.py
│   ├── orchestration/
│   │   ├── pipeline.py
│   │   ├── context.py
│   │   ├── waves.py
│   │   └── events.py
│   ├── media/
│   │   ├── imagen_service.py
│   │   ├── music_service.py
│   │   ├── tts_service.py
│   │   └── storage_service.py
│   ├── assembly/
│   │   ├── assembler.py
│   │   ├── pdf_generator.py
│   │   ├── timeline_builder.py
│   │   └── archive_builder.py
│   ├── models/
│   │   ├── agent_models.py
│   │   ├── conflict_models.py
│   │   ├── pipeline_models.py
│   │   └── media_models.py
│   ├── integrations/
│   │   ├── gemini.py
│   │   ├── firestore_client.py
│   │   └── gcs_client.py
│   └── tests/
│       ├── test_agents/
│       ├── test_conflict_engine.py
│       ├── test_pipeline.py
│       └── conftest.py
│
├── infrastructure/
│   ├── terraform/
│   │   ├── main.tf
│   │   ├── cloud_run.tf
│   │   ├── firestore.tf
│   │   ├── storage.tf
│   │   ├── iam.tf
│   │   └── variables.tf
│   └── firebase/
│       ├── firestore.rules
│       ├── firestore.indexes.json
│       └── firebase.json
│
├── scripts/
│   ├── seed-agents.py
│   ├── test-pipeline.py
│   └── demo-brief.json
│
├── turbo.json
├── package.json
├── pnpm-workspace.yaml
├── .gitignore
└── README.md
```

### API Endpoints Summary

**Next.js BFF:**
```
POST   /api/projects                    → Create project, start pipeline
GET    /api/projects/:id                → Project status + outputs
GET    /api/projects/:id/stream         → SSE live event stream
GET    /api/projects/:id/agents/:agentId → Specific agent output
GET    /api/projects/:id/conflicts      → Conflict transcripts
GET    /api/projects/:id/outputs        → Final output URLs
POST   /api/projects/:id/feedback       → User feedback
```

**Python Orchestrator (Cloud Run, internal):**
```
POST   /pipeline/start                  → Start orchestration
POST   /agents/:agentId/execute         → Execute single agent
POST   /conflicts/detect                → Run conflict detection
POST   /conflicts/resolve               → Run argument + arbitration
POST   /assembly/compile                → Assemble final outputs
POST   /media/imagen                    → Generate image
POST   /media/music                     → Generate music
POST   /media/tts                       → Generate speech
GET    /health                          → Health check
```

### Firestore Data Model

```
/projects/{project_id}
  ├── brief, project_type, status, user_id, created_at, completed_at
  ├── /agents/{agent_id}
  │     ├── status, started_at, completed_at, output (full JSON), token_usage
  ├── /conflicts/{conflict_id}
  │     ├── tension_type, agent_a, agent_b, transcript, resolution
  ├── /outputs/
  │     ├── storyboard_urls, music_urls, voice_urls, pitch_deck_url, archive_url
  └── quality_report
```

---

## SECTION 8: THE HACKATHON DEMO PLAN

### 5-Minute Demo Script

**0:00 — The Hook:**
"What if 20 AI creative minds could argue about your movie idea and produce a professional pitch package in under an hour? This is SYMPHONY."

**0:30 — One-Sentence Input (live on stage):**
> "A jazz musician in 1950s Harlem discovers she can hear the memories of the dead through their favorite songs — but the FBI wants to weaponize her gift during the Cold War."

Click "Deploy SYMPHONY."

**1:00 — The Swarm Comes Alive:**
Show live dashboard. 20 agent cards light up in waves. "The Creative Director is parsing the brief... Now the Worldbuilder, Character Psychologist, and Visual Style Director are running in parallel. Watch the agent grid — blue means working, green means done."

**2:00 — The Conflict Reveal (pre-run demo in conflict phase):**
"Here's what makes SYMPHONY different. The Character Psychologist says our protagonist should be guarded, speaks in short sentences. But the Dialogue Writer wrote her a passionate monologue. Watch them argue..."

Show 2-3 argument rounds live. Creative Director rules: "She IS guarded — but the monologue happens when she's channeling a dead person's memory. It's not her voice, it's theirs."

**3:00 — Output Gallery Reveal (pre-run completed project):**
- Scroll 12 storyboard frames — consistent cinematic style
- Play 15-second music cue: melancholic jazz with cold-war tension
- Play TTS voice clip of protagonist's key line
- Show pitch deck: 12 slides, professional layout, market analysis

**4:00 — The Numbers:**
"20 agents. 8 creative conflicts resolved. 35 storyboard images. 6 music cues. Full pitch deck. Quality score: 82/100. Creative tension score: 71 — the arguments genuinely improved the work. Total time: 47 minutes."

**4:30 — The Close:**
"SYMPHONY doesn't replace human creativity — it gives every creator a team of 20 collaborators who argue, challenge, and push each other until the vision is real. Because the best creative work doesn't come from agreement. It comes from tension."

### Demo Safety Net
- Pre-run 3 complete projects as backup
- If live demo fails, seamlessly switch to pre-run project
- Keep the "Deploy" button wired to a pre-cached result for demo reliability

---

## SECTION 9: 24-HOUR BUILD SPRINT PLAN

### Phase 1: Foundation (Hours 0-6)

| Hour | Task | Deliverable |
|------|------|-------------|
| 0-1 | Project scaffolding: Next.js + Python FastAPI + Firebase init + GCS bucket + deps | Running dev servers |
| 1-2 | Firebase Auth (Google sign-in), Firestore data model, GCS client wrapper | Auth + DB working |
| 2-3 | `base_agent.py` — SymphonyAgent base class with Gemini client, execute/critique/revise | Base class + unit tests |
| 3-4 | Implement Wave 0-1 agents: creative_director, worldbuilder, character_psychologist, visual_style_director, audience_research_agent | 5 agents producing JSON |
| 4-5 | Wave 2 agents: narrative_architect, cinematographer, casting_agent, production_designer + registry | Wave 2 agents work |
| 5-6 | Pipeline orchestrator v1 — wave deployment, dependency injection, SSE skeleton | Waves 0-2 execute end-to-end |

### Phase 2: Frontend Core (Hours 6-12)

| Hour | Task | Deliverable |
|------|------|-------------|
| 6-7 | Landing + Create page with VisionInput, ProjectTypeSelector, DeployButton | User can submit brief |
| 7-8 | Agent Grid dashboard with AgentCard, status badges (static mock) | 20 agent cards render |
| 8-9 | SSE integration: useProjectStream hook, live agent status updates | Cards update in real time |
| 9-10 | Wave 3 agents: dialogue_writer, composer, sound_designer, vfx_concept_agent, marketing_agent, distribution_strategist | All Wave 3 agents work |
| 10-11 | Conflict Engine v1: detect_conflicts + run_argument for 2 priority pairs | Conflicts generate transcripts |
| 11-12 | Conflict Theater UI component with live argument rendering | Arguments appear live on dashboard |

### Phase 3: Media & QC (Hours 12-18)

| Hour | Task | Deliverable |
|------|------|-------------|
| 12-13 | Imagen 3 integration: ImagenService, style prefix, batch gen, GCS upload | Storyboard images generate |
| 13-14 | Google Cloud TTS integration: TTSService, voice parameter mapping | Character voice clips generate |
| 14-15 | Music generation: MusicService with Vertex AI (fallback: royalty-free library) | Music cues generate |
| 15-16 | Wave 4 agents: continuity, legal_ip, localization, quality_control, pitch_deck | Full pipeline Waves 0-4 |
| 16-17 | Output Gallery UI: StoryboardViewer, AudioPlayer, PitchDeckViewer | Gallery displays all outputs |
| 17-18 | Assembly pipeline: OutputAssembler, PDF generation, ZIP archive | Final package downloads |

### Phase 4: Polish & Demo Prep (Hours 18-24)

| Hour | Task | Deliverable |
|------|------|-------------|
| 18-19 | Director arbitration + CTS calculation + all 8 conflict pairs | Full conflict system working |
| 19-20 | QC revision cycles (score < 75 triggers targeted revision) | QC loop works end-to-end |
| 20-21 | Framer Motion animations, responsive layout, UI polish | Cinematic UI |
| 21-22 | Pre-run 3 demo projects (film, game, album), fix pipeline issues | 3 complete demos ready |
| 22-23 | Performance: parallel execution tuning, SSE throttling, error handling | Reliable < 60 min execution |
| 23-24 | Demo rehearsal, fallback prep, final bugs, README | Demo-ready product |

---

## SECTION 10: PRODUCTION & SCALE

### Scaling Strategy

| Scale Level | Users | Infrastructure | Cost |
|-------------|-------|---------------|------|
| **MVP/Hackathon** | 1-10 concurrent | 1 Cloud Run instance (min 1), 1 Firestore DB | ~$50/month base |
| **Early Growth** | 50 concurrent | 3 Cloud Run instances, CDN for assets | ~$500/month |
| **Growth** | 500 concurrent | Auto-scaling Cloud Run (max 20), Redis for SSE fan-out, Cloud CDN | ~$5K/month |
| **Enterprise** | 5000 concurrent | Multi-region Cloud Run, Pub/Sub for event distribution, dedicated Vertex AI endpoints | ~$50K/month |

### Key Scaling Decisions
- **SSE at scale**: Move from in-memory queues to **Cloud Pub/Sub** for event distribution at 100+ concurrent projects
- **Agent execution**: Consider **Cloud Tasks** for individual agent execution (retry, timeout built-in) at 500+ concurrent projects
- **Image caching**: Common style prefixes → shared Imagen 3 outputs (mood boards reusable across projects with similar styles)
- **Firestore sharding**: Projects naturally shard by project_id. No special handling needed until 10K+ documents/second

---

## SECTION 11: TECHNICAL RISKS & MITIGATIONS

### Risk 1: Imagen 3 Visual Consistency
**Problem:** Characters look different across storyboard frames.
**Mitigations:**
1. Style prefix injection from Visual Style Director on every prompt
2. Fixed character appearance descriptions in every prompt featuring that character
3. Consistent negative prompts from style guide
4. QC Agent flags visual inconsistencies, triggers re-generation
5. Fallback: fewer hero frames (8-10) rather than many inconsistent ones

### Risk 2: Music Quality Variance
**Problem:** AI-generated music can be generic or tonally inappropriate.
**Mitigations:**
1. Highly specific prompts (BPM, key, instruments, genre, reference style)
2. Generate 2 variants per cue, use Gemini Flash to select better match
3. Keep cues short (15-45 seconds) — short cues are consistently higher quality
4. Fallback: curated library of 50 royalty-free cues tagged by mood/tempo/genre

### Risk 3: Agent Conflict Resolution Loops
**Problem:** Revision cycles could loop infinitely.
**Mitigations:**
1. Hard cap: max 1 revision cycle per agent
2. Wall-clock deadlines per phase (enforced timeouts)
3. Max 3 argument rounds per conflict
4. Monotonic convergence check — if quality decreases after revision, keep pre-revision output

```python
PHASE_TIMEOUTS = {
    "wave_0": 180, "wave_1": 420, "wave_2": 600,
    "conflict_1": 300, "wave_3": 720, "conflict_2": 300,
    "wave_4": 480, "revision": 300, "assembly": 300,
}
```

### Risk 4: Output Coherence
**Problem:** 20 independent agents produce stitched-together feeling.
**Mitigations:**
1. Vision document as universal anchor (every agent receives it)
2. Continuity Agent specifically catches contradictions
3. Shared entity IDs across all agents (scene_id, character names)
4. Two conflict resolution rounds before downstream consumption
5. QC Agent scores coherence and triggers targeted revision

### Risk 5: API Rate Limits & Latency
**Mitigations:**
1. Wave parallelism (4-6 concurrent, not 20 at once)
2. Model tiering (4 Pro + 16 Flash)
3. Token budget caps per agent
4. Retry with exponential backoff (1s, 2s, 4s)
5. Graceful degradation — pipeline continues without failed agents

### Risk 6: Cold Start Latency
**Mitigations:**
1. Cloud Run min-instances = 1
2. Keep-alive pings every 5 minutes via Cloud Scheduler
3. Slim Python base image, lazy-load heavy deps

### Risk 7: Cost Explosion
**Mitigations:**
1. Per-project cost cap ($10 max, 3x normal)
2. Rate limiting (3 concurrent, 10/day on Creator plan)
3. Firebase Auth required (no anonymous usage)
4. Stripe prepayment before pipeline starts

---

## SECTION 12: BUSINESS MODEL

### Pricing Tiers

| Plan | Price | Projects | Key Features |
|------|-------|----------|-------------|
| **Creator** | $29/project | Pay-per-use | Full 20-agent pipeline, all outputs, 30-day storage |
| **Studio** | $199/month | 10/month | Gemini Pro for key agents, priority queue, revision requests, 90-day storage |
| **Enterprise** | $2K-$10K/month | Unlimited | Custom agents, dedicated instances, SSO, permanent storage, webhook integrations, white-label |

### Unit Economics

```
Cost per project (estimated):
  Gemini 1.5 Pro (4 agents):     ~$0.80
  Gemini 1.5 Flash (16 agents):  ~$0.40
  Imagen 3 (35 images):          ~$1.40
  Cloud TTS (20 clips):          ~$0.10
  Vertex AI music (6 cues):      ~$0.60
  Cloud Run compute:             ~$0.15
  GCS storage (30 days):         ~$0.05
  ─────────────────────────────────
  Total COGS per project:        ~$3.50

  Creator Plan margin: ($29 - $3.50) / $29 = 88%
  Studio Plan margin: ($199 - $35) / $199 = 82%
```

### Go-to-Market
- **Launch**: ProductHunt, Hacker News, Twitter/X demo video
- **Target**: Independent filmmakers, game designers, ad agencies, screenwriting students
- **Partnerships**: Film schools, game dev bootcamps, creative agencies
- **Flywheel**: Gallery of anonymized best outputs as marketing (with permission)

### Hollywood & Gaming Pipeline
- **Year 2**: Studio API — integrate SYMPHONY into existing development pipelines
- **Year 3**: Custom agent training on studio IP guidelines, brand voice, franchise bibles

---

## SECTION 13: JUDGING CRITERIA + PITCH STRUCTURE

### Hackathon Judging Criteria Alignment

| Criteria | How SYMPHONY Scores |
|----------|-------------------|
| **Innovation** | First multi-agent creative collaboration system with built-in productive conflict. No competitor has this. |
| **Technical Complexity** | 20-agent orchestration, 5 Google APIs (Gemini, Imagen, TTS, Music, Firestore), real-time SSE dashboard, conflict resolution engine |
| **Use of Google APIs** | Deep integration: Gemini Pro + Flash for agents, Imagen 3 for storyboards, Vertex AI for music, Cloud TTS for voice, Firestore for state, Cloud Run for compute, GCS for assets |
| **Completeness** | Full working pipeline: input → 20 agents → conflicts → resolution → multimodal output → download |
| **UI/UX** | Cinematic dashboard showing AI agents arguing in real time — unlike anything judges have seen |
| **Business Potential** | $4.4B TAM, 88% gross margins, clear path from indie creators to studio enterprise |
| **Demo Quality** | One sentence in → dramatic live process → professional creative package out. Narrative arc in the demo itself. |

### Pitch Deck Structure (for SYMPHONY itself)

1. **"Imagine..."** — the problem (solo creator vs. 20-person studio)
2. **The Demo** — live one-sentence input
3. **The Swarm** — dashboard reveal
4. **The Argument** — conflict theater moment
5. **The Output** — gallery showcase
6. **The Numbers** — quality scores, CTS, timing
7. **The Tech** — architecture slide (20 agents, 5 Google APIs)
8. **The Market** — $4.4B TAM
9. **The Close** — "The best creative work comes from tension."

---

## SECTION 14: APPENDICES

### Appendix A: Orchestration Pipeline Timing

```
Total: 60 minutes
├── Phase 1: Brief Intake + Director Vision    [0:00-3:00]    3 min
├── Phase 2: Wave 1 Foundation Agents          [3:00-10:00]   7 min
├── Phase 3: Wave 2 Structure Agents           [10:00-20:00]  10 min
├── Phase 4: Conflict Resolution Round 1       [20:00-25:00]  5 min
├── Phase 5: Wave 3 Detail Agents              [25:00-37:00]  12 min
├── Phase 6: Conflict Resolution Round 2       [37:00-42:00]  5 min
├── Phase 7: Wave 4 QC & Assembly Agents       [42:00-50:00]  8 min
├── Phase 8: Targeted Revision (if needed)     [50:00-55:00]  5 min
└── Phase 9: Final Assembly & Export           [55:00-60:00]  5 min
```

### Appendix B: Agent Wave Deployment

```
Wave 0: creative_director
Wave 1: worldbuilder, character_psychologist, visual_style_director, audience_research_agent  (parallel)
Wave 2: narrative_architect, cinematographer, casting_agent, production_designer  (parallel)
  → Conflict Resolution Round 1
Wave 3: dialogue_writer, composer, sound_designer, vfx_concept_agent, marketing_agent, distribution_strategist  (parallel)
  → Conflict Resolution Round 2
Wave 4: continuity_agent, legal_ip_agent, localization_agent, quality_control_agent, pitch_deck_agent  (sequential)
```

### Appendix C: Environment Variables

```env
# Google AI
GEMINI_API_KEY=
GOOGLE_CLOUD_PROJECT=
GOOGLE_APPLICATION_CREDENTIALS=

# Firebase
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=

# Cloud Services
GCS_BUCKET_NAME=symphony-outputs
CLOUD_RUN_ORCHESTRATOR_URL=

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
```

### Appendix D: Critical Implementation Files

1. `services/orchestrator/agents/base_agent.py` — Core agent base class (execute/critique/revise)
2. `services/orchestrator/orchestration/pipeline.py` — Central pipeline orchestrator
3. `services/orchestrator/conflict/engine.py` — Creative Conflict Engine
4. `apps/web/app/project/[id]/page.tsx` — Live agent dashboard
5. `services/orchestrator/assembly/assembler.py` — Final output assembler
6. `services/orchestrator/integrations/gemini.py` — Gemini API client wrapper
7. `services/orchestrator/media/imagen_service.py` — Imagen 3 integration
8. `apps/web/app/project/[id]/hooks/useProjectStream.ts` — SSE real-time hook

### Appendix E: Verification Plan

1. **Unit tests**: Each agent produces valid JSON matching its output schema given a test brief
2. **Integration test**: Full pipeline executes Waves 0-4 with a sample brief, all agents produce output
3. **Conflict test**: At least 2 conflicts are detected and resolved with argument transcripts
4. **Media test**: At least 1 image, 1 music cue, and 1 voice clip generate and upload to GCS
5. **E2E test**: Submit brief via frontend → watch dashboard → view gallery → download ZIP
6. **Performance test**: Full pipeline completes in < 60 minutes with a complex brief
7. **Demo rehearsal**: Run the exact demo script 3 times end-to-end before presentation
