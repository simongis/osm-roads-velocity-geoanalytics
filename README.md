# OSM Roads for ArcGIS Velocity and GeoAnalytics Engine

A Python notebook that extracts OpenStreetMap road networks from Geofabrik PBF files and generates the schema required for both **ArcGIS Velocity's Snap to Network** and **GeoAnalytics Engine's Snap Tracks** tools.

## Overview

When working with real-time GPS streams or historical vehicle tracks, you often need to snap points to a road network. ArcGIS Velocity and GeoAnalytics Engine both provide powerful snapping tools, but they require different schema fields. This notebook creates a single feature class that supports both tools.

**Schema Generated:**

| Tool | Required Fields | Purpose |
|------|----------------|---------|
| **Velocity Snap to Network** | `OBJECTID`, `F_AUTOMOBILES`, `T_AUTOMOBILES` | Controls directional traversal |
| **GeoAnalytics Snap Tracks** | `from_node`, `to_node`, `direction` | Defines network connectivity |

## Features

- ✅ Reads directly from Geofabrik PBF files (fast, works offline)
- ✅ Supports any Australian city via bounding box configuration
- ✅ Generates direction fields for Velocity (F_AUTOMOBILES, T_AUTOMOBILES)
- ✅ Generates connectivity fields for GeoAnalytics (from_node, to_node, direction)
- ✅ Filters to drivable roads only (configurable highway types)
- ✅ Excludes driveways automatically
- ✅ Outputs to File Geodatabase ready for Portal publishing
- ✅ Full validation checks

## Requirements

- ArcGIS Pro 3.x
- Python 3.11+
- Libraries:
  - `pyogrio` 0.12+
  - `geopandas` 1.1+
  - `pandas` 2.x
  - `arcpy` (included with ArcGIS Pro)

## Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/simongis/osm-roads-for-velocity-geoanalytics.git
   cd osm-roads-for-velocity-geoanalytics
   ```

2. Install required libraries in ArcGIS Pro's Python environment:
   ```bash
   conda activate arcgispro-py3
   conda install -c conda-forge pyogrio geopandas
   ```

3. Download the Australia PBF from Geofabrik:
   - URL: https://download.geofabrik.de/australia-oceania/australia-latest.osm.pbf
   - Size: ~886 MB (as of March 2024)
   - Update the `INPUT_PBF` path in Step 1 of the notebook

## Usage

### Quick Start

1. Open `OSM_Roads_Clean_v1.ipynb` in ArcGIS Pro
2. Configure Step 1:
   ```python
   # Choose your bounding box
   BBOX = (152.6, -27.7, 153.3, -27.2)  # Brisbane metro
   
   # Set your PBF path
   INPUT_PBF = r"C:\Data\australia-latest.osm.pbf"
   
   # Set output location
   WORK_DIR = r"C:\GIS\Projects"
   OUTPUT_NAME = "brisbane_roads"
   ```
3. Run all cells
4. Output feature class: `{WORK_DIR}\BrisbaneRoads.gdb\BrisbaneRoadsNetwork`

### Pre-configured Bounding Boxes

Uncomment the city you need in Step 1:

```python
# BBOX = (153.2, -28.3, 153.7, -27.8)  # Gold Coast
# BBOX = (152.6, -27.7, 153.3, -27.2)  # Brisbane metro
# BBOX = (151.1, -33.0, 151.3, -32.8)  # Newcastle
# BBOX = (152.4, -28.3, 153.7, -26.5)  # SEQ region
# BBOX = (115.7, -32.1, 116.0, -31.8)  # Perth metro
# BBOX = (144.8, -37.9, 145.1, -37.7)  # Melbourne metro
# BBOX = (151.1, -34.0, 151.3, -33.8)  # Sydney metro
# BBOX = (138.5, -35.0, 138.7, -34.8)  # Adelaide metro
```

## Notebook Steps

1. **Configuration** - Set bounding box, PBF path, output location
2. **Environment Check** - Validate Python libraries and PBF file
3. **Extract Roads** - Read from PBF using pyogrio with highway filtering
4. **Velocity Direction Fields** - Generate F_AUTOMOBILES, T_AUTOMOBILES from OSM oneway tags
5. **GeoAnalytics Connectivity** - Generate from_node, to_node, direction from endpoints
6. **Prepare Output Schema** - Select and clean output columns
7. **Write to File Geodatabase** - Export via temporary GeoPackage
8. **Validation** - Verify all required fields present

## Publishing to Portal

Once the feature class is created:

1. Add to ArcGIS Pro map
2. **Share As Web Layer** → Feature (hosted)
3. Enable **Feature Access**
4. Publish to ArcGIS Enterprise or ArcGIS Online

The published feature service can be consumed by:
- **ArcGIS Velocity** via Snap to Network
- **ArcGIS GeoAnalytics Engine** via Snap Tracks  
- **GeoAnalytics Fabric** (same as Engine, Microsoft Fabric deployment)

## Schema Details

### Velocity Snap to Network

| Field | Type | Values | Meaning |
|-------|------|--------|---------|
| `OBJECTID` | Integer | Auto-generated | Unique ID (required) |
| `F_AUTOMOBILES` | String | Y/N | From-direction closed |
| `T_AUTOMOBILES` | String | Y/N | To-direction closed |


### GeoAnalytics Snap Tracks

| Field | Type | Values | Meaning |
|-------|------|--------|---------|
| `from_node` | Integer | Node ID | Start node |
| `to_node` | Integer | Node ID | End node |
| `direction` | String | FT/TF/B/N | Direction allowed |

**Direction values:**
- `FT` → Forward (from_node → to_node)
- `TF` → Backward (to_node → from_node)
- `B` → Both directions
- `N` → No direction (closed)


## Acknowledgements

- **OpenStreetMap** contributors for community-contributed road data
- **Geofabrik** for reliable PBF extracts

