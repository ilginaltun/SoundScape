# SoundScape — Urban Sound Architecture of the Kadıköy–Moda Coastline

An environmental sonification archive along a **~1.14 km pedestrian route** on the İstanbul coastline between Kadıköy and Moda, capturing six urban morphology channels per frame and translating them into real-time generative audio. The project records, for every frame, its geographic coordinates, building mass (left / right), sky openness, vegetation cover, pedestrian presence, and vehicle density — and turns this into an interactive web application with two modes: a pre-recorded route playback and a free-draw route analysis engine.

---

## Research Question

> **How can the spatial and sensory qualities of an urban coastal walk be encoded as sound — and what does the city *sound like* when its visual morphology becomes the score?**

---

## Dataset

| | |
|---|---|
| Rows (frames) | **31** |
| Columns | **11** |
| Primary key | `frame_id` (0–30) |
| Geographic scope | Kadıköy–Moda coastline, İstanbul — ~1.14 km linear route |
| Route polyline points | **62** (`ROAD_LL` array) |
| CRS | WGS84 (EPSG:4326) |
| Capture period | 2025–2026 |

**Source & method.** Frame data was extracted from street-level panoramic imagery retrieved through the **OpenStreetMap** ecosystem. A computer vision pipeline performed semantic pixel segmentation (building, sky, vegetation) and object detection (persons, vehicles) for each frame. The six resulting numeric values per frame are mapped to a generative audio synthesis engine built on the **Web Audio API**, using sine, triangle, and sawtooth oscillators, percussive synthesis, stereo panning, and convolution reverb.

The **Draw Mode** extends the static dataset dynamically: when a user draws a custom route, a live call to **OSRM** builds the road geometry and **Overpass API** queries OSM features within the bounding box, generating a procedurally interpolated 25-point frame dataset for any street in any city covered by OSM.

**Columns:** `frame_id`, `lat`, `lon`, `distance_from_start`, `building_ratio`, `sky_ratio`, `vegetation_ratio`, `building_left`, `building_right`, `person_count`, `vehicle_count`.
Full definitions in [`dataDictionary.json`](dataDictionary.json).

---

## Data Consistency Check

- **IDs.** All 31 frame IDs are sequential integers from 0 to 30. No gaps, no duplicates. The primary key is internally consistent.
- **Coordinates.** All `lat` values fall within the expected Kadıköy–Moda bounding box (lat ≈ 41.0364–41.0441, lon ≈ 28.9858–28.9906). No stray points detected outside the coastal corridor.
- **Distance monotonicity.** `distance_from_start` increases strictly from 0.0 m (frame 0) to 1135.88 m (frame 30). No backward jumps or duplicated distances. Inter-frame spacing ranges from ~23 m (frames 0–1) to ~93 m (frames 26–27), reflecting variable GPS sampling density.
- **Value domains.** All ratio fields (`building_left`, `building_right`, `sky_ratio`, `vegetation_ratio`, `building_ratio`) remain within [0, 1] — confirmed by inspection; the application additionally clamps these values in code. `person_count` maximum is 12; `vehicle_count` maximum is 12. The audio engine normalises these counts against ceilings of 10 and 8 respectively.
- **Polyline continuity.** The `ROAD_LL` array of 62 coordinate pairs forms a continuous, non-self-intersecting path. Its start point matches `frame_id = 0` and its end point matches `frame_id = 30`. No spatial discontinuities.
- **Left–right split consistency.** `building_left + building_right` never exceeds 1.0 for any frame, confirming the split is correctly computed from non-overlapping halves of the pixel area.

---

## Missing Data Analysis

