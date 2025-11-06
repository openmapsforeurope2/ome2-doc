# data-tools – Technical Documentation

## Overview

The **data-tools** repository is part of the Open Maps For Europe 2 (OME2) project, which aims to develop a pan-European, large-scale (1:10,000) cartographic reference dataset. The repository provides a suite of command-line tools for managing and processing spatial data in a PostgreSQL/PostGIS environment, with a focus on executing and managing SQL scripts related to OME2 workflows.

---

## Main Functionalities

The application consolidates several OME2 functionalities for executing SQL scripts. The primary tools included are:

- **create_table**: Generates and executes scripts to create all necessary tables for the OME2 project.
- **border_extract**: Extracts objects near borders from a source table to a target table.
- **integrate**: Reinserts data processed in a working table back into the source table.
- **revert**: Reverts changes associated with a specified 'step' and all subsequent steps.
- **copy_table**: Copies tables from a specified schema to the public schema.
- **clean**: Tools for cleaning and preparing data (e.g., removing duplicates, fixing geometries).
- **integrate_from_validation**: Integrates data only if it passes validation checks.
- **prepare_data**: Prepares and initializes the database schema and working tables for OME2 processes.

Each tool is executed via command-line scripts and is typically parameterized via JSON configuration files.

---

## Command-Line Usage

All tools are run from the command line, for example:

```sh
python3 script/create_table.py -c conf.json -m mcd.json -T au
python3 script/border_extract.py -c conf.json -T tn -t road_link -b ad -d 3000 -n fr
python3 script/integrate.py -c conf_v6.json -T au -t administrative_unit_area_1 -s 30
python3 script/copy_table.py -c path/to/conf.json au.administrative_unit_area_1 ib.international_boundary_line
```

**Parameters** are typically:
- `-c`: Configuration file (mandatory)
- `-T`: Theme or table group (e.g., `au`, `hy`, `ib`, `tn`)
- `-t`: Table name
- `-d`: Distance (for border extraction)
- `-b`: Border country code
- `-s`: Step or sequence in the workflow
- Other arguments depending on the tool

---

## Example Workflows

Shell scripts in the `use_case/` directory (e.g., `au_matching.sh`, `clean_hy_tn.sh`, `init_tables_win.bat`) provide examples of end-to-end workflows for administrative unit matching, data extraction, cleaning, and integration across country borders.

---

## Database and Environment Setup

The repository includes a `Dockerfile` for reproducible setup. The Docker image is based on Debian, installs PostgreSQL client libraries, Python 3, and required Python dependencies from `requirements.txt`.

```dockerfile
FROM debian:10
RUN apt update
RUN apt install -y libpq-dev python3 python3-pip
ENV APP_DIR_SRC=/usr/local/src/data-tools/
WORKDIR $APP_DIR_SRC/../
COPY . $APP_DIR_SRC
WORKDIR $APP_DIR_SRC
RUN python3 -m pip install -r ./requirements.txt
```

A helper script (`docker/build-docker-image.sh`) allows building Docker images for different branches or environments.

---

## Configuration

Configuration JSON files define database connections, schema details, table lists, and workflow parameters. Each operation references these files for reproducible, flexible execution.

---

## Reference

- [README.md](https://github.com/openmapsforeurope2/data-tools/blob/main/README.md)
- [Example usage scripts](https://github.com/openmapsforeurope2/data-tools/tree/main/use_case)
- [Dockerfile](https://github.com/openmapsforeurope2/data-tools/blob/main/docker/Dockerfile)

---
