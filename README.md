# Interpretable composition-descriptor equations for oxide glasses

**Status: work in progress.** This is an ongoing project. The results and equations here are preliminary and will be updated as the work continues.

Symbolic-regression equations for **Young's modulus**, **density** and **liquidus temperature** of oxide glasses. They are built from five general descriptors of each composition, and benchmarked carefully against the machine-learning models of

> Z. Kang *et al.*, "Interpretable machine learning accelerates development of high-specific modulus glass",
> *Computational Materials Science* **246** (2025) 113482, doi:[10.1016/j.commatsci.2024.113482](https://doi.org/10.1016/j.commatsci.2024.113482)

## Idea

Each glass is converted to mole fractions $x_i$ over 17 oxides. For each property, the same property of the *pure* oxide, $p_i$ (from the literature, see `pure_oxide_properties.csv`), is combined into five descriptors:

| Descriptor | Formula |
|---|---|
| `mean_p` | $\sum_i x_i p_i$ |
| `gmean_p` | $\exp(\sum_i x_i \ln p_i)$ |
| `hmean_p` | $1/\sum_i (x_i/p_i)$ |
| `var_p` | $\sum_i x_i p_i^2 - (\sum_i x_i p_i)^2$ |
| `S_mix` | $-\sum_i x_i \ln x_i$ |

Because only one literature value per oxide is needed, an equation in these descriptors can in principle be applied to glasses containing oxides it was never trained on.

## Results (nested, grouped 5-fold cross-validation; every score is on unseen glasses)

| Property | Paper's model (raw oxides) | Our long equation | Our short equation |
|---|---|---|---|
| Young's modulus, R² | **0.897** | 0.682 | 0.619 |
| Density, R² | **0.935** | 0.752 | 0.705 |
| Liquidus temperature, R² | **0.954** | 0.595 | 0.580 |
| Specific modulus E/ρ, R² (7,301 glasses with both measured) | **0.850** | 0.553 | 0.457 |

Short equations (constants fitted on all data, `results/main/short_equations_cv.csv`):

- Young's modulus: $E = (2.443\,S_{mix}\,\overline{E}_{geo} + 214.5)^{0.709}$
- Density: $\rho = \overline{\rho} + 3.857/\overline{\rho}_{harm} - 1.612$
- Liquidus temperature: $T_{liq} = \overline{T}_m + \sqrt{S_{mix}}\,(-1216 + 1.179\times10^6/\overline{T}_{m,geo})$

The bars mark composition-weighted means: no subscript for the arithmetic mean, *geo* for the geometric mean, *harm* for the harmonic mean.

**What we found:**
- The paper's models reproduce under this stricter grouped nested cross-validation.
- The five-descriptor equations are interpretable and general, but clearly less accurate.
- The clearest physical finding: the liquidus temperature falls below the averaged melting point in proportion to $\sqrt{S_{mix}}$, a mixing (eutectic-type) depression. A two-constant formula captures almost all of what the long equation does.

## Contents

| Path | What it is |
|---|---|
| `glass_property_pipeline.ipynb` | Main notebook. Builds the descriptors, runs nested grouped CV with baselines, optional XGBoost (Optuna) and PySR symbolic regression, then fits the final equations. It saves after every trial or run and resumes after interruptions, and has a built-in self-test. |
| `combined_property_check.ipynb` | Compares the equations with the paper's models: the experimental glass, specific modulus E/ρ of 7,301 measured glasses, and the CaO–MgO–Al₂O₃–SiO₂ design space. |
| `pure_oxide_properties.csv` | Pure-oxide density, melting point and Young's modulus, with a source for every value. |
| `results/main/` | CV scores, per-fold metrics, final and short equations, all Pareto-front equations, figures, and the run record (settings, versions, data checksums). |
| `results/combined/` | Results of the combined-property comparison. |

## How to run

```bash
pip install -r requirements.txt     # PySR installs Julia automatically on first import
```

1. Open `glass_property_pipeline.ipynb`, then **Restart Kernel → Run All** with `SELF_TEST = True`. This takes about 10 min and ends with a PASS/FAIL table.
2. Set `SELF_TEST = False` and Run All for the full run (roughly 18 h with the default PySR settings).
3. Run `combined_property_check.ipynb` from the same folder.

The datasets are **not redistributed here**. The notebooks download them from the authors' repository, [kzzzynobad/High-Specific-Modulus-Glass](https://github.com/kzzzynobad/High-Specific-Modulus-Glass), and verify checksums.

## Data notes and limitations

- **Mixed sources for Young's modulus.** Moduli of Li₂O, Na₂O, K₂O and BaO are DFT values (de Jong *et al.*, *Sci. Data* 2 (2015) 150009); this calculation method typically underestimates oxide moduli by ~7–10%. The SiO₂, B₂O₃ and P₂O₅ moduli are glass values, and all others are polycrystal or single-crystal values.
- **K₂O "melting point".** K₂O decomposes at 350 °C, and that decomposition temperature is used in place of a melting point.
- **ZrO column.** The dataset column `ZrO` is interpreted as ZrO₂.
- **Short-equation selection.** The short equations were chosen after inspecting the Pareto fronts, so treat them as a sensitivity analysis.

## Related work

- Hu *et al.*, *npj Comput. Mater.* 6 (2020) 25: composition-weighted (Hölder-mean) descriptors for densities and elastic moduli of SiO₂-based glasses.
- Vitoria *et al.*, arXiv:2608.14853 (2026): physics-informed symbolic regression for Tg of alkali borate glasses.
