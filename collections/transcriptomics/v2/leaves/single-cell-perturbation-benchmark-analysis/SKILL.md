---
name: single-cell-perturbation-benchmark-analysis
description: Use when comparing drug-responsive cell-type prioritization methods on
  scRNA-seq perturbation data where ground-truth target cell types are known and paired
  pre/post-treatment baselines must be evaluated against untreated-only inference.
when_to_use_negative:
- The goal is single-method application rather than cross-method benchmarking — just
  run the scRank pipeline (CreateScRank -> Constr_net -> rank_celltype) without comparator
  methods
- The drug acts as an agonist or has multiple targets — scRank's inhibitor-focused
  tpGRN model and its agonist/multi-target extensions are only preliminary (Figures
  S14, S17)
- No known ground-truth target cell types exist for the datasets, since the averaged-rank
  validation metric requires target-cell-type ranks
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: Untreated-only drug-responsive cell-type prioritization via target-perturbed
    GRN (tpGRN); ranking read from obj@cell_type_rank
  repo: https://github.com/ZJUFanLab/scRank
- name: Augur
  role: Paired-condition baseline that determines cell-type response priority using
    separability within a high-dimensional space
- name: rTensor
  role: R dependency for tensor operations used by scRank's Constr_net GRN reconstruction
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Python module integrated in the Constr_net() function for network reconstruction
- name: Seurat
  role: R/single-cell data handling for the perturbation datasets and DE baselines
    (Wilcoxon, MAST, bimod)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:bdaa0f0933273fb3f79fb4886d20fe26b4f780f3d24fd39da7e12a36628b440e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - Augur
  - R
  - Seurat
  - rTensor
  - scRank
  - tensorly
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-perturbation-benchmark-analysis

## Summary

Benchmarks scRank's untreated-only target-perturbed GRN approach against paired-condition cell-type prioritization baselines (Augur, Wilcoxon, MAST, bimod, DESeq2) on simulated and real drug-perturbation scRNA-seq datasets with known target cell types.

## When to use

Use when comparing drug-responsive cell-type prioritization methods on scRNA-seq perturbation data where ground-truth target cell types are known and paired pre/post-treatment baselines must be evaluated against untreated-only inference.

## When NOT to use

- The goal is single-method application rather than cross-method benchmarking — just run the scRank pipeline (CreateScRank -> Constr_net -> rank_celltype) without comparator methods
- The drug acts as an agonist or has multiple targets — scRank's inhibitor-focused tpGRN model and its agonist/multi-target extensions are only preliminary (Figures S14, S17)
- No known ground-truth target cell types exist for the datasets, since the averaged-rank validation metric requires target-cell-type ranks

## Inputs

- Scenario-4 simulated drug-perturbation scRNA-seq dataset with known responsive cell type
- Five real drug-perturbation scRNA-seq datasets with known ground-truth target cell types (nine target cell types total)
- Known drug target gene per dataset (e.g., resolvable via utile_database)
- Pre/post-treatment paired-condition expression matrices for baseline methods

## Outputs

- averaged_rank_comparison.csv containing per-method averaged ranks of known target cell types
- Figure panels reproducing Figures 3A-3D as PNG files
- Supplementary condition comparison panel reproducing Figure S4 (scGen, scVIDR, CellOracle)
- Per-dataset cell-type rankings from obj@cell_type_rank for scRank runs

## How to apply

Load the scenario-4 simulated drug-perturbation dataset and the five real drug-perturbation scRNA-seq datasets, each with nine known ground-truth responsive cell types in total. For each dataset, run scRank on the untreated condition only by building an S4 object with CreateScRank, reconstructing the GRN with Constr_net, and calling rank_celltype with the known drug target gene, reading the ranking from obj@cell_type_rank. Run the paired-condition comparators (Augur, Wilcoxon, MAST, bimod, DESeq2) on pre/post conditions, and scGen, scVIDR, and CellOracle for supplementary comparisons. Convert each method's per-dataset rankings into the rank of the known target cell type, average ranks per method across datasets, and reproduce the averaged-rank panels of Figures 3A-3D and Figure S4. Validation succeeds when scRank (untreated-only) ranks all nine known target cell types at the top, outperforming the paired-data baselines in most cases.

## Related tools

- **scRank** (Untreated-only drug-responsive cell-type prioritization via target-perturbed GRN (tpGRN); ranking read from obj@cell_type_rank) — https://github.com/ZJUFanLab/scRank
- **Augur** (Paired-condition baseline that determines cell-type response priority using separability within a high-dimensional space)
- **rTensor** (R dependency for tensor operations used by scRank's Constr_net GRN reconstruction) — https://github.com/rikenbit/rTensor
- **tensorly** (Python module integrated in the Constr_net() function for network reconstruction)
- **Seurat** (R/single-cell data handling for the perturbation datasets and DE baselines (Wilcoxon, MAST, bimod))

## Examples

```
devtools::install_github("ZJUFanLab/scRank"); obj <- CreateScRank(untreated_seurat_obj); obj <- Constr_net(obj); obj <- rank_celltype(obj, drug_target = "target_gene"); obj@cell_type_rank
```

## Evaluation signals

- On the five real datasets, scRank (untreated-only) ranks all nine known drug-targeted cell types at the top, exceeding paired-condition baselines (Wilcoxon, MAST, bimod, DESeq2, Augur) in most cases (Figures 3C-3D)
- The reproduced averaged-rank ordering across methods matches the reported Figures 3A-3D and Figure S4
- Each scRank run produces a non-empty obj@cell_type_rank covering all annotated cell types in the dataset
- averaged_rank_comparison.csv contains one averaged-rank row per method, computed as the mean target-cell-type rank across the simulation and five real datasets
- Ranks for comparator methods are computed on paired pre/post conditions while scRank uses only the untreated condition, preserving the benchmark's asymmetry

## Limitations

- scRank primarily focuses on inhibitor drugs due to data availability; agonist tests are preliminary only
- Agonist modeling uses a simplifying assumption (setting out-edge weights of the target gene node to maximum value)
- Multi-target perturbation requires specifically tailored methodologies not yet developed; preliminary results only (Figure S17)
- Baseline DE methods require disease-treatment paired datasets and ignore prior knowledge of the drug target, which is why untreated-only scRank performance is the key benchmark contrast

## Evidence
