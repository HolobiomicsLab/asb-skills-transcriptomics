---
name: paired-differential-expression-benchmarking
description: Use when benchmarking cell-type drug-response prioritization methods
  on single-cell perturbation datasets with known responsive cell types, or when comparing
  untreated-only inference against paired pre/post-treatment DE-based baselines.
when_to_use_negative:
- The dataset has no known drug target gene or no ground-truth responsive cell types,
  so target-cell-type ranks cannot be computed as a benchmark metric.
- The drug acts as an agonist rather than an inhibitor — scRank's perturbation model
  is primarily designed for inhibitors; agonist modeling is a simplifying assumption.
- Only treated (post-perturbation) data are available with no untreated condition,
  since scRank requires disease-condition untreated scRNA-seq as input.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: Untreated-only cell-type prioritization via target-perturbed GRN (CreateScRank
    -> Constr_net -> rank_celltype); rankings read from obj@cell_type_rank
  repo: https://github.com/ZJUFanLab/scRank
- name: Augur
  role: Paired-condition baseline that ranks cell-type response by separability within
    a high-dimensional space
- name: Seurat
  role: R/single-cell framework hosting the differential expression comparators (Wilcoxon,
    MAST, bimod) and dataset handling
- name: rTensor
  role: Dependent R package required for scRank installation (used in GRN reconstruction
    via tensor decomposition)
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Python module integrated in the Constr_net() function for GRN construction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:1332db50afa574219c94d4ef58d68c68a95d646d941eb676b5d5d8a37f08e7a5
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# paired-differential-expression-benchmarking

## Summary

Benchmarks drug-responsive cell-type prioritization by comparing scRank — which uses only untreated scRNA-seq data via a target-perturbed GRN — against paired-condition baselines (Augur, Wilcoxon, MAST, bimod, DESeq2) on datasets with known target cell types, scoring methods by the rank of the ground-truth target cell type.

## When to use

Use when benchmarking cell-type drug-response prioritization methods on single-cell perturbation datasets with known responsive cell types, or when comparing untreated-only inference against paired pre/post-treatment DE-based baselines.

## When NOT to use

- The dataset has no known drug target gene or no ground-truth responsive cell types, so target-cell-type ranks cannot be computed as a benchmark metric.
- The drug acts as an agonist rather than an inhibitor — scRank's perturbation model is primarily designed for inhibitors; agonist modeling is a simplifying assumption.
- Only treated (post-perturbation) data are available with no untreated condition, since scRank requires disease-condition untreated scRNA-seq as input.

## Inputs

- scenario-4 simulated drug-perturbation single-cell dataset with known ground-truth responsive cell type
- five real drug-perturbation scRNA-seq datasets with nine known target cell types in total
- known drug target gene for each dataset (e.g., from the utile_database)
- paired pre/post-treatment condition labels for baseline methods (Augur, Wilcoxon, MAST, bimod, DESeq2)

## Outputs

- per-dataset, per-method rank of the known target cell type
- averaged-rank comparison table across methods (averaged_rank_comparison.csv)
- figure panels reproducing Figures 3A-3D and the scGen/scVIDR/CellOracle comparison of Figure S4 (PNG files)

## How to apply

Load datasets with known ground-truth responsive cell types (e.g., the scenario-4 simulated drug-perturbation dataset and five real drug-perturbation scRNA-seq datasets covering nine target cell types). For scRank, run only the untreated condition via CreateScRank -> Constr_net -> rank_celltype using the known drug target gene, and read the ranking from obj@cell_type_rank. For comparators requiring disease-treatment paired data, run Augur and differential expression methods (Wilcoxon, MAST, bimod, DESeq2) on pre/post conditions, plus scGen, scVIDR, and CellOracle for supplementary comparisons. Convert each method's per-dataset cell-type ordering into the rank of the known target cell type and average the ranks per method across datasets. Validate by checking that the reproduced averaged-rank ordering matches Figures 3A-3D and Figure S4, with scRank outperforming the paired-data comparators.

## Related tools

- **scRank** (Untreated-only cell-type prioritization via target-perturbed GRN (CreateScRank -> Constr_net -> rank_celltype); rankings read from obj@cell_type_rank) — https://github.com/ZJUFanLab/scRank
- **Augur** (Paired-condition baseline that ranks cell-type response by separability within a high-dimensional space)
- **Seurat** (R/single-cell framework hosting the differential expression comparators (Wilcoxon, MAST, bimod) and dataset handling)
- **rTensor** (Dependent R package required for scRank installation (used in GRN reconstruction via tensor decomposition)) — https://github.com/rikenbit/rTensor
- **tensorly** (Python module integrated in the Constr_net() function for GRN construction)

## Examples

```
devtools::install_github("ZJUFanLab/scRank"); obj <- CreateScRank(untreated_seurat_obj); obj <- Constr_net(obj); obj <- rank_celltype(obj, drug_target_gene); obj@cell_type_rank
```

## Evaluation signals

- scRank, run on untreated data only, ranks all nine known drug-targeted cell types at the top of the five real datasets, exceeding paired-method ranks in most cases (Figures 3C and 3D).
- The averaged-rank ordering across methods matches the published Figures 3A-3D ordering.
- averaged_rank_comparison.csv contains one averaged rank per method aggregated across the simulation and the five real datasets.
- scRank ranking output is read from obj@cell_type_rank after rank_celltype runs with the correct known drug target gene supplied.
- Supplementary scGen/scVIDR/CellOracle comparison reproduces the Figure S4 condition ordering.

## Limitations

- Paired-condition baselines (Augur, Wilcoxon, MAST, bimod, DESeq2) require disease-treatment paired datasets and ignore prior knowledge of the drug target, limiting their applicability to untreated diseased samples.
- scRank primarily focuses on inhibitor drugs due to data availability; agonist tests are preliminary and use a simplifying assumption (setting out-edge weights of the target gene to maximum value).
- Performance of competing tools (e.g., scDEAL, beyondcell on sorafenib) can be influenced by the quality and imbalance of the bulk datasets they utilized.
- Multi-target perturbation requires tailored methodologies not yet developed; only preliminary results exist.

## Evidence
