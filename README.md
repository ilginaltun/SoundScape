# SoundScape — Translating Urban Morphology into Generative Audio

An environmental sonification study exploring how **spatial data from urban environments can be encoded as real-time generative sound**, and what acoustic narratives emerge when visual morphology becomes the primary score. The project captures six channels of urban morphological data per geographic frame and maps them directly to synthesis parameters (oscillator frequency, waveform type, pan position, amplitude), asking: can an observer recognize spatial transformation through sound alone?

---

## Research Question

> **Can spatial morphology be perceived and understood through sound without visual reference? What does sonification reveal about urban texture that visualization obscures—and what does it lose?**

More specifically:
- **Encoding**: How do we map six-dimensional morphological data (building mass, sky openness, vegetation, pedestrian density, vehicle density) to a perceptually coherent and navigable soundscape?
- **Legibility**: Can a listener track their movement through an urban sequence by ear alone, identifying key moments and zones from the acoustic signature?
- **Revelation**: What patterns in the data become *audible* through sonification that remain hidden in numerical tables or statistical plots? What spatial truths emerge when we "listen to the city"?
- **Loss**: What critical information—materiality, phenomenological richness, social dynamics—cannot be captured by morphological measurement and therefore cannot be sonified?

---

## Methodology

**Two routes, one sonification engine.** The project applies identical audio synthesis rules to two distinct urban corridors—a coastal transition and an urban commercial-to-cultural gradient—to test whether the sonification reveals universal patterns of spatial transformation or is specific to each site's character.

**6-Channel Data → Sound Mapping:**
- `building_mass` (left/right) → sine oscillator (low frequency, panned) — "pressure"
- `sky_ratio` → triangle oscillator (high frequency) — "openness"
- `vegetation_ratio` → triangle oscillator (mid frequency) — "texture"
- `pedestrian_count` → percussive spikes (temporal density) — "liveliness"
- `vehicle_count` → sawtooth oscillator (harmonic richness) — "mechanical presence"

**Playback modes:** (1) Route replay at variable speeds (0.25x–4x), with reverb and layer toggles. (2) Free-draw mode: user draws custom route, OSRM + Overpass API generate morphological data, sonification generates in real-time.

---

## Hypothesis

**H1: Morphological coherence is audible.** A listener hearing two different routes will perceive their distinct character—coastal vs. urban, green vs. built, quiet vs. dense—*solely through sound*.

**H2: Transitions are sonically salient.** Key spatial boundaries (park entry, density drop, vehicle vanishing point) produce recognizable acoustic shifts that listeners can identify without visual context.

**H3: Sonification amplifies certain patterns while suppressing others.** Patterns that are visually subtle (e.g., a 10% increase in vegetation ratio) may become acoustically dominant; conversely, spatially significant features (e.g., material texture, architectural detail, social activity) that cannot be measured may produce no acoustic signal.

---

## Key Questions Under Investigation

1. **Perceptual mapping**: Do listeners' acoustic judgments (e.g., "this section sounds denser") correlate with actual morphological values?

2. **Route memory**: After listening to a full route, can a listener identify moments they heard before (the "park entrance" or "traffic density peak") when played in random order?

3. **Aesthetic coherence**: Does sonification of morphological data produce musically coherent sound, or is it primarily analytical—interesting as data visualization but not as listening experience?

4. **Embodied navigation**: Does sonified morphology aid or confuse navigation? Can you walk a real route while listening to its sonification without the visual field?

5. **Comparative morphology**: Do two routes with *different* morphological profiles produce *audibly different* soundscapes? Do two routes with *similar* profiles sound alike?

6. **The sonification paradox**: Information that is trivial visually (a color change, a subtle shift in building height) may be imperceptible sonically; conversely, data that is numerically small (3% vegetation) may sound significant. What gets amplified in the translation?

---

## Dataset Structure

**Two routes, identical frame structure:**
- 31 frames per route
- ~1.1–1.15 km each
- 6 morphological channels per frame
- Temporal alignment: ~400 ms per frame at playback speed 1x

**Data sources:** OpenStreetMap street-level imagery + semantic segmentation (building, sky, vegetation) + object detection (persons, vehicles).

**Variability:** Routes selected to represent different urban typologies—one coastal/natural, one urban/built—to test whether sonification is generic (works for any morphology) or site-specific (reveals character unique to each place).

---

## Application & Design Rationale

**Single-file HTML application** with:
- **Route selector**: choose pre-recorded route or draw custom route
- **Playback controls**: speed (0.25x–4x), reverb depth, master volume
- **Layer toggles**: isolate individual channels (listen to building pressure alone; listen to person-spikes alone)
- **"What have you heard?" analysis**: after playback, view stacked area chart of all 6 channels across the route's timeline, with audio playback of each channel's characteristic tone
- **Map display**: route shown geographically, with user's position in the soundscape overlaid

