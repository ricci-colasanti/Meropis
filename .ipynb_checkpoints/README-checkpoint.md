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

## **Key Features**

### **1.  Voronoi‑Based map generator**
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

NUMBER_OF_POINTS: total number of seed points.
create_clustered_points(... ) arguments: number of clusters, spread, and allowed centroid range.
midpoint and radius: define the circular clipping region.




#### Outputs:

Console prints the random seed used, the count of polygons kept, and confirmation of the saved GeoJSON file.
An interactive Matplotlib window displays the filtered Voronoi diagram.
bounded_polygons.geojson appears in the working directory.




### 5. Output Structure (GeoJSON)
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


