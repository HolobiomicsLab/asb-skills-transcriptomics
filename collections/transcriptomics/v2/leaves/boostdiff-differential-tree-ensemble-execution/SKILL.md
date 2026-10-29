---
name: boostdiff-differential-tree-ensemble-execution
description: Use when you have paired disease/control tab-separated gene expression
  matrices (genes as rows, samples as columns, identical feature sets) and need to
  infer a differential network between the two conditions.
when_to_use_negative:
- You need an undirected co-expression or single-condition GRN rather than a differential
  network between two labeled conditions — use GENIE3/ARACNE-style inference instead
- Either condition has fewer than ~15 samples (BoostDiff requires min_samples=15 per
  condition) or the two input matrices do not share the same gene feature set
- The dataset is very large (e.g. the Crohn's disease GSE126124 example) without access
  to a high-performance computing cluster
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: BoostDiff
  role: 'Core method: builds the modified AdaBoost ensemble of differential trees
    with differential variance improvement (DVI) splitting criterion to infer the
    differential network'
  repo: https://github.com/scibiome/boostdiff_inference
- name: conda
  role: 'Environment management: create the bdenv environment from environment.yml
    before installing the package'
- name: pip
  role: Install the BoostDiff package from the cloned git repository inside bdenv
- name: GEOquery
  role: Optionally download GEO datasets (e.g. GSE126124) to build the two condition-specific
    expression matrices
- name: biomaRt
  role: Map probe IDs to HGNC gene symbols so both condition matrices share a common
    'Gene' identifier column
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:853a3985b26a50d9c2caab312aeeefc03a65bf3a63b9aaece0e03835c3b73b86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - R
  - biomaRt
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# boostdiff-differential-tree-ensemble-execution

## Summary

Run BoostDiff, a tree-based method that infers differential networks from two gene expression conditions using modified AdaBoost ensembles of differential trees with differential variance improvement (DVI) as splitting criterion, to obtain a ranked edge list discriminating disease vs. control.

## When to use

Use when you have paired disease/control tab-separated gene expression matrices (genes as rows, samples as columns, identical feature sets) and need to infer a differential network between the two conditions.

## When NOT to use

- You need an undirected co-expression or single-condition GRN rather than a differential network between two labeled conditions — use GENIE3/ARACNE-style inference instead
- Either condition has fewer than ~15 samples (BoostDiff requires min_samples=15 per condition) or the two input matrices do not share the same gene feature set
- The dataset is very large (e.g. the Crohn's disease GSE126124 example) without access to a high-performance computing cluster

## Inputs

- Disease-condition expression matrix: tab-separated file, rows = genes, columns = samples, first column named 'Gene'
- Control-condition expression matrix: tab-separated file with the same set of features as the disease file
- Parameters: n_estimators (recommended 50), n_features, n_subsamples, keyword, n_processes, optional regulators list, normalize (default 'unit_variance'), loss ('square'/'exponential'/'linear')

## Outputs

- Raw differential network (ranked edge list with variable-importance-based scores) written to output_folder under the given keyword
- Filtered differential network for target genes with a more predictive model for the target condition
- Diagnostics: correlation distributions per target condition and histogram of mean difference in prediction error

## How to apply

Create the conda environment (bdenv) from environment.yml and install the package from GitHub via `pip install .`. Prepare two tab-separated inputs with a 'Gene' first column and identical feature sets across disease and control files; aim for at least 30 samples per condition (minimum 15 per condition required by the algorithm). Call `model.run(file_disease, file_control, output_folder, n_estimators, n_features, n_subsamples, keyword, n_processes)` with `n_estimators=50` to avoid overfitting on real transcriptomics data, e.g. n_features=2500, n_subsamples=20, loss='exponential', and 'unit_variance' normalization for the B. subtilis salt stress demo. Run BoostDiff twice — once with the disease condition as target and once with control as target — then filter the raw network for target genes where BoostDiff found a more predictive model for the target condition (e.g. `filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)`).

## Related tools

- **BoostDiff** (Core method: builds the modified AdaBoost ensemble of differential trees with differential variance improvement (DVI) splitting criterion to infer the differential network) — https://github.com/scibiome/boostdiff_inference
- **conda** (Environment management: create the bdenv environment from environment.yml before installing the package)
- **pip** (Install the BoostDiff package from the cloned git repository inside bdenv)
- **GEOquery** (Optionally download GEO datasets (e.g. GSE126124) to build the two condition-specific expression matrices)
- **biomaRt** (Map probe IDs to HGNC gene symbols so both condition matrices share a common 'Gene' identifier column)

## Examples

```
from boostdiff.main_boostdiff import BoostDiff; model = BoostDiff(); model.run(file_etha, file_ctrl, output_folder, n_estimators=50, n_features=2500, n_subsamples=20, keyword='saltvssmm', n_processes=4)
```

## Evaluation signals

- The two input tab-separated files parse with a 'Gene' first column and identical feature sets across disease and control conditions
- At least 15 samples per condition are present (ideal ≥30); output files are generated in output_folder under the chosen keyword
- Run completes with n_estimators=50 and produces a ranked network whose top edges can be filtered (p=3, n_top_edges=150) and visualized
- On simulated benchmarks, precision/recall/F1 on the top-100 edges should reproduce BoostDiff leading in the 150-, 300-, and 500-node settings while RF Difftrees leads at 50 nodes
- Histogram of mean difference in prediction error and per-condition correlation distributions are plotted and non-degenerate

## Limitations

- BoostDiff was tested on Linux only
- The recommended low ensemble size (n_estimators=50) is advised for real data to avoid overfitting; larger datasets like Crohn's disease GSE126124 require a high-performance computing cluster
- The two input expression matrices must have the same set of features and a 'Gene' first column; genes may need mapping/filtering (e.g. via GEOquery/biomaRt) beforehand

## Evidence

- [abstract] AdaBoost ensemble with DVI criterion: "BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition."
- [readme] Installation procedure: "conda env create --name bdenv --file environment.yml
conda activate bdenv

pip install ."
- [readme] Recommended ensemble size rationale: "we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid overfitting."
- [readme] Input format requirement: "Each input is a tab-separated file. The rows correspond to genes, while the columns correspond to samples. Note that the two inputs should have the same set of features, where the first column should"
- [methods] Dual-run and filtering workflow: "Run 1: The disease condition will be used as the target condition (with control condition as baseline)"