| Column | Zero / effectively missing values | Note |
|---|---|---|
| `sky_ratio` | **1 frame = 0.000** (frame 6) | Fully enclosed urban canyon — a genuine morphological event, not a sensor failure. |
| `vegetation_ratio` | **2 frames ≈ 0.000** (frames 1, 6) | Dense built fabric at the Kadıköy departure point. |
| `building_left` | **5 frames = 0.000** | Open seafront or wide plaza sections — no left-side building wall visible. |
| `building_right` | **3 frames = 0.000** | Same coastal-edge logic; frames 13, 23, 27. |
| `person_count` | **14 frames = 0** | Quiet stretches; not missing data but a genuine spatial signal. |
| `vehicle_count` | **8 frames = 0** | Pedestrian-priority zones or park interiors. |
| `lat`, `lon`, `distance_from_start`, `building_ratio` | **0** | Complete across all 31 frames. |

There are **no literal null values** in the dataset. All zero values reflect genuine morphological conditions rather than failed measurements.

Beyond literal zeros, the dataset carries **structural under-coding** worth stating openly:

- **Temporal flattening.** Each frame is a single photographic snapshot. The same location at rush hour vs. midday would yield radically different `person_count` and `vehicle_count`. Time of day is not recorded; the archive captures *one temporal slice* of the route, not a time-averaged portrait.
- **2-D pixel area ≠ volumetric mass.** `building_left` and `building_right` measure projected pixel area. A low wide building and a narrow tall building can produce identical ratios. Depth, material, and fenestration are absent.
- **Vegetation conflation.** `vegetation_ratio` does not distinguish street trees, planted verges, park interiors, or coastal scrub. A single large tree can produce the same ratio as a dense garden.
- **Agent counting without classification.** `person_count` does not separate stationary from moving persons, or cyclists from pedestrians. `vehicle_count` conflates taxis, buses, motorcycles, and private cars.

---

## Outlier Analysis

- **`person_count` spike at frame 2** records 12 — the dataset maximum — while adjacent frames 1 and 3 record 0 and 0 respectively. This isolated spike (> 4× the mean of 1.97) likely reflects a momentary crowd event (market, bus stop discharge) rather than a stable neighbourhood characteristic. The audio engine caps percussive hits at 4 per frame, smoothing this outlier perceptually.
- **`vehicle_count` = 12 at frames 9 and 23** — both record the maximum, at different points along the route. Both locations correspond to road-adjacent segments of the coastal promenade where vehicle traffic from the main artery is fully visible. These are genuine high-traffic moments, not sensor artefacts.
- **`sky_ratio` = 0.000 at frame 6** — the only fully enclosed frame in the dataset. This is a true urban canyon moment and produces the most acoustically dense single point in playback: both building oscillators fire at maximum depth with no sky or vegetation counterpoint.
- **Near-zero building mass band (frames 20–28).** The latter third of the route shows `building_left` and `building_right` consistently below 0.065 while `vegetation_ratio` rises above 0.30. This is the clearest zone transition: the street fabric gives way to the Moda coastal park, and the soundscape shifts from low sine tones to mid-register triangle-wave foliage sounds.
- **IQR outlier test (`person_count`).** IQR = 2.5. Upper fence = 2.5 + 1.5 × 2.5 = 6.25. Frames 0 (10) and 2 (12) fall above this threshold — confirmed statistical outliers. No lower-fence outliers (minimum = 0 is within range).
- **IQR outlier test (`vehicle_count`).** IQR = 7.5. Upper fence = 8 + 1.5 × 7.5 = 19.25. No frames exceed this; the two peaks at 12 are high but not formally outside the fence. `vehicle_count` distribution is right-skewed but not outlier-heavy.

---

## Distribution Plots & Statistical Summary

**Numeric summary (31 frames)**

