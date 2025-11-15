# Winnipeg Postal-Code Route Mapper (Stage 1)

<p align="left">
  <a href="https://www.python.org/"><img alt="Python" src="https://img.shields.io/badge/Python-3.8%2B-blue.svg" /></a>
  <a href="#jupyter--notebook-recommended-for-stage-1"><img alt="Jupyter" src="https://img.shields.io/badge/Works%20in-JupyterLab-orange" /></a>
  <a href="https://pypi.org/project/pgeocode/"><img alt="pgeocode" src="https://img.shields.io/badge/pgeocode-CA%20geocoding-lightgrey" /></a>
  <a href="#license"><img alt="License" src="https://img.shields.io/badge/License-MIT-green" /></a>
  <a href="https://github.com/" ><img alt="PRs welcome" src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" /></a>
</p>

Turn a list of Canadian postal codes (Winnipeg or anywhere in CA) into an optimized visiting order and an interactive Leaflet map. Stage 1 operates at **postal-code level** using straight-line distance with a **Nearest-Neighbor + 2-opt** improvement. It also exports a driver-friendly CSV itinerary.

> Stage 2 (address-level + road-network times with OR-Tools) can be added later without changing the input format.

---

## Features
- Geocode Canadian postal codes to latitude/longitude using **pgeocode**.
- Build an initial tour via **Nearest-Neighbor**, then improve it with **2-opt** (keeps depot fixed start/end).
- Export **interactive Leaflet map** (`route_postal_map.html`) with numbered markers and a route polyline.
- Export **ordered CSV** (`route_postal_order.csv`) with sequence, coordinates, segment km, and estimated minutes.
- **Robust geocoding**: try full postal; fall back to **FSA** (first 3 chars); if depot fails, use **stops centroid** with a warning.
- **Jupyter-friendly**: works as a module (functions) or as a CLI script with argparse.

---

## Repository structure
```
.
├─ postal_route_mapper.py      # Jupyter-friendly script (functions + optional CLI)
├─ examples/
│  └─ postal_codes.csv         # Example input (header: postal_code)
├─ docs/                       # Demo assets (optional)
│  ├─ demo.gif                 # Short screen recording of the map
│  └─ screenshot.png           # Static image
├─ outputs/
│  ├─ route_postal_map.html    # Generated Leaflet map (example)
│  └─ route_postal_order.csv   # Generated itinerary (example)
└─ README.md
```

> You can change output paths; the script writes to the current working directory unless you specify otherwise.

---

## Requirements
- Python 3.8+
- Packages: `pgeocode` (geocoding), standard library only for the rest.

Install:
```bash
pip install pgeocode
```

> **Windows/Notebook tip:** If you copy code from the README/chat, avoid “smart” quotes or dashes in code. Use plain ASCII quotes (" ") and hyphens (-).

---

## Input format (CSV)
**Header:** `postal_code`

Example `examples/postal_codes.csv`:
```
postal_code
R3T 2N2
R2C 3A2
R2X 1Z2
R3E 0W2
R3J 0S4
R2N 1L7
R3K 0Y2
```

---

## Usage

### A) Jupyter / Notebook (recommended for Stage 1)
1) Ensure `pgeocode` is installed.
2) Load the script cell OR import the module if saved to file.

Call with a Python list:
```python
postal_list = [
    "R3T 2N2","R2C 3A2","R2X 1Z2","R3E 0W2",
    "R3J 0S4","R2N 1L7","R3K 0Y2"
]
route = route_from_postals(
    postal_list,
    depot="R3C 4T3",               # your depot (full code or FSA is fine)
    out_html="route_postal_map.html",
    out_csv="route_postal_order.csv",
)
```

Or from a CSV:
```python
route = run_from_csv(
    "examples/postal_codes.csv",
    depot="R3C 4T3",
    out_html="route_postal_map.html",
    out_csv="route_postal_order.csv",
)
```

Display the map inline (Jupyter):
```python
from IPython.display import IFrame
IFrame("route_postal_map.html", width="100%", height=600)
```

