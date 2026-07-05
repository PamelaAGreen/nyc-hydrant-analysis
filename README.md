# NYC Hydrant Density Analysis

This repo compares two ways to answer the same spatial question for New York City fire hydrants and Neighborhood Tabulation Areas (NTAs): a PostGIS/SQL workflow and a GDAL/GeoPandas workflow.

This code was built as a project under the [Modern GIS Accelerator](https://forrest.nyc/go/accelerator/) and can be updated and adopted for pipelines with similar data.

## Project questions

* Where are hydrants most dense across NYC neighborhoods?
* Which neighborhoods are closest to the median hydrant density?
* Which neighborhoods have the lowest share of area within 100 meters of a hydrant?

## Repository Contents

* nyc_hydrant_analysis_sql-2.ipynb - SQL/PostGIS version using ST_Contains, ST_Area, ST_Buffer, ST_Union, and ST_Intersection to summarize hydrant counts, density, median density, and 100 m coverage.
* nyc_hydrant_analysis_gdal.ipynb - GDAL/GeoPandas version using GeoJSON inputs, spatial joins, buffering, dissolve, overlay, choropleths, and GeoParquet export.
* README.md - project overview and run notes.
* LICENSE - open source license for use

## The data

- **NYC Neighborhoods:** 262 polygons (Source: NYC Open Data)
- **NYC Fire Hydrants:** 109,725 points (Source: NYC Open Data)
- License: NYC Open Data Terms of Use
- All data in EPSG:4326

## Methodology

Built the same analysis twice:

- **SQL (PostGIS):** four progressive queries in `nyc_hydrant_analysis_sql.ipynb`, going
  from simple filter to spatial join to area-normalized density to
  100m-buffer coverage analysis.
- **Python (GeoPandas):** equivalent pipeline in `analnyc_hydrant_analysis_gdal.ipynb`,
  with a static choropleth and interactive `.explore()` map. Final
  output exported to GeoParquet.

Both pipelines produce the same calculations values to within rounding and follow this workflow:

* Count hydrants in each neighborhood.
* Calculate area in square kilometers and hydrants per square kilometer.
* Find the neighborhood closest to the median hydrant density.
* Build a dissolved 100 m hydrant buffer and measure percent neighborhood coverage.
* Create choropleth maps and export final outputs to GeoParquet

## Findings

* The top SQL density results include Gramercy, SoHo-Little Italy-Hudson Square, Tribeca-Civic Center, West Village, and Financial District-Battery Park City.

* The lowest density neighborhoods include New Springville-Willowbrook-Bulls Head-Travis, Tottenville-Charleston, Co-op City, Todt Hill-Emerson Hill-Lighthouse Hill-Manor Heights, and Mariners Harbor-Arlington-Graniteville.

* The median-density neighborhood is Glendale, Queens, at about 185.8 to 185.9 hydrants per square kilometer depending on rounding between workflows.

* The lowest 100 m coverage results are concentrated in Staten Island, with New Springville-Willowbrook-Bulls Head-Travis at about 48.4 to 48.5 percent covered.

![Hydrant density map](images/hydrant-density.png)

## How to run it

1. Load the NYC neighborhoods and hydrants data into PostGIS for the SQL notebook, or keep the GeoJSON files available for the GDAL notebook.

2. Update database connection settings in the SQL notebook (#2 Open PostGIS database).

3. Run the SQL notebook to reproduce the PostGIS workflow and maps.

4. Run the GDAL notebook to reproduce the GeoPandas workflow, maps, and GeoParquet export.

You can run PostGIS in Docker if access to local PostGIS is not available.

\`\`\`bash
git clone https://github.com/{your-username}/nyc-hydrant-analysis.git
cd nyc-hydrant-analysis

If using Docker then:
# Start the PostGIS
docker compose -f docker/postgis/docker-compose.yml up -d

# Load the data, run the SQL pipeline, then the GDAL pipeline
make load
jupyter lab nyc_hydrant_analysis_sql.ipynb
jupyter lab nyc_hydrant_analysis_gdal.ipynb
\`\`\`

## What I learned

Implementing equivalent spatial analysis pipelines in PostGIS/SQL and GeoPandas/GDAL, then validating that both stacks produce matching results.

Taking raw GeoJSON through a full point‑in‑polygon workflow: spatial joins, aggregation, density calculations, and hydrant coverage metrics.

Selecting and applying appropriate projections and buffers for area and distance questions, and understanding how CRS choices impact density and 100 m coverage.

Defining a robust median‑density neighborhood concept and making “closest to the median” behave consistently in both SQL (percentiles/window functions) and pandas, including resolving rounding and area calculation differences.

How to turn analysis into usable outputs: choropleth maps, clear printed summaries, and a GeoParquet table that can plug into downstream tools.

## Stack

- Python, Jupyter, GeoPandas, pandas, matplotlib, psycopg.
- PostGIS / PostgreSQL for the SQL workflow.
- GeoParquet for final output.
