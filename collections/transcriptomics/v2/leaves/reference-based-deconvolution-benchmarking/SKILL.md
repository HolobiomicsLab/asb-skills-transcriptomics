---
name: reference-based-deconvolution-benchmarking
description: Use when simulated ST pixels with known ground-truth cell-type proportions
  are available and you need to quantify how deconvolution accuracy degrades for reference-based
  methods (SPOTlight, RCTD, spatialDWLS) when the scRNA-seq reference is missing cell
  types or mismatched to the tissue.
when_to_use_negative:
- No simulated ST data with ground-truth cell-type proportions exists, so accuracy
  cannot be quantified and only qualitative comparison is possible
- A suitable single-cell transcriptomics reference is available for the tissue — reference-based
  methods are expected to perform comparably to STdeconvolve rather than degrade
- Input is sub-cellular resolution ST data, where the article notes STdeconvolve and
  this benchmarking design are not best suited
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: SPOTlight
  role: Supervised deconvolution of ST pixels using cell-type marker genes from an
    scRNA-seq reference to seed NMF
- name: RCTD
  role: Supervised deconvolution building a probabilistic model from cell-type-specific
    mean expression of marker genes in the scRNA-seq reference
- name: spatialDWLS
  role: Supervised deconvolution enriching pixels for likely cell types via signature
    genes then fitting proportions by DWLS
- name: STdeconvolve
  role: Reference-free LDA-based baseline; its pixel-level proportions serve as the
    comparison target in Diebold–Mariano tests
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Implementation environment for all deconvolution methods, RMSE computation,
    and Diebold–Mariano tests
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs STdeconvolve from Bioconductor for the reference-free baseline
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: remotes
  role: Installs STdeconvolve from GitHub as an alternative to Bioconductor
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:67a74fa0397f9aad944dd15935638fc817643c3c5b05df16f11fd65596a24bb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - RCTD
  - SPOTlight
  - STdeconvolve
  - remotes
  - spatialDWLS
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# reference-based-deconvolution-benchmarking

## Summary

Benchmark supervised, reference-based deconvolution methods (SPOTlight, RCTD, spatialDWLS) against the reference-free STdeconvolve baseline on simulated multi-cellular pixel-resolution ST data, using pixel-level RMSE against known ground-truth proportions and one-sided Diebold–Mariano tests to quantify performance loss when suitable scRNA-seq references are unavailable.

## When to use

Use when simulated ST pixels with known ground-truth cell-type proportions are available and you need to quantify how deconvolution accuracy degrades for reference-based methods (SPOTlight, RCTD, spatialDWLS) when the scRNA-seq reference is missing cell types or mismatched to the tissue.

## When NOT to use

- No simulated ST data with ground-truth cell-type proportions exists, so accuracy cannot be quantified and only qualitative comparison is possible
- A suitable single-cell transcriptomics reference is available for the tissue — reference-based methods are expected to perform comparably to STdeconvolve rather than degrade
- Input is sub-cellular resolution ST data, where the article notes STdeconvolve and this benchmarking design are not best suited

## Inputs

- Simulated multi-cellular pixel-resolution ST dataset (e.g., simulated MPOA) with known ground-truth pixel-level cell-type proportions
- scRNA-seq reference (a): MPOA single-cell reference with excitatory and inhibitory neurons removed
- scRNA-seq reference (b): mouse brain scRNA-seq atlas reference

## Outputs

- Pixel-level cell-type proportion estimates (theta) for SPOTlight, RCTD, spatialDWLS, and STdeconvolve under each reference condition
- RMSE values per method vs. ground-truth simulated proportions
- One-sided Diebold–Mariano test p-values comparing each supervised method against STdeconvolve (expected p < 2.2e-16)
- Summary table (e.g., supervised_deconvolution_rmse_dm.csv) analogous to Fig. 2g, h

## How to apply

