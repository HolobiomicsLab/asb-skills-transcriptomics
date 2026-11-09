---
name: cell-line-drug-sensitivity-benchmarking
description: Use when comparing drug-responsive cell-type/cell-line inference tools
  on cancer cell line scRNA-seq data with known sensitivity labels (e.
when_to_use_negative:
- When the goal is to rank drug-responsive cell types in a single exploratory disease
  dataset without ground-truth sensitivity labels — the benchmarking skill specifically
  requires known resistant/sensitive or CTRP labels to score accuracy
- When the drug of interest acts as an agonist or has multiple targets — scRank primarily
  focuses on inhibitor drugs, and multi-target perturbation requires methodologies
  not yet developed
- When paired disease-treatment scRNA-seq data are already available and differential-expression-based
  approaches are appropriate — this skill targets inference from untreated scRNA-seq
  data
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: 'Primary method under benchmark: infers drug-responsive cell types from untreated
    scRNA-seq via target-perturbed gene regulatory network (tpGRN) in silico drug
    perturbation'
  repo: https://github.com/ZJUFanLab/scRank
- name: beyondcell
  role: Baseline drug-response prediction method run on the same cell line datasets
    to produce comparison rankings
- name: scDEAL
  role: Baseline drug-response prediction method run on the same cell line datasets
    to produce comparison rankings
- name: devtools
  role: R dependency used to install scRank from GitHub
- name: rTensor
  role: R dependency required before installing scRank
  repo: https://github.com/rikenbit/rTensor
- name: R
  role: Runtime environment for the scRank, beyondcell, and scDEAL pipelines
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:e614e3548afc1088c12ffd75f857f7021db062e439a9fba5cbda33f405581291
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - Seurat
  - beyondcell
  - devtools
  - rTensor
  - scDEAL
  - scRank
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-line-drug-sensitivity-benchmarking

## Summary

Benchmarks drug-response-predicting single-cell methods (scRank, beyondcell, scDEAL) on cancer cell line scRNA-seq datasets by scoring each method's top-ranked drug-responsive cell line against known drug-sensitivity ground truth (e.g., CTRP labels), yielding per-dataset and overall accuracies.

## When to use

Use when comparing drug-responsive cell-type/cell-line inference tools on cancer cell line scRNA-seq data with known sensitivity labels (e.g., GSE110894 resistant/sensitive leukaemic cells or the Kinker 179-cell-line × 53-drug CTRP evaluation), to quantify which method's top-ranked prediction matches ground truth.

## When NOT to use

- When the goal is to rank drug-responsive cell types in a single exploratory disease dataset without ground-truth sensitivity labels — the benchmarking skill specifically requires known resistant/sensitive or CTRP labels to score accuracy
- When the drug of interest acts as an agonist or has multiple targets — scRank primarily focuses on inhibitor drugs, and multi-target perturbation requires methodologies not yet developed
- When paired disease-treatment scRNA-seq data are already available and differential-expression-based approaches are appropriate — this skill targets inference from untreated scRNA-seq data

## Inputs

- Cancer cell line scRNA-seq gene expression matrices (e.g., GSE110894 leukaemic cells with BET inhibitor resistant/sensitive labels; Kinker 179-cell-line dataset)
- Cell type/cell line labels for each dataset
- Drug names or drug target genes (resolvable via scRank's utile_database)
- Ground-truth drug sensitivity labels (resistant/sensitive for GSE110894; CTRP drug sensitivity for the Kinker 179-cell-line × 53-drug evaluation)

## Outputs

- Drug-responsive cell type/cell line rankings per method (scRank rankings in obj@cell_type_rank; analogous rankings from beyondcell and scDEAL)
- Per-dataset and overall accuracy scores per method
- Compiled benchmark_accuracy.csv of accuracies for scRank, beyondcell, and scDEAL across datasets

## How to apply

Install scRank from the ZJUFanLab GitHub repository with devtools::install_github after installing the rTensor dependency. For each cell line scRNA-seq dataset, create an scRank S4 object with CreateScRank supplying the gene expression profile, cell line labels, species, and the drug target (or drug name resolved via the package's utile_database), keeping the default antagonist mode-of-action assumption. Reconstruct the gene regulatory network with Constr_net, then rank cell lines by in silico drug perturbation with rank_celltype and read rankings from obj@cell_type_rank. Run beyondcell and scDEAL baselines on the same datasets to obtain their drug-responsive rankings. Score each method's top-ranked cell line against ground truth (resistant/sensitive labels for GSE110894; CTRP drug sensitivity for the Kinker evaluation) and compute per-dataset and overall accuracies, compiling them into benchmark_accuracy.csv. In this benchmark, scRank achieved 77.8% mean accuracy versus 72.2% for beyondcell and 64.8% for scDEAL across the three cancer cell line datasets plus the Kinker evaluation.

## Related tools

- **scRank** (Primary method under benchmark: infers drug-responsive cell types from untreated scRNA-seq via target-perturbed gene regulatory network (tpGRN) in silico drug perturbation) — https://github.com/ZJUFanLab/scRank
- **beyondcell** (Baseline drug-response prediction method run on the same cell line datasets to produce comparison rankings)
- **scDEAL** (Baseline drug-response prediction method run on the same cell line datasets to produce comparison rankings)
- **devtools** (R dependency used to install scRank from GitHub)
- **rTensor** (R dependency required before installing scRank) — https://github.com/rikenbit/rTensor
- **R** (Runtime environment for the scRank, beyondcell, and scDEAL pipelines)

## Examples

```
devtools::install_github("rikenbit/rTensor"); devtools::install_github("ZJUFanLab/scRank")
```

## Evaluation signals

- scRank's mean accuracy across the three cancer cell line datasets is 77.8%, higher than beyondcell (72.2%) and scDEAL (64.8%)
- Per-dataset top-ranked cell type matches the known ground-truth label (resistant/sensitive for GSE110894; CTRP sensitivity for the Kinker 179-cell-line × 53-drug evaluation)
- Rankings are retrievable from obj@cell_type_rank after running rank_celltype on each CreateScRank S4 object
- The compiled benchmark_accuracy.csv contains accuracies for all three methods across the three cell line datasets plus the Kinker evaluation
- scRank pipeline completes the expected sequence: CreateScRank → Constr_net → rank_celltype with default antagonist MOA

## Limitations

- scRank primarily focuses on drugs acting as inhibitors, partly due to the nature of available data; agonist tests are preliminary only
- Low performances of scDEAL and beyondcell on some drugs (e.g., sorafenib) are influenced by the quality and imbalance of the bulk datasets they utilized
- Multi-target drug perturbation requires specifically tailored methodologies not yet developed; only preliminary results exist
- scRank's adaptability across clustering methodologies and its drug-resistance integrated score are only shown in supplementary figures (Figure S15, S16)

## Evidence

- [results] Benchmark accuracy comparison: [evidence span withheld — anchor status: not_found]
- [methods] Installation steps: "install dependent packages `devtools` and [`rTensor`](https://github.com/rikenbit/rTensor)"
- [methods] CreateScRank object creation: "scRank start with create a S4 object by `CreateScRank` function"
- [methods] Network construction step: "the first is to reconstruct the gene regulatory network from expression ptrofiles using `Constr_net` function"
- [methods] Ranking step: "the second step is to estimate the extent of the in silico drug perturbation for GRNs in each cell type using `rank_celltype` function"
- [methods] Demo dataset with ground truth labels: "a demo dataset (GSE110894) containing BET inhibitor resistant and sensitive leukaemic cells"
- [discussion] Baseline comparison framing: "In our comparison of scRank, beyondcell, and scDEAL, we observed disparities in the predictive performances."
- [discussion] Inhibitor-focused limitation: "One limitation of scRank is its primary focus on drugs acting as inhibitors, which is partly due to the nature of the available data."
- [readme] Install command from README: "devtools::install_github("ZJUFanLab/scRank")"
