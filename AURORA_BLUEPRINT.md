# AURORA: Climate Visualization Platform — Master Blueprint

## Context

Climate communication has failed for 30 years because of the **abstraction problem**: graphs, statistics, and global averages don't move people to action. 2.8 degrees Celsius of warming is meaningless to most humans. But showing someone a photorealistic image of *their own neighborhood* underwater in 2050 — that changes behavior, votes, and policy.

AURORA solves this by combining 40 years of satellite imagery (Google Earth Engine), climate projection data (CMIP6/ERA5), multimodal AI reasoning (Gemini 1.5 Pro), and photorealistic image generation (Imagen 3) into a single pipeline: **type any city, see your future.**

**Target audiences**: citizens (emotional engagement), city planners (data-driven decisions), insurance companies (risk assessment).

**Why 2026 is the moment**: Earth Engine's petabyte satellite catalog + Imagen 3's reference-conditioned generation + Gemini's multimodal reasoning = the first time this pipeline is technically possible.

---

## SECTION 1: PROBLEM DEFINITION & VISION

### 1.1 The Abstraction Problem
- 30 years of IPCC reports with charts and graphs have not produced proportional policy response
- Psychological research shows humans respond to **concrete, personal, visual** threats — not abstract statistical ones
- "Sea levels will rise 0.5m by 2050" vs. "Your street will look like this in 2050" — the second creates action

### 1.2 Scale of Risk
- **3.5 billion people** live in cities facing measurable climate risk by 2050 (IPCC AR6)
- Economic costs: **$23 trillion annually by 2050** in damage from climate-related disasters (Swiss Re)
- 570+ cities with populations >1M face at least one major climate hazard (heat, flood, drought, storms)

### 1.3 Where Existing Tools Fail
| Tool | What It Does | Where It Fails |
|------|-------------|----------------|
| Climate Central Surging Seas | Blue flood overlays on maps | Abstract — no photorealism, no personal connection |
| Google Earth Timelapse | Historical satellite timelapses | Shows the past, not the future |
| NASA WorldView | 1000+ scientific data layers | Designed for scientists, not citizens |
| En-ROADS / Climate Interactive | Policy simulation models | No geographic personalization, no imagery |
| Tomorrow.io | Weather forecasting | Short-term weather, not climate projection |

### 1.4 Core Insight
When people can **see** their future neighborhood — their street, their buildings, their trees — under two different climate scenarios, they:
- Increase willingness to support climate policy by 30-40% (Yale Climate Communication studies)
- Make different housing, insurance, and investment decisions
- Demand accountability from local government

### 1.5 The Vision
A world where every city council meeting starts with AURORA's projection. Where homebuyers check AURORA before signing. Where students show AURORA to their parents. The world's definitive climate visualization platform.

### 1.6 Three Target Audiences

| Audience | What They Need | AURORA Delivers |
|----------|---------------|-----------------|
| **Citizens** | Emotional, personal, shareable | Street-level future photos, social media sharing, comparison slider |
| **City Planners** | Data-driven, exportable, scenario-based | Climate metrics dashboard, PDF reports, policy comparison view |
| **Insurance/Real Estate** | Risk quantification, API access, bulk analysis | REST API, confidence intervals, batch city analysis |

### 1.7 Why 2026 Is the Moment
- **Earth Engine**: 50+ petabytes of satellite data, free for research, Python API
- **Imagen 3**: Reference image conditioning (style transfer) — can anchor on real Street View photos
- **Gemini 1.5 Pro**: Multimodal reasoning over satellite imagery + climate data simultaneously
- **CMIP6**: Latest generation climate models with SSP scenarios through 2100
- **ERA5 on BigQuery**: 1979-present global reanalysis data queryable in seconds

---

## SECTION 2: CORE TECHNICAL ARCHITECTURE

### 2.1 End-to-End Pipeline

```
USER INPUT (city name)
    |
    v
[Step 1] Geocoding (Google Maps Geocoding API)         ~200ms
    | Output: { lat, lng, formatted_address, place_id }
    v
[Step 2] Parallel Fan-Out (3 concurrent paths)
    |
    |---> [2a] Street View Acquisition                  ~300ms
    |       Street View Static API
    |       4 headings (0, 90, 180, 270) x 2 pitches (0, -10)
    |       = 8 images @ 640x640
    |
    |---> [2b] Satellite Data (Earth Engine)             ~2-5s
    |       Landsat 8 + Sentinel-2
    |       10km bounding box
    |       Compute: NDVI, NDWI, LST, land cover
    |
    |---> [2c] Climate Data (BigQuery)                   ~1-3s
    |       ERA5 historical + CMIP6 projections
    |       YALE UHI dataset
    |       Sea level rise for coastal cities
    |
    v
[Step 3] Gemini Scene Analysis                           ~2-4s
    Input: Street View images + satellite metrics
    Output: Structured scene description (vegetation, buildings,
            infrastructure, climate stress indicators)
    |
    v
[Step 4] Gemini Impact Reasoning                         ~3-5s
    Input: scene + climate projections + satellite metrics
    Output: Per (year, scenario) transformation specs +
            Imagen prompts + narratives
    |
    v
[Step 5] Imagen 3 Rendering (parallel)                   ~5-8s
    6 images: 3 years x 2 scenarios
    Street View as style reference
    |
    v
[Step 6] Storage + Delivery                              ~1-2s
    Cloud Storage + Redis cache + SSE push to frontend

TOTAL: ~15-25s first render | ~200ms cached
```

### 2.2 Earth Engine Integration

**Image Collections:**
- `LANDSAT/LC08/C02/T1_TOA` — Landsat 8, 30m resolution, 2013-present
- `COPERNICUS/S2_SR` — Sentinel-2, 10m resolution, 2017-present
- `MODIS/061/MOD11A1` — Land Surface Temperature, 1km, 2000-present
- `YALE/YCEO/UHI/UHI_monthly_averaged_v4` — Urban Heat Island intensity

**Band Selection:**
| Index | Formula | Bands (Landsat 8) | What It Measures |
|-------|---------|-------------------|------------------|
| NDVI | (NIR-Red)/(NIR+Red) | (B5-B4)/(B5+B4) | Vegetation health/density |
| NDWI | (Green-NIR)/(Green+NIR) | (B3-B5)/(B3+B5) | Water presence |
| NDBI | (SWIR-NIR)/(SWIR+NIR) | (B6-B5)/(B6+B5) | Built-up area density |
| LST | From MODIS thermal bands | MOD11A1 Band 31 | Surface temperature |

**Time Series Strategy:**
- Query 5-year composites: 1985, 1990, 1995, 2000, 2005, 2010, 2015, 2020, 2025
- Use median composites to handle cloud cover
- Compute per-pixel trends (linear regression) for NDVI and LST
- Detect acceleration/deceleration in greening/browning trends

**Missing Data Handling:**
- Cloud masking via QA_PIXEL band (Landsat) / SCL band (Sentinel-2)
- If <10 clear observations in a 5-year window, expand window to 7 years
- For locations with no Sentinel-2 coverage, fall back to Landsat only
- For pre-2013 data, use Landsat 5/7 with cross-sensor calibration

### 2.3 Climate Modeling Layer

