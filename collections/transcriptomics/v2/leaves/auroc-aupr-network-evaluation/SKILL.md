---
name: auroc-aupr-network-evaluation
description: Use when you need to score predicted cell-specific regulatory networks
  against dyngen ground-truth GRNs, e.g. benchmarking SSN, LIONESS, or pySCENIC on
  the 42-dataset dyngen CSNI benchmark.
when_to_use_negative:
- You lack cell-specific ground-truth regulatory networks — AUROC/AUPR cannot be computed
  without known per-cell GRN edges as dyngen provides
- You are evaluating bulk network inference or non-network tasks (e.g. RNA velocity
  or trajectory alignment), which the manuscript evaluates with different protocols
- You need runtime or robustness benchmarks rather than network accuracy against ground
  truth
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: dyngen
  role: Generates the 42 benchmark datasets (14 backbones × 3 seeds) with per-cell
    transcriptomics and cell-specific ground-truth regulatory networks used as evaluation
    targets
  repo: https://github.com/dynverse/dyngen
- name: tidyverse
  role: Used in R together with basic R functions to implement the SSN leave-one-out
    Pearson-correlation differencing and per-cell scoring
- name: dyngen_manuscript
  role: Contains analysis/usecase_network_inference scripts reproducing the cell-specific
    network inference evaluation
  repo: https://github.com/dynverse/dyngen_manuscript
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:eb3555211be9cf6c6ae6698ae0af1c838ca8bcff516a211367a521d2da355a19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - R
  - dyngen
  - dyngen_manuscript
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# auroc-aupr-network-evaluation

## Summary

Quantitatively evaluates predicted cell-specific gene regulatory networks (CSNI methods such as SSN) against dyngen's cell-specific ground-truth GRNs by computing per-cell AUROC and AUPR over the top-scored interactions and averaging them per dataset.

## When to use

Use when you need to score predicted cell-specific regulatory networks against dyngen ground-truth GRNs, e.g. benchmarking SSN, LIONESS, or pySCENIC on the 42-dataset dyngen CSNI benchmark.

## When NOT to use

- You lack cell-specific ground-truth regulatory networks — AUROC/AUPR cannot be computed without known per-cell GRN edges as dyngen provides
- You are evaluating bulk network inference or non-network tasks (e.g. RNA velocity or trajectory alignment), which the manuscript evaluates with different protocols
- You need runtime or robustness benchmarks rather than network accuracy against ground truth

## Inputs

- 42 dyngen benchmark datasets with per-cell transcriptomics profiles
- Cell-specific ground-truth regulatory networks for each dyngen dataset
- Per-cell predicted network scores from the CSNI method under test (e.g. SSN, LIONESS, pySCENIC)
- R scripts from dynverse/dyngen_manuscript analysis/usecase_network_inference

## Outputs

- Per-cell AUROC and AUPR scores over the top 10,000 interactions
- Per-dataset mean AUROC and mean AUPR for each method condition (e.g. SSN)
- A scores table with one entry per dataset per method condition

## How to apply

Load each of the 42 dyngen datasets (14 backbones × 3 seeds), each providing per-cell transcriptomics profiles and corresponding cell-specific ground-truth regulatory networks. Run the inference method per cell (for SSN in R, use a leave-one-out differencing approach restricted to Pearson correlation, implemented with basic R and tidyverse functions, mirroring LIONESS). For each cell, retain only the top 10,000 predicted interactions by score, then compare against the ground-truth cell-specific GRN to compute AUROC and AUPR. Average per-cell AUROC and AUPR within each dataset to obtain mean AUROC and mean AUPR for the method condition, following the manuscript's CSNI evaluation protocol.

## Related tools

- **dyngen** (Generates the 42 benchmark datasets (14 backbones × 3 seeds) with per-cell transcriptomics and cell-specific ground-truth regulatory networks used as evaluation targets) — https://github.com/dynverse/dyngen
- **tidyverse** (Used in R together with basic R functions to implement the SSN leave-one-out Pearson-correlation differencing and per-cell scoring)
- **dyngen_manuscript** (Contains analysis/usecase_network_inference scripts reproducing the cell-specific network inference evaluation) — https://github.com/dynverse/dyngen_manuscript

## Examples

```
git clone https://github.com/dynverse/dyngen_manuscript && cd dyngen_manuscript && devtools::install("package/", dependencies = TRUE)
```

## Evaluation signals

- Each of the 42 datasets yields a mean AUROC and mean AUPR value in [0, 1]
- The resulting scores table contains an entry for every dataset in the method condition (e.g. SSN)
- Per-cell scoring is computed over exactly the top 10,000 predicted interactions by score
- Per-dataset means equal the average of per-cell AUROC/AUPR scores over all individual cells
- Comparison against baselines: SSN should score worse than LIONESS and pySCENIC on the dyngen CSNI benchmark

## Limitations

- SSN performs poorly on the dyngen CSNI benchmark: subtracting one cell from the whole dataset yields correlation differences that are mostly noise, performing worse than LIONESS and pySCENIC
- dyngen assumes cells are well-mixed and independent from each other, so evaluations reflect simulated conditions only
- No software version numbers were reported for any tool, complicating exact reproducibility

## Evidence

- [results] Mean AUROC/AUPR computed across individual cells: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [full_text] SSN's poor CSNI performance rationale: [evidence span withheld — anchor status: not_found]
- [full_text] Top-k interaction cutoff: [evidence span withheld — anchor status: not_found]
- [readme] Reproducible analysis location in manuscript repo: [evidence span withheld — anchor status: external_doc]
