---
name: haseman-elston-regression-scrna-seq
description: Use when you need a fast, method-of-moments cross-check of cell type-specific
  variance differentiation calls from REML-based LMM fits on scRNA-seq pseudobulk
  data, e.g. to confirm Bonferroni-significant genes across two independent fitting
  methods.
when_to_use_negative:
- Sample sizes below ~50 individuals, where CTMM is not reliable and the Free test
  is slightly inflated
- Rare cell types that cannot provide >10 cells per individual-cell type pair, since
  CTMM is limited to common cell types
- Lowly-expressed genes that are severely non-Gaussian at available cell counts, violating
  the Gaussian approximation underlying the pseudobulk LMM
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: ctmm
  role: Python package providing the ctp.free_HE() function that fits the Free CTMM
    model on CTP inputs using Haseman-Elston regression and returns Wald p-values
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridges ctmm to R routines required for HE fitting; R 3.6-4.4 uses rpy2 3.5.x,
    R >= 4.5 uses rpy2 >= 3.6
  repo: https://rpy2.github.io/
- name: R
  role: Runtime for the statistical routines CTMM calls through rpy2 during HE regression
- name: Snakemake
  role: Snakefile orchestrates the simulation and iPSC analysis steps in which ctp.free_HE()
    is invoked
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:f48b59aa9dc49db32b6cdef126841ae34c2f05eb5b60035034cc09cb839ceae3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - Snakemake
  - ctmm
  - mvtnorm
  - numDeriv
  - rpy2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# haseman-elston-regression-scRNA-seq

## Summary

Fits the Free cell type-specific linear mixed model on cell type-specific pseudobulk (CTP) scRNA-seq data using Haseman-Elston (HE) regression — a method-of-moments alternative to REML — to obtain an independent set of Wald p-values for differentiation of variance (V) and mean expression (ct_beta) between cell types, used to cross-validate REML-based hits.

## When to use

Use when you need a fast, method-of-moments cross-check of cell type-specific variance differentiation calls from REML-based LMM fits on scRNA-seq pseudobulk data, e.g. to confirm Bonferroni-significant genes across two independent fitting methods.

## When NOT to use

- Sample sizes below ~50 individuals, where CTMM is not reliable and the Free test is slightly inflated
- Rare cell types that cannot provide >10 cells per individual-cell type pair, since CTMM is limited to common cell types
- Lowly-expressed genes that are severely non-Gaussian at available cell counts, violating the Gaussian approximation underlying the pseudobulk LMM

## Inputs

- CTP: cell type-specific pseudobulk gene expression matrix (one column per cell type, headerless, e.g. .gz)
- ctnu: variance of measurement noise for each individual-cell type pair (one column per cell type, headerless)
- P: cell type proportions for each individual (one column per cell type, headerless)
- Optionally fixed_covars_d (e.g. PCA on OP) and random_covars_d (e.g. experiment batch) files

## Outputs

- Dictionary of model parameter estimates from the Free model (beta, ct_beta, hom2, V, loglikelihood l)
- Dictionary of Wald test p-values per gene: p['V'] for variance differentiation between cell types and p['ct_beta'] for mean expression differentiation
- Ranked hit table (e.g. ipsc_ctp_hit_table.csv) with gene, REML (JK) p-values, HE p-values, and variance-only flag

## How to apply

Summarize the scRNA-seq data into cell type-specific pseudobulk (CTP), the measurement-noise variance matrix (ctnu), and the cell type proportion matrix (P), after filtering individuals with <=100 cells and individual-cell type pairs with <=10 cells, and standardizing via preprocess.std. Import the ctmm ctp module and call ctp.free_HE() with the CTP, P, and ctnu files to fit the Free model under Haseman-Elston regression; HE is a method-of-moments approach that avoids iterative likelihood optimization, giving an independent second set of Wald p-values per gene for variance differentiation (V) and mean differentiation (ct_beta). Apply Bonferroni correction across all tested genes (e.g. 11,231 genes in the iPSC-to-endoderm analysis) within each method, and retain candidate genes significant in both HE and REML (with jack_knife=True) — this dual-method requirement guards against optimization artifacts. Rank the surviving genes by p-value and flag those whose mean expression test is not significant (p>0.01 in REML), yielding variance-only hits such as the 85 genes reported for the iPSC-to-endoderm differentiation data.

## Related tools

- **ctmm** (Python package providing the ctp.free_HE() function that fits the Free CTMM model on CTP inputs using Haseman-Elston regression and returns Wald p-values) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridges ctmm to R routines required for HE fitting; R 3.6-4.4 uses rpy2 3.5.x, R >= 4.5 uses rpy2 >= 3.6) — https://rpy2.github.io/
- **R** (Runtime for the statistical routines CTMM calls through rpy2 during HE regression)
- **Snakemake** (Snakefile orchestrates the simulation and iPSC analysis steps in which ctp.free_HE() is invoked) — https://github.com/Minhui-Chen/CTMM

## Examples

```
from ctmm import ctp; free_he, p_he = ctp.free_HE(y_f='CTP.gz', P_f='P.gz', ctnu_f='ctnu.gz')
```

## Evaluation signals

- Bonferroni-corrected HE p-values and REML (jack_knife=True) p-values agree on the same set of candidate genes across all tested genes (11,231 in the iPSC analysis)
- Top-ranked candidates include POU5F1, HSPA8, and NDUFB4 with the reported p-values in the ranked hit table
- Count of genes significant for variance (V) but not for mean expression (REML p>0.01) equals 85 for the iPSC-to-endoderm data
- Output contains both expected dictionaries: parameter estimates (including hom2, V, l) and Wald p-value entries p['V'] and p['ct_beta']
- Input files follow the headerless format: one column per cell type for CTP, ctnu, and P

## Limitations

- HE p-values from the moment-based fit can differ from likelihood-based fits; the workflow requires concordance with jackknife-based REML rather than trusting HE alone
- CTMM (including the HE route) is not reliable for sample sizes below ~50, and the Free test is slightly inflated, so results require biological validation and replication
- Requiring >10 cells per individual-cell type pair limits the analysis to common cell types, and pseudobulk aggregation discards cell-level information, sacrificing statistical power within cell types
- Inferring cell types directly from the scRNA-seq data (as in the PBMC analysis) induces circularity that deflates cell type-specific variance estimates

## Evidence

- [readme] HE is a method-of-moments approach; ctp.free_HE() provides an independent second set of Wald p-values: "with **ML** (maximum likelihood), **REML** (restricted maximum likelihood), and **HE** (Haseman-Elston regression, a method-of-moments) methods."
- [readme] ctp module usage example: "To fit CTP, import the ``ctp`` module and call the function in the same way. For example, ``ctp.full_ML()`` to fit Full model using ML."
- [readme] Wald tests target V and ct_beta differentiation: "The second one contains p values from Wald test on e.g. differentiation of mean expression between cell types (ct_beta) and differentiation of expression variance between cell types (V)."
- [results] Candidate genes required significance in both REML (JK) and HE after Bonferroni correction: [evidence span withheld — anchor status: not_found]
- [abstract] 85 genes lack significant mean differentiation: "including 85 genes that do not have significant stage-specific mean expression"
- [discussion] Sample size reliability caveat: "CTMM is not reliable for sample sizes below ~50"
- [methods] Cell count filter thresholds: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
