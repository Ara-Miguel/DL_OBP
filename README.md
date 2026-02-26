# HP OBP Force Plate Dashboard Suite
### Driveline Open Biomechanics Project — Baseball Athlete Analysis
**1,162 Athletes · High School · College · Pro · 2021–2024**

---

## Data Source

This data is all built on the **Driveline Open Biomechanics Project (OBP)** — publicly available force plate data from baseball athletes across the development pipeline. We're working with 1,934 raw sessions across 1,162 unique athletes, collapsed down to one averaged record per athlete after cleaning.

A handful of artifacts were flagged and fixed before any analysis ran:
- Three RSI-Modified values were nulled out — they were physically impossible (>30 m/s)
- Stiffness and eccentric RFD got nulled for any session where countermovement depth was shallower than −20 cm, since a minimal dip artificially inflates the stiffness calculation. One known exception was left in intentionally.

---

## Data Quality Thresholds

On top of the initial cleaning, there's a second layer of quality logic baked into all four dashboards at render time. Before any chart or stat runs, values get checked against what's physiologically plausible — using population distributions, world-record context, and some known force plate (Hawkin Dynamics or VALD) software quirks as reference points.

Values either get hard-removed or flagged depending on how outstanding they are.

### Hard Removals

These get set to `NA` before anything else runs — not in any chart, not counted in any summary, gone.

| Metric | Threshold | Why | Athletes Affected |
|:------:|:---------:|:---:|:-----------------:|
| CMJ Jump Height | > 75 cm | NBA combine max is ~71 cm — nobody in a baseball sample is clearing 75+ legitimately | 0 |
| SJ Jump Height | > 65 cm | SJ is structurally lower than CMJ (no SSC) — anything above 65 is an artifact | 0 |
| HT Jump Height | > 55 cm | ATHLETE_0028 came in at 127.9 cm (z = 14.4) — every other HT metric was normal, so it's clearly a flight-time calculation error. P99 of the clean population is 44.3 cm | 1 |
| HT Active Stiffness (sentinel) | == 99,999.99 N/m | Force Plate outputs this exact value when stiffness can't be computed for a session — it's a software cap, not real data | 3 |
| HT Active Stiffness (artifact) | > 200,000 N/m | ATHLETE_0349 hit 888,107 N/m (z = 24.7) while their RSI and jump height were completely normal. Two more athletes in the same boat above 200k | 3 |

### Flagged Values

These stay in the plots but show up as amber diamonds with a hover note — "High — verify data." They're still included in summary stats. The flag gives you a heads-up to pull the raw session file before drawing conclusions on that athlete.

| Metric | Flag Threshold | Why | Athletes Flagged |
|:------:|:-------------:|:---:|:---------------:|
| CMJ Jump Height | > 60 cm | Above P99.5 of the full population (59.1 cm) — possible, but worth a second look | 2 |
| SJ Jump Height | > 55 cm | Above P99 of the SJ population (50.5 cm) | 2 |
| HT Jump Height | > 45 cm | Above P99 of the clean HT population (44.3 cm) | 7 |
| HT Active Stiffness | > 100,000 N/m | Above the 90th percentile of plausible values — not impossible, but flag it | 15 |

### Noteworthy Specific Artifacts

| Athlete | Test | Metric | Raw Value | Status | Notes |
|:-------:|:----:|:------:|:---------:|:------:|:-----:|
| ATHLETE_0028 | HT | Jump Height | 127.9 cm | Removed | RSI and contact time were fine — isolated flight-time error |
| ATHLETE_0349 | HT | Active Stiffness | 888,107 N/m | Removed | 24.7 SDs out; RSI and jump height were totally normal |
| ATHLETE_0523 | HT | Active Stiffness | 235,336 N/m | Removed | Same pattern |
| ATHLETE_1079 | HT | Active Stiffness | 340,130 N/m | Removed | Same pattern |
| ATHLETE_0434 | HT | Active Stiffness | 99,999.99 N/m | Removed | HD sentinel value |
| ATHLETE_0505 | HT | Active Stiffness | 99,999.99 N/m | Removed | HD sentinel value |
| ATHLETE_0507 | HT | Active Stiffness | 99,999.99 N/m | Removed | HD sentinel value |

---

## Test Definitions

### CMJ — Countermovement Jump

Standard bilateral CMJ — athlete dips down and jumps as high as possible. The countermovement loads the stretch-shortening cycle, which is why CMJ is almost always higher than SJ. It's the primary test in this dataset and probably the most common force plate assessment you'll see in applied sport science.

**Key metrics:**

**Jump Height** — from impulse-momentum; main output

**RSI-Modified** — jump height divided by total time on plate; captures efficiency, not just height

**Peak Power / BM** — peak concentric power relative to body mass; best predictor of jump height in this dataset (r = 0.956)