| | lat | lon | distance (m) | building_left | building_right | sky_ratio | vegetation_ratio | person_count | vehicle_count |
|---|---|---|---|---|---|---|---|---|---|
| **mean** | 41.0402 | 28.9886 | 567.9 | 0.136 | 0.135 | 0.143 | 0.244 | 1.97 | 4.55 |
| **std** | 0.0025 | 0.0013 | 338.2 | 0.187 | 0.196 | 0.107 | 0.158 | 2.92 | 4.01 |
| **min** | 41.0364 | 28.9858 | 0.0 | 0.000 | 0.000 | 0.000 | 0.000 | 0 | 0 |
| **median** | 41.0400 | 28.9889 | 567.9 | 0.034 | 0.030 | 0.121 | 0.261 | 1 | 4 |
| **max** | 41.0441 | 28.9906 | 1135.9 | 0.651 | 0.612 | 0.341 | 0.559 | 12 | 12 |

**Plots**

![Spatial distribution](main/spatial_distribution.png)
![Channel distributions along route](main/channel_distribution.png)
![Zone strip](main/zone_strip.png)
![Ratio fields box plot](main/ratio_boxplot.png)
![Agent count histograms](main/count_histograms.png)
![Correlation matrix](main/correlation_matrix.png)

**Morphological character by zone**

- **Frames 0–6 (0–278 m) — Dense urban approach.** High building ratios on both sides, `sky_ratio` near zero at frame 6, near-zero vegetation. The soundscape is dominated by low-register sine oscillators panned hard left and right, evoking the acoustic shadow of a street canyon.
- **Frames 7–16 (348–649 m) — Transitional park edge.** `vegetation_ratio` rises sharply (0.19–0.52). Sky opens. Building mass drops on both sides. Triangle-wave foliage tones emerge; the sonic texture lightens.
- **Frames 17–30 (696–1136 m) — Coastal park.** Vegetation dominant (0.27–0.56), buildings nearly absent, sky consistently present (0.06–0.34). Pedestrian presence persists at moderate density. The audio landscape is the most spectrally sparse — deliberate contrast to the opening.

**Notable correlation.** `building_left` and `building_right` show moderate positive correlation (r ≈ 0.55), consistent with the street-canyon geometry where both walls tend to appear together. `vegetation_ratio` and `building_left` / `building_right` show negative correlation (r ≈ −0.35 to −0.42), confirming that greener frames are structurally less built.

---

## Repeated Data Check

- **Fully duplicated rows: 0.** No two frames are identical across all fields.
- **Repeated zero combinations.** Frames 27 and 28 both record `building_left = 0.000`; frames 13, 23, and 27 record `building_right = 0.000`. These are consecutive open-seafront positions where no building wall appears — genuine repetition of a spatial condition, not data-entry duplication.
- **No repeated coordinates.** All 31 `(lat, lon)` pairs are unique. The route does not double back on itself.
- **Recurring zero `person_count`.** 14 of 31 frames record zero persons. Of these 14 frames, 11 have `vegetation_ratio > 0.20` — the quieter, greener frames are systematically less populated. This is a genuine spatial signal rather than a data artefact, and is the strongest intra-dataset pattern after the zone transition.
- **`vehicle_count` = 0** appears in 8 frames, all clustered in the park-interior segment (frames 4, 11, 17, 18, 27, 29, 30), confirming this as a pedestrian-priority zone rather than a data gap.

---

## Bias Check

- **Single-route, single-time sample.** The dataset covers one walk, captured once. It cannot represent the route at different times of day, seasons, or weather conditions. A rainy weekday morning would produce radically different `person_count` and `vehicle_count` values.
- **Direction bias.** The route is walked in one direction only (Kadıköy → Moda). The left/right building split (`building_left` / `building_right`) would reverse for the return walk; the dataset encodes a directional viewpoint, not a neutral spatial record.
- **Pixel-area dominance bias.** Because ratios are computed from pixel area, large nearby surfaces overwhelm small but spatially significant distant features. The dataset systematically over-represents the *nearest* dominant surface rather than the *volumetrically most significant* spatial element.
- **Agent detection under-count.** Person and vehicle counts depend on visibility. Occluded pedestrians (behind trees, inside vehicles, in doorways) are not counted. The dataset likely **under-counts agents**, particularly in vegetation-dense frames where foliage occludes parts of the visual field.
- **Coastline survivorship.** Only frames with sufficient image quality from the source imagery are included. Frames where imagery was unavailable, obscured by scaffolding, or poorly lit are absent — the archive over-represents well-documented, open-façade sections of the route.
- **Skew to be aware of when reading any chart.** `vegetation_ratio` mean (0.244) is higher than `sky_ratio` mean (0.143) and substantially higher than `building_left` mean (0.136) — which correctly reflects the green-dominant character of the Moda coastal park section, but would misrepresent the character of the opening urban segment if taken as a whole-route average. The route is spatially lopsided: the first third is built, the last two-thirds are green.