**Data Sources:**
| Source | What | Resolution | Access |
|--------|------|-----------|--------|
| CMIP6 (via BigQuery/CDS) | Future projections 2025-2100 | ~100km native, 6km downscaled | BigQuery public datasets |
| ERA5 (ARCO-ERA5) | Historical reanalysis 1979-present | ~30km, hourly | BigQuery: `bigquery-public-data.arco_era5` |
| YALE YCEO UHI | Urban heat island intensity | City-level, monthly | Earth Engine |
| Climate Central | Sea level rise by location | ~1m DEM | API/pre-downloaded |

**SSP Scenarios Used:**
| Scenario | Meaning | AURORA Label | Temp by 2100 |
|----------|---------|-------------|-------------|
| SSP2-4.5 | Moderate action, middle road | "Optimistic" / "If we act" | +2.7C |
| SSP5-8.5 | Business as usual, fossil-fuel heavy | "Pessimistic" / "Current policy" | +4.4C |

**Uncertainty Quantification:**
- Use CMIP6 multi-model ensemble: 5th, 25th, 50th, 75th, 95th percentiles
- Display median projection as primary, with ±range as confidence band
- Label all projections: "Illustrative projection, not a forecast"
- Show ensemble spread in data panel charts

### 2.4 Gemini Vision Reasoning Layer

**Scene Analysis Prompt Architecture:**
```
You are an expert urban geographer and climate scientist analyzing
street-level imagery of {city_name} at coordinates ({lat}, {lng}).

Analyze these 8 street-view images and the satellite metrics below
to produce a structured scene assessment.

Satellite Metrics:
- NDVI (vegetation index): {ndvi_mean} (trend: {ndvi_trend}/decade)
- Land Surface Temperature: {lst_mean}C
- Water fraction: {water_fraction}%
- Green cover: {green_fraction}%
- Urban density: {urban_fraction}%

Return JSON: { vegetation, buildings, infrastructure,
climate_indicators, overall_narrative }
```

**Impact Reasoning Prompt Architecture:**
```
You are a climate impact specialist. Given the current scene analysis
and climate projections below, determine EXACTLY what physical changes
will be visible at this location in {year} under {scenario}.

CONSTRAINTS:
- Temperature will change by EXACTLY {temp_delta}C (not more, not less)
- Sea level will rise by EXACTLY {slr_m} meters
- Precipitation will change by {precip_delta}%
- Do NOT invent numbers. Use ONLY the data provided.
- Be conservative. Describe changes that are physically certain given
  these numbers.

Output: JSON with visual_modifications[], imagen_prompt, narrative
```

### 2.5 Imagen 3 Rendering Pipeline

**Conditioning Strategy:**
- Use hero Street View image as `referenceType: "STYLE"` — preserves architectural identity
- Imagen prompt describes the *changes* to apply: flooding, vegetation loss, infrastructure adaptation
- `personGeneration: "dont_allow"` — avoid generating people (ethical/legal)
- `safetyFilterLevel: "block_few"` — allow climate disaster imagery

**Prompt Construction:**
```
Photorealistic street view of {city_name} in {year} under {scenario_desc}.
Same architecture and buildings as the reference image but showing:
{visual_modifications joined by '. '}
Photorealistic, high detail, natural lighting, 4K quality.
Street-level perspective matching the reference image angle.
```

**Quality Control:**
- Gemini 1.5 Pro validates generated image against prompt (stretch goal)
- Pre-render and manually review demo cities
- Include "photorealistic" and "natural lighting" anchors in every prompt
- Use reference image to maintain geographic identity

### 2.6 Street View Integration

- Primary: Street View Static API with 4 headings x 2 pitches = 8 images
- Select "hero" image: heading with most visual interest (most vegetation + buildings)
- Check metadata endpoint first — if no panorama within 500m, trigger fallback
- **Fallback**: Use Earth Engine RGB composite as top-down reference + Gemini to describe the scene

### 2.7 Time Slider Architecture

- Pre-compute key years: 2035, 2050, 2075
- Frontend interpolates between years for smooth slider animation (crossfade)
- Each year + scenario = 1 pre-rendered image
- Total: 6 images per city (3 years x 2 scenarios)
- On-demand rendering for custom years is a stretch goal

---

## SECTION 3: GOOGLE API INTEGRATION PLAN

### 3.1 Google Earth Engine (Python EE API)

**Use Case:** Query 40 years of satellite imagery, compute vegetation/thermal/water indices

**Setup:**
```python
import ee
ee.Authenticate()  # Or service account:
# credentials = ee.ServiceAccountCredentials(email, key_file)
# ee.Initialize(credentials, project='aurora-climate')
```

**Sample Query — NDVI Time Series:**
```python
point = ee.Geometry.Point([lng, lat])
bbox = point.buffer(5000)  # 5km radius

landsat = (ee.ImageCollection('LANDSAT/LC08/C02/T1_TOA')
    .filterBounds(bbox)
    .filterDate('2020-01-01', '2025-01-01')
    .filter(ee.Filter.lt('CLOUD_COVER', 20)))

def compute_ndvi(image):
    ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI')
    return image.addBands(ndvi)

ndvi_collection = landsat.map(compute_ndvi)
ndvi_mean = ndvi_collection.select('NDVI').mean().reduceRegion(
    reducer=ee.Reducer.mean(), geometry=bbox, scale=30
).getInfo()
```

**Integration Point:** `services/pipeline/app/services/earth_engine.py`

### 3.2 Gemini 1.5 Pro Vision

**Use Case:** Analyze satellite + street-view imagery, reason about climate impacts

**Sample Request:**
```python
import google.generativeai as genai
genai.configure(api_key=GEMINI_API_KEY)
model = genai.GenerativeModel('gemini-1.5-pro')

response = model.generate_content([
    streetview_image_1,  # PIL Image or bytes
    streetview_image_2,
    "Analyze these street-level images of Miami Beach. "
    "Identify vegetation types, building materials, flood vulnerability..."
])
```

**Integration Points:**
- `services/pipeline/app/services/scene_analyzer.py` — scene understanding
- `services/pipeline/app/services/impact_reasoner.py` — climate impact reasoning

### 3.3 Vertex AI Forecast

**Use Case:** Time series projection of temperature, precipitation, NDVI trends

**Note for Hackathon:** Skip Vertex AI Forecast for MVP. Use CMIP6 ensemble projections directly — they already contain multi-model time series forecasts through 2100. Vertex AI Forecast adds value for custom local calibration in production.

### 3.4 Google Maps Platform

**Use Case:** Base map layer, address geocoding, location search

**Endpoints:**
- Geocoding: `https://maps.googleapis.com/maps/api/geocode/json?address={city}&key={KEY}`
- Places Autocomplete: Via `@react-google-maps/api` in frontend
- Maps JavaScript API: Interactive map with marker

**Integration Points:**
- Frontend: `apps/web/components/map/MapView.tsx`
- Backend proxy: `apps/web/app/api/geocode/route.ts`

### 3.5 Street View Static API

**Use Case:** Fetch current ground-level imagery as baseline for future rendering

**Sample Request:**
```
GET https://maps.googleapis.com/maps/api/streetview
  ?location=25.7907,-80.1300
  &size=640x640
  &heading=90
  &pitch=0
  &key={KEY}
```

