
<div align="center">

```
                          N
                          ▲
                      NW  │  NE
                        ╲ │ ╱
                   W ◄────┼────► E
                        ╱ │ ╲
                      SW  │  SE
                          ▼
                          S

   0        250       500       750      1000 commits
   ├─────────┼─────────┼─────────┼─────────┤
```

# Tom Baker

### `arcpy.management.SelectLayerByLocation("humans", "INTERSECT", "maps")`

*GIS developer · cartography apologist · person who will ask "what's the CRS?" before saying hello*

![CRS](https://img.shields.io/badge/CRS-EPSG%3A4326-2ea44f?style=flat-square)
![Axis Order](https://img.shields.io/badge/axis%20order-it's%20complicated-orange?style=flat-square)
![Topology](https://img.shields.io/badge/topology-valid%20(mostly)-blue?style=flat-square)
![Schema Lock](https://img.shields.io/badge/schema%20lock-probably-yellow?style=flat-square)
![Errors](https://img.shields.io/badge/ERROR-999999-red?style=flat-square)
![Datum](https://img.shields.io/badge/datum-emotionally%20shifted-purple?style=flat-square)

</div>

---

## 📍 You Are Here

GitHub renders GeoJSON natively, so my bio is a Feature. Click the marker.

```geojson
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [-94.6, 39.0]
      },
      "properties": {
        "name": "Tom Baker",
        "role": "GIS Developer",
        "stack": "ArcGIS · Python · Swift · ColdFusion",
        "currently": "Reprojecting something that was already in the right projection",
        "favorite_projection": "Equal Earth (EPSG:8857)",
        "least_favorite_file_format": "You know the one. It has 4-7 files.",
        "coffee_units": "liters per hectare",
        "marker-color": "#2ea44f",
        "marker-symbol": "star"
      }
    }
  ]
}
```

<sub>39.0°N, 94.6°W. Not to be confused with <a href="https://en.wikipedia.org/wiki/Null_Island">Null Island</a> (0°N, 0°E), where all my ungeocoded records go to retire.</sub>

---

## 🧭 Metadata (ISO 19115-ish)

| Element | Value |
|---|---|
| **Title** | Tom Baker |
| **Abstract** | Turns coordinates into decisions and decisions into choropleths |
| **Spatial Reference** | WGS 84 at work, State Plane when the surveyors are watching |
| **Spatial Resolution** | Sub-meter before coffee, roughly county-level after 5 pm |
| **Temporal Extent** | `[first_shapefile_trauma, ∞)` |
| **Lineage** | Paper maps → ArcMap (RIP) → ArcGIS Pro → "wait, I can *script* this?" → ArcPy → now there's a Swift app too |
| **Positional Accuracy** | ± 1 datum shift |
| **Completeness** | Some features missing attributes; see `TODO` field (truncated to 10 chars) |
| **Use Constraints** | Not for navigation |

---

## 🛠️ The Stack (draw order matters)

![ArcGIS](https://img.shields.io/badge/ArcGIS-Pro%20·%20Online%20·%20Enterprise-2C7AC3?style=for-the-badge&logo=arcgis&logoColor=white)
![Python](https://img.shields.io/badge/Python-ArcPy%20·%20ArcGIS%20API-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Swift](https://img.shields.io/badge/Swift-ArcGIS%20Maps%20SDK%20·%20MapKit-F05138?style=for-the-badge&logo=swift&logoColor=white)
![ColdFusion](https://img.shields.io/badge/ColdFusion-yes%2C%20really-0B3D91?style=for-the-badge)

| Layer | Tool | What it does in my table of contents |
|---|---|---|
| 🗺️ **Basemap** | **ArcGIS** | Where the data lives, gets analyzed, and gets published. Geodatabases, feature services, and a Pro project with 14 maps named `Map`, `Map1`, `Map2`… |
| 🐍 **Automation** | **Python** | ArcPy and the ArcGIS API for Python. If I've clicked it twice in a geoprocessing pane, it's a script by Friday. |
| 📱 **Field** | **Swift** | Native iOS mapping apps. SwiftUI on top, ArcGIS Maps SDK underneath, CoreLocation quietly draining the battery. |
| 🧊 **Web tier** | **ColdFusion** | The load-bearing CFML that has been serving map data since before "web GIS" was a phrase. It has outlived three JavaScript frameworks and it will outlive yours. |

---

## 🐍 ArcPy: How I Find Lunch

```python
import arcpy

arcpy.env.overwriteOutput = True          # living dangerously

arcpy.management.MakeFeatureLayer("restaurants", "lunch_lyr",
                                  "cuisine <> 'sad desk salad'")

# GEODESIC = real meters on the ellipsoid, no projection gymnastics required
arcpy.management.SelectLayerByLocation(
    in_layer="lunch_lyr",
    overlap_type="WITHIN_A_DISTANCE_GEODESIC",
    select_features="my_desk",
    search_distance="800 Meters",          # walkable
    selection_type="NEW_SELECTION",
)

count = int(arcpy.management.GetCount("lunch_lyr")[0])
arcpy.AddMessage(f"{count} options. Going to the same place as yesterday.")
```

## ✅ Pre-flight Checklist

```python
import arcpy

fc = r"C:\GIS\definitely_clean_data.gdb\parcels"

sr = arcpy.Describe(fc).spatialReference
assert sr.name != "Unknown", "Narrator: it was Unknown."

arcpy.management.RepairGeometry(fc)       # self-intersecting polygons. Again.

# Never compute area in degrees. Square degrees are not a unit. They are a cry for help.
arcpy.management.CalculateGeometryAttributes(
    fc, [["area_km2", "AREA_GEODESIC"]], area_unit="SQUARE_KILOMETERS"
)
```

## 📱 Swift: A Map in 12 Lines

```swift
import SwiftUI
import ArcGIS

struct ContentView: View {
    @State private var map: Map = {
        let map = Map(basemapStyle: .arcGISTopographic)
        // Latitude first here. Longitude first in GeoJSON. I contain multitudes.
        map.initialViewpoint = Viewpoint(latitude: 39.0, longitude: -94.6, scale: 1e6)
        return map
    }()

    var body: some View {
        MapView(map: map)
    }
}
```

## 🧊 ColdFusion: Still Serving Features

```cfml
<cfhttp url="https://services.arcgis.com/ORG_ID/arcgis/rest/services/Parcels/FeatureServer/0/query"
        method="get" result="res">
    <cfhttpparam type="url" name="where"     value="1=1"><!--- the most-typed SQL in all of GIS --->
    <cfhttpparam type="url" name="outFields" value="*">
    <cfhttpparam type="url" name="outSR"     value="4326">
    <cfhttpparam type="url" name="f"         value="geojson">
</cfhttp>

<cfset features = deserializeJSON(res.fileContent).features>

<cfoutput>
    #arrayLen(features)# features returned.
    Times ColdFusion has been declared dead: many. Times it has noticed: 0.
</cfoutput>
```

---

## 🌐 Projection Opinions (strongly held, conformally preserved)

| Projection | Verdict |
|---|---|
| **Web Mercator** (EPSG:3857) | Fine for slippy tiles. Not fine for your thematic world map. Greenland is not the size of Africa; Africa is about 14× larger. |
| **Equal Earth** (EPSG:8857) | Equal-area, pleasant to look at, and has an actual EPSG code. The grown-up choice. |
| **Winkel Tripel** | The compromise candidate. Distorts everything a little so nothing is distorted a lot. |
| **Albers Equal Area Conic** (EPSG:5070) | The only acceptable way to draw the contiguous US. Look at that gentle curve on the Canadian border. *Chef's kiss.* |
| **Goode Homolosine** | The orange peel. Oceans hate it. I love it. |
| **Spilhaus** | One ocean, one map. For when the land is the negative space. |
| **Dymaxion** | No "up," no "down," no apologies. |
| **Plate Carrée** | Not a projection so much as a refusal to choose one. |

---

## 📜 Laws I Live By

- **Tobler's First Law** — everything is related, but near things more so. Also applies to bugs in adjacent modules.
- **MAUP** (Modifiable Areal Unit Problem) — change the boundaries, change the story. Every choropleth is a little bit of a lie; the job is making it an honest one.
- **The Ecological Fallacy** — the county is not the person.
- **The Coastline Paradox** — the length of my backlog depends entirely on the ruler you measure it with.
- **The Four Color Theorem** — four is enough. I will still use a ColorBrewer ramp with seven.
- **RFC 7946 §3.1.6** — exterior rings counterclockwise, holes clockwise. Esri JSON does the exact opposite. I have made peace with neither.
- **Define Projection ≠ Project** — one relabels, one transforms. Confusing them is a rite of passage and a fireable offense, in that order.
- **Always normalize your choropleths.** A map of raw counts is just a population map wearing a costume.

---

## 🐛 Known Issues

- [ ] Says "lat/long" out loud, types `lon, lat` in code, has been hurt by both
- [ ] `ERROR 999999: Something unexpected caused the tool to fail.` — yes, thank you, very helpful
- [ ] Cannot delete the feature class because of a schema lock held by… me, in another window
- [ ] Still has ArcMap muscle memory; reaches for the Editor toolbar that isn't there
- [ ] Writes `<cfoutput>` in a `.swift` file roughly once a quarter
- [ ] Cannot look at a restaurant placemat map without checking for a scale bar
- [ ] Notices when a movie's "satellite view" is obviously Web Mercator tiles
- [ ] Rainbow color ramps cause a visible flinch
- [ ] Nervously waiting to see what the NAD 83 → NATRF2022 transition does to every dataset ever
- [x] Finally stopped storing production data in shapefiles (the `.gdb` is fine, don't open it in Explorer)

---

## 📊 Attribute Table

<div align="center">

<img height="165" src="https://github-readme-stats.vercel.app/api?username=trbaker&show_icons=true&theme=default&hide_border=true" alt="GitHub stats" />
<img height="165" src="https://github-readme-stats.vercel.app/api/top-langs/?username=trbaker&layout=compact&hide_border=true" alt="Top languages" />

</div>

---

## 📡 Georeference Me

| Channel | Endpoint |
|---|---|
| 🌍 Website | `https://your-site.example` |
| 🗺️ ArcGIS Online | `https://YOUR_ORG.maps.arcgis.com` |
| 💼 LinkedIn | `https://linkedin.com/in/YOUR_LINKEDIN` |
| 📬 Email | `you@example.com` |

---

<div align="center">

```
┌──────────────────────────────────────────────────────────────┐
│  LEGEND                                                      │
│   ★  Me            ─── Commit history    ░░░ Unfinished      │
│   ●  Side project  - - Abandoned branch      side projects   │
├──────────────────────────────────────────────────────────────┤
│  Projection: Opinionated Conformal Conic                     │
│  Datum: D_Caffeine_1984        Units: commits                │
│  This README is not to scale. Not for navigation.            │
└──────────────────────────────────────────────────────────────┘
```

*Made with ♥, an unreasonable number of `.prj` files, and at least one silent datum transformation.*

<sub>Esri, HERE, Garmin, FAO, NOAA, USGS, © OpenStreetMap contributors, and the GIS User Community (force of habit)</sub>

</div>