**Design philosophy:**
- **Dark/minimal aesthetic** (black + yellow) to reduce visual distraction — the listener should attend to sound
- **Monospace typography** (IBM Plex Mono) to evoke a measurement/analysis tool, not a consumer app
- **Stereo spatial encoding** (`building_left` panned hard L, `building_right` hard R) to map the street's geometry into headspace
- **Percussive moments** (person-spikes) to punctuate the continuous oscillators, making pedestrian presence a lived/embodied signal

---

## Expected Outcomes

### Affirmative Results (H1, H2, H3 supported):
- Listeners reliably identify route character by ear
- Transitions produce audible discontinuities
- Sonification creates emergent patterns (e.g., "traffic peaks are harmonically dense") that weren't explicit in the input data

### Negative Results (hypotheses falsified):
- Routes sound homogeneous despite morphological difference
- Transitions produce no perceptual shifts
- Sonification obscures rather than clarifies spatial pattern

### Nuanced Results (most likely):
- Some morphological features (density gradients, area changes) are audibly legible
- Others (categorical transitions, semantic shifts) produce no sonic signal
- The sonification is *interesting* as a representation but *impoverished* compared to visual/embodied navigation
- Sonification reveals correlations that visualization misses (e.g., person-density + vehicle-density are **not** correlated in space; sonically, they fight for auditory attention)

---

## Broader Implications

**For urban design**: Can sonification become a tool for assessing spatial quality? Current methods are visual (photography, rendering) or embodied (walking). Sonification offers a third mode—a portable, replayable representation that abstracts away the visual but preserves morphological essence.

**For sound art & composition**: Can urban morphology be understood as *pre-composed music*? The city's density gradient, traffic rhythm, and spatial rhythm already encode something like musical form. Sonification simply *plays* what is already there.

**For accessibility**: Sonification could make urban spatial structure legible to blind or low-vision users in a way photography and visual maps cannot.

**For data visualization**: Sonification is data visualization—it answers the same question ("what story does the data tell?") through a different sensory channel. The project tests whether that channel reveals or obscures.

---

## Critical Framework

The project sits at the intersection of:
- **Sonification** (representing data as sound)
- **Geospatial analysis** (mapping urban morphology)
- **Sound design** (coherent listening experience)
- **Perception research** (what is audible? what is meaningful?)

It is *not* primarily a music composition project, though it produces sound. It is *not* primarily a data visualization project, though it encodes data. It is a **methodological experiment**: can we parse the city by ear?

---

## Repository Structure

```
dataset.pkl              Morphological data for pre-recorded routes
dataset.csv              Convenience CSV copy
dataDictionary.json      Per-column definitions
metadata.json            Provenance, routes, synthesis mapping
README.md                Full documentation (this file)
soundscape.html          Single-file interactive application
plots/
  spatial_distribution.png
  channel_distribution.png
  zone_strip.png
  correlation_matrix.png
  [etc.]
```

**Reproducing the sonification:** The HTML file is standalone. No server, no API key, no installation required. Open in any modern browser. (Draw Mode requires internet for OSRM + Overpass API; pre-recorded routes work offline.)

**Modifying the synthesis:** Mapping rules are editable in the JavaScript section of the HTML. Change oscillator types, frequency ranges, pan positions, reverb parameters — and re-listen to the result.

**Adding new routes:** Provide 31 frames of (lat, lon, distance, building_ratio, sky_ratio, vegetation_ratio, building_left, building_right, person_count, vehicle_count) and the sonification will render immediately.

---

## Author & Context

**ılgın altun** · ITU MBL549E Special Topics in Architectural Design Computing, 2026.

A final project exploring the intersection of **spatial data, urban morphology, and sound design**. The question at the heart of the project is not "what does İstanbul sound like?" but rather "what can we learn about space *by* listening to it?"

---

## References & Inspiration

- Cage, Pauline (1969). *4'33"* — silence as composition; what we hear in the absence of intentional sound
- Ligeti, György (1962). *Atmosphères* — textural, morphological orchestration
- Cardew, Evan (1967). *The Great Learning* — graphic notation; the score as spatial object
- Wishart, Trevor (1996). *Audible Design* — sonification principles
- de Certeau, Michel (1984). *The Practice of Everyday Life* — the walk as text; spatial narrative
- Attali, Jacques (1985). *Noise: The Political Economy of Music* — sound as a mirror of social order