**Metadata Check:**
```
GET https://maps.googleapis.com/maps/api/streetview/metadata
  ?location=25.7907,-80.1300
  &key={KEY}
```

**Integration Point:** `services/pipeline/app/services/streetview.py`

### 3.6 Imagen 3

**Use Case:** Generate photorealistic future neighborhood images

**Sample Request:**
```python
from google.cloud import aiplatform

endpoint = f"projects/{PROJECT}/locations/us-central1/publishers/google/models/imagen-3.0-generate-001"

request = {
    "instances": [{
        "prompt": imagen_prompt,
        "referenceImages": [{
            "referenceId": 1,
            "referenceType": "STYLE",
            "image": {"bytesBase64Encoded": streetview_b64}
        }]
    }],
    "parameters": {
        "sampleCount": 1,
        "aspectRatio": "1:1",
        "safetyFilterLevel": "block_few",
        "personGeneration": "dont_allow"
    }
}
```

**Cost:** ~$0.04/image, 6 images/render = $0.24/render for generation
**Integration Point:** `services/pipeline/app/services/image_generator.py`

### 3.7 BigQuery

**Use Case:** Climate dataset storage and fast querying

**Key Datasets:**
- `bigquery-public-data.arco_era5` — ERA5 reanalysis
- `aurora_climate.cmip6_projections` — our loaded CMIP6 data
- `aurora_climate.city_climate_summary` — pre-computed city summaries

**Integration Point:** `services/pipeline/app/utils/bigquery_client.py`

### 3.8 Cloud Storage

**Use Case:** Store rendered images, Street View captures, Earth Engine exports

**Bucket Structure:**
```
aurora-renders/
  streetview/{placeId}/{heading}_{pitch}.jpg
  satellite/{placeId}/{year}_rgb.png
  renders/{jobId}/{year}_{scenario}.png
```

**Integration Point:** `services/pipeline/app/services/storage.py`

### 3.9 Google Translate (Stretch Goal)

**Use Case:** Multilingual interface for global audience
**Integration:** `next-intl` package with Google Translate API for dynamic content

### 3.10 Looker Studio (Production Feature)

**Use Case:** Professional climate dashboards for city planners
**Integration:** Embed Looker dashboards via iframe, connected to BigQuery

### 3.11 Cloud Run

**Use Case:** Host Python pipeline and Node.js gateway as serverless containers
**Config:** Min instances = 1 (avoid cold start), max = 10, 2GB RAM, 2 vCPU

### 3.12 Firebase

**Use Case:** User auth (Google Sign-In), Firestore for render history, Firebase Hosting for frontend
**Integration:** `firebase-admin` SDK in gateway, `firebase/app` in frontend

---

## SECTION 4: RENDERING PIPELINE DETAIL

### 4.1 City Name to Photorealistic Image — Step by Step

1. User types "Miami Beach" → Google Places Autocomplete returns structured result
2. User selects result → frontend sends `POST /api/v1/render` with coordinates + placeId
3. Gateway creates job in Redis queue, returns jobId + SSE stream URL
4. Python pipeline picks up job, starts parallel fan-out:
   - **Path A**: Fetch 8 Street View images (4 headings x 2 pitches)
   - **Path B**: Query Earth Engine for NDVI, NDWI, LST, land cover in 10km bbox
   - **Path C**: Query BigQuery for ERA5 baseline + CMIP6 projections
5. All three paths complete → feed into Gemini Scene Analysis
6. Scene analysis + climate data → feed into Gemini Impact Reasoning
7. Impact Reasoner outputs 6 transformation specs (3 years x 2 scenarios) with Imagen prompts
8. Fire 6 Imagen 3 requests in parallel, each with hero Street View as style reference
9. As each image completes → upload to Cloud Storage → push SSE event to frontend
10. Frontend progressively renders: first streetview baseline, then data, then generated images

### 4.2 Spatial Resolution

- Street View: 640x640 pixels per image (max free tier)
- Generated images: 1024x1024 (Imagen 3 standard)
- Satellite: 10m (Sentinel-2), 30m (Landsat), 1km (MODIS thermal)
- Climate: ~100km (CMIP6 native), downscaled to ~6km for precipitation

### 4.3 Cities Without Street View

**Fallback chain:**
1. Expand search radius to 500m, then 1km
2. Use Earth Engine true-color RGB composite (top-down satellite view) as reference
3. Gemini describes the scene from satellite imagery alone
4. Imagen generates based on satellite reference + Gemini description
5. Label output: "Satellite-based visualization (no street-level imagery available)"

### 4.4 Multi-Location Compositing

- **Primary view**: Street-level (Street View → Imagen 3)
- **Context view**: Satellite overview (Earth Engine RGB) shown in map strip
- **Future**: Side-by-side street + satellite comparison

### 4.5 Policy Comparison

- Two-panel split screen: "Current Policy (SSP5-8.5)" | "If We Act (SSP2-4.5)"
- Same year selected via timeline slider
- Identical camera angle — only the climate effects differ
- This is the emotional core: the delta between inaction and action

### 4.6 Time Slider Strategy

**Hackathon (MVP):** Pre-render 3 fixed years (2035, 2050, 2075) x 2 scenarios = 6 images
**Production:** Pre-render every 5 years (2030, 2035, 2040, 2045, 2050, 2055, 2060, 2065, 2070, 2075) = 20 images, crossfade interpolation between

### 4.7 Caching Strategy

| Layer | Key | TTL | What's Cached |
|-------|-----|-----|---------------|
| Redis | `render:{placeId}` | 24 hours | Full render result JSON + image URLs |
| Redis | `climate:{placeId}` | 7 days | Climate data (changes slowly) |
| Redis | `satellite:{placeId}` | 7 days | Satellite metrics |
| Cloud Storage | `renders/{jobId}/` | Permanent | Generated images |
| CDN (Cloud CDN) | Image URLs | 1 hour | Signed URLs for images |
| Frontend | localStorage | Session | Recent searches |

### 4.8 Resolution Settings

| Setting | Image Size | Use Case | Cost/Image |
|---------|-----------|----------|------------|
| Demo/Fast | 512x512 | Testing, iteration | ~$0.02 |
| Standard | 1024x1024 | Production, hackathon | ~$0.04 |
| HD | 2048x2048 | City planner exports | ~$0.08 |

---

## SECTION 5: FRONTEND ARCHITECTURE

### 5.1 Map Interface
- Full-width Google Maps embed with custom styling (dark theme)
- Google Places Autocomplete integrated into search bar
- Click-to-drop pin on map → triggers render for that location
- Marker shows selected location with info window

### 5.2 Time Slider Component
- Horizontal track: 2025 (Today) ---|--- 2035 ---|--- 2050 ---|--- 2075
- Animated thumb with year label
- Temperature delta badge above each stop (+1.5C, +2.8C, +4.4C)
- Below track: scenario toggle pills [If We Act] [Current Policy]
- Built with CSS custom properties for smooth animation

### 5.3 Split-Screen Comparison
- Default: Slider comparison (drag divider left/right)
- Option: Side-by-side (two panels)
- Option: Toggle (crossfade animation between now/future)
- Labels: "Today" on left, "2050 — Current Policy" on right
- Keyboard accessible: left/right arrows move divider

