---
name: cell-specific-regulatory-network-inference
description: Use when evaluating or comparing cell-specific network inference algorithms
  on single-cell expression data where per-cell ground-truth regulatory networks (or
  a simulator such as dyngen that provides them) are available for AUROC/AUPR scoring.
when_to_use_negative:
- No per-cell ground truth exists and no simulator (dyngen or similar) can provide
  it — the method requires cellwise GRN annotations for AUROC/AUPR scoring
- The underlying base method requires motif/prior data that is unavailable (e.g. PANDA
  is inapplicable here because dyngen provides no motif data)
- Input is bulk or aggregated expression without single-cell resolution, since leave-one-out
  cell-specific differencing is meaningless
edam_operation: http://edamontology.org/operation_2492
edam_topics:
- http://edamontology.org/topic_3794
tools:
- name: dyngen
  role: Simulates the 42 benchmark datasets (Gillespie SSA over a generated GRN) with
    cell-specific ground-truth regulatory networks used for scoring
  repo: https://github.com/dynverse/dyngen
- name: lionessR
  role: Runs LIONESS with Pearson correlation as the base network inference method
    on each dataset to produce cell-specific GRNs
- name: tidyverse
  role: Used with basic R to implement the SSN leave-one-out Pearson differencing
    approach and to aggregate per-cell scores
- name: pySCENIC
  role: Third CSNI condition included in the benchmark comparison; significantly outperforms
    LIONESS and SSN on mean AUROC and AUPR
- name: dyngen_manuscript
  role: Repository containing the reproduction code for the network-inference use
    case (analysis/usecase_network_inference)
  repo: https://github.com/dynverse/dyngen_manuscript
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:f48b420d62e806bbdd75d12000183b99707beed5c0495c0a20a3b6bfe6675f08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - R
  - dyngen
  - dyngen_manuscript
  - lionessR
  - pySCENIC
  - tidyverse
  edam_operation: http://edamontology.org/operation_2492
  edam_topics:
  - http://edamontology.org/topic_3794
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cell-specific-regulatory-network-inference

## Summary

Benchmark cell-specific gene regulatory network inference (CSNI) methods — SSN, LIONESS + Pearson, and pySCENIC — against dyngen's cellwise ground-truth GRNs, scoring each method by mean AUROC and mean AUPR averaged over individual cells across 42 simulated datasets (14 backbones × 3 seeds).

## When to use

Use when evaluating or comparing cell-specific network inference algorithms on single-cell expression data where per-cell ground-truth regulatory networks (or a simulator such as dyngen that provides them) are available for AUROC/AUPR scoring.

## When NOT to use

- No per-cell ground truth exists and no simulator (dyngen or similar) can provide it — the method requires cellwise GRN annotations for AUROC/AUPR scoring
- The underlying base method requires motif/prior data that is unavailable (e.g. PANDA is inapplicable here because dyngen provides no motif data)
- Input is bulk or aggregated expression without single-cell resolution, since leave-one-out cell-specific differencing is meaningless

## Inputs

- 42 dyngen simulated datasets (14 backbones × 3 seeds), each with a per-cell transcriptomics count matrix
- Cell-specific ground-truth regulatory networks per dataset with regulatory effects bounded in [-1, 1]
- Normalised expression data as provided by dyngen, with no extra cell or feature filtering

## Outputs

- Per-dataset mean AUROC and mean AUPR tables over the 42 datasets for each CSNI condition (e.g. lioness_pearson_auroc_aupr.csv with 42 rows and both score columns)
- Per-cell AUROC and AUPR scores computed over the top 10,000 retained interactions
- Ranked method comparison (SSN vs LIONESS+Pearson vs pySCENIC) supporting significance statements

## How to apply

Generate or load the 42 dyngen benchmark datasets, each built from one of 14 predefined backbones (linear, bifurcating, cyclic, converging, disconnected, etc.) across three seeds, containing a transcriptomics count matrix plus cell-specific ground-truth regulatory networks with regulatory effects bounded in [-1, 1]. Run each CSNI method on every dataset: implement SSN in R with basic R and tidyverse as a leave-one-out Pearson-correlation differencing approach (one cell subtracted from the whole dataset), run LIONESS via lionessR with Pearson correlation as the base method (PANDA is inapplicable since dyngen provides no motif data), and run pySCENIC for comparison. For every cell, retain the top 10,000 predicted interactions by score, then compute AUROC and AUPR of the retained predictions against the cell-specific ground-truth GRN. Average the per-cell scores within each dataset to obtain mean AUROC and mean AUPR per dataset (42 values per method), as the article's CSNI evaluation prescribes. Interpretation decision point: in the paper's benchmark, SSN performs poorly because its correlation differences are mostly noise, LIONESS significantly outperforms SSN on mean AUROC and AUPR, and pySCENIC significantly outperforms both.

## Related tools

- **dyngen** (Simulates the 42 benchmark datasets (Gillespie SSA over a generated GRN) with cell-specific ground-truth regulatory networks used for scoring) — https://github.com/dynverse/dyngen
- **lionessR** (Runs LIONESS with Pearson correlation as the base network inference method on each dataset to produce cell-specific GRNs)
- **tidyverse** (Used with basic R to implement the SSN leave-one-out Pearson differencing approach and to aggregate per-cell scores)
- **pySCENIC** (Third CSNI condition included in the benchmark comparison; significantly outperforms LIONESS and SSN on mean AUROC and AUPR)
- **dyngen_manuscript** (Repository containing the reproduction code for the network-inference use case (analysis/usecase_network_inference)) — https://github.com/dynverse/dyngen_manuscript

## Examples

```
git clone https://github.com/dynverse/dyngen_manuscript && cd dyngen_manuscript && Rscript -e 'devtools::install("package/", dependencies = TRUE)'
```

## Evaluation signals

- Each of the 42 datasets yields a mean AUROC and mean AUPR value within the valid [0, 1] range computed over the top 10,000 interactions per cell
- The resulting scores table contains an entry for every dataset in each CSNI condition (e.g. the LIONESS CSV contains exactly 42 rows with mean AUROC and mean AUPR columns)
- Dataset count invariant: exactly 42 complete datasets (14 backbones × 3 seeds), each with a count matrix plus ground-truth trajectory and cellwise GRN annotations
- Qualitative check: SSN scores should be markedly worse than LIONESS, consistent with SSN's correlation differences being mostly noise; pySCENIC should exceed LIONESS on mean AUROC and AUPR

## Limitations

- SSN performs poorly on the dyngen CSNI benchmark because subtracting one cell from the whole dataset yields correlation differences that are mostly noise
- PANDA is inapplicable as a LIONESS base method here because dyngen provides no motif data
- dyngen assumes cells are well-mixed and independent from each other, so inferred performance carries that simulator assumption
- No software version numbers are reported for any tool, which may affect exact reproducibility

## Evidence

- [results] Mean AUROC/AUPR aggregation over cells: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [abstract] dyngen supports CSNI benchmarking: "three applications: aligning cell developmental trajectories, cell-specific regulatory network inference and estimation of RNA velocity"
- [results] Motivation for quantitative CSNI assessment: "a quantitative assessment of their performance is until now lacking"
- [readme] Reproduction code location: [evidence span withheld — anchor status: external_doc]
- [discussion] Simulator assumption: "One major assumption of dyngen is that cells are assumed to be well-mixed and independent from each other"
