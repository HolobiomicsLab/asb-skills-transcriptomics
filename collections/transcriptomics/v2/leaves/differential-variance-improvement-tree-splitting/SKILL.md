---
name: differential-variance-improvement-tree-splitting
description: Use when two matched gene expression matrices (disease/target and control/baseline,
  rows = genes, columns = samples, first column 'Gene', tab-separated) are available
  and the goal is to find condition-specific predictor genes via tree-based differential
  network inference.
when_to_use_negative:
- Only one condition's expression data is available — DVI requires paired disease
  and control sample sets from the same feature set
- Fewer than min_samples=15 samples per condition (BoostDiff's stated minimum; ~30
  per condition is the ideal input)
- The goal is standard differential expression analysis rather than finding regulators
  whose predictive (tree-based) relationship with a target gene changes between conditions
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
tools:
- name: BoostDiff
  role: Implements the differential tree learner with the DVI splitting criterion
    and the modified AdaBoost ensemble wrapping it; invoked via boostdiff.main_boostdiff.BoostDiff.run()
  repo: https://github.com/scibiome/boostdiff_inference
- name: Python
  role: Language in which the 5-tuple node representation, DVI split scans, and recursive
    partitioning are implemented (pandas/numpy for matrix I/O and variance scans)
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:6c80a0b482e72440c3c78548ab31c3d870e338cf117195350ce1c0a5b6a790f6
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# differential-variance-improvement-tree-splitting

## Summary

Grow differential regression trees that split nodes using differential variance improvement (DVI) — the difference between the best variance reduction achieved in the target condition and in the baseline condition — to identify regulators whose predictive relationship with a gene changes between two biological conditions.

## When to use

Use when two matched gene expression matrices (disease/target and control/baseline, rows = genes, columns = samples, first column 'Gene', tab-separated) are available and the goal is to find condition-specific predictor genes via tree-based differential network inference.

## When NOT to use

- Only one condition's expression data is available — DVI requires paired disease and control sample sets from the same feature set
- Fewer than min_samples=15 samples per condition (BoostDiff's stated minimum; ~30 per condition is the ideal input)
- The goal is standard differential expression analysis rather than finding regulators whose predictive (tree-based) relationship with a target gene changes between conditions

## Inputs

- Disease/target expression matrix (tab-separated, rows = genes, columns = samples, first column named 'Gene')
- Control/baseline expression matrix (tab-separated, same feature set as the target matrix)
- Optional regulator list ('all' or a list of gene names to be used as predictors)

## Outputs

- Grown differential tree with 5-tuple nodes (SD, SC, split gene g*, τD*, τC*)
- tree_structure.csv recording split gene, both condition-specific thresholds, and node sample sets
- Per-node differential variance improvement (DVI) scores used for variable-importance edge ranking

## How to apply

Represent each tree node as the 5-tuple (SD, SC, split gene g*, disease threshold τD*, control threshold τC*), keeping separate per-condition sample sets and thresholds because a single shared threshold would skew the two expression distributions to opposite sides of the split. At each candidate split, compute DVI(g') = maxτ VarRed(g',τ,SD) − maxτ VarRed(g',τ,SC) via variance-reduction scans over genes and thresholds in each condition, and select the gene/threshold pair maximizing DVI. Recursively partition both conditions' sample sets at their respective thresholds, terminating per BoostDiff defaults: min_samples_leaf=2, min_samples_split=6, max_depth=2 (max_depth=0 yields stumps), with min_samples=15 per condition and 'unit_variance' normalization applied beforehand. In the boosting context (task_002), only target-condition samples are re-weighted across AdaBoost.R2 iterations while baseline samples retain uniform weight. Serialize each internal node's split gene, both condition-specific thresholds, and node sample membership (e.g., to tree_structure.csv) so the tree is auditable.

## Related tools

- **BoostDiff** (Implements the differential tree learner with the DVI splitting criterion and the modified AdaBoost ensemble wrapping it; invoked via boostdiff.main_boostdiff.BoostDiff.run()) — https://github.com/scibiome/boostdiff_inference
- **Python** (Language in which the 5-tuple node representation, DVI split scans, and recursive partitioning are implemented (pandas/numpy for matrix I/O and variance scans))

## Examples

```
from boostdiff.main_boostdiff import BoostDiff; model = BoostDiff(); model.run(file_disease, file_control, output_folder, n_estimators=50, n_features=2500, n_subsamples=20, max_depth=2)
```

## Evaluation signals

- tree_structure.csv contains records with fields for split gene, both condition-specific thresholds (τD*, τC*), and node sample sets
- Every selected split maximizes DVI(g') = maxτ VarRed(g',τ,SD) − maxτ VarRed(g',τ,SC), i.e., the target-condition variance reduction exceeds the baseline-condition variance reduction at the chosen gene/threshold pair
- Termination respects the defaults: min_samples_leaf=2, min_samples_split=6, max_depth=2, and min_samples=15 per condition
- Both input matrices passed validation: same feature set, first column named 'Gene', unit_variance normalization applied before growing
- The grown tree runs end-to-end on a sample expression-matrix pair without skewing the two conditions' expression distributions to opposite sides of any split

## Limitations

- Using a single shared split threshold would skew the expression distributions of the two conditions to opposite sides of the split, hence the dual-threshold (τD*, τC*) design
- For real transcriptomics datasets, a low number of base learners (e.g., n_estimators=50) is recommended to avoid overfitting; the Crohn's disease dataset was too large for a desktop run and required an HPC cluster
- BoostDiff was tested on Linux only

## Evidence

- [readme] DVI splitting criterion as the core novelty: "BoostDiff modifies regression trees to use differential variance improvement (DVI) as the novel splitting criterion."
- [methods] 5-tuple node structure and dual thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] Boosting context: target-condition weighting: "BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition."
- [readme] Termination parameters: "`min_samples_split`: *int (default = 6)*, the minimum no. of samples required to split an internal node"