### 5.4 Rendering Progress UI
- **Phase 1 (0-5s)**: Skeleton loader with pulsing gradient + "Analyzing satellite data..."
- **Phase 2 (5-10s)**: Current Street View appears + climate metrics populate + "Building climate model..."
- **Phase 3 (10-20s)**: Blurred placeholder → sharp images crossfade in one by one + "Rendering future..."
- Each SSE event transitions the UI to the next phase

### 5.5 Personal Impact Panel
Metric cards showing:
- Temperature: 25.3C → 28.1C (+2.8C)
- Extreme Heat Days: 45 → 98 days/year
- Sea Level Rise: +0.5m (coastal cities only)
- Precipitation Change: -12%
- Green Cover: 28% → 18%
- Flood Risk Score: Low → High

### 5.6 Local Action Engine (Stretch Goal)
- "What changes the outcome?" section
- Lists city-specific policy actions: tree planting targets, sea wall investment, building codes
- Shows how each action shifts the projection
- Data source: C40 Cities action database

### 5.7 Sharing Features
- "Share Your City's Future" button
- Generates OG-image-optimized comparison image (before/after composite)
- Copy link to clipboard
- Direct share to Twitter/X, Facebook, LinkedIn
- Share URL: `aurora.app/city/miami-beach?year=2050&scenario=pessimistic`

### 5.8 City Planner Dashboard (Stretch Goal)
- PDF report generation with all metrics, images, and narratives
- CSV export of climate data
- Batch analysis: run multiple locations in a city
- Embeddable widget for city government websites

### Component Tree
```
<RootLayout>
  <Header />
  <main>
    <LandingPage>
      <HeroSection>
        <CitySearchBar />
        <DemoCityGrid>
          <DemoCityCard /> x5
      <HowItWorksSection />
      <DataSourcesSection />

    <ExplorePage>
      <MapStrip>
        <MapView />
        <StreetViewPreview />
        <LocationInfo />
      <VisualizationPanel>
        <ControlBar>
          <TimelineSlider />
          <ScenarioToggle />
        <ImageComparison>
          <ProgressiveLoader />
          <BeforeImage />
          <AfterImage />
          <ComparisonSlider />
        <NarrativePanel />
      <DataPanel>
        <ClimateMetrics>
          <MetricCard /> x6
        <TemperatureChart />
        <PrecipitationChart />
        <SeaLevelGauge />
        <DataSourceBadge />

    <AboutPage>
      <MethodologySection />
      <LimitationsSection />
  </main>
  <Footer />
```

### State Management (Zustand)
```typescript
interface VisualizationStore {
  selectedCity: GeocodedPlace | null;
  jobId: string | null;
  jobStatus: 'idle' | 'pending' | 'processing' | 'completed' | 'failed';
  progress: number;
  currentStep: string;
  streetViewUrl: string | null;
  satelliteMetrics: SatelliteMetrics | null;
  climateBaseline: ClimateBaseline | null;
  results: Record<string, RenderResult>;  // "2050_optimistic" -> result
  selectedYear: 2035 | 2050 | 2075;
  selectedScenario: 'optimistic' | 'pessimistic';
  comparisonMode: 'slider' | 'sideBySide' | 'toggle';
}
```

---

## SECTION 6: COMPLETE FILE & FOLDER STRUCTURE

```
aurora/
├── package.json                          # Workspace root (npm workspaces)
├── turbo.json                            # Turborepo build config
├── .gitignore
├── .env.example                          # All environment variable templates
├── docker-compose.yml                    # Local dev: Redis + backend + frontend
├── Dockerfile.api                        # Python FastAPI container
├── Dockerfile.gateway                    # Node.js gateway container
│
├── apps/
│   ├── web/                              # === NEXT.JS 14 FRONTEND ===
│   │   ├── package.json
│   │   ├── next.config.js
│   │   ├── tailwind.config.ts
│   │   ├── tsconfig.json
│   │   ├── .env.local
│   │   ├── public/
│   │   │   ├── favicon.ico
│   │   │   ├── aurora-logo.svg
│   │   │   ├── og-image.png
│   │   │   └── demo/                    # Pre-rendered demo images (WebP)
│   │   │       ├── miami-2035-opt.webp
│   │   │       ├── miami-2050-opt.webp
│   │   │       ├── miami-2075-opt.webp
│   │   │       ├── miami-2035-pes.webp
│   │   │       ├── miami-2050-pes.webp
│   │   │       ├── miami-2075-pes.webp
│   │   │       └── ... (5 cities x 6 variants = 30 images)
│   │   ├── app/
│   │   │   ├── layout.tsx               # Root: fonts, metadata, providers
│   │   │   ├── page.tsx                 # Landing page with search hero
│   │   │   ├── globals.css              # Global styles + Tailwind
│   │   │   ├── explore/
│   │   │   │   └── page.tsx             # Main visualization page
│   │   │   ├── about/
│   │   │   │   └── page.tsx             # Methodology + limitations
│   │   │   └── api/
│   │   │       ├── geocode/route.ts     # Proxy Google Geocoding
│   │   │       ├── render/route.ts      # POST: trigger render
│   │   │       ├── render/[jobId]/route.ts  # GET: poll status
│   │   │       └── streetview/route.ts  # Proxy Street View
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── Header.tsx
│   │   │   │   └── Footer.tsx
│   │   │   ├── search/
│   │   │   │   ├── CitySearchBar.tsx
│   │   │   │   └── SearchSuggestions.tsx
│   │   │   ├── visualization/
│   │   │   │   ├── TimelineSlider.tsx
│   │   │   │   ├── ScenarioToggle.tsx
│   │   │   │   ├── ImageComparison.tsx
│   │   │   │   ├── RenderCanvas.tsx
│   │   │   │   ├── ProgressiveLoader.tsx
│   │   │   │   └── NarrativePanel.tsx
│   │   │   ├── data/
│   │   │   │   ├── ClimateMetrics.tsx
│   │   │   │   ├── MetricCard.tsx
│   │   │   │   ├── TemperatureChart.tsx
│   │   │   │   ├── SeaLevelGauge.tsx
│   │   │   │   └── DataSourceBadge.tsx
│   │   │   ├── map/
│   │   │   │   ├── MapView.tsx
│   │   │   │   └── StreetViewPreview.tsx
│   │   │   └── ui/
│   │   │       ├── Button.tsx
│   │   │       ├── Card.tsx
│   │   │       ├── Skeleton.tsx
│   │   │       └── Toast.tsx
│   │   ├── hooks/
│   │   │   ├── useRenderJob.ts          # SSE stream + Zustand updates
│   │   │   ├── useClimateData.ts
│   │   │   ├── useGeocoding.ts
│   │   │   └── useStreetView.ts
│   │   ├── lib/
│   │   │   ├── api-client.ts            # Typed fetch wrapper
│   │   │   ├── constants.ts             # Demo cities, labels, colors
│   │   │   ├── types.ts                 # TypeScript interfaces
│   │   │   └── utils.ts                 # Formatting helpers
│   │   └── stores/
│   │       └── visualization-store.ts   # Zustand store
│   │
│   └── gateway/                          # === NODE.JS API GATEWAY ===
│       ├── package.json
│       ├── tsconfig.json
│       └── src/
│           ├── index.ts                 # Express server entry
│           ├── routes/
│           │   ├── render.ts            # /api/v1/render (orchestrate)
│           │   ├── climate.ts           # /api/v1/climate
│           │   └── health.ts            # /api/v1/health
│           ├── middleware/
│           │   ├── rate-limit.ts        # Redis-backed rate limiter
│           │   └── cache.ts             # Redis cache middleware
│           └── services/
│               ├── queue.ts             # Bull job queue
│               └── cache.ts             # Redis operations
│
├── services/
│   └── pipeline/                         # === PYTHON FASTAPI (ML PIPELINE) ===
│       ├── pyproject.toml
│       ├── requirements.txt
│       ├── Dockerfile
│       ├── app/
│       │   ├── main.py                  # FastAPI entry, CORS, lifespan
│       │   ├── config.py                # Pydantic Settings
│       │   ├── dependencies.py          # DI: EE client, Gemini, etc.
│       │   ├── routers/
│       │   │   ├── render.py            # POST /render — full pipeline
│       │   │   ├── satellite.py         # GET /satellite/{lat}/{lng}
│       │   │   ├── climate.py           # GET /climate/{lat}/{lng}
│       │   │   └── health.py            # GET /health
│       │   ├── services/
│       │   │   ├── geocoding.py         # Maps Geocoding wrapper
│       │   │   ├── streetview.py        # Street View (8 angles)
│       │   │   ├── earth_engine.py      # EE: NDVI, NDWI, LST
│       │   │   ├── climate_data.py      # BigQuery: ERA5 + CMIP6
│       │   │   ├── mock_climate_data.py # Hardcoded IPCC data (fallback)
│       │   │   ├── scene_analyzer.py    # Gemini scene analysis
│       │   │   ├── impact_reasoner.py   # Gemini climate reasoning
│       │   │   ├── image_generator.py   # Imagen 3 via Vertex AI
│       │   │   ├── storage.py           # Cloud Storage upload
│       │   │   └── cache.py             # Redis cache
│       │   ├── models/
│       │   │   ├── schemas.py           # Pydantic request/response
│       │   │   ├── climate.py           # Climate data models
│       │   │   └── scene.py             # Scene analysis models
│       │   ├── prompts/
│       │   │   ├── scene_analysis.py    # Gemini scene prompt
│       │   │   ├── impact_reasoning.py  # Gemini impact prompt
│       │   │   └── imagen_template.py   # Imagen prompt templates
│       │   └── utils/
│       │       ├── ee_helpers.py        # Earth Engine helpers
│       │       ├── bigquery_client.py   # BQ connection + queries
│       │       └── image_processing.py  # Resize, encode, compose
│
├── packages/
│   └── shared-types/                    # Shared TypeScript types
│       ├── package.json
│       └── src/
│           ├── index.ts
│           ├── climate.ts
│           ├── render.ts
│           └── geo.ts
│
├── infrastructure/
│   ├── cloud-run-api.yaml              # Cloud Run config (pipeline)
│   ├── cloud-run-gateway.yaml          # Cloud Run config (gateway)
│   └── cloudbuild.yaml                 # CI/CD pipeline
│
└── scripts/
    ├── setup-gcp.sh                    # Enable APIs, create SA
    ├── seed-demo-data.sh               # Pre-render 5 demo cities
    ├── load-climate-data.sh            # Load ERA5/CMIP6 into BQ
    └── ee-authenticate.py              # Earth Engine auth helper
```

