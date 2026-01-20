# Documentation: `spiro_features_extraction`

The `spiro_features_extraction` class provides a modular architecture for extracting advanced mathematical features from forced expiratory (FE) spirometry signals. It includes:

* Area under the Flow-Volume Loop (% predicted)
* Angle of Collapse (AC)
* Deflating Balloon Model (ongoing development)

All modules assume the FE signal is standardized and oriented such that:

* Volume is in litres (TLC at 0, RV positive)
* Flow is in litres/s and positive
* Signal is right-skewed (i.e., normal expiratory flow direction)

---

## Class: `spiro_features_extraction`

Main container class encapsulating multiple feature extraction models.

---

### Subclass: `areaFE`

Calculates the area under the expiratory flow-volume loop and its predicted value using ECCS93 equations.

#### Initialization

```python
area = spiro_features_extraction.areaFE(FE_volume, FE_flow, sex, age, height)
```

#### Methods

* `calc_AreaPred()`

  * Returns predicted AreaFE using demographic inputs.

* `calc_areaFE()`

  * Computes area under the FE curve using trapezoidal integration.

---

### Subclass: `angle_of_collapse`

Implements a data-driven geometric fitting model to compute the angle of collapse after PEF.

#### Initialization

```python
ac = spiro_features_extraction.angle_of_collapse(FE_volume, FE_flow)
```

#### Methods

* `generate_linemodel(x, y, index)`

  * Constructs a piecewise linear model intersecting point `(x, y)`

* `min_line_model_error(plotProcess=False)`

  * Loops through all post-PEF points to find best-fitting point minimizing squared error

* `get_angle(x_p, y_p)`

  * Computes the geometric angle between two segments joined at `(x_p, y_p)`

* `calc_AC(plotModel=False, plotProcess=False)`

  * Returns computed angle of collapse and squared error

---

### Subclass: `deflating_baloon`

Models the FE signal using second-order ODE dynamics. Simulates the lungs as a deflating balloon.

> Note: the class name in code is spelled `deflating_baloon` (single "o" in "baloon"). This matches the implementation and should be used when instantiating the class.

#### Initialization

```python
db = spiro_features_extraction.deflating_baloon(FE_time, FE_volume, FE_flow)
```

#### Core Methods

* `orient_and_snip_signal()`

  * Prepares volume/flow signals for modeling by standardizing orientation

* `reorient_model()`

  * Reverts simulated signal to original coordinate system

* `get_excitation_phase(T1, params)`

  * Models early phase of expiration (excitation) based on dynamic type

* `calc_hypothesis(params)`

  * Simulates the flow-volume signal using the selected model and parameters

* `Cost_Function(params)`

  * Computes error between predicted and actual volume/flow to be minimized

* `run_model(excitation_type, plot_model=False, ...)`

  * Fits model using `differential_evolution` optimizer and plots results

* `run_simulation(...)`

  * Runs sensitivity analysis by varying one model parameter

* `calc_FEV1_FVC()`

  * Computes interpolated FEV1 and final FVC from model output

* `plot_model(only_FVL, add_title_text)`

  * Plots comparison between actual and simulated flow/volume signals

---

## Excitation Types

* `Linear`: (Deprecated)
* `Exponential pressure`: (Deprecated)
* `Non linear`: Nonlinear ramping of PEF
* `Default`: Constant initial condition

Each model type influences the cost function structure and parameter interpretation.

---

## Optimization Notes

All modeling is done via `scipy.optimize.differential_evolution`. Fit metrics include:

* Mean Squared Error (MSE)
* R² Score (flow and volume)

---

## Example Usage

```python
# Compute angle of collapse
ac = spiro_features_extraction.angle_of_collapse(volume, flow)
angle, cost = ac.calc_AC(plotModel=True)

# Compute AreaFE % predicted
af = spiro_features_extraction.areaFE(volume, flow, sex=1, age=35, height=170)
area_pred = af.calc_AreaPred()
area_actual = af.calc_areaFE()

# Fit balloon model
db = spiro_features_extraction.deflating_baloon(time, volume, flow)
db.run_model(excitation_type="Non linear", plot_model=True)
```

---

## Dependencies

* `numpy`
* `matplotlib.pyplot`
* `scipy.optimize.differential_evolution`
* `sklearn.metrics` (used to compute MSE and R² inside `run_model`)
* `utilities` (custom plotting utility used inside `angle_of_collapse`)

---

## Known issues / Implementation notes (from code)

1. Several excitation types are marked as discarded or deprecated in the code. Prefer using the `Non linear` or default fitting options for stable behavior.

2. The run_model implementation contains a typographical mistake in the condition that checks excitation types when computing fit metrics. The string used in the list does not match the literal "Exponential pressure"; as a result, the code path that computes MSE/R² for that specific excitation type may not be selected as intended. The code still imports `sklearn.metrics` and computes metrics, but behavior might not match expectations for the "Exponential pressure" branch. If you rely on that branch, inspect the implementation or adjust the source.

3. The deflating balloon module performs non-linear optimization and may be sensitive to initial conditions and bounds. Expect longer runtimes for differential evolution on large signals.

---

## References

* AreaFE: [DOI:10.2147/COPD.S51453](https://www.dovepress.com/area-under-the-forced-expiratory-flow-volume-loop-in-spirometry-indica-peer-reviewed-fulltext-article-COPD)
* Angle of Collapse: [DOI:10.1186/1465-9921-14-131](https://respiratory-research.biomedcentral.com/articles/10.1186/1465-9921-14-131)

---

## Licensing

This tool is intended for research and educational purposes. Ensure clinical validation before diagnostic use.
