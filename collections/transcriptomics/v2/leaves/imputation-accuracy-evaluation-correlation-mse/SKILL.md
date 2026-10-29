---
name: imputation-accuracy-evaluation-correlation-mse
description: Decision support for choosing an imputation method to fill missing ctp/ctnu
  entries (individual-cell type pairs with <=10 cells, set to missing via preprocess.pseudobulk
  with ct_cut=10) before fitting CTMM to pseudobulk data.
when_to_use_negative:
- The data has no missing ctp/ctnu entries (e.g. all individual-cell type pairs exceed
  the 10-cell threshold), so no imputation or accuracy evaluation is needed
- The goal is imputing raw single-cell counts or per-cell expression rather than individual-level
  pseudobulk (ctp) and noise variance (ctnu) matrices used as CTMM input
tools:
- name: ctmm
  role: Python package providing preprocess.pseudobulk (with ind_cut=100, ct_cut=10)
    and preprocess.softimpute used to generate, mask, and impute the ctp/ctnu matrices
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: softImpute
  role: R matrix-completion program wrapped by preprocess.softimpute to fill missing
    ctp/ctnu entries; chosen after benchmarking against alternatives (e.g. MVN)
- name: rpy2
  role: Python-R bridge through which CTMM calls the R softImpute routines
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required for the softImpute routine invoked via rpy2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:ad134a62438851800d9f89633a53114457d40c7ebb71a34fdadde23cbd41ea1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - ctmm
  - rpy2
  - softImpute
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# imputation-accuracy-evaluation-correlation-mse

## Summary

Benchmark matrix-completion imputation of cell type-specific pseudobulk (ctp) and cell type-specific noise variance (ctnu) matrices by holding out masked entries as ground truth and scoring transcriptome-wide softImpute against alternatives using Pearson correlation and mean squared error. This establishes that transcriptome-wide softImpute is the imputation approach used to fill individual-cell type pair entries with <=10 cells before CTMM fitting.

## When to use

Decision support for choosing an imputation method to fill missing ctp/ctnu entries (individual-cell type pairs with <=10 cells, set to missing via preprocess.pseudobulk with ct_cut=10) before fitting CTMM to pseudobulk data.

## When NOT to use

- The data has no missing ctp/ctnu entries (e.g. all individual-cell type pairs exceed the 10-cell threshold), so no imputation or accuracy evaluation is needed
- The goal is imputing raw single-cell counts or per-cell expression rather than individual-level pseudobulk (ctp) and noise variance (ctnu) matrices used as CTMM input

## Inputs

- ctp: cell type-specific pseudobulk expression matrix (individuals x cell types) with entries masked as missing for individual-cell type pairs with <=10 cells
- ctnu: cell type-specific noise variance matrix (individuals x cell types) masked by the same ct_cut=10 rule
- Gene expression filter: restriction to genes expressed in all retained cell types

## Outputs

- Imputed ctp and ctnu matrices ready for CTMM model fitting
- imputation_metrics.csv summarizing Pearson correlation and mean squared error between imputed and held-out true masked entries

## How to apply

Load the ctp and ctnu matrices produced by preprocess.pseudobulk with ind_cut=100 and ct_cut=10, so entries are missing for individual-cell type pairs with <=10 cells. Restrict entries to genes expressed in all retained cell types before imputation, and hold out the masked entries as ground truth. Run matrix completion with preprocess.softimpute, which wraps the R softImpute program called from Python via rpy2 (softImpute is installed via install.packages with abind, mvtnorm, numDeriv, and tensor). Compare imputed versus held-out true values across all genes using Pearson correlation and mean squared error, mirroring Supplementary Fig. 12, including MVN as a comparison approach. The comparison showed transcriptome-wide softImpute performed best with MVN performing similarly, which justified using transcriptome-wide softImpute in practice before CTMM model fitting.

## Related tools

- **ctmm** (Python package providing preprocess.pseudobulk (with ind_cut=100, ct_cut=10) and preprocess.softimpute used to generate, mask, and impute the ctp/ctnu matrices) — https://github.com/Minhui-Chen/ctmm_python
- **softImpute** (R matrix-completion program wrapped by preprocess.softimpute to fill missing ctp/ctnu entries; chosen after benchmarking against alternatives (e.g. MVN))
- **rpy2** (Python-R bridge through which CTMM calls the R softImpute routines) — https://rpy2.github.io/
- **R** (Runtime required for the softImpute routine invoked via rpy2)

## Examples

```
from ctmm import preprocess; ctp = preprocess.softimpute( ctp )
```

## Evaluation signals

- Pearson correlation between imputed and held-out true masked entries across all genes is a value between 0 and 1
- Mean squared error between imputed and true masked entries is non-negative and finite
- The correlation/MSE summary is written to imputation_metrics.csv
- Transcriptome-wide softImpute should outperform (or match) the MVN comparison on both correlation and MSE, consistent with Supplementary Fig. 12
- Only genes expressed in all retained cell types are included in the imputation and evaluation

## Limitations

- Missing ctp/ctnu entries arise because CTMM requires >10 cells per individual-cell type pair, which limits CTMM to common cell types
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation and complicating noise variance estimation for sparse pairs
- The iid comparison model's code is not thoroughly tested; MVN imputation performs similarly to softImpute, so the choice is not sharply decisive

## Evidence

- [methods] softImpute wrapper invocation: "# ctp = preprocess.softimpute( ctp )"
- [methods] masking rule: "set ctp and ctnu to missing for individual-cell type pairs with <=10 cells"
- [readme] R dependency install: "install.packages(c("abind", "mvtnorm", "numDeriv", "softImpute", "tensor"))"
- [methods] softImpute as matrix completion method: "using the program softImpute"
- [full_text] workflow step from source card: [evidence span withheld — anchor status: not_found]
