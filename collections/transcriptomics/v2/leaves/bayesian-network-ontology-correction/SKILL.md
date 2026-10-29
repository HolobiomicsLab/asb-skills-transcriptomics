---
name: bayesian-network-ontology-correction
description: Use when benchmarking generative Bayesian-network correction (BNC, URSA)
  of per-term Cell Ontology classifier probabilities against discriminative baselines
  (independent classifiers, IR, TPR) on human RNA-seq data.
when_to_use_negative:
- 'You need accurate out-of-the-box cell type annotation: BNC significantly degraded
  validation-set performance versus independent classifiers, IR, and TPR, so use CellO''s
  default ''IR'' algorithm instead of reproducing the BNC comparison.'
- 'Your data are not human primary cell RNA-seq: CellO''s classifiers were trained
  on human primary bulk RNA-seq from the SRA, and no comprehensive curated mouse training
  set exists yet.'
- 'Your goal is calibrated binary yes/no cell type decisions: the paper notes performance
  with F1-score lags average precision, so average-precision-based ontology-correction
  comparisons do not certify decision quality.'
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0089
tools:
- name: URSA
  role: Provides the Bayesian network correction (BNC) implementation that combines
    per-term classifier probabilities over the ontology graph; the generative scheme
    being benchmarked
- name: CellO
  role: Supplies the independent per-term logistic regression classifiers and the
    IR ('IR' default algorithm) baseline against which BNC is compared on the validation
    partition
  repo: https://github.com/deweylab/CellO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:522180aec4a7a299bed1d3e2c300ec800fe204932419ebcaed02a0051281952b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deweylab/CellO
  repo_url: https://github.com/deweylab/CellO
  tools:
  - CellO
  - URSA
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# bayesian-network-ontology-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the generative Bayesian network correction (BNC, as implemented in URSA) to combine per-cell-type classifier probabilities over the Cell Ontology graph, benchmarked in CellO against independent classifiers and discriminative corrections (IR, TPR). In the CellO validation, BNC significantly degraded average precision, making it chiefly a cautionary baseline showing discriminative ontology correction is preferred.

## When to use

Use when benchmarking generative Bayesian-network correction (BNC, URSA) of per-term Cell Ontology classifier probabilities against discriminative baselines (independent classifiers, IR, TPR) on human RNA-seq data.

## When NOT to use

- You need accurate out-of-the-box cell type annotation: BNC significantly degraded validation-set performance versus independent classifiers, IR, and TPR, so use CellO's default 'IR' algorithm instead of reproducing the BNC comparison.
- Your data are not human primary cell RNA-seq: CellO's classifiers were trained on human primary bulk RNA-seq from the SRA, and no comprehensive curated mouse training set exists yet.
- Your goal is calibrated binary yes/no cell type decisions: the paper notes performance with F1-score lags average precision, so average-precision-based ontology-correction comparisons do not certify decision quality.

## Inputs

- Pre-training/validation partitioned human primary bulk RNA-seq expression data (Zenodo deposit 10.5281/zenodo.4289064)
- Cell type labels mapped to Cell Ontology terms
- Per-cell-type probability outputs from CellO's independent per-term logistic regression classifiers

## Outputs

- Per-cell-type BNC-corrected posterior probabilities on the validation partition
- Per-cell-type average precision values for all four schemes (figure3_average_precision.csv)
- Per-cell-type precision-recall curves overlaying IR, TPR, BNC, and independent classifiers (figure3_pr_curves.png)

## How to apply

Train independent per-term logistic regression classifiers on the pre-training partition of human primary bulk RNA-seq samples (Zenodo deposit 10.5281/zenodo.4289064) with cell type labels mapped to Cell Ontology terms, following CellO's hierarchical classification framework. Treat the ontology as a Bayesian network and apply the Bayesian network correction (BNC) as implemented in URSA (Lee et al., 2013) to combine parent and child probabilities into consistent posteriors over the Cell Ontology graph. Compute per-cell-type probabilities on the held-out validation partition for BNC alongside three baselines: uncorrected independent classifiers, isotonic regression (CellO's default 'IR' algorithm), and True Path Rule (TPR) propagation to ontology ancestors. Score each scheme with average precision and overlay per-cell-type precision-recall curves. The expected outcome, per the paper, is that BNC significantly degrades average precision relative to the discriminative schemes, i.e., 'discriminative methods outperformed the generative BNC approach implemented in URSA', so this skill is mainly used as a baseline/negative control justifying CellO's default IR correction.

## Related tools

- **URSA** (Provides the Bayesian network correction (BNC) implementation that combines per-term classifier probabilities over the ontology graph; the generative scheme being benchmarked)
- **CellO** (Supplies the independent per-term logistic regression classifiers and the IR ('IR' default algorithm) baseline against which BNC is compared on the validation partition) — https://github.com/deweylab/CellO

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- IR and TPR average precision values match the independent-classifier baseline within a small tolerance (no degradation), per the workflow's validation step.
- BNC average precision is lower than the independent-classifier baseline, reproducing the paper's finding that discriminative methods outperformed the generative BNC approach.
- Overlaid precision-recall curves (figure3_pr_curves.png) show BNC curves below the IR/TPR/independent baselines for most cell types.
- All corrected outputs remain valid probabilities in [0,1] keyed to Cell Ontology terms, and figure3_average_precision.csv contains one average-precision value per cell type per scheme.

## Limitations

- BNC significantly degraded performance relative to the independent classifiers, IR, and TPR in the CellO validation benchmark, so it should not be used as a production correction.
- CellO's classifiers are trained only on nearly all human primary cell bulk RNA-seq data in the SRA; extension to other species (e.g., mouse) awaits curated training data.
- Classifier calibration gaps remain: CellO performance measured with F1-score lags its average-precision performance.

## Evidence

- [discussion] Discriminative methods outperformed the generative BNC approach implemented in URSA: "we found that discriminative methods outperformed the generative BNC approach implemented in URSA (Lee et al., 2013)"
- [intro] URSA is the only ontology-graph method: "the only method that utilizes the graph structure of an ontology for the task at hand is URSA (Lee et al., 2013)"
- [methods] CellO makes hierarchical predictions against the Cell Ontology: "CellO makes hierarchical predictions against the [Cell Ontology](http://www.obofoundry.org/ontology/cl.html)."
- [readme] Training data are human primary bulk RNA-seq from SRA: "These classifiers were trained on nearly all of the human primary cell, bulk RNA-seq data in the [Sequence Read Archive](https://www.ncbi.nlm.nih.gov/sra)."