---

## SECTION 7: DATA SCIENCE METHODOLOGY

### 7.1 Validation Against Historical Outcomes
- Compare CMIP6 hindcast (1990-2020 projections) against ERA5 actual observations
- Compute bias correction factors per region
- Apply bias correction to forward projections
- Publish validation metrics in About page

### 7.2 Preventing Sensationalist Imagery
- Imagen prompts are constrained by **exact numerical values** from CMIP6 (e.g., "0.5m flooding", not "catastrophic flooding")
- Gemini is instructed to be conservative: "If uncertain, understate rather than overstate"
- Default view is SSP2-4.5 (moderate), not SSP5-8.5 (worst case)
- Both scenarios always shown together — no cherry-picking

### 7.3 Scientific Advisory Layer
- All projections cite specific CMIP6 models and SSP scenarios
- Data sources listed on every visualization
- IPCC AR6 as the authority — AURORA visualizes IPCC data, doesn't create its own projections
- Methodology page explains every step

### 7.4 Confidence Scoring
- Multi-model ensemble spread shown as confidence bands in charts
- Labels: "High confidence" (ensemble agrees), "Moderate" (some spread), "Low" (large spread)
- All images labeled: "AI-generated illustration based on CMIP6 projections. Not a photograph of the future."

### 7.5 Edge Cases

| Geography | Challenge | Approach |
|-----------|----------|----------|
| Island nations (Maldives) | Entire islands could disappear | Show sea level overlay, not just street-level |
| Desert cities (Dubai) | Already hot, changes are about extremes | Focus on extreme heat days, sandstorm frequency |
| Arctic cities (Tromsoe) | Warming is amplified (Arctic amplification) | Apply 2x temperature delta, show permafrost thaw |
| Mountain cities (La Paz) | Glacier retreat, water supply | Focus on water availability, not temperature |
| Inland cities (Omaha) | Flooding from rivers, not sea level | Use precipitation data, skip sea level metrics |

### 7.6 Ethical Obligations
- **Disclaimer on every page**: "These visualizations are AI-generated illustrations informed by peer-reviewed climate science. They represent plausible futures, not predictions."
- **No fearmongering**: Always show the optimistic scenario alongside pessimistic
- **Attribution**: Every data source cited, every model named
- **Limitations section**: What AURORA cannot predict (exact timing, local microclimate effects, adaptation responses)
- **Privacy**: No personal data collected for rendering. Location searches are not linked to user identity unless they opt in.

---

## SECTION 8: HACKATHON DEMO PLAN

### 8.1 Demo City Selection

| Rank | City | Why It's Perfect |
|------|------|-----------------|
| 1 | **Miami Beach, FL** | Iconic streetscape + severe sea level threat + universally recognized |
| 2 | **Phoenix, AZ** | Extreme heat story + desert expansion + visually dramatic |
| 3 | **San Francisco, CA** | Wildfire haze + drought + tech audience relates |
| 4 | **Amsterdam, NL** | Below sea level + engineering response + international appeal |
| 5 | **Mumbai, India** | 20M population at risk + monsoon intensification + global scale |

### 8.2 Pre-Rendering (12 hours before demo)
- Run `scripts/seed-demo-data.sh` for all 5 cities
- Generate 30 images total (5 cities x 3 years x 2 scenarios)
- Cherry-pick best results, re-run any that look unrealistic
- Save as static WebP files in `public/demo/` as offline fallback
- Cache in Redis for instant loading during demo

### 8.3 The "Personal City" Moment
- After showing pre-rendered cities, say: "But this is about YOUR city."
- Ask a judge: "Where do you live?" or "Where did you grow up?"
- Type their city live → render in real-time while talking about the pipeline
- If it fails → smoothly redirect to a pre-rendered city nearby
- This moment is the emotional climax — it makes it personal

### 8.4 Five-Minute Pitch Arc

