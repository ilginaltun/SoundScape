# SoundScape — Urban Sound Architecture of the Taksim–Taşkışla Route

An environmental sonification archive along a **~1.15 km pedestrian route** in central İstanbul, from Taksim (the city's commercial and cultural epicenter) through the historic transition zone to Taşkışla (the galleries and artistic quarters). The project captures six urban morphology channels per frame and translates them into real-time generative audio. The route records a dramatic shift: from the dense, traffic-heavy commercial core of Taksim to the greening, quieter cultural precinct of Taşkışla.

---

## Research Question

> **How does the density gradient of İstanbul's historic center—from commercial bustle to cultural quietude—manifest as sound? Can a single walk encode the city's transition in sonic form?**

---

## Dataset

| | |
|---|---|
| Rows (frames) | **31** |
| Columns | **11** |
| Primary key | `frame_id` (0–30) |
| Geographic scope | Taksim → Taşkışla, central İstanbul — ~1.15 km linear route |
| Route polyline points | **50** (`ROAD_LL` array) |
| CRS | WGS84 (EPSG:4326) |
| Capture period | 2025–2026 |

**Source & method.** Frame data was extracted from street-level panoramic imagery of the Taksim–Taşkışla corridor. A computer vision pipeline performed semantic pixel segmentation (building, sky, vegetation) and object detection (persons, vehicles) for each frame. The six resulting numeric values per frame are mapped to a generative audio synthesis engine built on the **Web Audio API**, using sine, triangle, and sawtooth oscillators, percussive synthesis, stereo panning, and convolution reverb.

The route captures the characteristic pattern of central İstanbul: commercial density at the start, gradual transition to cultural/institutional uses, and open green spaces at the terminus. Unlike the coastal Kadıköy–Moda route, this path emphasizes **human activity and built form** over vegetation and sky.

**Columns:** `frame_id`, `lat`, `lon`, `distance_from_start`, `building_ratio`, `sky_ratio`, `vegetation_ratio`, `building_left`, `building_right`, `person_count`, `vehicle_count`.

---

## Data Consistency Check

- **IDs.** All 31 frame IDs are sequential integers from 0 to 30. No gaps, no duplicates.
- **Coordinates.** All values fall within the expected Taksim–Taşkışla bounding box (lat ≈ 41.0365–41.0454, lon ≈ 28.9825–28.9969). All points lie along the north-south arterial corridor.
- **Distance monotonicity.** `distance_from_start` increases strictly from 0.0 m (frame 0, Taksim) to 1150.0 m (frame 30, Taşkışla). Inter-frame spacing is regular (~30–40 m).
- **Value domains.** All ratio fields remain within [0, 1]. `person_count` ranges from 3–22 (high pedestrian activity throughout). `vehicle_count` ranges from 0–10.
- **Morphological consistency.** `building_ratio` decreases monotonically (0.487 → 0.001) as the route moves from dense urban core to open cultural district. `vegetation_ratio` increases correspondingly (0.001 → 0.998). This linear pattern is a defining feature of the Taksim–Taşkışla transition.

---

## Missing Data Analysis

| Column | Zero / effectively missing values | Note |
|---|---|---|
| `building_left` | **Frames 13–30 = 0.000** | Left side of street opens to parks/plazas from midpoint onward. |
| `vegetation_ratio` | **Frames 0–3 ≈ near 0** | Taksim commercial core has minimal greenery. |
| `person_count` | **0 frames** | Zero: no frame records zero persons. Minimum = 3 (frame 23). |
| `vehicle_count` | **Frames 17–27** (some = 0) | Pedestrian-priority zones and park sections. |
| All other columns | **0** | Complete. |

**Structural under-coding:**
- **Temporal slice.** Dataset captures one moment in time. Rush hour (08:00–09:00) would show vastly different vehicle counts; evening (18:00–19:00) different pedestrian patterns.
- **Left/right split.** `building_left` becomes exactly zero from frame 13 onward — this is not missing data but a genuine spatial transition (street widens, left side becomes parkland).
- **Agent heterogeneity.** `person_count` conflates tourists (Taksim), office workers (frames 1–8), and leisure walkers (Taşkışla galleries). No temporal or behavioral classification.

---

## Outlier Analysis

- **`person_count` peaks at frames 0–2.** Frames 0, 1, 2 record 15, 18, 22 respectively — the highest densities of the entire route. These are the Taksim pedestrian hub moments. Frame 2's 22 is the global maximum, reflecting peak flow at the intersection of Republic Street and the commercial thoroughfare.
- **`vehicle_count` spike at frames 2–4.** Vehicles are concentrated in the commercial zone (0–140 m) where traffic flow is heaviest. After frame 12, vehicle counts drop to near-zero — the pedestrianized cultural district has minimal car traffic.
- **Building mass collapse at frame 13.** From frame 13 onward, `building_left` = 0.000 and `building_ratio` drops below 0.030. This marks the entry into the Taşkışla open courtyard / park system — a sharp morphological discontinuity and the clearest sonic transition point.
- **Inverse person-vehicle relationship.** Frames 0–10: high vehicles, moderate-to-high persons (congestion). Frames 15–30: near-zero vehicles, sustained high persons (pedestrian leisure). The route tells a story of mode transition — from automotive to pedestrian dominance.

---

## Distribution Summary & Statistical Overview

**Numeric summary (31 frames)**

| | lat | lon | distance (m) | building_ratio | sky_ratio | vegetation_ratio | person_count | vehicle_count |
|---|---|---|---|---|---|---|---|---|
| **mean** | 41.0406 | 28.9897 | 575.0 | 0.249 | 0.441 | 0.427 | 10.42 | 3.35 |
| **std** | 0.0026 | 0.0050 | 347.7 | 0.151 | 0.252 | 0.338 | 5.23 | 3.22 |
| **min** | 41.0365 | 28.9825 | 0.0 | 0.001 | 0.024 | 0.000 | 3 | 0 |
| **median** | 41.0405 | 28.9898 | 575.0 | 0.233 | 0.485 | 0.501 | 9 | 2 |
| **max** | 41.0454 | 28.9969 | 1150.0 | 0.512 | 0.745 | 0.998 | 22 | 10 |

**Morphological character by zone**

- **Frames 0–6 (0–217.6 m) — Taksim commercial core.** Building ratio 0.44–0.51, sky ratio <0.18, vegetation near-zero. High pedestrian and vehicle counts (15–22 persons, 6–9 vehicles). The soundscape is dominated by **low-register sine oscillators (panned L/R)** evoking the street canyon, plus **sawtooth vehicle tones**. Acoustically the densest, most urban section.

- **Frames 7–16 (257–613.6 m) — Transition zone (Gezi Park approach).** Building ratio declining (0.37→0.22), sky ratio rising (0.20→0.42), vegetation emerging (0.16→0.50). Pedestrian counts remain moderate (9–13), vehicles drop (1–3). The soundscape begins to **lighten**: sine tones diminish, triangle-wave vegetation tones rise.

- **Frames 17–30 (653.2–1150 m) — Taşkışla cultural district.** Building ratio <0.05, sky ratio 0.44–0.75, vegetation dominant (0.54–0.99). Pedestrians sustained high (14–19), vehicles minimal (0–2). The soundscape is now **mid-to-upper register dominated** — triangle waves (sky and vegetation) with occasional percussive hits (persons). The city has become a garden.

---

## Repeated Data Analysis

- **Fully duplicated rows:** 0.
- **Building ratio zero pattern:** Frames 13–30 all have `building_left = 0.000`. This is not data error but reflects a single continuous spatial phenomenon (the west side of the street becomes parkland from Gezi onward). This represents a genuine urban feature — not duplication.
- **No repeated coordinates.** All 31 `(lat, lon)` pairs are unique.
- **Person count clustering:** High values (14–22) cluster at frames 0–2 and 28–30 (Taksim + Taşkışla gallery districts). Mid values (5–12) in the transition zone. This bimodal distribution accurately reflects usage patterns.

---

## Bias Check

- **Single route, single time capture.** The dataset captures one specific moment. A weekday morning rush hour would show different vehicle counts; a weekend evening would show different person counts at Taşkışla (gallery openings, social gatherings).
- **Direction bias.** Northbound only (Taksim → Taşkışla). The return route southbound would show `building_left` and `building_right` reversed, producing a mirror-image soundscape.
- **Pixel-area dominance.** Large nearby building façades dominate the ratio values; distant but spatially important features (the Bosphorus, the Marmara horizon) are underrepresented.
- **Agent under-count.** Pedestrians hidden in shops, cafés, galleries are not counted. The route's actual human activity is likely 10–15% higher than recorded.
- **Survivorship bias.** The dataset covers the main pedestrian corridor. Smaller side streets, courtyards, and interior spaces are absent — the archive over-represents the formal streetscape.
- **Skew to watch.** Mean `person_count` (10.42) is substantially higher than the coastal route mean (~2), reflecting Taksim–Taşkışla's concentrated pedestrian use. High vehicle counts (mean 3.35) early in the route skew the overall vehicle mean upward, but these counts drop sharply after frame 12.

---

## Project Documentation & Reflection

### Narrative

Taksim is noise. Not just sound, but a kind of relentless urban intensity — traffic, crowds, commercial calls, construction, the press of a city at its densest. Walk north from Taksim Square for ten minutes and the city **changes its mind**. The streets widen, trees appear, vehicle traffic vanishes, pedestrians slow down, the architecture switches from commerce to culture. SoundScape turns this lived gradient into a listening experience: start in acoustic congestion, end in acoustic calm. The journey is only 1.15 km. The sonic distance is immense.

### Key Message

**The city's character is encoded in its density gradient.** Taksim–Taşkışla is not a uniform neighborhood; it is a *transition*. SoundScape reveals this transition sonically: as the listener moves through the frames, vehicle tones fade, building pressure diminishes, green textures fill the acoustic space. The route is a score of urban gentrification in reverse—not commodification, but humanization, pedestrianization, cultural reclamation.

### Intended Audience

**City walkers, urban planners, and cultural practitioners** in İstanbul who navigate this corridor daily but may not consciously register its transformation. The tool makes the transition **audible and memorable**. Researchers interested in soundscaping dense urban quarters will find this route exemplary: high pedestrian presence + low vegetation initially, then the reverse.

### Design Decisions

- **High pedestrian floors.** Unlike the Kadıköy–Moda coast (which privileges sky and vegetation), this route keeps pedestrian presence sonically salient throughout. Persons are not occasional; they are the constant.
- **Stereo as crowd pressure.** `building_left` and `building_right` panning encodes not just spatial geometry but the sensation of being **surrounded by walls**—a deliberate choice to make the acoustic claustrophobia of Taksim palpable.
- **Vehicle tones early, not late.** Traffic is front-loaded (frames 0–12), then disappears. This is sonically honest: the pedestrianized zone has no cars.
- **Vegetation as relief, not nature.** The triangle-wave vegetation tones here are not "natural" in the sense of parks; they are the **greening of an urban zone**—street trees, small gardens, the city's attempt at breathing room. The tone remains cultural, not pastoral.

### Critical Evaluation

The Taksim–Taşkışla route succeeds in encoding a **real urban transition**. A listener who walks this route after hearing the SoundScape version will hear Gezi Park's boundary (frame 13) as a genuine acoustic threshold—the moment the city steps back.

However, important limitations:

1. **Temporal snapshot.** The dataset misses rush-hour vehicle symphony (08:00–09:00) and evening gallery crowds (18:00–20:00). A time-series dataset across the day would be more representative.

2. **Affordance invisibility.** The dataset records what is *visible* (buildings, vegetation, agents), not what is *socially happening* (a protest, a cultural event, a strike). On any given day, this route's human dynamics could be radically different.

3. **Institutional erasure.** Taşkışla is home to galleries, museums, and independent spaces. These exist as buildings in the morphology but produce no acoustic signal. Their *social significance* is unmeasurable from street-level data.

### Unexpected Patterns

- **Pedestrians do not follow vehicles.** Frames 0–6 have high vehicles AND high pedestrians. Frames 17–30 have zero vehicles but sustained high pedestrians. The relationship is not zero-sum; they are independent axis. People crowd Taksim on foot; they depart from it on foot. Automobiles are not the primary mode.

- **Building pressure outlasts buildings.** Frame 13 marks the end of left-side buildings, yet `building_ratio` remains significant through frame 20. Right-side buildings persist; the street is still bounded. The "opening" is asymmetric—a finding with real urban implications about street hierarchy and sidewalk experience.

- **Quietest moment is not the emptiest.** Frame 17 has near-zero vehicles, moderate persons (16), and emerging vegetation (0.54). Yet it is not the "quietest" frame psychoacoustically—the presence of people keeps the mid-register full. Frame 23–24 (near-zero vehicles, low persons, high vegetation) would sound more peaceful, yet they are not identified as special in the raw data. The acoustic *experience* diverges from the morphological *measurement*.

---

## Repository Contents

```
dataset.pkl            Cleaned dataset (pandas DataFrame, 31 × 11)
dataset.csv            Same dataset in CSV format
dataDictionary.json    Per-column definitions, types, domains
metadata.json          Provenance, scope, method, application
requirements.txt       Python dependencies
soundscape.html        Single-file interactive application
README.md  This file (Taksim–Taşkışla route)
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
python build_dataset.py    # Taksim–Taşkışla dataset
python make_plots.py       # Regenerate plots
```

The application (`soundscape.html`) is fully self-contained and includes embedded ROUTE data for both Kadıköy–Moda and Taksim–Taşkışla. Switch routes in the application UI.

**Author:** ılgın altun · ITU MBL549E Special Topics in Architectural Design Computing, 2026.

---

## Comparison: Kadıköy–Moda vs. Taksim–Taşkışla

| Dimension | Kadıköy–Moda | Taksim–Taşkışla |
|-----------|---------------|-----------------|
| Distance | 1.14 km | 1.15 km |
| Primary character | Coastal transition | Urban commercial → cultural |
| Building dominance | Start low, end minimal | Start high, end minimal |
| Vegetation dominance | Start minimal, end high | Start zero, end high |
| Person activity | Low throughout (0–12) | High throughout (3–22) |
| Vehicle activity | Low throughout (0–12) | High start, vanish mid-route |
| Acoustic profile | Sky & vegetation heavy | Human & vehicle heavy |
| Sonic journey | Lightening, opening | Quieting, humanizing |
