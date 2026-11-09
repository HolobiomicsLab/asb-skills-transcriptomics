---
name: binary-classifier-metric-evaluation
description: Use when a trained binary classifier (e.g. a Keras/TensorFlow viral-read
  model) must be scored on a labeled held-out test partition and compared to baselines
  under identical data.
when_to_use_negative:
- Input sequences come from organisms other than human or virus — the paper states
  model predictions for such sequences are not defined and models for other organisms
  are left to future work
- Reads are not fixed-length 48 bp fragments or have not been mapped/removed against
  the human genome (hg19) and phiX contaminant first — the model was precomputed for
  48 bp reads on unmapped reads
- Reads are short and detection requires no more than approximately five viral reads
  — the paper only claims sensitivity if more than five viral reads are present
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: viRNAtrap
  role: Framework supplying the trained TensorFlow/Keras viral-read classifier that
    is evaluated and benchmarked on the 48 bp test set
  repo: github.com/AuslanderLab/virnatrap
- name: TensorFlow
  role: Deep learning backend used to build, train, and evaluate the viral-read prediction
    model
- name: Keras
  role: API for constructing the 1D-convolutional network and saving weights in .hdf5
    format for evaluation
  repo: https://github.com/keras-team/keras
- name: NumPy
  role: Array handling for fragment labels, scores, and metric computation
- name: DeepVirFinder
  role: Baseline viral-identification classifier retrained on the same 48 bp fragments
    for comparison (best on precision)
- name: DeepViFi
  role: Baseline viral-identification classifier retrained on the same 48 bp fragments
    for comparison
- name: ViraMiner
  role: Baseline viral-identification classifier retrained on the same 48 bp fragments
    for comparison
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:f619ac898bbd894dcffbf8e5f8f0074fe5efacaadf03f7acd88a4c7987333df3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepViFi
  - DeepVirFinder
  - Keras
  - NumPy
  - TensorFlow
  - ViraMiner
  - viRNAtrap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# binary-classifier-metric-evaluation

## Summary

Evaluate a binary viral-vs-human read classifier by computing AUROC, AUPRC, accuracy, recall, precision, and F1 on a held-out test set of fixed-length (48 bp) RNA-seq fragments, and compare against baseline classifiers on the identical test set.

## When to use

Use when a trained binary classifier (e.g. a Keras/TensorFlow viral-read model) must be scored on a labeled held-out test partition and compared to baselines under identical data.

## When NOT to use

- Input sequences come from organisms other than human or virus — the paper states model predictions for such sequences are not defined and models for other organisms are left to future work
- Reads are not fixed-length 48 bp fragments or have not been mapped/removed against the human genome (hg19) and phiX contaminant first — the model was precomputed for 48 bp reads on unmapped reads
- Reads are short and detection requires no more than approximately five viral reads — the paper only claims sensitivity if more than five viral reads are present

## Inputs

- Labeled 48 bp RNA-seq fragments (viral positives, human negatives) split into training and test partitions
- Trained Keras/TensorFlow model weights saved as model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5
- Retrained baseline models (DeepVirFinder, DeepViFi, ViraMiner, off-the-shelf Seq2Seq) fit on the same 48 bp training fragments

## Outputs

- Per-read viral/non-viral predictions or scores on the shared 48 bp test set
- Metrics table (benchmark_comparison_metrics.csv) with AUROC, AUPRC, accuracy, recall, precision, and F1 for each model
- Fitted model weights in HDF5 format (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)

## How to apply

Segment positive (viral) and negative (human) transcripts into fixed 48 bp fragments and split them into training and test partitions. Train (or load) the 1D-convolutional Keras/TensorFlow model with the reference hyperparameters encoded in the checkpoint name model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5 — learning rate 0.005, pooling 5, embedding 25, L2 regularization 0.02, 64 units — using binary cross-entropy loss. Run the trained model on the held-out 48 bp test set to obtain per-read viral/non-viral scores, then compute AUROC, AUPRC, accuracy, recall, precision, and F1. For a fair benchmark, retrain baselines (DeepVirFinder, DeepViFi, ViraMiner, off-the-shelf Seq2Seq) on the exact same 48 bp training fragments and evaluate all models on the same test set, compiling the metrics into a single table (e.g. benchmark_comparison_metrics.csv). The rationale is that all methods see identical training and test data, so metric differences reflect modeling rather than data-processing differences.

## Related tools

- **viRNAtrap** (Framework supplying the trained TensorFlow/Keras viral-read classifier that is evaluated and benchmarked on the 48 bp test set) — github.com/AuslanderLab/virnatrap
- **TensorFlow** (Deep learning backend used to build, train, and evaluate the viral-read prediction model)
- **Keras** (API for constructing the 1D-convolutional network and saving weights in .hdf5 format for evaluation) — https://github.com/keras-team/keras
- **NumPy** (Array handling for fragment labels, scores, and metric computation)
- **DeepVirFinder** (Baseline viral-identification classifier retrained on the same 48 bp fragments for comparison (best on precision))
- **DeepViFi** (Baseline viral-identification classifier retrained on the same 48 bp fragments for comparison)
- **ViraMiner** (Baseline viral-identification classifier retrained on the same 48 bp fragments for comparison)

## Examples

```
virnatrap-predict --input input_fastq/ --output output_contigs/
```

## Evaluation signals

- Held-out test metrics reproduce the published reference values within reasonable tolerance: AUROC 0.81, AUPRC 0.82, accuracy 0.71, recall 0.83, precision 0.67, F1 0.74
- Reproduced benchmark ranking matches Fig. 1b/c: viRNAtrap outperforms all baselines on every measure except precision, where DeepVirFinder is best
- Saved weights follow the naming pattern model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5, encoding the hyperparameters actually used
- All models (including baselines) were evaluated on the identical 48 bp test set, so the metrics table rows are directly comparable

## Limitations

- Comparison baselines must be retrained on the same 48 bp training fragments following each method's own original instructions; reusing their published models on different data breaks the comparison
- Metrics are specific to the viral-vs-human 48 bp fragment task; the authors note it is not possible to model all laboratory contaminants, and sensitivity is only guaranteed when more than five viral reads are present
- DeepVirFinder's superior precision despite lower overall performance means no single model dominates all measures — selection depends on which metric matters for the downstream application

## Evidence

- [intro] positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [intro] We compared the performance of this model to previous models trained to identify viruses, namely DeepViFi16, DeepVirFinder15, ViraMiner21: "We compared the performance of this model to previous models trained to identify viruses, namely DeepViFi16, DeepVirFinder15, ViraMiner21"
- [readme] Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads.: "Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads."
- [discussion] Future work could train models to identify viruses from a variety of other organisms: "Future work could train models to identify viruses from a variety of other organisms"