```
[0:00-0:30]  HOOK
  "What will your neighborhood look like when your kids are your age?"
  Show Miami Beach 2050 pessimistic fullscreen. Let it breathe.

[0:30-1:30]  PROBLEM
  "Climate change is invisible. 2.8 degrees means nothing.
   But THIS is your street in 2050. AURORA makes climate change
   visible, personal, and local."

[1:30-3:00]  LIVE DEMO
  Type "Phoenix, AZ". Explain pipeline while loading.
  Walk through timeline slider: 2035 → 2050 → 2075
  Toggle optimistic vs pessimistic.
  Highlight specific details: cracked roads, dead vegetation.
  Scroll to data panel: "Real IPCC data, not fiction."

[3:00-4:00]  TECH ARCHITECTURE
  Quick architecture slide: EE → Gemini → Imagen 3
  "40 years of NASA satellite data + CMIP6 climate models +
   Gemini's reasoning + Imagen 3's photorealism."
  Name-drop Google APIs. Show API count.

[4:00-5:00]  IMPACT + CLOSE
  "Show a city council what their downtown looks like in 2050.
   Show a homebuyer what that waterfront really costs.
   Climate DATA is everywhere. Climate UNDERSTANDING is missing."
  End on Amsterdam 2075 OPTIMISTIC: "This is what happens if we act."
  Hope is the final frame.
```

### 8.5 Policy Comparison Reveal
- The money shot: split-screen showing same street, same year, two futures
- Left: "Current Policy" — flooding, dead vegetation, damaged infrastructure
- Right: "If We Act" — green infrastructure, managed waterways, thriving canopy
- Linger on this image. It tells the entire story without words.

### 8.6 Demo Fallback Plan

| If This Fails... | Do This Instead |
|-------------------|----------------|
| APIs are all down | Play pre-recorded screen capture video (record during hour 22) |
| Internet is spotty | All demo cities load from `public/demo/` static files |
| Imagen returns bad image | Show Street View with CSS overlay (sepia=heat, blue=flood) + narrative text |
| Pipeline is slow | Talk through architecture while loading, show charts first |
| Judge asks for obscure city | "Great city! Let me show you [nearest pre-rendered city] which faces similar risks." |

---

## SECTION 9: 24-HOUR BUILD SPRINT

### Pre-Sprint (Hour -2 to 0)
- Create GCP project, enable all APIs
- Create service account with Earth Engine + Vertex AI + BigQuery + Storage roles
- Set up Firebase project
- Create Cloud Storage bucket
- **Team roles**: Person A (frontend), Person B (Python pipeline), Person C (gateway + infra)

### Hour 0-3: Foundation
| Who | Deliverable |
|-----|-------------|
| **A** | Next.js 14 + TypeScript + Tailwind scaffold. CitySearchBar with Places Autocomplete working. Header, Footer, landing page layout. |
| **B** | FastAPI skeleton. `geocoding.py` + `streetview.py` working. Earth Engine connected. `earth_engine.py` returns NDVI for a test coordinate. |
| **C** | Express gateway skeleton. Redis in docker-compose. Health endpoints. Cloud Storage upload helper tested. |

### Hour 3-6: Core Data Pipeline
| Who | Deliverable |
|-----|-------------|
| **A** | ExplorePage layout. TimelineSlider + ScenarioToggle. Zustand store. `useRenderJob` hook skeleton with SSE. |
| **B** | `climate_data.py` with BigQuery OR `mock_climate_data.py` with hardcoded IPCC values for 20 cities. `scene_analyzer.py` with Gemini. Both tested. |
| **C** | Bull job queue + Redis. SSE endpoint streaming mock updates. Gateway routes proxying to Python. |

### Hour 6-10: The Rendering Engine (Critical Path)
| Who | Deliverable |
|-----|-------------|
| **A** | ImageComparison with drag slider. ProgressiveLoader. NarrativePanel. Connected to SSE stream — images appear as generated. |
| **B** | `impact_reasoner.py` — Gemini generates transformation specs + Imagen prompts. `image_generator.py` with Imagen 3. **Full pipeline end-to-end tested with Miami Beach.** |
| **C** | Frontend wired to real backend. CORS fixed. Error handling. Rate limiting. |

### Hour 10-14: Data Panel + Polish
| Who | Deliverable |
|-----|-------------|
| **A** | ClimateMetrics cards, TemperatureChart (recharts), SeaLevelGauge. DataSourceBadge. Mobile responsive pass. |
| **B** | Pre-render 5 demo cities. Iterate Imagen prompts for quality. Save best results. Fallback prompt templates. |
| **C** | Finalize mock/real climate data. Firebase Hosting deploy. Cloud Run deploy if time. |

### Hour 14-18: Integration + Hardening
| Who | Deliverable |
|-----|-------------|
| **A** | Landing page: HeroSection with dramatic background, DemoCityGrid, HowItWorks. Loading states + error states everywhere. |
| **B** | Error handling for every API call. Timeouts + retries. Fallback to pre-rendered images. Prompt quality iteration. |
| **C** | End-to-end testing all 5 demo cities. Integration bugs fixed. Performance profiling. |

### Hour 18-22: Demo Prep
| Who | Deliverable |
|-----|-------------|
| **A** | Animations (framer-motion). Color theme polish. Typography. About page with methodology. |
| **B** | Final high-quality renders for all demo cities. Verify narratives. Export static fallback set. |
| **C** | Production deploy. Pre-warm Redis. Load test. Record screen capture backup video. |

### Hour 22-24: Rehearsal
| All | Practice pitch 3-4 times. Time sections. Identify "wow" moments. Test on venue WiFi if possible. Backup laptop with localhost ready. |

### Cut List (If Behind Schedule)
| Priority | Cut This | Keep This Instead |
|----------|----------|-------------------|
| P1 (first to cut) | City planner dashboard | Just the citizen view |
| P2 | Google Translate integration | English only |
| P3 | BigQuery real data | Mock climate data (IPCC values) |
| P4 | Node.js gateway | Direct frontend → Python calls |
| P5 | Vertex AI Forecast | CMIP6 projections directly |
| P6 | Temperature/precipitation charts | Just metric cards with numbers |
| P7 | User auth / Firebase | Anonymous access only |
| NEVER CUT | Imagen 3 rendering | This IS the product |
| NEVER CUT | Gemini reasoning | This creates the prompts |
| NEVER CUT | Pre-rendered demo cities | This saves the demo |

---

## SECTION 10: PRODUCTION & SCALE

### 10.1 Rendering Job Queue
- **Cloud Tasks** or **Pub/Sub** for job distribution
- Worker pool on Cloud Run: auto-scale 1-50 instances
- Each worker handles one render pipeline (not CPU-bound — mostly API calls)
- Priority queue: paid users > free users > anonymous

### 10.2 Cost Per Render
| Component | Cost |
|-----------|------|
| Geocoding | $0.005 |
| Street View (8 images) | $0.056 |
| Earth Engine | Free |
| BigQuery (~50MB scan) | $0.0003 |
| Gemini 1.5 Pro (scene + reasoning) | ~$0.08 |
| Imagen 3 (6 images) | $0.24 |
| Cloud Storage | $0.0001 |
| Cloud Run compute (~30s) | $0.003 |
| **TOTAL** | **~$0.38/render** |