---

## Project Documentation & Reflection

### Narrative

A coastal walk in İstanbul is rarely *just* a walk. It is a compression of the city's most contradictory qualities — the density of Kadıköy's commercial blocks giving way, gradually, to the breathing room of Moda's planted seafront. SoundScape turns this gradient into something you can *hear* without seeing: a score generated from the city's own spatial data, played back in the time it takes to walk the route. The project argues that urban morphology is already a kind of music — it simply hasn't been listened to yet.

### Key Message

**The city already has a score. SoundScape plays it.** Six channels of spatial data — building mass, sky openness, vegetation, pedestrian presence, vehicle density — are sufficient to generate a meaningfully differentiated sonic portrait of an urban sequence. The soundscape is not an illustration of the walk; it *is* the walk, re-encoded as audio. The tool argues that sonification is not decoration; it is analysis made audible.

### Intended Audience

**Urban designers, sound artists, and everyday walkers** interested in experiencing the city through a non-visual register. The tool is designed for two levels of engagement: casual listeners who simply press play and let the route unfold, and researchers who want to toggle individual channels, adjust reverb and speed, and inspect the "What have you heard?" escape chart to understand the route's morphological composition. The Draw Mode extends the tool to any city covered by OpenStreetMap, making every street a potential score.

Design decisions are calibrated for a data-literate but non-specialist audience: the channel parameters (building mass, sky ratio, etc.) are visible in real time in the Parameters panel, but require no numerical literacy to hear — the building walls *sound heavy*, the park *sounds open*.

### Design Decisions

- **Dark editorial aesthetic.** The interface is opaque black (`#0a0a0a`) with yellow (`#ffff00`) as the sole accent. The darkness references the acoustic shadow of the city's back streets and ensures that the data's subtle tonal distinctions read clearly on screen. Yellow functions as a signal colour — the same logic as construction tape or wayfinding markers — directing attention without adding noise.
- **IBM Plex Mono typography.** Monospaced text reads like instrumentation, not consumer UI. The interface should feel like a measurement device — precise, neutral, technical — because it *is* one.
- **Stereo spatial encoding.** Building mass is split left/right, mapped to hard-panned oscillators. The listener hears the street canyon's geometry in their headphones: a wide building on the left registers as a pressure on the left ear. This spatial fidelity is the core design claim of the project.
- **Escape as reflection.** The `Escape` key is repurposed from its conventional "exit" function into an act of post-journey analysis. Pressing it reveals the "What have you heard?" chart — a retrospective overlay of all six channels across the route's timeline. The question is asked *after* the walk, not before, so the chart is encountered as memory rather than preview.
- **Layer toggles.** Each of the six channels can be muted independently. This is not merely a usability feature; it is an analytical one. Muting all layers except `vegetation_ratio`, for instance, lets the listener isolate the park's acoustic signature and hear where it begins and ends along the route.
- **Draw Mode generality.** The pre-recorded route is a demonstration; the tool's real ambition is the Draw Mode, where any user can compose their own walk anywhere in the world and immediately hear it as sound. The design deliberately keeps the draw interface minimal — waypoints, an analyze button, a play button — so the focus remains on listening.

### Critical Evaluation

