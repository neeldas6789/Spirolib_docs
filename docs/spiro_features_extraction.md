# Documentation: `spiro_features_extraction`

The `spiro_features_extraction` class provides a modular architecture for extracting advanced mathematical features from forced expiratory (FE) spirometry signals. It includes:

* Area under the Flow-Volume Loop (% predicted)
* Angle of Collapse (AC)
* Deflating Balloon Model (ODE-based dynamic simulation)

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

Models the FE signal using a second-order ODE dynamics, simulating the lungs as a deflating balloon. Supports:

* `orient_and_snip_signal()`
* `reorient_model()`
* `get_excitation_phase(T1, params)`
* `calc_hypothesis(params)`
* `Cost_Function(params)`
* `run_model(excitation_type, plot_model=False, ...)`
* `run_simulation(sim_param, sim_type, ...)`
* `calc_FEV1_FVC()`
* `plot_model(only_FVL, add_title_text)`

These methods enable parameter optimization via `scipy.optimize.differential_evolution` and visualization of model fits, including sensitivity analyses (MSE, R² metrics).

---

## Excitation Types

* `Linear` (deprecated)
* `Exponential pressure` (deprecated)
* `Non linear` (nonlinear PEF ramp-up)
* `Default` (constant initial conditions)

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
* `sklearn.metrics`
* `utilities` (custom plotting utility used inside `angle_of_collapse`)

Ensure these libraries are installed before using this class.

---

## References

* AreaFE: [DOI:10.2147/COPD.S51453](https://www.dovepress.com/area-under-the-forced-expiratory-flow-volume-loop-in-spirometry-indica-peer-reviewed-fulltext-article-COPD)
* Angle of Collapse: [DOI:10.1186/1465-9921-14-131](https://respiratory-research.biomedcentral.com/articles/10.1186/1465-9921-14-131)

---

## Licensing

This tool is intended for research and educational purposes. Ensure clinical validation before diagnostic use.