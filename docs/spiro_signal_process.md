The `spiro_signal_process` class provides tools to analyze spirometry data, particularly for handling flow-volume loops (FVLs), forced expiratory manoeuvres (FE), and computing standard spirometry parameters such as FEV1, FVC, and PEF.

## Class Initialization

```python
sp = spiro_signal_process(time, volume, flow, patientID, trialID, flag_given_signal_is_FE, scale=1, scale2=1, scale3=1)
```

### Parameters

* `time`: Time array of the spirometry manoeuvre (list or 1D array). The constructor multiplies this by `scale`.
* `volume`: Volume array of the manoeuvre (list or 1D array). The constructor multiplies this by `scale2`.
* `flow`: Flow array (list or 1D array). The constructor multiplies this by `scale3`.
* `patientID`: Unique identifier for the patient
* `trialID`: Identifier for the trial
* `flag_given_signal_is_FE`: Boolean flag indicating if the signal is forced expiration only
* `scale`: Multiplicative factor applied to `time` on construction (useful to convert units, e.g., ms -> s uses 0.001)
* `scale2`: Multiplicative factor applied to `volume` on construction (e.g., ml -> L uses 0.001)
* `scale3`: Multiplicative factor applied to `flow` on construction (e.g., ml/s -> L/s uses 0.001)

---

## Core Methods

### Data Preprocessing

* `correct_data_positioning(flip_vol=False, flip_flow=False)`

  * Ensures signal orientation is standard (expiratory FVL right skewed and PEF positive)

* `standerdize_units()`

  * Converts all input data units to litres and seconds (divides internal arrays by 1000). Note: Depending on the units you pass at construction, you can use the `scale`, `scale2`, and `scale3` constructor arguments to pre-scale inputs so that subsequent calls to `standerdize_units()` behave as expected.

* `manual_trim(begin_time=0, end_time=None)`

  * Trims the signal between the given time bounds

### Plotting

* `plotFVL(only_FVL=False, show_ID=True, add_text="", only_FE=False, color='black', dpi=100, figsize=None, grid_on=True)`

  * Plots the flow-volume loop (FVL) or time-series representations of volume and flow

### Signal Processing Helpers

* `butter_lowpass(cutoff, fs, order)` / `butter_lowpass_filter(data, cutoff, fs, order)`

  * Implements low-pass filtering using Butterworth filters

* `smooth_FVL_start(time, vol, flow)`

  * Smoothens the FVL at the start of FE for better shape quality

### Index Detection

* `get_Indexes_In_1s(start_index=0)`

  * Returns index at 1 second from the given start point

* `get_PEF_index(indx1, indx2)`

  * Identifies the index corresponding to Peak Expiratory Flow (PEF)

* `get_FE_start_end(...)`

  * Determines the indices corresponding to the start and end of forced expiration

* `get_FI_start(index1=None)`

  * Gets the start index of forced inspiration (for combined FI-FE signals)

### FE Signal Extraction

* `get_FE_signal(start_type=None, thresh_percent_begin=2, thresh_percent_end=0.25, plot=False)`

  * Returns FE segment (time, volume, flow) and optionally plots it

### Trimming & Thresholding

* `backExtrapolate_FEstart()`

  * Uses back-extrapolation to determine FE start and validate BEV criteria

* `threshPEF_FEstart(thresh_percent_begin)`

  * Uses PEF threshold to identify FE start

* `trim_FE_end(thresh_percent_end)`

  * Uses PEF threshold to identify FE end

### Acceptability Checks

* `check_rise_to_PEF()`

  * Confirms presence of a rise to PEF at signal start

* `check_largest_time_interval(max_time_interval=1, FE_time_duration=4)`

  * Checks time gaps in the signal to ensure uniform sampling

* `check_acceptability_of_spirogram(min_FE_time=6, thresh_percent_end=0.5)`

  * Evaluates acceptability of the spirometry signal

### Spirometry Parameter Computation

* `calc_FEV1_FVC()`

  * Calculates FEV1 and FVC using interpolated 1-second volume

* `calc_flow_parameters()`

  * Computes PEF, FEF25, FEF50, FEF75, and FEF25-75

### Reference Prediction (ECCS93)

* `calc_ECCS93_ref(param)`

  * Returns predicted reference value for a given parameter using ECCS93 formulas

### Finalization

* `finalize_signal(sex=None, age=None, height=None)`

  * Finalizes signal after processing and calculates all flow/volume metrics and predicted values

### Internal Attributes (Post-finalization)

* `FEV1`, `FVC`, `Tiff`, `PEF`, `FEF25`, `FEF50`, `FEF75`, `FEF25_75`
* Reference prediction percentages: `FEV1_PerPred`, `FVC_PerPred`, etc.
* `index1`, `index2`: Start and end indices of FE segment
* `signal_finalized`: Flag indicating processing completion

---

## Notes

* The constructor now accepts `scale`, `scale2`, and `scale3` to pre-scale the input time, volume and flow arrays respectively. Use these to convert input units to the internal units expected by the class (for example: ms -> s and ml -> L).
* It is important to run `correct_data_positioning()` and `standerdize_units()` before performing calculations or acceptability checks if your inputs are not already in litres and seconds.
* Plotting methods help visualize raw and processed signals for verification
* ECCS93 reference computations depend on gender, age, and height
* All array attributes are assumed to be NumPy arrays internally

---

## Example Workflow

```python
# If your inputs are in ms and ml, scale them to seconds and litres on construction:
sp = spiro_signal_process(time_ms, volume_ml, flow_ml_per_s, patientID='P1', trialID='T1', flag_given_signal_is_FE=False, scale=0.001, scale2=0.001, scale3=0.001)
sp.correct_data_positioning()
sp.standerdize_units()
accepted, reason = sp.check_acceptability_of_spirogram()
if accepted:
    sp.finalize_signal(sex=1, age=35, height=175)
    print(sp.FEV1, sp.FVC, sp.PEF)
```

---

## Dependencies

* `numpy`
* `matplotlib.pyplot`
* `scipy.signal.butter`, `scipy.signal.lfilter`
* `peakutils`

Make sure these libraries are installed before using the class.

---

## Licensing

This module is intended for educational and research purposes. If used clinically or commercially, ensure proper validation and compliance with medical device regulations.
