# au_matching – Technical Documentation

## Overview

**au_matching** is a C++ application, part of the Open Maps For Europe 2 (OME2) project, dedicated to the automated matching and reconciliation of administrative unit boundaries across European countries. It implements spatial algorithms for cross-border feature matching, geometry refinement, and robust data integration, and is intended for use in large-scale (1:10,000) pan-European mapping workflows.

---

## Architecture & Build

### Build System

- Uses **CMake** for cross-platform build configuration.
- Depends on several spatial and database libraries:  
  - Boost, GEOS, CGAL, GMP, MPFR, Shapelib, PostgreSQL (libpq), SQLite, and several OME2/IGN libraries (e.g., Socle, LibEPG).
- The project is built as a single executable.

**Example build (Unix):**
```sh
./build-unix-release.sh
```
Or using Docker:
```sh
docker build --build-arg GIT_BRANCH=main --build-arg NB_PROC=4 -t au_matching .
```

### Docker Support

A `Dockerfile` is provided for reproducible builds and deployment. The Docker image is built on top of `libepg` and compiles the project using the provided CMake configuration.

---

## Main Components

### Core Matching Logic

- **src/app/calcul/AuMatchingOp.cpp/.h**  
  Implements the main matching operator, with logic for boundary alignment, angle computation, and matching operations.
  - Key methods:
    - `Compute(std::string countryCode, bool verbose)`
    - Internal steps: initialization, matching computation, geometry feature extraction.

- **src/app/calcul/InitLandmaskCoastOp.cpp**  
  Specialized operator for initializing landmask and coastal boundary features as part of the matching process.

- **src/app/detail/refining.cpp**, **extractNotTouchingParts.cpp**, etc.  
  Helper modules for geometry refinement, extracting non-touching parts, and other spatial processing routines.

- **src/main.cpp**  
  Application entry point. Handles parsing of configuration/parameter files, logging setup, and orchestrates the matching steps through a suite of "steps" (processing stages).

### External Dependencies

- **OME2/IGN Libraries**: Provides foundational geometry, database, and utility classes.
- **Boost**: Progress reporting and other utilities.
- **GEOS/CGAL**: Geometry processing and computational geometry.
- **PostgreSQL/SQLite**: Database access for feature storage and queries.

---

## Build & Install

- **CMakeLists.txt**:  
  Handles include/library paths, links against dependencies, and configures the build.  
  Output executable is placed in `bin/Release` or `bin/Debug` depending on build type.

- **build-unix-release.sh** and **build-unix-package.sh**:  
  Shell scripts to automate build and (optionally) packaging via CPack.

---

## Usage

### Command-line

The application is typically run with configuration files specifying:
- EPG parameters (`--c`)
- Theme parameters (`--t`)
- Step codes, country codes, and other options

**Example:**
```sh
./au_matching --c path/to/epg_parameters.ini --t path/to/theme_parameters.ini --stepcode STEP_X --cc DE
```
For detailed options, run with `--help`.

### Input/Output

- Reads/writes features to/from spatial databases (PostgreSQL/PostGIS, SQLite).
- Operates on shapefiles as needed for testing or debugging.

---

## Detailed Pipeline Steps

### 1. Initialization
- Load EPG (environment/project) parameters and theme/step configurations.
- Initialize logging, workspace, and database connections.
- Parse processing steps to execute (e.g., boundary matching, refinement).

### 2. Data Preparation
- Load source administrative units from provided spatial databases.
- Optionally, apply geometry cleaning and attribute normalization.

### 3. Landmask and Coastline Initialization
- Extract and process landmask and coastal features.
- Prepare for correct handling of coastal boundaries during matching.

### 4. Boundary Matching
- Identify cross-border features and their neighbor relationships.
- Compute angles and geometric metrics for alignment.
- Apply spatial algorithms to align and match boundaries between countries.
- Store matching results, including matched feature IDs and geometry adjustments.

### 5. Geometry Refinement
- Refine the resulting geometries to remove slivers, overlaps, and other topological errors.
- Use geometric operations (buffering, union, difference) for cleanup.

### 6. Feature Extraction & Output
- Extract unified/matched features for output.
- Write results to target spatial database or output files.
- Generate logs and statistics for the matching process.

### 7. Finalization
- Clean up resources, close database connections.
- Summarize process results and errors in log output.

---

## Contributing

See [CONTRIBUTING.md](https://github.com/openmapsforeurope2/au_matching/blob/main/CONTRIBUTING.md) for guidelines, style guides, and information on joining the project team.

---

## References

- [CMakeLists.txt](https://github.com/openmapsforeurope2/au_matching/blob/main/CMakeLists.txt)
- [Dockerfile](https://github.com/openmapsforeurope2/au_matching/blob/main/docker/Dockerfile)
- [CONTRIBUTING.md](https://github.com/openmapsforeurope2/au_matching/blob/main/CONTRIBUTING.md)