### B) CLI (optional)
If you saved the script as `postal_route_mapper.py`, you can run:
```bash
python postal_route_mapper.py --csv examples/postal_codes.csv --depot "R3C 4T3" --out route_postal_map.html --ordercsv route_postal_order.csv
```

Use a built-in sample set:
```bash
python postal_route_mapper.py --use-sample --depot "R3C 4T3"
```

---

## Demo

Add a quick visual to your repo so people see it immediately.

**Animated GIF** (recommended):

```markdown
![Demo](docs/demo.gif)
```

**Static screenshot**:

```markdown
![Route map screenshot](docs/screenshot.png)
```

> Place the files in a `docs/` folder at the repo root so GitHub finds them easily.

### How to capture the demo GIF
- **Windows:** use [ScreenToGif](https://www.screentogif.com/) → record your browser showing `route_postal_map.html` → save to `docs/demo.gif`.
- **macOS:** use QuickTime (Screen Recording) + [gifify](https://github.com/jclem/gifify) or [Gifski](https://gif.ski/).
- **Linux:** use [Peek](https://github.com/phw/peek) or `byzanz-record`.

### Suggested flow for the recording
1. Run the notebook cell to generate `route_postal_map.html`.
2. Open the map in your browser and click a few numbered markers.
3. Pan/zoom to show the whole route and the depot labels.
4. Keep it **5–10 seconds** so the GIF stays small.

---

## Outputs
- **route_postal_map.html**: Leaflet map with markers
  - Marker labels: `#n — POSTAL` for stops, and `Depot (Start/End)` for depot.
- **route_postal_order.csv**: ordered visits
  - Columns: `sequence, label, lat, lon, km_from_prev, est_travel_min`.

> Time estimates use straight-line distance at a configurable city speed (default 35 km/h). Adjust `ASSUMED_SPEED_KMH` for rush hour or winter.

---

## Algorithm details
- **Seed:** Nearest-Neighbor (greedy) from the depot.
- **Polish:** 2-opt local search (keeps depot fixed). Stops are reversed between i..k if it shortens path length.
- **Distance metric:** Haversine (great-circle) between centroids.

### Why it’s not exact
- Postal (and FSA) centroids are coarse locations; multiple addresses share a centroid.
- No road network is used in Stage 1; results are approximations. Stage 2 will add road travel times and constraints.

---

## Configuration
Open `postal_route_mapper.py` and edit these constants if needed:
```python
ASSUMED_SPEED_KMH = 35          # city average speed for time estimates
OUTPUT_HTML_DEFAULT = "route_postal_map.html"
OUTPUT_CSV_DEFAULT  = "route_postal_order.csv"
```

You can also add your **default depot** in your notebook wrapper if desired.

---

## Troubleshooting
- **ModuleNotFoundError: postal_route_mapper**: either import from the same folder, or run the script cell directly so functions are in scope.
- **Geocoding warnings** like `[warn] 'R3T 2N2' not found at full precision; using FSA centroid 'R3T'.` are normal. The script falls back safely.
- **Depot fails to geocode**: the script uses the **centroid of the stops** as a fallback depot so you can still generate a route.
- **SyntaxError with quotes/dashes**: replace smart quotes/dashes with plain ASCII. In code, strings must be quoted and comments start with `#`.

---

## Roadmap (Stage 2+)
- Address-level geocoding (full addresses) with time windows and service times.
- Road-network travel-time matrix via OSRM/Google/GraphHopper.
- OR-Tools VRPTW solver (capacities, shift windows, penalties for lateness).
- Live “next stop” logic and partial re-optimization.

---

## License
MIT (or your preferred license). Add a `LICENSE` file if publishing.

---

## Acknowledgments
- Map tiles by OpenStreetMap contributors.
- Geocoding via `pgeocode`.

---

## Citation (resume-friendly snippet)
> Built a Python/Jupyter postal-code route optimizer (pgeocode + Leaflet) with NN + 2-opt, exporting driver-ready HTML maps and CSV itineraries; foundation for address-level OR-Tools optimization.

