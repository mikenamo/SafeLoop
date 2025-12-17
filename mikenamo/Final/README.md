# SafeLoop — USC (University Park) 

SafeLoop is a routing system that generates safer loops for running routes around the University of Southern California (University Park). It combines street network data with crime, collision, and contextual risk information to produce time-aware routes that balance safety and efficiency.

---

## Project Overview

SafeLoop:

- Builds a safety score for each street segment using nearby crime and collision data  
- Adjusts safety scoring based on time of day and season (daylight vs. darkness)  
- Uses time-aware safety scores directly during route generation  
- Looks ahead across neighboring streets to anticipate nearby risk  
- Generates multiple looped route options for a target distance  
- Visualizes routes with clear safety tradeoffs:
  - **Green:** safest (likely slower)  
  - **Blue:** balanced  
  - **Red:** most efficient (least safe)

---

## How Safety Is Modeled

### Crime and Collision Risk

Safety scores are built from two components:

- **Crime risk:** crime incidents near street segments  
- **Collision risk:** traffic collisions near street segments  

Incidents are counted within a **50-meter buffer** around each street segment and normalized **per kilometer** to account for varying street lengths. Keeping crime and collision separate allows the model to capture different types of risk (personal safety vs. traffic exposure).

---

### Time of Day and Seasonality

Safety scoring explicitly incorporates time context.

The user specifies:
- Month of the year  
- Hour of the day  

A simple seasonal daylight approximation is applied:
- Winter months assume shorter daylight hours  
- Summer months assume longer daylight hours  

Weighting behavior:
- **After dark:** crime risk is weighted more heavily  
- **During daylight:** crime and collision risk are weighted more evenly  

As a result, the same street can receive different safety scores depending on when the run is assumed to occur, and those time-aware scores are used directly during route generation.

---

### Looking Ahead a Couple Streets

SafeLoop does not evaluate streets independently.

To anticipate nearby risk, the model:
- Converts safety scores into risk values  
- Averages risk across neighboring street segments that share nodes  
- Combines each segment’s own risk with the risk of adjacent streets  

This allows routes to avoid clusters of risky streets rather than reacting only to isolated segments.

---

## Outputs

### Scored Street Outputs

- `usc_edges_scored.geojson`  
- `usc_edges_scored.csv`  

Generated once per safety scoring run. These files reflect the selected **month**, **hour**, and **weighting scheme** used for time-aware safety scoring.

---

### Route Outputs

- `safeloops_Xmi.geojson`  

Generated once per requested route distance. Different distances (e.g., 3.0mi, 4.0mi, 5.0mi) each produce their own file.  
Multiple files indicate multiple distances explored, not duplication errors.

---

## Notebooks

### 01_build_safety_scores.ipynb

Builds time-aware safety scores for street segments.

**Main steps:**
- Count crimes and collisions within 50 meters of each street  
- Normalize risk per kilometer  
- Apply time-of-day and seasonal weighting  
- Produce a composite safety score scaled from 0–100  
- Save scoring context (month, hour, weights) for transparency  

**Outputs:**
- `outputs/usc_edges_scored.geojson`  
- `outputs/usc_edges_scored.csv`  

---

### 02_generate_safeloop.ipynb

Generates SafeLoop routes using scored street segments.

**Main steps:**
- Filter streets to the USC DPS patrol zone  
- Convert safety scores into risk values  
- Smooth risk across neighboring streets (look ahead a couple streets)  
- Build a routing graph balancing distance, risk, and road type  
- Generate multiple loop options using different safety–efficiency tradeoffs (`alpha`)  

**Outputs:**
- `outputs/safeloops_Xmi.geojson`  

---

### 03_safeloop_map_builder.ipynb (Google Colab)

Visualizes SafeLoop routes interactively.

**Main steps:**
- Load SafeLoop route GeoJSON files  
- Clip routes to the USC DPS patrol zone  
- Display routes with consistent color semantics  
- Add offset start and end markers to avoid overlap  
- Show hover tooltips with distance, safety score, and alpha value  
- Export a shareable HTML map  

**Output:**
- `safeloop_map.html`  

---

## How to Run

1. Run `01_build_safety_scores.ipynb` in Databricks or locally  
2. Run `02_generate_safeloop.ipynb` to generate routes for desired distances  
3. Open `03_safeloop_map_builder.ipynb` in Google Colab  
4. Upload:
   - A `safeloops_Xmi.geojson` file from `outputs/`  
   - `usc_dps_upc_zone.geojson` from `data/`  
5. Run the notebook to display and save the interactive map  

---

## Notes and Limitations

- Longer distances may produce fewer viable loop options.  
- Daylight conditions are estimated by season, not real-time data 
- Crime and collision data are based on past records and location averages  
- SafeLoop is intended as a decision-support tool, not a safety guarantee

---

## Folder Structure

```text
SafeLoop/
├── notebooks/
│   ├── 01_build_safety_scores.ipynb
│   ├── 02_generate_safeloop.ipynb
│   └── 03_safeloop_map_builder.ipynb
│
├── data/
│   ├── usc_street_edges.geojson
│   ├── usc_crime_points.csv
│   ├── usc_collision_points.csv
│   └── usc_dps_upc_zone.geojson
│
├── outputs/
│   ├── usc_edges_scored.geojson
│   ├── usc_edges_scored.csv
│   ├── safeloops_3.0mi.geojson
│   ├── safeloops_4.0mi.geojson
│   ├── safeloops_5.0mi.geojson
│   └── ...
