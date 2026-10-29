---
name: single-cell-signalling-entropy-estimation
description: Use when scRNA-Seq expression matrices need potency ordering of single
  cells (e.g. pluripotent vs multipotent vs differentiated, developmental stage ordering,
  cancer-vs-microenvironment contrast) without prior marker or timepoint knowledge.
when_to_use_negative:
- When the goal is pseudo-temporal trajectory inference from well-established differentiation
  hierarchies where algorithms such as Monocle or MPath are the intended choice
- When only transcriptomic entropies without network information (as used in StemID
  or SLICE) are required, since SCENT specifically relies on the interaction network
  context
- When prior marker expression or timepoint information is already available and sufficient,
  as SCENT is designed for settings lacking such biological knowledge
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCENT
  role: R package implementing the signalling entropy (SR) algorithm on a PPI network
    to estimate differentiation potency of single cells and bulk samples
  repo: https://github.com/aet21/SCENT
- name: CCAT
  role: Companion tool used in the SCENT tutorial for estimating differentiation potency
    of single cells
- name: R
  role: Runtime environment for SCENT; used to load expression matrices, run SignallingEntropy,
    perform Wilcoxon rank-sum and permutation tests, and write result CSVs
- name: BiocStyle
  role: Bioconductor document style used in the SCENT/CCAT tutorial vignette
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:376b141bdb11da04375ad325e34f0599778a6f22ca0bd72c4afc591b4bc5e9ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-signalling-entropy-estimation

## Summary

Estimate the differentiation potency of individual single-cell RNA-Seq profiles by computing signalling entropy (SR) — the signalling promiscuity of a cell's transcriptome in the context of a protein interaction network — using the SCENT algorithm, without feature selection or training.

## When to use

Use when scRNA-Seq expression matrices need potency ordering of single cells (e.g. pluripotent vs multipotent vs differentiated, developmental stage ordering, cancer-vs-microenvironment contrast) without prior marker or timepoint knowledge.

## When NOT to use

- When the goal is pseudo-temporal trajectory inference from well-established differentiation hierarchies where algorithms such as Monocle or MPath are the intended choice
- When only transcriptomic entropies without network information (as used in StemID or SLICE) are required, since SCENT specifically relies on the interaction network context
- When prior marker expression or timepoint information is already available and sufficient, as SCENT is designed for settings lacking such biological knowledge

## Inputs

- scRNA-Seq expression/count matrix (e.g. Tirosh melanoma: 3,256 non-malignant and 1,257 cancer cells across 12 patients; Chu and Treutlein datasets)
- Cell-type / malignant-vs-non-malignant annotations (optional, for contrast tests)
- Protein interaction (PPI) network required by SCENT for entropy estimation
- Matched bulk RNA-Seq expression matrices (for MRH bulk-vs-single-cell analysis)

## Outputs

- Per-cell signalling entropy (SR) scores
- Per-cell-type summary tables of mean SR (e.g. tirosh_sr_scores.csv)
- Wilcoxon rank-sum P-values contrasting SR between cell populations (e.g. P<1e-500 for cancer vs non-malignant)
- Permutation null distributions and summary tables (e.g. permutation_significance_results.csv)
- MRH z-scores and empirical/permutation P-values for bulk samples (e.g. mrh_zscore_results.csv)

## How to apply

Load the scRNA-Seq expression matrix (count matrix) and a protein interaction network into R and install the SCENT package from GitHub (aet21/SCENT). Run SCENT's network-based SignallingEntropy routine to compute the SR score for every cell in the context of the PPI network, explicitly without feature selection or training. SR is high in pluripotent/stem-like cells and decreases with developmental stage, so rank cells by SR to order them by differentiation potency, or use rank-sum tests (e.g. Wilcoxon) to contrast SR distributions between cell types (e.g. cancer cells vs ECs, CAFs, T cells, B cells, macrophages, NK cells). For null testing, generate network-constrained permutations by randomly reshuffling gene expression values over the PPI network while preserving network structure (e.g. 100 permutations for the Chu set, 1,000 for the Treutlein set) and verify that discrimination power is lost, confirming dependence on network-constrained expression patterns. Optionally extend to bulk samples via the MRH z-score: standardize each bulk SR against the single-cell SR distribution and assess significance with empirical/permutation P-values. Write per-cell SR scores and per-cell-type summary tables (e.g. permutation_significance_results.csv, tirosh_sr_scores.csv) as outputs.

## Related tools

- **SCENT** (R package implementing the signalling entropy (SR) algorithm on a PPI network to estimate differentiation potency of single cells and bulk samples) — https://github.com/aet21/SCENT
- **CCAT** (Companion tool used in the SCENT tutorial for estimating differentiation potency of single cells)
- **R** (Runtime environment for SCENT; used to load expression matrices, run SignallingEntropy, perform Wilcoxon rank-sum and permutation tests, and write result CSVs)
- **BiocStyle** (Bioconductor document style used in the SCENT/CCAT tutorial vignette)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- SR loses power to discriminate pluripotent from multipotent/differentiated cells (Chu set) and loses its consistent decrease with developmental stage (Treutlein set) under 100–1,000 network-constrained expression reshuffling permutations, confirming the finding depends on network-constrained expression patterns
- Cancer-vs-non-malignant contrast reproduces the dramatic entropy increase (Wilcoxon rank-sum P<1e-500 over 4,513 Tirosh melanoma cells), with non-malignant cell types rank-ordered by mean SR in a potency-graded ordering with ECs and CAFs at the top
- MRH negative controls pass: bulk-average profile is non-significant (P ≈ 0.30) and the AML population SR (0.88) exceeds the maximum single-cell SR (0.82), while melanoma bulk samples are significant (combined Fisher test P<1e-6)
- All genes are mapped onto the protein interaction network before entropy estimation, and no feature selection or training step is applied
- Output CSVs contain per-cell SR values plus per-cell-type summaries, z-scores, and P-values with expected schema

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- No existing assay can measure all single cells in a population, limiting validation of the bulk-vs-single-cell synergy
- Criticality hallmarks (e.g. an SI peak before a cell-fate transition) remain untested, pending upcoming high-resolution time course and genome-wide scRNA-Seq data

## Evidence

- [readme] SCENT package purpose: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [abstract] Abstract network-entropy claim: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network, without the need"
- [discussion] No feature selection or training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [methods] Tutorial title (methods): "A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT"
- [readme] Installation command (readme): "devtools::install_github("aet21/SCENT")"
