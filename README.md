# EF5-RTI: EF5-FLASH Input Preparation Toolkit

This repository contains helper scripts and a guided notebook for preparing hydrologic model inputs for EF5-style flash flood simulation workflows.

The core intent is to:
- pull forcing/observation data (MRMS precipitation and USGS streamflow),
- convert and organize files into model-ready formats, and
- run a repeatable, notebook-driven preprocessing workflow.

## Background: EF5-FLASH (high-level)

EF5 is a distributed hydrologic modeling framework commonly used for event-based and real-time flood simulation. In many operational or research workflows, EF5 is paired with high-frequency precipitation products (such as MRMS) and streamflow observations (such as USGS NWIS Instantaneous Values) to support setup, calibration, and evaluation tasks.

In that context, this project focuses on the *data preparation* side of an EF5-FLASH-style workflow:
- preparing precipitation forcing inputs from MRMS PrecipRate archives,
- preparing observed streamflow time series for comparison,
- and walking through raster/model input preparation in the notebook.

## Repository layout

- `prepare_model.ipynb`
  - Primary workflow notebook.
  - Contains the step-by-step process to prepare model inputs (raster clipping, conversion, and related preprocessing tasks).
  - **Recommended path:** follow this notebook sequentially from top to bottom.

- `download_mrms_preciprate.sh`
  - Bash helper script to download MRMS PrecipRate `.gz` files from IEM mtarchive for a date range.
  - Supports dry-run mode and skips files that already exist locally.
  - Decompresses downloaded `.gz` files at the end of a real run.

- `fetch_usgs_from_control.py`
  - Python helper script to download USGS NWIS instantaneous streamflow (`parameterCd=00060`) for one gauge and date range.
  - Converts discharge from cfs to cms and writes a CSV.
  - Exports **all available timesteps** returned by USGS in the requested interval.

- `requirements.txt`
  - Frozen Python package list exported from the working environment used for this project.
  - Use this file to recreate a compatible environment for notebook and script execution.

- `control_files/`
  - Example control/configuration files used for EF5 execution contexts.
  - Useful as templates/reference when connecting prepared inputs into an EF5 run.

- `__pycache__/`
  - Python bytecode cache artifacts.

## Recommended workflow (important)

1. Start with `prepare_model.ipynb` and follow cells in order.
2. Use helper scripts to fetch raw forcing/observation data.
3. Return to notebook steps for conversion/formatting and final model input preparation.

The notebook is the orchestrator for the full pipeline; the scripts are supporting utilities.

## Helper script usage

### 1) Download MRMS precipitation

Script: `download_mrms_preciprate.sh`

Make executable (one-time):

```bash
chmod +x download_mrms_preciprate.sh
```

Show help:

```bash
./download_mrms_preciprate.sh --help
```

Dry-run first (recommended):

```bash
./download_mrms_preciprate.sh \
  --start-date 2022-07-27 \
  --end-date 2022-07-30 \
  --dest-dir ~/MRMS_preciprate \
  --dry-run
```

Run actual download:

```bash
./download_mrms_preciprate.sh \
  --start-date 2022-07-27 \
  --end-date 2022-07-30 \
  --dest-dir ~/MRMS_preciprate
```

Behavior notes:
- Creates destination directory if it does not exist.
- Skips files that already exist (either `.gz` or already decompressed version).
- Prints a compact summary with skipped/downloaded counts.
- In dry-run mode, no files are downloaded or decompressed.

### 2) Download USGS streamflow observations

Script: `fetch_usgs_from_control.py`

Show help:

```bash
python3 fetch_usgs_from_control.py --help
```

Example:

```bash
python3 fetch_usgs_from_control.py \
  --gauge 04085200 \
  --start-date 2022-07-27 \
  --end-date 2022-07-30 \
  --outdir ~/Kewaunee/observations
```

Behavior notes:
- Accepted date formats: `YYYYMMDDHHMMSS`, `YYYY-MM-DD`, or ISO-8601.
- Output is UTC and includes all available USGS timesteps in the requested interval.
- Output file pattern:
  - `Streamflow_Time_Series_CMS_UTC_USGS_<gauge>.csv`

## Dependencies and environment

- `download_mrms_preciprate.sh` requires common shell tools and `wget`, `grep`, `sed`, `gunzip`.
- `fetch_usgs_from_control.py` uses Python 3 standard library only.
- Notebook and geospatial preprocessing steps rely on additional Python packages listed in `requirements.txt`.

### Create and install the Python environment

From the `EF5-RTI` repository root:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Optional (for Jupyter notebook kernel selection):

```bash
python -m ipykernel install --user --name ef5-rti --display-name "Python (ef5-rti)"
```

Then open `prepare_model.ipynb` and select the installed `Python (ef5-rti)` kernel.

## Quality, validation, and known limitations

This project is practical and useful for iterative modeling work, but it should be treated as an evolving workflow rather than a fully hardened production system.

Please keep in mind:
- some paths, assumptions, and examples are environment-specific,
- edge cases across all basins/events may not be fully tested,
- upstream data service behavior/availability can change,
- there is room for improvement in robustness, error handling, and broader test coverage.

Before operational use, validate outputs for your basin/event and review intermediate products in the notebook.

## Suggested future improvements

- Add automated tests for date parsing, download logic, and CSV output schema.
- Add retry/backoff logic for transient network/API failures.
- Add structured logging and optional verbose/quiet modes.
- Add notebook checks to verify required files/directories before heavy processing steps.
