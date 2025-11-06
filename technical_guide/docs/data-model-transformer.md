# Data Model Transformer – Technical Documentation

## Overview

The **data-model-transformer** is a tool designed for transforming source data into a standardized target data model, typically for pan-European or cross-national datasets. It automates extraction, transformation, and loading (ETL) processes between country-specific schemas and a common target schema.

---

## Architecture

### Core Steps

1. **Extract**  
   Relevant source data is extracted into `.json` files stored in a temporary directory (`output/tmp`).

2. **Transform & Dump**  
   SQL dump files are created from the JSON data and saved in the output directory.

3. **Load**  
   The dump files are restored into the target database.

---

## Configuration

- **Configuration Files**:  
  Each country-theme has its own configuration file, which defines:
  - Source/target table mapping.
  - Field-level mapping (direct, expression, or function).
  - Options for mocking, additional filtering, geometric transforms, and field fetching.
- **Function Directory**:  
  For complex field mappings, configuration files can reference reusable Python functions placed in the `functions/` directory.

#### Example mapping types:
- Direct in config:  
  `"national_code": {"eval": "data['nationalcode'] if data['nationalcode'] != '' else 'void_unk'"}`
- Via function:  
  `"name": {"function": "inspire_xx_name"}`
- Fixed value:  
  `"national_level_code": {"eval": "'void_unk'"}`
- Field-to-field:  
  `"w_national_identifier": "inspireid"`

**Template parameters** such as `${a_parameter}` in configs are resolved at load time.

---

## Main Components

### Python Scripts (in `script/`)

- **transform.py**:  
  Orchestrates the process: loads configuration, merges with theme-specific settings, manages output directories, and calls extraction and transformation logic.

- **dump.py**:  
  Converts transformed JSON data to SQL dumps.  
  - Handles mock tables, writes SQL statements, and manages schema.

- **utils.py**:  
  Utility functions for:
  - Loading and templating configuration files.
  - Path management and OS compatibility.
  - Dynamic function loading.

### Function Modules (in `functions/`)

Reusable field-mapping functions, each encapsulated in a separate file (e.g., `es_tn_road_label.py`).  
These accept a `context` dict and return the appropriate transformed value.

**Example:**  
```python
def function_name(context):
    ome2_vial = context['data']['ome2_vial']
    # ... transformation logic ...
    return value
```

---

## Docker Support

A `Dockerfile` is provided for reproducible setup, including:
- Python 3, PostgreSQL client.
- All dependencies installed via `requirements.txt`.
- App runs as non-root user.

---

## Usage

1. Ensure configuration files exist for each country-theme.
2. Set the output directory in `conf.json`.
3. Run the transformation pipeline (typically via `transform.py`).
4. Review and load the generated SQL dump files in your target database.

**Note:**  
- The temporary directory (`output/tmp`) is deleted at the end of the process.
- Ensure sufficient disk space for output and temp files.
- Use the `mock` parameter to skip tables; `where` for filtering; `mapping` for field logic.

---

## Extending

- Add new transformation logic by placing new Python functions in the `functions/` directory.
- Update configuration files to reference new logic as needed.

---

## References

- [README.md](https://github.com/openmapsforeurope2/data-model-transformer/blob/main/README.md)
- [Example functions/](https://github.com/openmapsforeurope2/data-model-transformer/tree/main/functions)
- [Dockerfile](https://github.com/openmapsforeurope2/data-model-transformer/blob/main/docker/Dockerfile)
