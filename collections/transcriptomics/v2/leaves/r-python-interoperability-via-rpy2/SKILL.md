---
name: r-python-interoperability-via-rpy2
description: Use when a Python pipeline (e.g., ctmm's preprocess.softimpute for imputing
  missing cell type-specific pseudobulk ctp and ctnu matrices) must call R-only programs
  such as softImpute via rpy2, and you need to set up the R/rpy2 version pairing correctly.
when_to_use_negative:
- If no working R installation is available or R cannot be installed on the system,
  since CTMM hard-depends on calling R through rpy2
- If the needed routine exists natively in Python (pure-Python alternatives), adding
  the rpy2/R bridge and its version constraints is unnecessary overhead
- If the R version is >= 4.5 but rpy2 is left at the PyPI pin (rpy2 >= 3.4, < 3.6),
  the interop will not work without manually upgrading rpy2 to >= 3.6
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: rpy2
  role: Python-to-R bridge that lets ctmm call R routines; version must be matched
    to the R installation (3.5.x for R 3.6-4.4, >= 3.6 for R >= 4.5)
  repo: https://rpy2.github.io/
- name: R
  role: Runtime environment hosting the routines (softImpute, mvtnorm, numDeriv, abind,
    tensor) that ctmm invokes through rpy2
- name: softImpute
  role: R program for matrix completion, invoked via rpy2 by preprocess.softimpute
    to fill missing ctp/ctnu entries
- name: ctmm
  role: Python package whose preprocess module wraps the R calls made through rpy2
  repo: https://github.com/Minhui-Chen/ctmm_python
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:03f542ad71db4d2e7df79aa6fa0a621f20fef27e3adac572240ecce6424c80b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - R
  - ctmm
  - rpy2
  - softImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# r-python-interoperability-via-rpy2

## Summary

Enables the Python ctmm package to execute R-only routines (softImpute, mvtnorm, numDeriv, abind, tensor) through rpy2, which requires a working R installation and a version-matched R/rpy2 pairing. In this paper it is the bridge that lets Python-side preprocessing call R softImpute for matrix completion of missing CTP and ctnu entries.

## When to use

Use when a Python pipeline (e.g., ctmm's preprocess.softimpute for imputing missing cell type-specific pseudobulk ctp and ctnu matrices) must call R-only programs such as softImpute via rpy2, and you need to set up the R/rpy2 version pairing correctly.

## When NOT to use

- If no working R installation is available or R cannot be installed on the system, since CTMM hard-depends on calling R through rpy2
- If the needed routine exists natively in Python (pure-Python alternatives), adding the rpy2/R bridge and its version constraints is unnecessary overhead
- If the R version is >= 4.5 but rpy2 is left at the PyPI pin (rpy2 >= 3.4, < 3.6), the interop will not work without manually upgrading rpy2 to >= 3.6

## Inputs

- A working R installation (R 3.6-4.4 with rpy2 3.5.x, or R >= 4.5 with rpy2 >= 3.6)
- Python ctmm environment installed via 'pip install ctmm' (or from source via git clone https://github.com/Minhui-Chen/CTMM.git)
- ctp (cell type-specific pseudobulk) and ctnu (cell type-specific noise variance) matrices with missing entries for individual-cell type pairs with <=10 cells, to be imputed by R softImpute through rpy2

## Outputs

- Imputed ctp/ctnu matrices produced by the R softImpute program inside the Python session
- Imputation quality summaries (Pearson correlation and MSE between imputed and held-out true values, written to imputation_metrics.csv in the task workflow)

## How to apply

First check the installed R version: CTMM pins rpy2 >= 3.4, < 3.6, which supports R 3.6-4.4, whereas R >= 4.5 requires manually upgrading to rpy2 >= 3.6 after 'pip install ctmm' (the resulting dependency-conflict warning is expected and safe to ignore). Next, install the R packages the routines depend on with install.packages(c("abind", "mvtnorm", "numDeriv", "softImpute", "tensor")); MASS and Matrix ship with R. Then call the Python wrappers that bridge to R via rpy2, e.g. ctp = preprocess.softimpute(ctp), which wraps the R softImpute program for matrix completion. For the CTP/ctnu imputation use case, restrict entries to genes expressed in all retained cell types before imputation and validate by computing Pearson correlation and mean squared error between imputed and held-out true masked entries.

## Related tools

- **rpy2** (Python-to-R bridge that lets ctmm call R routines; version must be matched to the R installation (3.5.x for R 3.6-4.4, >= 3.6 for R >= 4.5)) — https://rpy2.github.io/
- **R** (Runtime environment hosting the routines (softImpute, mvtnorm, numDeriv, abind, tensor) that ctmm invokes through rpy2)
- **softImpute** (R program for matrix completion, invoked via rpy2 by preprocess.softimpute to fill missing ctp/ctnu entries)
- **ctmm** (Python package whose preprocess module wraps the R calls made through rpy2) — https://github.com/Minhui-Chen/ctmm_python

## Examples

```
from ctmm import preprocess; ctp = preprocess.softimpute(ctp)
```

## Evaluation signals

- Version pairing check: R 3.6-4.4 runs with rpy2 3.5.x (the PyPI pin rpy2 >= 3.4, < 3.6), and R >= 4.5 runs only with rpy2 >= 3.6
- All five R dependencies installed: abind, mvtnorm, numDeriv, softImpute, tensor (MASS and Matrix ship with R)
- A Python call such as preprocess.softimpute(ctp) returns an imputed matrix rather than raising an rpy2 conversion or R-runtime error
- For the imputation use case, Pearson correlation between imputed and held-out true masked entries falls between 0 and 1, and MSE is non-negative and finite
- When overriding the pin on R >= 4.5, pip prints a dependency-conflict warning ('ctmm 1.1.0 requires rpy2<3.6'); this warning is expected and safe to ignore

## Limitations

- The PyPI ctmm release pins rpy2 >= 3.4, < 3.6, which only supports R <= 4.4; newer R requires a manual rpy2 upgrade that triggers an expected dependency-conflict warning
- pip does not install R itself: a working R installation and the R packages abind, mvtnorm, numDeriv, softImpute, tensor must be provisioned separately
- Only R 3.6-4.4 and R >= 4.5 ranges are documented; intermediate or older R versions are untested in this context

## Evidence

- [methods] CTMM calls R routines through rpy2, so a working R installation is required.: "CTMM calls R routines through [rpy2](https://rpy2.github.io/), so a working R installation is required."
- [readme] The rpy2 pin determines the supported R range.: [evidence span withheld — anchor status: not_found]
- [readme] R packages required by the routines and how to install them.: "install.packages(c("abind", "mvtnorm", "numDeriv", "softImpute", "tensor"))"
- [readme] The dependency-conflict warning when upgrading rpy2 for newer R is expected.: "This is expected and safe to ignore: CTMM works with ``rpy2 >= 3.6``, and the pin is only a lower-bound safeguard for older R."
- [methods] Concrete Python entry point that bridges to R softImpute for CTP imputation.: "# ctp = preprocess.softimpute( ctp )"
