---
name: precision-recall-curve-metric-extraction
description: Use when benchmarking pathway enrichment methods against known true-positive
  pathways (e.g. similarity_sample-activated terms) and you need a recall-fixed precision
  summary under a specified noise level such as η=0.25.
when_to_use_negative:
- Exploratory enrichment analysis (e.g. TCGA breast cancer) where no ground-truth
  true-positive pathway set exists to define recall
- A single enrichment run without multiple benchmark replicates — the median-across-replicates
  aggregation step cannot be performed meaningfully
- Input is already a summarized PR curve or a single precision/recall point rather
  than per-pathway enrichment scores
tools:
- name: pareg
  role: Enrichment method under benchmarking; supplies network-regularized pathway
    enrichment scores and the similarity_sample helper used to define true positives
  repo: https://github.com/cbg-ethz/pareg
- name: tidyverse
  role: Used in R for constructing PR curves, interpolating precision at recall 0.80,
    and summarizing median values across replicates
- name: Snakemake
  role: Runs the benchmark workflows needed to reproduce the precision-at-recall results
  repo: https://github.com/cbg-ethz/pareg
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:8fd75031bc448e74f6fcfb2b78229d7eccaa6d5fa408e7fb229b752002172105
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - Snakemake
  - pareg
  - tidyverse
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# precision-recall-curve-metric-extraction

## Summary

Constructs per-method precision-recall curves from pathway enrichment scores on the synthetic benchmark and extracts the median precision at a fixed recall (e.g. 0.80) as a noise-robustness metric for methods like pareg, MGSA, FET and topGO.

## When to use

Use when benchmarking pathway enrichment methods against known true-positive pathways (e.g. similarity_sample-activated terms) and you need a recall-fixed precision summary under a specified noise level such as η=0.25.

## When NOT to use

- Exploratory enrichment analysis (e.g. TCGA breast cancer) where no ground-truth true-positive pathway set exists to define recall
- A single enrichment run without multiple benchmark replicates — the median-across-replicates aggregation step cannot be performed meaningfully
- Input is already a summarized PR curve or a single precision/recall point rather than per-pathway enrichment scores

## Inputs

- Synthetic pathway database with pronounced pathway clustering
- Simulated differential-expression p-value study (DE genes from Beta(0.1, 1), background from Uniform(0, 1))
- Pathway-level enrichment scores from pareg, MGSA, FET, topGO and blitzGSEA per replicate
- Active-pathway ground-truth labels selected via similarity_sample (Jaccard similarity matrix)

## Outputs

- Per-method precision-recall curves
- Median precision at recall = 0.80 per method (expected: pareg ≈ 94%, MGSA ≈ 37%, FET ≈ 12%, topGO ≈ 5%)
- Summary table of precision-at-recall values across benchmark replicates (Fig 1b/1d)

## How to apply

Run each enrichment method (pareg with network regularization on the Jaccard pathway-similarity matrix, MGSA, FET, topGO, blitzGSEA) on the synthetic benchmark replicate, where DE gene p-values are drawn from Beta(0.1, 1), background genes from Uniform(0, 1), and the active-pathway subset is chosen with similarity_sample at noise level η=0.25 and ρ=0.5. Treat pathways activated by similarity_sample as true positives and build a precision-recall curve per method from the pathway-level enrichment scores. Interpolate precision at recall = 0.80 from each curve rather than reading a single operating point, so the metric is independent of score-threshold choice. Take the median precision-at-80%-recall across benchmark replicates per method to absorb replicate variability, then compile the values into a summary table for comparison against the reported values (pareg 94%, MGSA 37%, FET 12%, topGO 5%; Fig 1b/1d).

## Related tools

- **pareg** (Enrichment method under benchmarking; supplies network-regularized pathway enrichment scores and the similarity_sample helper used to define true positives) — https://github.com/cbg-ethz/pareg
- **tidyverse** (Used in R for constructing PR curves, interpolating precision at recall 0.80, and summarizing median values across replicates)
- **Snakemake** (Runs the benchmark workflows needed to reproduce the precision-at-recall results) — https://github.com/cbg-ethz/pareg

## Examples

```
active_terms <- similarity_sample(mat_similarities, 5)
```

## Evaluation signals

- Median precision at recall 0.8 for pareg reproduces ≈ 94% and for FET ≈ 12%, topGO ≈ 5% under η=0.25, within the benchmark's replicate variability
- PR curves are built with pathways selected by similarity_sample as true positives, not with arbitrary score thresholds
- The metric is interpolated at a fixed recall value (0.80) rather than taken at the default decision threshold
- Values are medians across benchmark replicates, not single-replicate point estimates
- Aggregated results match the published summary (Fig 1b/1d) for all benchmarked methods simultaneously

## Limitations

- The metric depends on the synthetic benchmark design (Beta/Uniform p-value sampling, noise level η=0.25, ρ=0.5); results do not transfer directly to real studies
- Precision-at-recall values carry replicate variability, so agreement is judged within tolerance rather than exactly
- The reported robustness claim holds under the tested noise conditions ('pareg is more robust to noise than competing methods') and does not guarantee superiority at all noise levels

## Evidence

- [methods] True positives defined by similarity_sample-activated pathways: "we select a subset of pathways to be activated. In a performance evaluation, these would be considered to be true positives."
- [methods] Synthetic p-value distributions underlying the benchmark: "The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$. The p-values for all other genes are drawn from a Uniform distribution."
- [abstract] Noise robustness claim the PR metric substantiates: "We show that pareg is more robust to noise than competing methods."
- [readme] Reproducible benchmark workflows in the repository: "Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model."