The project succeeds in demonstrating that sonification can be a legible, spatially accurate mode of urban representation. A listener who walks the Kadıköy–Moda route after hearing the SoundScape version will recognise the canyon moment (frame 6), the park entry (around frame 7), and the traffic nodes (frames 9 and 22–23) as sonic memories. The correlation between visual morphology and audio texture is tight enough to be predictive.

However, three structural limitations remain unresolved:

1. **31 frames is a thin dataset.** At the default playback speed of 400 ms per frame, the full route lasts approximately 12 seconds — experientially brief. The reverb tail and smooth envelope shapes bridge the gaps perceptually, but the quantisation of a 1.14 km walk into 31 discrete samples is audible and imposes a step-wise quality on what should be a continuous spatial experience. A denser dataset (one frame per 5–10 m) would substantially improve the perceptual realism.

2. **Draw Mode data is an approximation.** Overpass API returns building centroids, not façade surfaces; the left/right split is computed from raw proximity counts rather than true directional raycast geometry. The resulting values are plausible but not photogrammetrically accurate. A user drawing a route in a city they know may find the audio portrait surprising — not always in ways that reflect genuine spatial difference.

3. **Simultaneous channels create auditory masking.** When all six layers are active, lower-frequency elements (building sine oscillators, vehicle sawtooth) tend to mask the upper-register sky and vegetation tones. The layer toggle system is a practical mitigation, but the fundamental psychoacoustic tension between spatial completeness and auditory clarity is not fully resolved. A future version could implement dynamic range compression or frequency-aware mixing to balance the layers automatically.

### Unexpected Patterns

- **The quietest moment is not the most open.** Frame 27 has zero buildings on both sides, zero pedestrians, and zero vehicles — yet `sky_ratio` is only 0.341 and `vegetation_ratio` is 0.559. The expected perceptual result would be a sparse, open sound dominated by sky tones. Instead, the dense vegetation ratio keeps the mid-register triangle tones persistently active, filling what should be the "most silent" frame of the route with organic, mid-frequency texture. Greenery, not silence, fills the most open section.

- **The acoustically densest moment is not the most populated one.** Frame 2 has the highest `person_count` (12), but frame 9 is perceptually louder: `vehicle_count = 12` combined with moderate building mass on both sides produces a dense mix where the sawtooth vehicle layer dominates. The busiest *sounding* frame is not the most inhabited one — vehicle sound, in the synthesis model, is heavier than human sound. This reflects a real urban truth that the data amplifies.

- **An inverse relationship between name and form, repeated here as form and function.** In the Kadıköy departure section, `building_ratio` is at its highest (frames 1 and 6 both exceed 0.55) while `person_count` and `vegetation_ratio` are at their lowest. The most urban-looking frames are the least inhabited ones in this dataset — the city's densest built fabric is, in this snapshot, its quietest in human terms. The park section, by contrast, is sparsely built but persistently occupied. The architecture does not follow the people.

---

## Repository Contents

```
dataset.pkl            Cleaned dataset (pandas DataFrame, 31 × 11)
dataset.csv            Same dataset in CSV format for convenience
dataDictionary.json    Per-column definitions, types, domains, and notes
metadata.json          Dataset provenance, scope, method, and application details
requirements.txt       Python dependencies
soundscape.html        Complete single-file interactive application
README.md              This file
plots/
  spatial_distribution.png
  channel_distribution.png
  zone_strip.png
  ratio_boxplot.png
  count_histograms.png
  correlation_matrix.png
```

### Reproducing

```bash
pip install -r requirements.txt
python build_dataset.py    # Recreates dataset.pkl and dataset.csv from raw data
python make_plots.py       # Generates all plots in plots/
```

The application (`soundscape.html`) is fully self-contained. Open it in any modern browser. An internet connection is required for the Leaflet tile layer, OSRM routing, and Overpass API in Draw Mode.

**Author:** ılgın altun · ITU MBL549E Special Topics in Architectural Design Computing, 2026.
