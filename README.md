# Spirolib

A Python library for processing and analyzing spirometry data.

## Features
- Batch processing of spirometry data
- Feature extraction from spiro signals
- Signal processing utilities

## Installation
Clone the repository and use the package in your Python projects. For local development, you can clone the repo and import the package from the cloned directory or add it to your PYTHONPATH. Example:

```bash
git clone https://github.com/neeldas6789/Spirolib_new.git
cd Spirolib_new
# then in Python scripts, import the package from the cloned repo
```

(If you plan to publish/pack the project, add a setup.py / pyproject.toml and use pip install.)

## Usage
Import the modules from the `spirolib` package in your Python scripts. The package __init__ exposes the main classes and utilities. Example:

```python
from spirolib import (
    spiro_signal_process,
    spiro_features_extraction,
    spiro_features_lite,
    spiro_trialsbatch_process,
    spiro_batch_process,
    utilities,
)

# Example instantiation of signal processor (note the required `scale` argument):
# sp = spiro_signal_process(time, volume, flow, patientID, trialID, flag_given_signal_is_FE, scale)
```

## Project Structure
- `spirolib/` - Main library code
  - `__init__.py` (exports main classes)
  - `spiro_signal_process.py`
  - `spiro_features_extraction.py`
  - `spiro_features_lite.py`
  - `spiro_batch_process.py`
  - `utilities.py`
- `docs/` - Documentation and guides

(Note: there is no top-level `spirolib.py` entrypoint in the repository; use the `spirolib` package.)

## License
Specify your license here.
