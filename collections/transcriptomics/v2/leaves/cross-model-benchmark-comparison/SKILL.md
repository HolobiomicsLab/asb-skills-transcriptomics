---
name: cross-model-benchmark-comparison
description: Use when multiple virus-read-classification models must be compared fairly
  on identical RNA-seq fragment data, e.g. to reproduce the finding that viRNAtrap
  beats baselines on all measures except precision, where DeepVirFinder wins.
when_to_use_negative:
- The goal is de novo viral discovery or contig assembly from unmapped RNA-seq reads
  rather than classifier benchmarking (that is viRNAtrap's prediction/assembly pipeline,
  not this comparison task).
- Models cannot be retrained on the same fragments — comparing models trained on different
  training data violates the shared-test-set design and makes the metrics non-comparable.
- Input reads are not fixed-length 48 bp fragments; the pretrained viRNAtrap model
  and the benchmark protocol are specific to 48 bp read lengths.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0781
tools:
- name: viRNAtrap
  role: Provides the TensorFlow/Keras deep learning model being benchmarked and the
    48 bp read-length convention; model saved as .hdf5 in the model/ directory.
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: DeepVirFinder
  role: Comparison baseline retrained on the same 48 bp fragments; expected to achieve
    the best precision in the reproduced ranking.
- name: DeepViFi
  role: Comparison baseline retrained on the same 48 bp fragments following its original
    training instructions.
- name: ViraMiner
  role: Comparison baseline retrained on the same 48 bp fragments following its original
    training instructions.
- name: TensorFlow
  role: Backend for building and training the viRNAtrap-style classifier used in the
    comparison.
- name: Keras
  role: High-level API for the viRNAtrap deep learning model training and evaluation.
  repo: keras-team/keras
- name: NumPy
  role: Numerical handling of fragment sequences and prediction scores during metric
    computation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:de714d62371bb8195218d823eb1de872f41a2a9eb794ceeba6b6d1468c744df3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - DeepViFi
  - DeepVirFinder
  - Keras
  - NumPy
  - TensorFlow
  - ViraMiner
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-model-benchmark-comparison

## Summary

Retrain viRNAtrap and competing virus-identification deep learning models (DeepVirFinder, DeepViFi, ViraMiner, off-the-shelf Seq2Seq) on the same 48 bp RNA-seq fragments and evaluate them on a shared test set to produce a single head-to-head metrics table (accuracy, precision, recall, F1, AUC), as in the viRNAtrap paper's benchmark comparison.

## When to use

Use when multiple virus-read-classification models must be compared fairly on identical RNA-seq fragment data, e.g. to reproduce the finding that viRNAtrap beats baselines on all measures except precision, where DeepVirFinder wins.

## When NOT to use

- The goal is de novo viral discovery or contig assembly from unmapped RNA-seq reads rather than classifier benchmarking (that is viRNAtrap's prediction/assembly pipeline, not this comparison task).
- Models cannot be retrained on the same fragments — comparing models trained on different training data violates the shared-test-set design and makes the metrics non-comparable.
- Input reads are not fixed-length 48 bp fragments; the pretrained viRNAtrap model and the benchmark protocol are specific to 48 bp read lengths.

## Inputs

- Positive (viral) transcript sequences segmented into 48 bp fragments (training and test splits)
- Negative (human) transcript sequences segmented into 48 bp fragments (training and test splits)
- Original training instructions/code for DeepVirFinder, DeepViFi, and ViraMiner
- Trained viRNAtrap TensorFlow/Keras model (.hdf5) or hyperparameters (lr=0.005, pool=5, emb=25, l2=0.02)

## Outputs

- benchmark_comparison_metrics.csv — per-model accuracy, precision, recall, F1, and AUC on the shared 48 bp test set
- Per-read viral/non-viral prediction scores for each model on the shared test set

## How to apply

Segment positive (viral) and negative (human) transcripts into 48 bp fragments and divide them into shared training and test sets, since the viRNAtrap model is built for fixed-length 48 bp reads. Train the viRNAtrap-style TensorFlow/Keras model on the 48 bp training fragments and save it as an .hdf5 file following the naming pattern model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5 (learning rate 0.005, pool 5, embedding 25, L2 0.02). Retrain DeepVirFinder, DeepViFi, and ViraMiner on the exact same 48 bp training fragments using each method's own original training instructions so that any performance difference reflects the models rather than the data. Run all four retrained models plus the off-the-shelf Seq2Seq baseline on the shared 48 bp test set to obtain per-read viral/non-viral predictions, then compute accuracy, precision, recall, F1, and AUC for each model. Compile the metrics into a single table (benchmark_comparison_metrics.csv) and check the ranking against the reported result: viRNAtrap outperforms all baselines on all evaluation measures except precision, on which DeepVirFinder outperforms all other methods.

## Related tools

- **viRNAtrap** (Provides the TensorFlow/Keras deep learning model being benchmarked and the 48 bp read-length convention; model saved as .hdf5 in the model/ directory.) — https://github.com/AuslanderLab/virnatrap.git
- **DeepVirFinder** (Comparison baseline retrained on the same 48 bp fragments; expected to achieve the best precision in the reproduced ranking.)
- **DeepViFi** (Comparison baseline retrained on the same 48 bp fragments following its original training instructions.)
- **ViraMiner** (Comparison baseline retrained on the same 48 bp fragments following its original training instructions.)
- **TensorFlow** (Backend for building and training the viRNAtrap-style classifier used in the comparison.)
- **Keras** (High-level API for the viRNAtrap deep learning model training and evaluation.) — keras-team/keras
- **NumPy** (Numerical handling of fragment sequences and prediction scores during metric computation.)

## Examples

```
git clone https://github.com/AuslanderLab/virnatrap.git && cd virnatrap && pip install .
```

## Evaluation signals

- benchmark_comparison_metrics.csv exists and contains accuracy, precision, recall, F1, and AUC for all four models plus the Seq2Seq baseline.
- Reproduced ranking matches Fig. 1b/c: viRNAtrap outperforms DeepVirFinder, DeepViFi, and ViraMiner on all evaluation measures except precision.
- DeepVirFinder achieves the highest precision among all compared methods.
- All models were trained and tested on the exact same 48 bp fragment training/test split (no per-model data leakage or differing splits).
- The viRNAtrap model file follows the documented naming pattern model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5.

## Limitations

- Precision is the one metric where viRNAtrap does not win; DeepVirFinder outperforms all other methods on precision, so a single-metric comparison can be misleading.
- Model predictions for sequences from organisms other than human/viral are not defined; the comparison is scoped to viral-vs-human discrimination.
- Each baseline must follow its own original training instructions, which complicates exact reproducibility across toolchains.

## Evidence

- [intro] comparison finding: "We compared the performance of this model to previous models trained to identify viruses, namely DeepViFi16, DeepVirFinder15, ViraMiner21"
- [intro] fragment construction: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [readme] model definition: "Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads."
- [readme] installation: "git clone https://github.com/AuslanderLab/virnatrap.git"
- [discussion] model scope: "Future work could train models to identify viruses from a variety of other organisms, and, with the viRNAtrap framework, achieve higher sensitivity for viral detection"
