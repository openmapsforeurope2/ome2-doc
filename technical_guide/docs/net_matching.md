# net_matching – Technical Documentation

Repository: [openmapsforeurope2/net_matching](https://github.com/openmapsforeurope2/net_matching)  
Primary Language: C++  
License: GNU General Public License v3.0

---

## Overview

**net_matching** is a C++ application forming part of the Open Maps For Europe 2 (OME2) initiative. It is tasked with the automated matching and reconciliation of network features (such as linear infrastructure or topological networks) across national borders, supporting the production of harmonized, pan-European datasets.

---

## Architecture

- **Language:** C++
- **Build System:** Likely CMake (typical for OME2 projects and C++ spatial applications)
- **License:** GPL-3.0

The repository is structured to support high-performance, batch-oriented spatial network matching and reconciliation. The core logic focuses on matching, aligning, and topologically correcting linear and network features from different national sources.

---

## Main Components

While the exact file structure is not enumerated, based on standard OME2 practices and the C++ focus, you can expect key components such as:

- **Source Directories:**  
  - `src/` or `source/` containing core logic.
  - Submodules for network parsing, topology, matching, and output.
- **Core Classes/Modules:**
  - **Network Loader:** Reads and parses spatial network data from databases or files.
  - **Network Matcher:** Implements algorithms for cross-border feature detection and alignment.
  - **Topology Validator:** Ensures network connectivity and the absence of topological errors (e.g., dangles, splits).
  - **Geometry Processor:** Handles snapping, line merging, and refinement of features.
  - **Output Writer:** Writes unified network data to output spatial databases or files.

---

## Build & Installation

**Typical build steps (to be confirmed in project-specific README):**
```sh
# Clone the repository
git clone https://github.com/openmapsforeurope2/net_matching.git
cd net_matching

# Configure and build (presumed CMake)
mkdir build && cd build
cmake ..
make -j4
```
**Dependencies:**  
Expect dependencies on Boost, GEOS, GDAL, and possibly OME2/IGN internal libraries. Ensure these are available before building.

---

## Usage

**Command-line interface:**  
The application is likely run with parameters specifying:
- Input/output database connection strings or file paths
- Configuration files controlling matching thresholds and process steps
- Country codes or area of interest

**Example:**
```sh
./net_matching --config config.ini --input-db input.sqlite --output-db output.sqlite --country DE
```

Check for a `--help` option for detailed usage.

---

## Processing Steps

### 1. Initialization
- Load configuration and parameters.
- Set up logging, workspace, and database connections.

### 2. Data Import & Preparation
- Read network features (lines, nodes) from source databases.
- Normalize geometry and attributes.

### 3. Candidate Detection
- Identify network features that are at or near country borders.
- Build spatial indices for efficient neighbor queries.

### 4. Cross-Border Matching
- For each border feature, locate candidate matches in neighboring datasets.
- Compute geometric similarity (distance, angle, shape metrics).
- Score and select best matches using configurable thresholds.

### 5. Alignment & Snapping
- Adjust geometries to ensure matched features are coincident at the border.
- Snap nodes and endpoints as needed to guarantee connectivity.

### 6. Topological Correction
- Run topology validation to detect and repair inconsistencies (e.g., splits, overlaps, gaps, dangles).
- Merge or split lines as necessary to create a seamless network.

### 7. Output Generation
- Write the harmonized and topologically correct network to the output database or file.
- Export logs and statistics.

### 8. Finalization
- Close connections, summarize processing results, and clean temporary resources.

---

## Best Practices

- Always validate input data for geometry and attribute completeness.
- Use version control for configuration and parameter files.
- Document all manual decisions and overrides during the matching process.
- Test outputs visually and with automated topology checks.

---

## References

- [OME2 Project](https://github.com/openmapsforeurope2)
- [GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)

---

*For detailed API, developer guidelines, or advanced configuration, please consult the repository README or reach out to the OME2 technical team.*