Load a simulated multi-cellular pixel-resolution ST dataset (e.g., simulated MPOA) with known ground-truth cell-type proportions. Run the three supervised methods — SPOTlight (NMF seeded with cell-type marker genes), RCTD (probabilistic model on cell-type-specific marker means), and spatialDWLS (signature-gene enrichment followed by Dampened Weighted Least Squares) — twice: once with a scRNA-seq reference missing cell types (excitatory/inhibitory neurons removed) and once with a mismatched reference (mouse brain scRNA-seq atlas). Run STdeconvolve (LDA on significantly overdispersed genes) as the reference-free baseline on the same pixels. Compute RMSE between each method's estimated pixel-level proportions and the simulated ground truth, then perform one-sided Diebold–Mariano tests comparing each supervised method's errors to STdeconvolve's; significantly higher RMSE (p < 2.2e-16) demonstrates that reference quality, not supervision per se, drives performance. Assemble RMSE values and DM p-values into a results table (e.g., supervised_deconvolution_rmse_dm.csv).

## Related tools

- **SPOTlight** (Supervised deconvolution of ST pixels using cell-type marker genes from an scRNA-seq reference to seed NMF)
- **RCTD** (Supervised deconvolution building a probabilistic model from cell-type-specific mean expression of marker genes in the scRNA-seq reference)
- **spatialDWLS** (Supervised deconvolution enriching pixels for likely cell types via signature genes then fitting proportions by DWLS)
- **STdeconvolve** (Reference-free LDA-based baseline; its pixel-level proportions serve as the comparison target in Diebold–Mariano tests) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Implementation environment for all deconvolution methods, RMSE computation, and Diebold–Mariano tests) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs STdeconvolve from Bioconductor for the reference-free baseline) — https://github.com/JEFworks-Lab/STdeconvolve
- **remotes** (Installs STdeconvolve from GitHub as an alternative to Bioconductor) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
require(remotes); remotes::install_github('JEFworks-Lab/STdeconvolve')
```

## Evaluation signals

- All three supervised methods (SPOTlight, RCTD, spatialDWLS) show significantly higher RMSE than STdeconvolve under the neuron-removed and mouse-brain references, with one-sided Diebold–Mariano p < 2.2e-16 (matching Fig. 2g, h)
- STdeconvolve baseline runs on identical pixels as supervised methods, so RMSE values are directly comparable per pixel
- Estimated proportions are non-negative and sum to 1 within each pixel for every method
- Results reproduce the qualitative claim: reference-based methods degrade when the reference lacks cell types (missing neurons) or is tissue-mismatched (mouse brain atlas)
- Output CSV (supervised_deconvolution_rmse_dm.csv) contains one RMSE and DM p-value row per method-per-reference condition

## Limitations

- When a suitable single-cell reference exists, reference-based methods perform comparably to STdeconvolve, so the inflated-RMSE result is specific to unsuitable references
- STdeconvolve baseline depends on LDA assumptions: performance depends on dataset size (number of pixels and genes), and it may fail when cell-type proportions are homogeneous across pixels, data are too sparse with high stochastic drop-out, or thin structures cover only a few pixels
- The comparison requires ground truth; on real ST data without simulation, this RMSE/DM benchmarking design cannot be applied directly

## Evidence

- [full_text] DM p-value threshold: [evidence span withheld — anchor status: not_found]
- [intro] Method reliance on references: "As such, these approaches rely on the availability of a suitable single-cell reference, which may present limitations if such a reference does not exist"
- [intro] Mechanisms of supervised methods: "Of these, SPOTlight3 uses cell-type marker genes derived from a single-cell RNA-sequencing (scRNA-seq) reference to seed a non-negative matrix factorization."
- [readme] STdeconvolve baseline method: "`STdeconvolve` enables reference-free cell-type deconvolution of multi-cellular pixel-resolution spatial transcriptomics data."
- [methods] Installation path for baseline: "remotes::install_github('JEFworks-Lab/STdeconvolve')"
