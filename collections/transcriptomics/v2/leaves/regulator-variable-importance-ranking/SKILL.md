---
name: regulator-variable-importance-ranking
description: Use when you have BoostDiff output from two conditions (e.g., salt-stress
  target vs. SMM baseline) and need to rank regulator-to-target edges by aggregating
  weighted DVI across the boosted differential trees into a VIM table.
when_to_use_negative:
- The two expression matrices do not share the same set of features, or data comes
  from a single condition — BoostDiff's differential trees require two conditions
  compared simultaneously with the first column named 'Gene'.
- The goal is differential expression per gene rather than ranking regulatory context;
  BoostDiff complements standard differential expression analyses but does not replace
  them.
- Dataset scale matches the Crohn's disease dataset (GSE126124), which the authors
  state is too large for a single machine and needs a high-performance computing cluster.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0659
tools:
- name: BoostDiff
  role: Builds the modified AdaBoost ensemble of differential trees whose DVI-based
    variable importances are aggregated into the regulator VIM ranking and raw network
    txt files
  repo: https://github.com/scibiome/boostdiff_inference
- name: pandas
  role: Loads the tab-separated salt-stress and SMM expression matrices and writes
    the aggregated VIM table as CSV
- name: numpy
  role: Supports aggregation of weighted DVI across the 50 boosted trees into per-target-gene
    regulator importances
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:03a755e85c23c6ba78091849567ffe4398647c6df09a50ef09c5d15be33cccfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - Python
  - conda
  - numpy
  - pandas
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# regulator-variable-importance-ranking

## Summary

Rank candidate regulators of each target gene by aggregating BoostDiff's differential variance improvement (DVI) across its 50-tree modified AdaBoost ensemble into per-target-gene variable importance (VIM) scores. These weighted importances are what turn the boosted differential trees into the final differential gene regulatory network.

## When to use

Use when you have BoostDiff output from two conditions (e.g., salt-stress target vs. SMM baseline) and need to rank regulator-to-target edges by aggregating weighted DVI across the boosted differential trees into a VIM table.

## When NOT to use

- The two expression matrices do not share the same set of features, or data comes from a single condition — BoostDiff's differential trees require two conditions compared simultaneously with the first column named 'Gene'.
- The goal is differential expression per gene rather than ranking regulatory context; BoostDiff complements standard differential expression analyses but does not replace them.
- Dataset scale matches the Crohn's disease dataset (GSE126124), which the authors state is too large for a single machine and needs a high-performance computing cluster.

## Inputs

- Tab-separated disease/target expression matrix (genes as rows, samples as columns, first column named 'Gene'), e.g. exprs_salt.txt
- Tab-separated control/baseline expression matrix with the same gene set, e.g. exprs_smm.txt
- Raw BoostDiff outputs: boostdiff_network txt file and differences txt file per target-condition run

## Outputs

- Per-target-gene regulator variable importance (VIM) table saved as CSV
- Raw boostdiff_network txt files (ranked regulator-target edges) in 'disease' and 'control' subfolders
- Filtered ranked edge list (filter_network, p=3, n_top_edges=150) per target condition

## How to apply

Run BoostDiff's modified AdaBoost ensemble twice — once with the disease/salt condition as target, once with the control/SMM condition as target — using n_estimators=50, n_features=2500, n_subsamples=20, min_samples_leaf=2, min_samples_split=6, max_depth=2, learning_rate=1.0, loss='exponential', regulators='all', and n_processes=4, writing results into 'disease' and 'control' subfolders with keyword 'saltvssmm'. In each tree, only the target-condition samples are re-weighted across boosting iterations while baseline samples retain uniform weight, and each tree contributes weight alpha_m. Aggregate the weighted differential variance improvement (DVI) across all 50 differential trees into a per-target-gene regulator variable importance (VIM) table and save it as CSV alongside the raw boostdiff_network txt files. Because the raw network includes all target genes, filter it for target genes in which BoostDiff found a more predictive model for the target condition (filter_network with p=3, n_top_edges=150) before interpreting the top-ranked regulator edges. Divergent edge rankings between the two target-condition runs are expected and flag condition-specific regulation.

## Related tools

- **BoostDiff** (Builds the modified AdaBoost ensemble of differential trees whose DVI-based variable importances are aggregated into the regulator VIM ranking and raw network txt files) — https://github.com/scibiome/boostdiff_inference
- **pandas** (Loads the tab-separated salt-stress and SMM expression matrices and writes the aggregated VIM table as CSV)
- **numpy** (Supports aggregation of weighted DVI across the 50 boosted trees into per-target-gene regulator importances)

## Examples

```
from boostdiff.main_boostdiff import BoostDiff; model.run(file_etha, file_ctrl, output_folder, n_estimators=50, n_features=2500, n_subsamples=20, keyword="saltvssmm", n_processes=4, loss="exponential")
```

## Evaluation signals

- Both target-condition runs complete and each of the 'disease' and 'control' subfolders contains a differences txt file and a raw boostdiff_network txt file.
- The VIM CSV contains exactly 50 boosted trees' worth of aggregated weighted DVI, with regulator candidates bounded by n_features=2500.
- filter_network(p=3, n_top_edges=150) yields at most 150 top-ranked edges per condition, restricted to target genes with a more predictive target-condition model.
- Top-ranked regulator edges differ between the disease-target and control-target runs, with conflicting edges expected for condition-specific regulation.
- Only target-condition samples show varying weights across boosting iterations; baseline samples remain uniformly weighted in every differential tree.

## Limitations

- For real transcriptomics datasets, n_estimators should stay low (e.g., 50 boosted differential trees) to avoid overfitting; the Crohn's disease dataset is too large without a high-performance computing cluster.
- The ideal input is at least 30 samples per condition (min_samples defaults to 15 per condition), and BoostDiff was tested on Linux only.
- The raw network must be filtered for target genes in which BoostDiff found a more predictive model for the target condition before rankings are interpreted.

## Evidence

- [abstract] abstract: "BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition."
- [methods] methods — filtering: "the raw network should be filtered for target genes in which BoostDiff found a more predictive model for the target condition"
- [methods] methods — filter call: "df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)"
- [readme] readme — DVI criterion: "BoostDiff modifies regression trees to use differential variance improvement (DVI) as the novel splitting criterion."
- [readme] readme — inputs: "the two inputs should have the same set of features, where the first column should be named "Gene""
