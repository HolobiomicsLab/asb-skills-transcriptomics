---
name: isotonic-regression-probability-calibration
description: Use when per-cell-type binary classifier outputs over a Cell Ontology
  term set violate the 'is a' hierarchy (child probability exceeds parent), and calibrated
  probabilities are needed for 10x/TSV/CSV/HDF5 RNA-seq expression matrices.
when_to_use_negative:
- A generative Bayesian network correction (BNC/URSA) is required — the paper found
  BNC significantly degraded validation average precision relative to IR and the independent
  classifiers
- The task is non-hierarchical flat classification without ontology structure to calibrate
  against
- Training data are mouse or non-human samples — CellO was trained only on human primary
  SRA bulk RNA-seq and no curated mouse training set exists yet
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: Python package implementing the IR hierarchical classification algorithm and
    per-term linear classifiers over the Cell Ontology; installed via pip as cello-classify
  repo: https://github.com/deweylab/CellO
- name: URSA
  role: Source of the BNC (Bayesian network correction) baseline used to evaluate
    whether IR calibration preserves or degrades average precision
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:102c11c3900365d872ac6a416ab1dd14f55bb6fdd14d6251126907939899e35e
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# isotonic-regression-probability-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply isotonic regression (IR), CellO's default hierarchical algorithm, to calibrate independent one-vs-rest logistic regression probabilities against the Cell Ontology so child probabilities respect the 'is a' parent-child ordering, while avoiding the performance-degrading Bayesian network correction (BNC).

## When to use

Use when per-cell-type binary classifier outputs over a Cell Ontology term set violate the 'is a' hierarchy (child probability exceeds parent), and calibrated probabilities are needed for 10x/TSV/CSV/HDF5 RNA-seq expression matrices.

## When NOT to use

- A generative Bayesian network correction (BNC/URSA) is required — the paper found BNC significantly degraded validation average precision relative to IR and the independent classifiers
- The task is non-hierarchical flat classification without ontology structure to calibrate against
- Training data are mouse or non-human samples — CellO was trained only on human primary SRA bulk RNA-seq and no curated mouse training set exists yet

## Inputs

- Gene expression matrix in TSV, CSV, HDF5, or 10x format (genes x cells or cells x genes via -r flag), with units COUNTS, CPM, LOG1_CPM, TPM, or LOG1_TPM
- Human primary bulk RNA-seq training set from the SRA (Zenodo deposit 10.5281/zenodo.4289064) for model training/benchmarking
- Cell Ontology 'is a' parent-child edge list for inconsistency checking

## Outputs

- <output_prefix>.probability.tsv: N x M table of hierarchy-calibrated per-cell-type probabilities
- <output_prefix>.binary.tsv: N xM binary decision matrix
- inconsistency_rates.csv: fraction of inconsistent edges, fraction of samples with >=1 inconsistent edge, fraction with >=1 severely inconsistent edge (child exceeds parent by >=0.25)

## How to apply

Train independent binary classifiers (one per Cell Ontology term) on human primary bulk RNA-seq data from the Sequence Read Archive, producing an N x M probability matrix over cells and cell types. Apply isotonic regression as the hierarchical correction ('IR' is CellO's default -a algorithm, distinct from 'CLR' cascaded logistic regression) to enforce monotonicity along 'is a' edges so a child's calibrated probability never exceeds its parent's. If benchmarking correction schemes, also compare against True Path Rule (TPR) ancestor propagation — which behaves similarly to IR and the independent baseline — and Bayesian network correction (BNC) from URSA, which significantly degraded average precision. Quantify residual inconsistency by flagging edges where the child probability exceeds the parent's, and severe inconsistency where the child exceeds by >= 0.25; IR should drive the ~12.1% inconsistent-edge rate of independent classifiers toward zero without degrading precision-recall performance.

## Related tools

- **CellO** (Python package implementing the IR hierarchical classification algorithm and per-term linear classifiers over the Cell Ontology; installed via pip as cello-classify) — https://github.com/deweylab/CellO
- **URSA** (Source of the BNC (Bayesian network correction) baseline used to evaluate whether IR calibration preserves or degrades average precision)

## Examples

```
cello_predict -a IR -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- After IR correction, no child probability exceeds its parent's on any Cell Ontology 'is a' edge (inconsistent-edge fraction drops from the ~12.1% of independent classifiers)
- Validation-set average precision of IR matches the independent-classifier baseline within a small tolerance (no degradation), mirroring the TPR result
- Fraction of samples with at least one severely inconsistent edge (child > parent by >=0.25) falls from ~34% toward zero
- Output files follow the expected schema: <prefix>.probability.tsv is N cells x M cell types with values in [0,1] and <prefix>.binary.tsv is 0/1
- The 'IR' algorithm is the selected -a option in cello_predict and runs without gene-set mismatch errors (input genes match the pretrained classifier's genes)

## Limitations

- The IR-corrected model's F1-score lags its average precision; calibration for binary yes-no decisions remains imperfect
- Inconsistencies can arise for cell types that share a parent via 'is a' but have divergent evolutionary origins
- Classifier calibration is expected to improve only as more training data per cell type become available; gene sets of the input must exactly match the pretrained classifier's genes or a new model must be trained with -t

## Evidence

- [discussion] independent classifiers can be inconsistent across 'is a' edges: [evidence span withheld — anchor status: not_found]
