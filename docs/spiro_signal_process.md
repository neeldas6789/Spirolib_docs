# Documentation: `spiro_signal_process`

The `spiro_signal_process` class provides tools to analyze spirometry data, particularly for handling flow-volume loops (FVLs), forced expiratory manoeuvres (FE), and computing standard spirometry parameters such as FEV1, FVC, and PEF.

## Class Initialization

```python
sp = spiro_signal_process(time, volume, flow, patientID, trialID, flag_given_signal_is_FE, scale)
```

### Parameters

* `time`: Time array of the spirometry manoeuvre (list or 1D array, e.g., raw units such as ms or s).
* `volume`: Volume array of the manoeuvre (list or 1D array, preferably in litres).
* `flow`: Flow array (list or 1D array, preferably in litres/sec).
* `patientID`: Unique identifier for the patient (string or number).
* `trialID`: Identifier for the trial (string or number; use `'Best'` for best/all trials).
* `flag_given_signal_is_FE`: Boolean flag indicating if the provided signal is forced expiration only.
* `scale`: Scale factor applied to the input `time` values (e.g., use `1` if `time` is already in seconds, or `0.001` if `time` is in milliseconds).

---

## Core Methods

### Data Preprocessing

* `correct_data_positioning(flip_vol=False, flip_flow=False)`

  * Ensures signal orientation is standard (expiratory FVL right skewed and PEF positive)

* `standerdize_units()`

  * Converts all input data units to litres and seconds

* `manual_trim(begin_time=0, end_time=None)`

  * Trims the signal between the given time bounds

### Plotting

* `plotFVL(only_FVL=False, show_ID=True, add_text="", only_FE=False, color='black', dpi=100, figsize=None, grid_on=True)`

  * Plots the flow-volume loop (FVL) or time-series representations of volume and flow

... (rest unchanged) ...
