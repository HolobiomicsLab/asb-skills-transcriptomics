---
name: diebold-mariano-model-comparison-testing
description: Use when two or more deconvolution/clustering methods have been run on
  the same pixels with known ground-truth cell-type proportions and you must test
  whether one method's per-pixel RMSE is significantly lower than another's.
when_to_use_negative:
- Deconvolved results are not comparable per pixel (methods run on different datasets
  or different pixel subsets), so paired error series cannot be formed
- No ground-truth cell-type proportions exist, so RMSE-based error series cannot be
  computed (e.g., purely exploratory real-tissue analyses without simulation)
- Input is single-cell-resolution or sub-cellular pixel-resolution ST data, which
  STdeconvolve-based benchmarks in this paper do not target
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution baseline whose per-pixel error series
    is compared against supervised methods using the Diebold–Mariano test
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: SPOTlight
  role: Supervised reference-based deconvolution method whose RMSE is DM-tested against
    STdeconvolve under unsuitable scRNA-seq references
- name: RCTD
  role: Supervised reference-based deconvolution method DM-tested against STdeconvolve
    in the missing-neurons and mouse-brain-atlas reference scenarios
- name: spatialDWLS
  role: Supervised reference-based deconvolution method DM-tested against STdeconvolve
    in the unsuitable-reference benchmark
- name: BayesSpace
  role: Resolution-enhanced clustering baseline whose RMSE is DM-tested against STdeconvolve
    on simulated 3-cell-type and MERFISH mouse brain data
- name: R
  role: Environment for running the deconvolution methods, computing per-pixel RMSE,
    performing the DM tests, and writing the results CSVs
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:71337b7929b9825ab18cbd327c259051175a042f9000ee909683c39437098eb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BayesSpace
  - BiocManager
  - R
  - RCTD
  - SPOTlight
  - STdeconvolve
  - remotes
  - spatialDWLS
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# diebold-mariano-model-comparison-testing

## Summary

Applies one-sided Diebold–Mariano tests to per-pixel error series (e.g., RMSE of deconvolved cell-type proportions from STdeconvolve vs. supervised methods like SPOTlight, RCTD, spatialDWLS, BayesSpace, and transcriptional clustering) to assess whether differences in deconvolution accuracy on simulated multi-cellular pixel-resolution ST data are statistically significant.

## When to use

Use when two or more deconvolution/clustering methods have been run on the same pixels with known ground-truth cell-type proportions and you must test whether one method's per-pixel RMSE is significantly lower than another's.

## When NOT to use

- Deconvolved results are not comparable per pixel (methods run on different datasets or different pixel subsets), so paired error series cannot be formed
- No ground-truth cell-type proportions exist, so RMSE-based error series cannot be computed (e.g., purely exploratory real-tissue analyses without simulation)
- Input is single-cell-resolution or sub-cellular pixel-resolution ST data, which STdeconvolve-based benchmarks in this paper do not target

## Inputs

- Per-pixel estimated cell-type proportion matrices (theta) from each compared method on the same pixels
- Ground-truth simulated pixel-level cell-type proportions for the MPOA, 3-cell-type, or MERFISH mouse brain (thalamus, n=716 pixels) datasets
- Method labels for supervised baselines (SPOTlight, RCTD, spatialDWLS, BayesSpace, transcriptional clustering) and the STdeconvolve baseline

## Outputs

- Per-method RMSE values between estimated and ground-truth proportions
- One-sided Diebold–Mariano test statistics and p-values comparing each supervised method against STdeconvolve (expected p < 2.2e-16)
- Summary CSV such as supervised_deconvolution_rmse_dm.csv or fig3_rmse_comparison.csv, plus figure panels (Fig. 2g,h; Fig. 3h,l)

## How to apply

For each method, compute per-pixel errors (RMSE) between estimated pixel-level cell-type proportions and the ground-truth simulated proportions, yielding paired error series over the same pixels. Run a one-sided Diebold–Mariano test of each supervised method's error series against the reference-free baseline STdeconvolve's errors, as done for SPOTlight, RCTD, and spatialDWLS under unsuitable scRNA-seq references (neurons removed, mouse brain atlas) and for BayesSpace and transcriptional clustering on the simulated 3-cell-type and 100 µm² MERFISH mouse brain data. The paper's expected outcome is highly significant p-values (p < 2.2e-16) in favor of STdeconvolve; assemble the RMSE values and DM p-values into a CSV (e.g., supervised_deconvolution_rmse_dm.csv, fig3_rmse_comparison.csv) and report alongside figure panels (Fig. 2g,h; Fig. 3h,l). The paired nature of the per-pixel errors on identical data is what justifies the DM test over unpaired comparisons.

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution baseline whose per-pixel error series is compared against supervised methods using the Diebold–Mariano test) — https://github.com/JEFworks-Lab/STdeconvolve
- **SPOTlight** (Supervised reference-based deconvolution method whose RMSE is DM-tested against STdeconvolve under unsuitable scRNA-seq references)
- **RCTD** (Supervised reference-based deconvolution method DM-tested against STdeconvolve in the missing-neurons and mouse-brain-atlas reference scenarios)
- **spatialDWLS** (Supervised reference-based deconvolution method DM-tested against STdeconvolve in the unsuitable-reference benchmark)
- **BayesSpace** (Resolution-enhanced clustering baseline whose RMSE is DM-tested against STdeconvolve on simulated 3-cell-type and MERFISH mouse brain data)
- **R** (Environment for running the deconvolution methods, computing per-pixel RMSE, performing the DM tests, and writing the results CSVs)

## Evaluation signals

- Per-pixel error series are paired: each method is evaluated on the identical set of simulated pixels with the same ground-truth proportions
- One-sided DM comparisons of supervised methods against STdeconvolve under unsuitable references reproduce p < 2.2e-16 (Fig. 2g,h)
- On simulated 3-cell-type and MERFISH thalamus data (n=716 pixels), STdeconvolve attains the lowest RMSE among the three compared approaches (Fig. 3h,l)
- Output CSV contains RMSE values plus DM statistics/p-values per method-dataset pair and is traceable to specific figure panels
- Reproduced panels match the published figures in ranking: STdeconvolve < clustering/BayesSpace and STdeconvolve < supervised methods with unsuitable references

## Limitations

- DM significance reflects the simulated benchmark's ground truth; on real tissue without ground truth, RMSE and DM tests cannot be computed
- Conclusions are specific to multi-cellular pixel-resolution ST; STdeconvolve may not be best suited for sub-cellular resolution data
- STdeconvolve (and thus the baseline being defended by DM tests) can fail when cell-type proportions are homogeneous across pixels, data are too sparse with high stochastic drop-out, or tissue slivers cover only a few pixels

## Evidence

- [results] Unsuitable-reference DM significance: [evidence span withheld — anchor status: not_found]
- [abstract] Comparable performance claim: "STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available, as well as potentially superior performance when suitable"
- [readme] README workflow for deconvolution outputs: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)
deconProp <- results$theta
deconGexp <- results$beta"
- [intro] Feature selection step feeding the compared baseline: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
