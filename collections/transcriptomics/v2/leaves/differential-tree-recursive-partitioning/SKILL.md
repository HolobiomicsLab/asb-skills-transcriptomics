---
name: differential-tree-recursive-partitioning
description: Use when you must identify regulators whose predictive relationship with
  a target gene differs between two conditions (e.g. disease vs. control) and have
  two tab-separated expression matrices with matching gene feature sets.
when_to_use_negative:
- You need a full boosted differential network rather than a single base-learner tree
  — use the complete BoostDiff AdaBoost ensemble (model.run with n_estimators) instead
- Only one condition's expression data is available; DVI requires paired per-condition
  variance-reduction scans (SD and SC)
- The two input matrices have non-matching feature sets or lack the required 'Gene'
  first column — preprocess/map gene IDs first (e.g. via GEOquery/biomaRt)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BoostDiff
  role: Reference implementation whose run() signature and default hyperparameters
    (min_samples_leaf=2, min_samples_split=6, max_depth=2, min_samples=15, unit_variance
    normalization) the grow_tree wrapper mirrors
  repo: github.com/scibiome/boostdiff_inference
- name: pandas
  role: Loads the two tab-separated expression matrices with rows-as-genes, columns-as-samples
    and the 'Gene' first column
- name: numpy
  role: Supports the variance-reduction scans over genes g' and thresholds tau used
    to compute DVI at each candidate split
- name: Python
  role: Implements the recursive partitioning, the 5-tuple node representation, and
    the grow_differential_tree.py wrapper script
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:9af704af1797bfea38a8a7c6cf0fefec98111ad67ba4d35f5a6fa60cb26993df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoostDiff
  - Python
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# differential-tree-recursive-partitioning

## Summary

Grows a BoostDiff differential tree by binary recursive partitioning of two condition-specific expression matrices using differential variance improvement (DVI) as the splitting criterion, yielding a tree whose nodes carry condition-specific split thresholds for the same predictor gene.

## When to use

Use when you must identify regulators whose predictive relationship with a target gene differs between two conditions (e.g. disease vs. control) and have two tab-separated expression matrices with matching gene feature sets.

## When NOT to use

- You need a full boosted differential network rather than a single base-learner tree — use the complete BoostDiff AdaBoost ensemble (model.run with n_estimators) instead
- Only one condition's expression data is available; DVI requires paired per-condition variance-reduction scans (SD and SC)
- The two input matrices have non-matching feature sets or lack the required 'Gene' first column — preprocess/map gene IDs first (e.g. via GEOquery/biomaRt)

## Inputs

- Disease-condition gene expression matrix (tab-separated, rows = genes, columns = samples, first column named 'Gene')
- Control-condition gene expression matrix (tab-separated, same feature set as the disease matrix)
- Optional list of regulator gene names, or the string 'all', to restrict candidate predictors
- Boolean/string normalize argument (default 'unit_variance' normalization)

## Outputs

- tree_structure.csv containing each internal node's split gene g*, per-condition thresholds tau_D* and tau_C*, and node sample membership
- A grown differential tree represented as 5-tuple nodes (SD, SC, g*, tau_D*, tau_C*)

## How to apply

Load the two tab-separated gene expression matrices (disease XD and control XC) with pandas, enforcing rows-as-genes, columns-as-samples, and a first column named 'Gene'; apply unit_variance normalization and enforce min_samples=15 per condition before splitting. Represent each node as the 5-tuple (SD, SC, g*, tau_D*, tau_C*) holding per-condition sample sets, the selected predictor gene, and its two condition-specific thresholds. At each candidate split compute DVI(g') = max_tau VarRed(g',tau,SD) - max_tau VarRed(g',tau,SC) via variance-reduction scans over genes g' and thresholds tau in each condition, selecting the gene/threshold pair maximizing DVI; two separate thresholds are used because a single threshold optimizing DVI would skew the expression distributions of the two conditions to opposite sides of the split. Recursively partition each node's per-condition sample sets at the chosen thresholds, terminating per BoostDiff defaults (min_samples_leaf=2, min_samples_split=6, max_depth=2, where max_depth=0 yields tree stumps). Serialize every internal node's split gene, tau_D* and tau_C*, and every node's sample membership to tree_structure.csv, wrapping the pipeline in a grow_tree(file_disease, file_control, output_folder, ...) function mirroring the BoostDiff run() signature (with min_samples_leaf, min_samples_split, max_depth, normalize, regulators arguments).

## Related tools

- **BoostDiff** (Reference implementation whose run() signature and default hyperparameters (min_samples_leaf=2, min_samples_split=6, max_depth=2, min_samples=15, unit_variance normalization) the grow_tree wrapper mirrors) — github.com/scibiome/boostdiff_inference
- **pandas** (Loads the two tab-separated expression matrices with rows-as-genes, columns-as-samples and the 'Gene' first column)
- **numpy** (Supports the variance-reduction scans over genes g' and thresholds tau used to compute DVI at each candidate split)
- **Python** (Implements the recursive partitioning, the 5-tuple node representation, and the grow_differential_tree.py wrapper script)

## Examples

```
tree = grow_tree('../data/disease_exprs.txt', '../data/control_exprs.txt', '../outputs_tree', min_samples_leaf=2, min_samples_split=6, max_depth=2, normalize='unit_variance', regulators='all')
```

## Evaluation signals

- The script runs end-to-end on a sample disease/control expression-matrix pair without error
- The emitted tree_structure.csv contains records with fields for the split gene, both condition-specific thresholds (tau_D* and tau_C*), and node sample sets
- Every internal node stores two distinct thresholds (tau_D*, tau_C*) rather than a single shared threshold, reflecting the DVI skew rationale
- All terminal leaves respect min_samples_leaf=2, every split respects min_samples_split=6, and tree depth does not exceed max_depth=2
- Each condition retains at least min_samples=15 samples per node before unit_variance-normalized splitting is attempted

## Limitations

- At least 15 samples per condition are required (default min_samples=15), and ideally 30+ per condition for reliable results
- The single-tree procedure is a base learner; the full BoostDiff method wraps it in a modified AdaBoost ensemble and variable-importance aggregation
- A single threshold optimizing DVI would skew the expression distributions of the two conditions to opposite sides of the split, hence the dual-threshold design
- BoostDiff was tested on Linux; very large datasets (e.g. the Crohn's disease dataset GSE126124) require a high-performance computing cluster

## Evidence

- [abstract] To build the differential trees, we propose differential variance improvement as a novel splitting criterion.: "To build the differential trees, we propose differential variance improvement as a"
- [readme] BoostDiff modifies regression trees to use differential variance improvement (DVI) as the novel splitting criterion.: "BoostDiff modifies regression trees to use differential variance improvement (DVI) as the novel splitting criterion."
- [readme] max_depth parameter default and stump behavior.: "the maximum depth of a differential tree; max_depth=0 means tree stumps will be built"
- [readme] Input format requirements for the two condition-specific expression matrices.: "The rows correspond to genes, while the columns correspond to samples."
- [methods] Two separate per-condition thresholds are needed because one threshold optimizing DVI would skew the conditions to opposite sides of the split.: [evidence span withheld — anchor status: not_found]