**Eccentric RFD** — how fast force develops during the loading/braking phase

**Lower-Limb Stiffness** — spring-like behavior of the lower limb during the dip

**Concentric Peak Force** — max force during the upward push

**Countermovement Depth** — depth of the dip; shallow dips inflate stiffness, so keep an eye on this

**Asymmetry metrics** — L/R differences in stiffness, eccentric deceleration impulse, and concentric impulse (Phase 1 and 2)

---

### SJ — Squat Jump

Same as CMJ but the athlete starts from a static hold at the bottom — no countermovement, no SSC contribution. The gap between CMJ and SJ (elasticity index) tells you how much an athlete is actually getting out of the stretch-shortening cycle.

**Key metrics:**

**Jump Height** — same derivation as CMJ; lower for most athletes

**Peak Power / BM** — power without the SSC assist

**Elasticity Index** — CMJ minus SJ jump height; positive = SSC benefit, zero or negative = SSC deficit

**Asymmetry metrics** — concentric impulse L/R differences in Phase 1 and Phase 2

**Interpretation:** On average, athletes in this dataset gain about 13% more height in CMJ vs SJ. If SJ equals or beats CMJ (elasticity index ≤ 0), that athlete has a true SSC deficit — reactive and plyometric work is probably suggested from exercise selection perspective.

---

### IMTP — Isometric Mid-Thigh Pull

No movement — athlete just pulls as hard and fast as possible against a fixed bar from mid-thigh. Pure isometric strength. Good for capturing maximal force production and early RFD without the movement variability of a jump.

**Key metrics:**

**Peak Vertical Force** — highest force during the pull; primary strength number

**Net Peak Vertical Force** — peak minus body weight; the force added beyond their own mass

**Force at 100 / 150 / 200ms** — force built at specific time points after pull initiation; reflects neural drive and early RFD

**% of Peak at each window** — what fraction of their max they've expressed at 100–200ms

**Interpretation:** Side note: all three levels (HS/college/pro) hit nearly identical percentages of their peak at each time window (~40% at 100ms, ~59% at 200ms). The difference between HS, College, and Pro is the absolute force ceiling, not how fast they climb to it relative to their max. Pro athletes have a bigger ceiling, but everyone reaches their peak at the same relative rate early on.

---

### HT — Hop Test

Repeated bilateral hopping on the force plates — athlete bounces continuously and we pull their best rep. More reactive and elastic than a single jump, and arguably more sport-relevant for a lot of baseball movements. The stiffness metric here is task-specific, which is why it doesn't correlate well with CMJ stiffness (r = 0.126) even though RSI does (r = 0.670).

**Key metrics:**
**RSI (Jump Height / Contact Time)** — best single reactive strength number; tracks well with CMJ RSI

**Best Jump Height (Flight Time)** — highest hop height recorded

**Active Stiffness** — spring stiffness across repeated contacts; reflects tendon and elastic lower-limb properties

**Interpretation:** HT RSI and CMJ RSI move together, which tells you reactive ability is a fairly stable quality across tasks. Stiffness doesn't transfer that way — it's specific to the movement pattern. Don't expect HT stiffness and CMJ stiffness to tell the same story.

---

## Inside the Dashboards

### Dashboard 01 — Level Comparison (`01_level_comparison.html`)

Straight comparison of CMJ and IMTP metrics across High School, College, and Pro. Every chart includes significance testing across all three level pairings, with corrections applied for multiple comparisons — so you know which differences actually hold up statistically.

**Overview** — Box plots for Jump Height, RSI-Modified, Peak Power/BM, and Eccentric RFD by level. The headline: HS → College gap is significant across the board, but College → Pro narrows considerably and doesn't hold on some metrics.

**Force Production** — Concentric peak force, stiffness, depth, and duration. Peak force is the strongest level discriminator. Pro athletes complete the concentric phase fastest (~267ms vs ~294ms in HS).

**Percentile Norms** — Sortable, downloadable normative tables (P10/P25/P50/P75/P90) for all CMJ metrics by level. Quick reference for comparing an individual athlete against the population.

**IMTP — Force at Time Windows** — Bar charts and box plots for force at 100ms, 150ms, and 200ms by level. Includes the percentage-of-peak chart showing that all three levels express force at nearly the same relative rate — the difference is in how high the ceiling is, not how fast they get there.

---

### Dashboard 02 — Multi-Test Relationships (`02_multi_test.html`)

How do the different tests relate to each other, and what does that reveal about an athlete's physical profile?

**CMJ vs SJ** — Elasticity index distributions by level and a CMJ vs SJ scatter with a diagonal reference line (above the line = SSC benefit). College athletes show the largest average SSC gain (+5.0 cm). 40 athletes (3.7%) are at or below zero — true SSC deficit.

