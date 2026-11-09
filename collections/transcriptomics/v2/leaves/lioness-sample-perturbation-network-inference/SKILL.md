---
name: lioness-sample-perturbation-network-inference
description: 'Use when evaluating cell-specific network inference (CSNI) methods on
  dyngen simulations: datasets carry transcriptomics plus per-cell ground-truth GRNs,
  and no motif data is available, so LIONESS must run with Pearson correlation rather
  than PANDA.'
when_to_use_negative:
- Motif data is available for the dataset and a PANDA-based LIONESS run is intended
  — dyngen benchmarking here is specifically the Pearson-only fallback because dyngen
  provides no motif data
- The evaluation target is RNA velocity accuracy or trajectory alignment (ABWAP),
  the other dyngen use cases, rather than cell-specific GRN inference
- 'You need method performance on real biological data: this skill''s ground truth
  exists only because dyngen''s synthetic data provides completely known cellwise
  regulatory networks'
edam_operation: http://edamontology.org/operation_3927
tools:
- name: dyngen
  role: Generates the 42 benchmark single-cell datasets with cell-specific ground-truth
    regulatory networks used to score LIONESS per cell
  repo: https://github.com/dynverse/dyngen
- name: lionessR
  role: R implementation of LIONESS run with Pearson correlation to infer the cell-specific
    GRN for every cell (PANDA is inapplicable without motif data)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:9da7e6facb1693aed208dcf0bc9456a151fa098ba1e66afa8fbe057da3959d20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - dyngen
  - lionessR
  edam_operation: http://edamontology.org/operation_3927
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lioness-sample-perturbation-network-inference

## Summary

Benchmark LIONESS, a sample (cell)-specific network inference method, on dyngen's 42 simulated single-cell datasets against their cell-specific ground-truth regulatory networks, scoring each cell with AUROC and AUPR and averaging per dataset. It exists because dyngen provides the cellwise regulatory ground truth that real single-cell data lacks for CSNI evaluation.

## When to use

Use when evaluating cell-specific network inference (CSNI) methods on dyngen simulations: datasets carry transcriptomics plus per-cell ground-truth GRNs, and no motif data is available, so LIONESS must run with Pearson correlation rather than PANDA.

## When NOT to use

- Motif data is available for the dataset and a PANDA-based LIONESS run is intended — dyngen benchmarking here is specifically the Pearson-only fallback because dyngen provides no motif data
- The evaluation target is RNA velocity accuracy or trajectory alignment (ABWAP), the other dyngen use cases, rather than cell-specific GRN inference
- You need method performance on real biological data: this skill's ground truth exists only because dyngen's synthetic data provides completely known cellwise regulatory networks

## Inputs

- 42 dyngen benchmark datasets (14 predefined backbones × 3 seeds), each with a single-cell transcriptomics profile
- Cell-specific ground-truth regulatory networks from dyngen for each dataset
- lionessR implementation of LIONESS with Pearson correlation as the base network inference method

## Outputs

- lioness_pearson_auroc_aupr.csv containing 42 rows with mean AUROC and mean AUPR columns (one row per dyngen dataset)
- Per-cell AUROC and AUPR scores computed against the cell-specific ground-truth GRN (averaged into dataset-level means)

## How to apply

Load the 42 dyngen benchmark datasets, which come from 14 predefined backbones each simulated with three different seeds, and for each dataset pair the transcriptomics profile with the corresponding cell-specific ground-truth regulatory network. Run LIONESS via the lionessR implementation using Pearson correlation as the underlying network inference method — PANDA is inapplicable here because dyngen provides no motif data. For every cell in each dataset, retain the top 10,000 interactions from the inferred cell-specific GRN. Compare these retained interactions against the cell-specific ground-truth GRN by computing AUROC and AUPR per cell, then average across cells to get the mean AUROC and mean AUPR for each of the 42 datasets, following the manuscript's CSNI evaluation scheme. Write the per-dataset mean scores to lioness_pearson_auroc_aupr.csv and use them to compare LIONESS against other CSNI methods such as SSN and pySCENIC.

## Related tools

- **dyngen** (Generates the 42 benchmark single-cell datasets with cell-specific ground-truth regulatory networks used to score LIONESS per cell) — https://github.com/dynverse/dyngen
- **lionessR** (R implementation of LIONESS run with Pearson correlation to infer the cell-specific GRN for every cell (PANDA is inapplicable without motif data))

## Examples

```
install.packages("dyngen"); devtools::install("package/", dependencies = TRUE)  # then run analysis/usecase_network_inference scripts with lionessR + Pearson
```

## Evaluation signals

- The output CSV lioness_pearson_auroc_aupr.csv contains exactly 42 rows, matching the 14 backbones × 3 seeds dataset layout
- Both a mean AUROC and a mean AUPR column are present in the output CSV
- All AUROC and AUPR values fall within the valid [0, 1] score range
- Per-dataset means are correctly computed as the average of per-cell AUROC/AUPR scores over all cells in that dataset
- Top-10,000 interactions are retained per cell before scoring against the cell-specific ground-truth GRN

## Limitations

- LIONESS significantly outperforms SSN on mean AUROC and AUPR, but pySCENIC significantly outperforms LIONESS; LIONESS only achieves a higher AUROC on a few datasets
- PANDA-based LIONESS cannot be used in this benchmark because dyngen provides no motif data, forcing the Pearson-correlation variant
- dyngen assumes cells are well-mixed and independent from each other, so benchmark results may not transfer to real tissues
- No software version numbers were reported for the tools used in the manuscript's analyses

## Evidence

- [results] Mean AUROC/AUPR per dataset: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [results] Motivation for CSNI benchmarking: "a quantitative assessment of their performance is until now lacking"
- [intro] dyngen's role as benchmarking engine: "dyngen is more flexible than current single-cell simulation engines, and allows better method development and benchmarking"
- [readme] Reproducibility via manuscript repo: "All source code for reproducing the results in this manuscript are available on [GitHub](https://github.com/dynverse/dyngen_manuscript)."
