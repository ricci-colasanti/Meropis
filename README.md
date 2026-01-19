# Meropis an Artificial Population Generator for Geospatial Simulation
## **Overview**
Meropis is an artificial population generator for geospatial simulation. The population is of the fictional island mentioned by ancient Greek writer Theopompus of Chios. 
These sets of notebooks implements an **artificial population generator** that create mythical but deomgrapicaly realistic **adult demographic profiles** (16+ years) for geospatial analysis and policy simulation. The system simulates diverse adult populations across different geographic area types, with detailed household structures and individual socioeconomic attributes.

**Project Context:** This artificial population framework is specifically designed to generate:
- A synthetic adult population (individuals 16+ and their households)
- A census summary of those households  
- Survey samples from the population
- Mythical but deomgrapicaly realistic geospatial distributions of the island of Meropis

## **Purpose**
Generate statistically realistic **adult-only populations** with:
- **Demographic diversity** across 9 socioeconomic classes
- **Geographic variation** based on 8 urban/rural area types  
- **Household composition** reflecting adult living arrangements
- **Compatible relationships** based on multi-dimensional similarity metrics
- **Mythical but deomgrapicaly realistic** geographic distributions of the island of Meropis


### Voronoi‑Based map generator
### Artificial population generator

## ** Voronoi‑Based map generator**
Jupyter notebook `Voronoi-json.ipynb`.   

This notebook creates a synthetic spatial dataset that mimics administrative regions (e.g., local‐authority zones) built from clustered points. It:

Generates a set of points grouped into a configurable number of clusters.
Computes the Voronoi diagram for those points.
Filters the resulting Voronoi cells, keeping only those whose entire polygon lies inside a predefined circular boundary.
Classifies each retained cell (urban, market‑town, agricultural) based on its proximity to cluster centroids and its area relative to the overall average.
Exports the filtered cells as a GeoJSON FeatureCollection, ready for GIS tools or further analysis.
Visualises the selected polygons with Matplotlib.

The workflow is useful for prototyping spatial analyses, generating test data for geoprocessing pipelines, or teaching concepts such as clustering, Voronoi tessellation, and spatial classification.