**CMJ vs IMTP** — IMTP peak force vs CMJ jump height, and time-window forces vs CMJ height. Moderate relationship overall. Later windows (200ms) predict CMJ better than early ones (100ms) — sustained force output matters more than first-contact speed.

**CMJ vs Hop Test** — RSI comparison (r = 0.670), jump height comparison, and the stiffness comparison (r = 0.126 — task-specific, as expected). Also includes IMTP net peak force vs SJ jump height.

**SSC Profiling** — Quadrant scatter of elasticity index vs peak power/BM. Four profiles: High EI/High PP (well-developed), Low EI/High PP (reactive training target — 210 athletes), High EI/Low PP (good SSC, limited power), Low EI/Low PP (general development — 333 athletes, majority HS). Profile counts by level included.

**Asymmetry** — Eccentric deceleration, stiffness, and SJ concentric impulse asymmetry by level with ±10% threshold lines. Eccentric deceleration is the most common issue — 39% of all athletes exceed ±10%. HS shows the highest asymmetry across every metric.

---

### Dashboard 03 — Physical Capacity Profile (`03_physical_capacity.html`)

Body composition, relative strength, and peak power as physical development markers across the pipeline.

**Overview** — Body weight and relative strength by level. HS: 155 lbs avg, College: 195 lbs, Pro: 208 lbs — 53 lbs from prep to pro. Relative strength (HS: 3.18, College: 3.60, Pro: 3.53) doesn't follow the same clean progression, which makes sense — body weight tends to develop faster than strength-to-weight ratio through the pipeline.

**Quadrant Analysis** — Relative strength vs peak power/BM, split by dataset medians. Athletes color-coded by quadrant, symbol by level. Quadrant count table by level.

**Key Correlations** — Four scatter plots: Peak Power/BM vs CMJ JH (r = 0.956 — strongest predictor in the dataset), Relative Strength vs IMTP Peak Force (r = 0.722), Body Weight vs CMJ JH (r = 0.502 — positive but partly a level confound), plus one more physical capacity comparison.

---

### Dashboard 04 — Longitudinal Subgroup (`04_longitudinal.html`)

The 176 athletes (out of 1,162) who registered in the original dataset with three or more sessions. Not large amounts of datapoints, but enough to see some meaningful trends.

**Subgroup:** 92 College (avg 3.78 sessions), 60 HS (avg 3.87), 24 Pro (avg 4.0). Max sessions: 10 (HS), 9 (College), 7 (Pro).

**Session Overview** — Session count histogram and days between first/last session by level.

**Trajectories** — Mean-by-session line plots with SE bars for Jump Height, RSI, Peak Power/BM, and Eccentric RFD. Only sessions with 5+ athletes per level are shown — keeps small-sample noise out of the group trends.

**First vs Last** — Box plots + individual data points showing change from first to last session. All three levels trend positive. HS shows the largest absolute gains (JH: +1.86 cm, RSI: +0.065 m/s), which tracks with them being earlier in development. Pro shows the smallest changes, starting from a higher floor.

**Change Summary Table** — Athlete-level table with change scores for all four metrics from first to last session. Green/red color-coded. Downloadable as CSV or Excel.

---

## Files

| File | What it is |
|:-----|:-----------|
| `hp_obp_combined.csv` | 1,162 athletes, session-averaged, identified |
| `hp_obp_anonymized.csv` | Same athletes just anonymized (hashed IDs, binned body weight, shifted dates) |
| `hp_obp_longitudinal.csv` | 676 sessions from the 176 athletes with 3+ tests |
| `hp_obp_changes.csv` | First-to-last change scores for the longitudinal subgroup |
| `hp_obp_bat_pitch.csv` | Bat and pitch speed data — set aside, not included in the dashboards, can be used later |
| `hp_obp_remap_PRIVATE.csv` | Maps original UIDs to assigned athlete IDs — **IDs are already anonymized, re-sharing to show further anonymization** |

---

## Running the Dashboards

You'll need R with these packages:
```r
install.packages(c("flexdashboard", "tidyverse", "DT", "plotly", "readxl"))
```

At the top of each Rmd setup chunk, update `PROCESSED_DIR` to wherever your CSV files live:
```r
PROCESSED_DIR <- "/path/to/your/DL-OBP"  # ← update this to your local path
```

Then render in order:
```r
rmarkdown::render("01_level_comparison.Rmd")
rmarkdown::render("02_multi_test.Rmd")
rmarkdown::render("03_physical_capacity.Rmd")
rmarkdown::render("04_longitudinal.Rmd")
```

Dashboard 04 also needs `hp_obp_longitudinal.csv` and `hp_obp_changes.csv` in the same directory.

---

*Dataset: Driveline Open Biomechanics Project — publicly available at openbiomechanics.baseball*
