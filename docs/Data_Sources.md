# Data Sources and Preparation

This document records the source datasets and preparation methods for the **DC Park Accessibility Planner**. The analysis evaluates park proximity in Washington, DC, using an 800-meter straight-line buffer and ranks populated underserved census block groups for potential park investment.

## Source datasets

| Dataset | Provider | Source / identifier | Use in project |
|---|---|---|---|
| Parks and Recreation Areas | District of Columbia GIS (DC GIS) | [Recreation_WebMercator MapServer, layer 9](https://maps2.dcgis.dc.gov/dcgis/rest/services/DCGIS_DATA/Recreation_WebMercator/MapServer/9) | Park polygons and accessibility buffers |
| District of Columbia boundary | U.S. Census Bureau, TIGERweb | [State_County MapServer, layer 1](https://tigerweb.geo.census.gov/arcgis/rest/services/TIGERweb/State_County/MapServer/1) | Study-area boundary |
| Census block groups | U.S. Census Bureau, TIGERweb | [Tracts_Blocks MapServer, layer 1](https://tigerweb.geo.census.gov/arcgis/rest/services/TIGERweb/Tracts_Blocks/MapServer/1) | Neighborhood-scale spatial units |
| Total population | U.S. Census Bureau, American Community Survey | **2024 ACS 5-Year, B01003 (Total Population)** | Population counts and density calculations |

The repository includes these ACS files in `data/`:

- `ACSDT5Y2024.B01003-Data.csv`
- `ACSDT5Y2024.B01003-Column-Metadata.csv`
- `ACSDT5Y2024.B01003-Table-Notes.txt`

## Data preparation

1. **Parks:** Added the DC GIS parks web layer and exported a local working copy (`DC_Parks_Raw`). The original source contained **252** records. The analysis ultimately used **147** park features after project-specific filtering and preparation; consult the ArcGIS workflow and notebook for the exact implemented criteria.
2. **DC boundary:** Selected the District of Columbia county-equivalent from TIGERweb using state FIPS **11** and exported a single polygon. Projected the working boundary to **EPSG:26985**.
3. **Block groups:** Selected DC block groups from TIGERweb using state FIPS **11** and exported **571** block groups for local analysis.
4. **Population:** Used ACS B01003 total-population data, matched to census geography using geographic identifiers, and calculated population density for priority scoring.
5. **Coordinate system:** Performed local spatial analysis in **NAD 1983 StatePlane Maryland FIPS 1900 (Meters), EPSG:26985**.

## Accessibility and prioritization

- Accessibility threshold: **800-meter straight-line buffer** around eligible park polygons, dissolved into a coverage area. This is a proximity measure, **not** a pedestrian-network or travel-time service area.
- Underserved block groups: **6** identified as outside the coverage area under the project's selection rule.
- Total population in underserved block groups: **4,237**.
- Populated underserved block groups ranked: **4**.
- Priority score weights: **60% population density** and **40% nearest-park distance**.

| Priority rank | Block-group GEOID | Priority score |
|---:|---|---:|
| 1 | `110010082002` | 87.70 |
| 2 | `110010022023` | 65.12 |
| 3 | `110010095102` | 62.53 |
| 4 | `110010022021` | 57.58 |

The published ArcPy notebook reports that the ArcPy and ModelBuilder priority results match for all four ranked locations.

## Reproducibility notes and limitations

- **Local geodatabases are not committed to GitHub.** Reproducing the full GIS workflow requires acquiring the source layers, creating the local feature classes, and configuring paths in ArcGIS Pro / ArcPy.
- **Service contents can change.** The live DC GIS and TIGERweb services may differ from the versions accessed during project development. Exact spatial-service extraction dates and TIGERweb boundary vintage have not been independently established here.
- **ACS and geography vintage may differ.** The population table is explicitly the **2024 ACS 5-Year** B01003 table; the exact vintage of the TIGERweb block-group boundaries should be checked before claiming strict vintage alignment.
- **Proximity is not walkability.** The 800-meter buffer does not account for streets, barriers, entrances, or actual pedestrian routes.
- **Block-group population is aggregated.** The analysis should not be interpreted as a parcel-level count of residents without park access.

For implementation details, see [`arcpy/DC_Park_Accessibility_Analysis.ipynb`](../arcpy/DC_Park_Accessibility_Analysis.ipynb) and the project README.