### 2. Key Concepts
ConceptWhat the script doesClustered point generationRandomly places n_clusters centroids within a user‑defined sub‑range of the unit square, then scatters n_points around each centroid using a Gaussian offset (cluster_spread).Voronoi tessellationUses scipy.spatial.Voronoi to partition the plane into convex polygons where each polygon contains all locations closer to its seed point than to any other seed.Circular boundary filterChecks every vertex of a Voronoi cell against a circle (center = (0.5, 0.5), radius ≈ 0.6). Only cells whose all vertices lie inside the circle are retained.Area calculationComputes polygon area via the shoelace formula (`0.5 *Administrative area assignmentAdds three extra “admin” points (simulating higher‑level boundaries) and assigns each retained cell to the nearest admin point using Euclidean distance.Urban / rural classification• If the cell’s seed point lies within 0.1 units of a cluster centroid → city.  • Else, compare cell area to half the mean area:   – smaller → market_town (treated as urban).   – larger → agricultural.Population attributeDraws a synthetic population value from a normal distribution (mean = 500, σ = 20).GeoJSON outputPacks each polygon (closed by repeating the first vertex) together with its properties (id, admin_area, vertices, area, urban, type, population) into a standard GeoJSON FeatureCollection saved as bounded_polygons.geojson.VisualizationPlots the retained polygons with semi‑transparent blue fill and black edges, labeling the figure “Voronoi of Clustered Points of LSOA in Meropis”.

### 3. Dependencies
LibraryVersion range (tested)Rolenumpy≥ 1.20Numerical arrays, random sampling, vectorised math.scipy (specifically spatial)≥ 1.6Voronoi construction and Euclidean distance.matplotlib≥ 3.3Rendering the final plot.json (standard library)—Serialising the GeoJSON output.random (standard library)—Seed handling and population simulation.
Install the required packages with:
pip install numpy scipy matplotlib

### 4. How to Run


Adjust parameters (optional) at the top of the script:

 - NUMBER_OF_LOCAL_AREAS = 300
 - NUMBER_OF_CITIES = 2
 - NUMBER_OF_ADMIN_AREAS = 3



#### Outputs:

Console prints the random seed used, the count of polygons kept, and confirmation of the saved GeoJSON file.
An interactive Matplotlib window displays the filtered Voronoi diagram.
bounded_polygons.geojson appears in the working directory.




### 5. Output Structure (GeoJSON)
OUTPUT_FILE = "bounded_polygons.geojson"   
Each feature follows this schema:
```json
{
  "type": "Feature",
  "geometry": {
    "type": "Polygon",
    "coordinates": [[[x0, y0], [x1, y1], ..., [x0, y0]]]
  },
  "properties": {
    "id": <int>,               // sequential identifier
    "admin_area": <int>,       // index of nearest admin point
    "vertices": <int>,         // number of polygon vertices
    "area": <float>,           // computed area (unit‑square units)
    "urban": <bool>,           // true for city/market_town
    "type": "<city|market_town|agricultural>",
    "population": <int>        // synthetic population estimate
  }
}
```

The file conforms to the GeoJSON specification and can be loaded into GIS software (QGIS, ArcGIS) or visualised with web mapping libraries (Leaflet, Mapbox GL).

### 6. Limitations & Caveats

Boundary filtering discards any Voronoi cell intersecting the circle, which may bias the retained sample toward centrally located clusters.
Area‑based classification uses a simple heuristic (half the mean area); real datasets often require more nuanced metrics.
Population model is a naïve Gaussian draw; replace with a demographic model if realism is required.
The script assumes a unit square domain; scaling to other extents requires consistent transformation of all geometric entities.


## Artificial population generator
Jupyter notebook `ArtificialPopulation_dev.ipynb`

### 1 Purpose
The notebook builds a synthetic population model for a set of geographic areas (cities, agricultural zones, market towns). It generates individual‑level demographic records, assembles them into realistic households, and then derives aggregate census‑style statistics and a small survey sample. The workflow is fully reproducible and driven by configurable probability tables.

### 2 Core Data Structures
| Structure | What it Holds | Key Fields |
| :--- | :--- | :--- |
| **demographic_profiles** | Parameter sets for nine socioeconomic groups (e.g., `affluent_professional`, `student`, `retired`). | Means/std‑dev for age, education, income; race distribution; family‑formation propensity; household‑type probabilities; typical children. |
| **area_types** | Composition of each geographic typology (`city`, `agricultural`, `market_town`). | Demographic mix, household‑type mix, urban flag, density, typical household size. |
| **household_types** | Blueprint of six household categories (`single`, `couple`, `nuclear family`, `single‑parent`, `shared`, `multi‑generational`). | Human‑readable description + size‑distribution (probability of 1‑6 occupants). |
| **Generated Individuals** | One dict per person. | `id`, `area`, `admin_area`, `demographic_id`, `age`, `education`, `gender`, `ethnicity`, `family_ratio`, `income`, plus household tracking fields (`in_household`, `household_id`, `relationship`). |
| **Households** | One dict per household. | `hh_id`, `area_id`, `admin_area`, `urban`, `members` (list of person dicts), derived `size`. |

### 3 Generation Pipeline

#### **Base Population (`generate_individuals`)**
- Uses a roulette‑wheel (cumulative probability) to draw a demographic profile for each person according to the area’s `demographic_distribution`.
- Draws continuous attributes from normal distributions (`age`, `education`, `income`) and categorical attributes (`gender`, `ethnicity`) from the profile’s defined probabilities.

#### **Household Formation (`create_area_population`)**
- Repeatedly selects a head from the remaining pool.
- Blends the head’s personal household‑type probabilities with the area‑level household distribution via `combine_probabilities` (simple Bayesian multiplication).
- Chooses a household type and size using roulette‑wheel sampling on the resulting distributions.
- Depending on the type, calls one of three helper routines:
  - `couple` – finds a compatible partner (heterosexual bias, similarity scoring).
  - `shared_household` – fills a non‑romantic house with similar adults.
  - `multi_generational` – adds a partner then older relatives.
- Updates each person’s `in_household`, `relationship`, and `household_id`.

#### **Census Summary (`calculate_census`)**
Walks every household to compute:
- Total population, urban vs. rural household counts.
- Household‑size histogram (1, 2, 3, 4+).
- Age‑by‑gender buckets (0‑30, 30‑60, >60).
- Education tier counts (non‑educated, low, middle, high).

#### **Survey Sample**
- Randomly selects **1 %** of households (`survay`).
- For each sampled household, `calculate_survey` produces the same set of aggregates as the full census, while also preserving the raw member list.

#### **Export**
Helper `dicts_to_csv` writes three CSV files:
1.  `artifical_cencus.csv` – census aggregates per area.
2.  `artifical_population.csv` – flat list of all generated individuals.
3.  `artifical_survay.csv` – survey‑level aggregates.
4.  `compleate_artifical_individual_survey.csv` – raw individuals from the survey sample.

### 4 Key Algorithms
| Algorithm | Role | Highlights |
| :--- | :--- | :--- |
| **Roulette‑Wheel Sampling** | Random selection proportional to defined probabilities (demographics, household types, size). | Simple cumulative‑sum approach; deterministic once the RNG seed (`np.random.default_rng(seed=42)`) is set. |
| **Similarity Scoring (`calculate_similarity_score`)** | Determines partner compatibility for `couple`‑type households. | Weighted combination of age, education, income (log‑scaled), ethnicity, and demographic profile similarity. Includes a **90 % hetero bias**. |
| **Non‑Gender Similarity (`calculate_similarity_score_non_gender`)** | Used for `shared_household` matching where gender is irrelevant. | Same weighted factors but without the gender filter. |
| **Bayesian Blending (`combine_probabilities`)** | Merges individual‑level and area‑level household probabilities. | Multiplication followed by renormalisation; falls back to uniform distribution if all products are zero. |

### 5 Execution Flow (Top‑Level Script)
```python
with open("bounded_polygons.geojson") as f:
    geojson_obj = json.load(f)

for each feature in geojson_obj["features"]:
    # 1. Resolve area type & urban flag
    # 2. Generate individuals & households via create_area_population(...)
    # 3. Derive census stats with calculate_census(...)
    # 4. Accumulate global lists (population, households, census)

### Export CSVs
dicts_to_csv(census, "artifical_cencus.csv")
dicts_to_csv(population, "artifical_population.csv")
```


### 6 Limitations & Assumptions

- **Static Distributions** – All probabilities are hard‑coded; real‑world calibration would require external data sources.
- **Simplified Ethnicity Handling** – Uses cumulative probabilities but stores ethnicity as an integer index only.
- **Gender Bias** – A 90 % heterosexual assumption is baked into `calculate_similarity_score`; this can be toggled if needed.
- **No Spatial Interaction** – Households are formed purely from the pool of individuals within an area; cross‑area commuting or migration is not modeled.
- **Age/Education Bounds** – Generated values are not explicitly clamped beyond the normal distribution; extreme outliers may appear.