### 10.3 CDN Strategy
- Cloud CDN in front of Cloud Storage for rendered images
- Cache-Control: public, max-age=3600 for signed URLs
- Regional buckets: us-central1 (primary), europe-west1, asia-east1
- WebP format for 40% size reduction

### 10.4 Data Update Pipeline
- **Monthly**: Re-ingest ERA5 latest month into BigQuery
- **Annually**: Update CMIP6 projections when new model runs published
- **On IPCC release**: Review and update city_climate_summary table
- **Continuous**: Invalidate cached renders older than 30 days

### 10.5 Partnership Model
| Tier | Customer | Pricing | Features |
|------|----------|---------|----------|
| Free | Citizens | $0 | 3 renders/day, standard resolution |
| Pro | Researchers, journalists | $29/mo | Unlimited renders, HD, export |
| City | City governments | $499/mo | Bulk analysis, PDF reports, embeddable widget, API access |
| Enterprise | Insurance, real estate | Custom | API access, batch analysis, custom scenarios, SLA |

---

## SECTION 11: TECHNICAL RISKS & MITIGATIONS

| # | Risk | Probability | Impact | Mitigation |
|---|------|------------|--------|------------|
| 1 | Imagen 3 produces unrealistic images | Medium | Critical | Iterate prompts aggressively. Use Street View as style reference. Include "photorealistic, natural lighting" anchors. Pre-render demo cities and cherry-pick. |
| 2 | Imagen 3 rate limits / latency | Medium | High | 6 parallel requests. 30s timeout + 1 retry. Pre-render demos. Show first image immediately while others load. CSS overlay fallback. |
| 3 | Earth Engine slow / fails | Low | Medium | Initialize at startup, not per-request. Cache 7 days. Pre-compute demo cities. Skip satellite if needed — use CMIP6 alone. |
| 4 | Street View unavailable | Medium | Medium | Check metadata first. Expand radius. Fall back to satellite RGB. Clear user messaging. |
| 5 | BigQuery data not loaded in time | High | Medium | `mock_climate_data.py` with real IPCC numbers for 20 cities. Latitude-based interpolation for unknowns. This IS the Day 1 default. |
| 6 | Gemini hallucinations | Medium | High | Constrain with exact numerical bounds. Never let Gemini invent numbers. Validate JSON schema. "If uncertain, be conservative." |
| 7 | Pipeline >30s latency | Medium | Medium | Progressive loading via SSE — user sees data in 5s. Perception of speed matters. Pre-cached demos load in 200ms. |
| 8 | Demo venue poor internet | Medium | Critical | Static WebP files in `public/demo/`. Offline detection auto-switches. Mobile hotspot backup. |
| 9 | API key exposure | Medium | Critical | All keys server-side only. Next.js API routes proxy everything. Env vars, never committed. |
| 10 | Cost overrun during hackathon | Low | Low | Budget: ~$40 total. Imagen is the main cost ($0.24/render x ~50 test renders = $12). Set billing alerts at $50. |

---

## SECTION 12: BUSINESS MODEL & IMPACT

### 12.1 Revenue Streams
1. **B2C**: Freemium for individuals. Pro tier ($29/mo) for unlimited HD renders and exports
2. **B2G**: City government subscriptions ($499/mo) for planning tools, bulk analysis, PDF reports
3. **B2B API**: Insurance and real estate companies pay per-query for risk assessment data

### 12.2 Partnership Pathway
- **IPCC / UNFCCC**: Official visualization partner for COP conferences
- **C40 Cities**: Climate action plans for member cities
- **FEMA**: Flood risk visualization for US communities
- **World Bank**: Climate resilience planning for developing nations

### 12.3 Grant Opportunities
- Google.org Impact Challenge (climate track)
- NSF Convergence Accelerator
- Bloomberg Philanthropies climate data initiatives
- EU Horizon Europe climate adaptation grants

### 12.4 Press Strategy
- The story writes itself: "AI shows you what your neighborhood looks like in 2050"
- Pitch to: Wired, MIT Technology Review, The Verge, Reuters climate desk
- Viral mechanic: people share their own city's future on social media
- Demo to climate journalists at COP31

### 12.5 Long-Term Vision
The world's definitive climate visualization platform. Every city government uses AURORA for planning. Every insurance company uses AURORA for risk. Every citizen checks AURORA before buying a home. Climate data becomes climate understanding.

---

## SECTION 13: JUDGING CRITERIA ALIGNMENT

### Pitch Structures

**1-sentence:** "AURORA turns any city name into a photorealistic visualization of what that neighborhood looks like in 2050 under different climate scenarios, using NASA satellite data, IPCC projections, and Google's AI."

**30-second:** "Climate change is invisible — 2.8 degrees means nothing to most people. AURORA makes it visible and personal. Type any city, and we use 40 years of Earth Engine satellite data, CMIP6 climate projections, Gemini's multimodal reasoning, and Imagen 3's photorealistic rendering to show you your actual neighborhood in 2035, 2050, and 2075 — under current policy versus if we act now. We make climate change impossible to ignore."

**3-minute:** Use the pitch arc from Section 8.4, condensed. Lead with the Miami Beach image. Demo Phoenix live. End with the policy comparison split-screen and the hope frame.

### What Makes AURORA Impossible to Ignore

1. **Emotional impact**: The first demo image creates an involuntary reaction. Judges remember how they felt.
2. **Technical depth**: 12+ Google APIs, satellite data pipeline, climate modeling, AI reasoning, photorealistic rendering — this is a full-stack AI application.
3. **Real science**: Every number is from CMIP6/ERA5/IPCC. This isn't speculation — it's visualization of peer-reviewed data.
4. **Scalability**: Works for any city on Earth. Not a one-off demo.
5. **Business model**: Clear path from hackathon to product to company.
6. **Timeliness**: Climate is the defining challenge. This tool is needed now.

---

## SECTION 14: APPENDICES

### A. Complete Tech Stack
| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14, TypeScript, TailwindCSS, Zustand, React Query, Recharts, Framer Motion |
| Maps | Google Maps JavaScript API, @react-google-maps/api |
| API Gateway | Node.js, Express, Bull (job queue), ioredis |
| ML Pipeline | Python 3.11, FastAPI, google-cloud-aiplatform, google-generativeai, earthengine-api, google-cloud-bigquery, google-cloud-storage |
| Database | BigQuery (climate data), Firestore (user data), Redis (cache) |
| AI Models | Gemini 1.5 Pro (reasoning), Imagen 3 (generation) |
| Data Sources | Earth Engine (Landsat, Sentinel-2, MODIS), ERA5, CMIP6, YALE UHI |
| Infrastructure | Cloud Run, Cloud Storage, Cloud CDN, Firebase Hosting |
| Dev Tools | Docker, docker-compose, Turborepo, Poetry |

### B. Environment Variables
```bash
# Google Cloud
GOOGLE_CLOUD_PROJECT=aurora-climate
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# API Keys
GEMINI_API_KEY=AIza...
GOOGLE_MAPS_API_KEY=AIza...
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=AIza...

# Earth Engine
EE_SERVICE_ACCOUNT=aurora@aurora-climate.iam.gserviceaccount.com
EE_KEY_FILE=/path/to/ee-key.json

# Redis
REDIS_URL=redis://localhost:6379

# Cloud Storage
GCS_BUCKET=aurora-renders
GCS_REGION=us-central1

# Firebase
FIREBASE_PROJECT_ID=aurora-climate
NEXT_PUBLIC_FIREBASE_API_KEY=AIza...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=aurora-climate.firebaseapp.com

# Pipeline
PIPELINE_URL=http://localhost:8000
GATEWAY_PORT=3001
```

