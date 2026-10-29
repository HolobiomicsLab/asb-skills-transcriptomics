---
name: boosted-tree-ensemble-training
description: Use when you have two matched tab-separated expression matrices (rows
  = genes, columns = samples, first column 'Gene') from disease vs. control conditions
  and need a condition-specific differential network with regulator-to-target edges.
when_to_use_negative:
- The two input matrices do not share the same feature set, or the first column is
  not named 'Gene' — BoostDiff requires identical features across conditions
- Either condition has fewer than 15 samples (BoostDiff's min_samples default), making
  target-condition re-weighting unreliable; prefer at least 30 per condition
- The goal is a static (single-condition) co-expression or GRN rather than a differential
  network discriminating two conditions
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: BoostDiff
  role: Trains the modified AdaBoost.R2 ensemble of differential regression trees
    with DVI splitting criterion; its run() method executes the two target-condition
    boosting runs and writes raw networks and VIM tables
  repo: https://github.com/scibiome/boostdiff_inference
- name: conda
  role: Creates the bdenv environment from the provided environment.yml before pip-installing
    BoostDiff
- name: pip
  role: Installs the BoostDiff package from the cloned git repository inside the conda
    environment
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:552df802a57cf05d3209ea3505197731911514b18c73deeba66273850820f90f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - Python
  - conda
  - matplotlib
  - networkx
  - numpy
  - pandas
  - pip
  - pygraphviz
  - seaborn
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5909090909090909
n_claims: 22
---

# boosted-tree-ensemble-training

## Summary

Train the BoostDiff modified AdaBoost ensemble of differential regression trees, which use differential variance improvement (DVI) as a novel splitting criterion, to infer a differential gene regulatory network between two expression conditions. Only the target-condition samples are re-weighted across boosting iterations while baseline samples keep uniform weight.

## When to use

Use when you have two matched tab-separated expression matrices (rows = genes, columns = samples, first column 'Gene') from disease vs. control conditions and need a condition-specific differential network with regulator-to-target edges.

## When NOT to use

- The two input matrices do not share the same feature set, or the first column is not named 'Gene' — BoostDiff requires identical features across conditions
- Either condition has fewer than 15 samples (BoostDiff's min_samples default), making target-condition re-weighting unreliable; prefer at least 30 per condition
- The goal is a static (single-condition) co-expression or GRN rather than a differential network discriminating two conditions

## Inputs

- Disease-condition expression matrix (tab-separated, rows = genes, columns = samples, first column named 'Gene')
- Control-condition expression matrix (tab-separated, same gene set as disease matrix, first column named 'Gene')

## Outputs

- Raw differential network file boostdiff_network_<keyword>.txt written into 'disease' and 'control' subfolders of the output folder
- Differences txt file per target-condition run
- Aggregated regulator variable-importance (VIM) table CSV per target gene

## How to apply

Install BoostDiff in a conda environment (bdenv from environment.yml, then pip install .) and import via 'from boostdiff.main_boostdiff import BoostDiff'. Load the disease and control matrices, letting BoostDiff apply its default 'unit_variance' normalization, then call model.run(file_disease, file_control, output_folder, ...) with n_estimators=50 (recommended for real transcriptomics data to avoid overfitting), n_features=2500, n_subsamples=20, min_samples_leaf=2, min_samples_split=6, max_depth=2, learning_rate=1.0, loss='exponential', regulators='all'. Run the boosting loop twice: once with the disease condition as target and once with control as target, writing into 'disease' and 'control' subfolders with a shared keyword so output files are named boostdiff_network_<keyword>.txt. Each differential tree re-weights only target-condition samples during AdaBoost.R2 boosting; the weighted DVI is aggregated across the 50 trees into a per-target-gene regulator variable-importance (VIM) table saved as CSV alongside the raw network txt files. Require at least ~30 samples per condition ideally (minimum 15 per condition via min_samples).

## Related tools

- **BoostDiff** (Trains the modified AdaBoost.R2 ensemble of differential regression trees with DVI splitting criterion; its run() method executes the two target-condition boosting runs and writes raw networks and VIM tables) — https://github.com/scibiome/boostdiff_inference
- **conda** (Creates the bdenv environment from the provided environment.yml before pip-installing BoostDiff)
- **pip** (Installs the BoostDiff package from the cloned git repository inside the conda environment)

## Examples

```
from boostdiff.main_boostdiff import BoostDiff; model = BoostDiff(); model.run('df_dis.txt', 'df_ctrl.txt', '../outputs_saltvssmm', n_estimators=50, n_features=2500, n_subsamples=20, keyword='saltvssmm', n_processes=4, loss='exponential', regulators='all', normalize='unit_variance')
```

## Evaluation signals

- Both target-condition runs complete and write a differences txt file plus a raw boostdiff_network txt file into the 'disease' and 'control' subfolders
- Exactly 50 boosted differential trees are built per run (n_estimators=50), matching the recommended setting for real transcriptomics datasets
- The VIM table aggregates weighted differential variance improvement across all trees into one importance value per regulator–target-gene pair
- Input files parse as tab-separated matrices with identical gene features in both conditions and the first column named 'Gene'
- Downstream filtering with filter_network (e.g. p=3) yields a target-gene-filtered edge list whose top edges can be validated against known biology (e.g. PPARG–PLIN4, PPARG–PCK1 edges and the CD36 hub in the TCGA BRCA case)

## Limitations

- The Crohn's disease dataset (GSE126124) is too large for a desktop demo and requires a high-performance computing cluster; BoostDiff was tested on Linux only
- A high n_estimators can overfit real transcriptomics datasets — 50 base learners is the recommended low setting
- Minimum of 15 samples per condition is enforced, and fewer than ~30 per condition is below the ideal input size

## Evidence

- [abstract] BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition.: "BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition."
- [readme] README run parameters: "For real transcriptomics datasets, we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid"
- [readme] README modified AdaBoost description: "The network is inferred by building modified AdaBoost ensembles of differential trees as base learners. BoostDiff modifies regression trees to use differential variance improvement (DVI) as the novel"
- [results] target-condition re-weighting finding: [evidence span withheld — anchor status: not_found]
- [methods] run-twice workflow: "Run 1: The disease condition will be used as the target condition (with control condition as baseline)"
