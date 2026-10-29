---
name: regulated-heterogeneity-z-score-computation
description: Use when matched single-cell and bulk RNA-Seq expression matrices are
  available and the goal is to test whether a bulk population's signalling entropy
  is significantly higher than expected from its constituent single cells.
when_to_use_negative:
- No matched single-cell data exist for the bulk sample, so no single-cell SR reference
  distribution can be built
- The goal is ordering cells in pseudo-time or ranking differentiation potency of
  individual cells rather than testing bulk-vs-single-cell entropy deviation
- Genes cannot be mapped onto the interaction network required by SCENT's entropy
  computation
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCENT
  role: Computes the network-based signalling entropy (SR) of single cells and bulk
    samples used to build the reference distribution and derive MRH z-scores
  repo: https://github.com/aet21/SCENT
- name: R
  role: Environment in which SCENT is run and in which matched expression matrices
    are loaded and z-scores/P-values are computed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:379feba0a1a216090f275c45b8f269ec49cc9f005b0b59e94a2c9e9b40adbd90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# regulated-heterogeneity-z-score-computation

## Summary

Computes a measure of regulated heterogeneity (MRH) z-score by standardizing the signalling entropy (SR) of a matched bulk RNA-Seq sample against the single-cell SCENT SR distribution, testing whether bulk entropy exceeds what one random single cell would give — evidence of regulated single-cell expression heterogeneity.

## When to use

Use when matched single-cell and bulk RNA-Seq expression matrices are available and the goal is to test whether a bulk population's signalling entropy is significantly higher than expected from its constituent single cells.

## When NOT to use

- No matched single-cell data exist for the bulk sample, so no single-cell SR reference distribution can be built
- The goal is ordering cells in pseudo-time or ranking differentiation potency of individual cells rather than testing bulk-vs-single-cell entropy deviation
- Genes cannot be mapped onto the interaction network required by SCENT's entropy computation

## Inputs

- Matched single-cell RNA-Seq expression matrix (gene x cell)
- Matched bulk RNA-Seq expression matrix (gene x sample)
- Protein interaction network compatible with SCENT entropy estimation

## Outputs

- Single-cell SR reference distribution per population
- Bulk-sample SR values and MRH z-scores
- Empirical/permutation P-values per bulk sample (and combined Fisher P-values)
- mrh_zscore_results.csv with all z-scores, P-values, and control values

## How to apply

Load matched single-cell and bulk expression matrices in R and map genes onto the protein interaction network required by SCENT. Compute the network-based signalling entropy (SR) of every individual cell with SCENT to build the reference single-cell SR distribution, then compute SR for each matched bulk sample. Derive the MRH z-score as the standardized deviation of the bulk SR from the single-cell SR distribution, and assign an empirical/permutation P-value to each bulk-sample z-score (e.g., per cell type such as melanoma T cells, B cells, and cancer cells, combining across patients with Fisher's test). Run negative controls to validate: a bulk-average profile should be non-significant (P approximately 0.30), and a true heterogeneous population (e.g., AML, population SR=0.88) should exceed the maximum single-cell SR (0.82). Significant melanoma cancer-cell MRH z-scores (combined Fisher P<1e-6 across 12 patients) indicate regulated expression heterogeneity.

## Related tools

- **SCENT** (Computes the network-based signalling entropy (SR) of single cells and bulk samples used to build the reference distribution and derive MRH z-scores) — https://github.com/aet21/SCENT
- **R** (Environment in which SCENT is run and in which matched expression matrices are loaded and z-scores/P-values are computed)

## Examples

```
devtools::install_github("aet21/SCENT"); library(SCENT)
```

## Evaluation signals

- Melanoma cancer-cell bulk samples yield significant MRH z-scores (combined Fisher test P<1e-6 across 12 patients)
- Bulk-average negative control is non-significant (P approximately 0.30)
- AML heterogeneous population SR (0.88) exceeds the maximum single-cell SR (0.82)
- Bulk entropies are much higher than expected from the single-cell SR distributions in T-cell and B-cell populations
- All z-scores, P-values, and control values are written to mrh_zscore_results.csv

## Limitations

- No existing assay can measure all single cells in a population, limiting direct validation of bulk-vs-single-cell entropy synergy
- The precise relation between single-cell signalling entropy and transcriptional entropy as defined by other measures is non-trivial and not established
- Genes must map onto the interaction network; SCENT's entropy requires network context, unlike transcriptomic entropy measures such as StemID or SLICE

## Evidence

- [abstract] MRH z-score as standardized bulk-vs-single-cell deviation: "It further reveals that expression heterogeneity within single-cell populations is regulated."
- [abstract] Entropy computed in context of an interaction network: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network"
- [abstract] SCENT estimates entropy for single cells and bulk samples: "signalling entropy allows in silico estimation of the differentiation potency and plasticity of single cells and bulk samples"
- [discussion] Population-level statistic from distribution over single cells: "we can assign single cells into potency states, from which a SI over the whole cell population (that is, using the distribution of potency states over single cells)"
- [readme] SCENT package installation: "devtools::install_github("aet21/SCENT")"