### C. Sample Earth Engine Query
```python
import ee

ee.Initialize(credentials=service_account_credentials, project='aurora-climate')

def get_satellite_metrics(lat: float, lng: float) -> dict:
    point = ee.Geometry.Point([lng, lat])
    bbox = point.buffer(5000)  # 5km radius

    # NDVI from Landsat 8 (2020-2025)
    landsat = (ee.ImageCollection('LANDSAT/LC08/C02/T1_TOA')
        .filterBounds(bbox)
        .filterDate('2020-01-01', '2025-12-31')
        .filter(ee.Filter.lt('CLOUD_COVER', 20)))

    ndvi = landsat.map(lambda img: img.normalizedDifference(['B5', 'B4']).rename('NDVI'))
    ndvi_stats = ndvi.mean().reduceRegion(
        reducer=ee.Reducer.mean(), geometry=bbox, scale=30
    ).getInfo()

    # Land Surface Temperature from MODIS
    modis = (ee.ImageCollection('MODIS/061/MOD11A1')
        .filterBounds(bbox)
        .filterDate('2020-01-01', '2025-12-31'))

    lst = modis.select('LST_Day_1km').mean().multiply(0.02).subtract(273.15)
    lst_stats = lst.reduceRegion(
        reducer=ee.Reducer.mean(), geometry=bbox, scale=1000
    ).getInfo()

    return {
        "ndvi_mean": ndvi_stats.get('NDVI', 0),
        "lst_mean_c": lst_stats.get('LST_Day_1km', 0),
    }
```

### D. Sample Imagen 3 Prompt Templates

**Coastal City (Sea Level Rise):**
```
Photorealistic street view of {city} in {year} under {scenario}.
Same architecture as reference but showing {slr_m}m of standing
seawater on the road surface. {vegetation_changes}. Buildings show
water damage marks at {slr_m}m height. Elevated walkways connect
buildings above flood level. Sandbag barriers at doorways. Rooftop
solar panels. Ground-floor shops boarded up. Emergency flood markers
on utility poles. Salt corrosion on metal surfaces.
Photorealistic, high detail, natural lighting, 4K quality.
```

**Desert City (Extreme Heat):**
```
Photorealistic street view of {city} in {year} under {scenario}.
Same architecture as reference but showing extreme heat effects.
Cracked and buckled asphalt. Dead and dying {vegetation_type}
trees. Heat shimmer visible over the road. {adaptation_features}.
Reflective white roof coatings on buildings. Covered pedestrian
walkways with shade structures. Dust haze reducing visibility.
Photorealistic, high detail, harsh sunlight with deep shadows.
```

**Temperate City (Mixed Effects):**
```
Photorealistic street view of {city} in {year} under {scenario}.
Same architecture as reference but showing {temp_delta}C warming
effects. {vegetation_changes}. {precipitation_effects}. {heat_effects}.
{adaptation_features}. Photorealistic, high detail, natural lighting.
```

### E. Climate Data Schema (Mock Data Example)
```python
DEMO_CITIES = {
    "miami_beach": {
        "name": "Miami Beach, FL",
        "lat": 25.7907, "lng": -80.1300,
        "coastal": True, "elevation_m": 1.2,
        "baseline": {"temp_c": 25.3, "precip_mm": 1520, "humidity_pct": 74},
        "projections": {
            "2035_ssp245": {"temp_c": 26.5, "delta_c": 1.2, "slr_m": 0.15, "heat_days": 62},
            "2035_ssp585": {"temp_c": 26.8, "delta_c": 1.5, "slr_m": 0.18, "heat_days": 71},
            "2050_ssp245": {"temp_c": 27.4, "delta_c": 2.1, "slr_m": 0.30, "heat_days": 78},
            "2050_ssp585": {"temp_c": 28.1, "delta_c": 2.8, "slr_m": 0.45, "heat_days": 98},
            "2075_ssp245": {"temp_c": 28.2, "delta_c": 2.9, "slr_m": 0.50, "heat_days": 95},
            "2075_ssp585": {"temp_c": 30.1, "delta_c": 4.8, "slr_m": 0.95, "heat_days": 145},
        }
    },
    # ... repeat for phoenix, san_francisco, amsterdam, mumbai
}
```

### F. Hackathon Budget Summary
| Item | Cost |
|------|------|
| Pre-rendered demo cities (5 x $0.38) | $1.90 |
| Prompt iteration (~50 test renders) | $19.00 |
| Live demo renders (~10) | $3.80 |
| Audience renders (~30) | $11.40 |
| Cloud Run (24hrs) | $2.00 |
| Redis (24hrs) | $0.50 |
| Maps API (~200 loads) | $1.40 |
| **TOTAL** | **~$40** |

---

## VERIFICATION PLAN

### How to Test End-to-End

1. **Earth Engine**: Run `python -c "import ee; ee.Initialize(); print('EE connected')"` — verify authentication
2. **Street View**: `curl "https://maps.googleapis.com/maps/api/streetview/metadata?location=25.79,-80.13&key=$KEY"` — verify `status: "OK"`
3. **Gemini**: Send a test image to scene_analyzer.py → verify structured JSON response
4. **Imagen 3**: Generate one test image with Miami Street View as reference → verify image returned
5. **Full Pipeline**: `POST /render` with Miami Beach → verify 6 images generated + narratives + metrics
6. **Frontend**: Navigate to explore page, search "Miami Beach" → verify progressive loading, image comparison, data panel
7. **Demo Mode**: Disconnect internet → verify demo cities load from static files
8. **Pre-render**: Run `scripts/seed-demo-data.sh` → verify all 30 images cached in Redis + static files

### Critical Path Items (Must Work for Demo)
1. CitySearchBar autocomplete
2. Pre-rendered demo cities load instantly
3. At least one live render completes in <30s
4. ImageComparison slider works on touch + mouse
5. Policy comparison (optimistic vs pessimistic) toggle works
6. Data panel shows real numbers with sources

---

## FILES TO MODIFY/CREATE

Since this is a greenfield project (empty workspace), ALL files listed in Section 6 need to be created. The **critical path files** in order of implementation priority:

1. `services/pipeline/app/services/image_generator.py` — Imagen 3 integration (the core product)
2. `services/pipeline/app/services/impact_reasoner.py` — Gemini climate reasoning (creates the prompts)
3. `services/pipeline/app/services/scene_analyzer.py` — Gemini scene analysis
4. `services/pipeline/app/services/earth_engine.py` — Satellite data pipeline
5. `services/pipeline/app/services/mock_climate_data.py` — Fallback climate data
6. `services/pipeline/app/routers/render.py` — Pipeline orchestrator
7. `apps/web/app/explore/page.tsx` — Main visualization page
8. `apps/web/components/visualization/ImageComparison.tsx` — Before/after slider
9. `apps/web/hooks/useRenderJob.ts` — SSE client for progressive loading
10. `apps/web/stores/visualization-store.ts` — Global state management
