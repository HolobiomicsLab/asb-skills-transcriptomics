---
name: tensorflow-keras-model-training
description: Use when building a supervised binary classifier (viral vs. human) over
  fixed-length sequence fragments (e.g., 48 bp RNA-seq reads) with TensorFlow/Keras,
  or when retraining baseline models on an identical training split for a fair benchmark.
when_to_use_negative:
- Reads must be classified for organisms other than human/viral — the model's predictions
  for sequences from other organisms are not defined, and the authors defer such models
  to future work
- The task is predicting viral contigs from an already-trained model rather than training
  — use the packaged virnatrap-predict command on *_unmapped.fastq files instead
- Only one model needs to be run with no fair-comparison requirement — retraining
  all baselines on the same split is unnecessary overhead
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: viRNAtrap
  role: Reference pipeline whose precomputed 48 bp TensorFlow model defines the target
    architecture and benchmark training procedure
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: TensorFlow
  role: Backend framework used to build and train the viral-read discrimination model
- name: Keras
  role: High-level API used to define, fit, and save the model as .hdf5
  repo: keras-team/keras
- name: NumPy
  role: Array handling for fragment-encoded training data
- name: DeepVirFinder
  role: Baseline model retrained on the identical 48 bp training fragments for the
    comparison table
- name: DeepViFi
  role: Baseline model retrained on the identical 48 bp training fragments for the
    comparison table
- name: ViraMiner
  role: Baseline model retrained on the identical 48 bp training fragments for the
    comparison table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:3fcc7f38f17d201b7bdeb2451c4e9fc09e38f13f845d1226eb564b749ab5ed1b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tensorflow-keras-model-training

## Summary

Train a TensorFlow/Keras deep learning model to discriminate viral RNAseq reads from human transcripts, as in viRNAtrap: 48 bp fragments of positive (viral) and negative (human) transcripts are split into training/test sets, and the trained model is persisted as an .hdf5 checkpoint for later read prediction and contig assembly.

## When to use

Use when building a supervised binary classifier (viral vs. human) over fixed-length sequence fragments (e.g., 48 bp RNA-seq reads) with TensorFlow/Keras, or when retraining baseline models on an identical training split for a fair benchmark.

## When NOT to use

- Reads must be classified for organisms other than human/viral — the model's predictions for sequences from other organisms are not defined, and the authors defer such models to future work
- The task is predicting viral contigs from an already-trained model rather than training — use the packaged virnatrap-predict command on *_unmapped.fastq files instead
- Only one model needs to be run with no fair-comparison requirement — retraining all baselines on the same split is unnecessary overhead

## Inputs

- Positive (viral) transcripts segmented into 48 bp fragments
- Negative (human) transcripts segmented into 48 bp fragments
- Train/test split definition shared across all compared models
- Hyperparameter settings (learning rate 0.005, pooling 5, embedding 25, L2 0.02, 64 units)

## Outputs

- Trained TensorFlow/Keras model saved as model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5
- Per-read viral/non-viral predictions on the shared 48 bp test set
- Compiled metrics table benchmark_comparison_metrics.csv (accuracy, precision, recall, F1, AUC per model)

## How to apply

Segment positive (viral) and negative (human) transcripts into fixed 48 bp fragments and divide them into shared training and test sets so that any comparison model sees identical data. Build the Tensorflow model with the viRNAtrap-style architecture, saving the fitted network as an .hdf5 file following the naming pattern model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5, which encodes learning rate 0.005, pooling 5, embedding 25, L2 0.02, and 64 units as hyperparameters. If the goal is a benchmark, retrain DeepVirFinder, DeepViFi, and ViraMiner on the exact same 48 bp training fragments following each method's own original instructions, then run all models on the shared test set. Compute accuracy, precision, recall, F1, and AUC per model and compile them into a single metrics table (benchmark_comparison_metrics.csv); in the source study viRNAtrap won all measures except precision, where DeepVirFinder was best.

## Related tools

- **viRNAtrap** (Reference pipeline whose precomputed 48 bp TensorFlow model defines the target architecture and benchmark training procedure) — https://github.com/AuslanderLab/virnatrap.git
- **TensorFlow** (Backend framework used to build and train the viral-read discrimination model)
- **Keras** (High-level API used to define, fit, and save the model as .hdf5) — keras-team/keras
- **NumPy** (Array handling for fragment-encoded training data)
- **DeepVirFinder** (Baseline model retrained on the identical 48 bp training fragments for the comparison table)
- **DeepViFi** (Baseline model retrained on the identical 48 bp training fragments for the comparison table)
- **ViraMiner** (Baseline model retrained on the identical 48 bp training fragments for the comparison table)

## Evaluation signals

- The saved checkpoint exists at model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5 and loads in Keras without error
- Training used the same 48 bp train/test split for all models, so reproduced metric ranking matches the reported finding: viRNAtrap best on all measures except precision, where DeepVirFinder is best
- benchmark_comparison_metrics.csv contains accuracy, precision, recall, F1, and AUC rows for viRNAtrap, DeepVirFinder, DeepViFi, ViraMiner, and the off-the-shelf Seq2Seq baseline
- All fragment inputs are exactly 48 bp in length, matching the precomputed model's expected read length

## Limitations

- The model was precomputed for 48 bp reads only; other read lengths are out of scope
- Model predictions for sequences from organisms other than human/viral are not defined; training models for other organisms is left to future work
- It is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures
- viroRNAtrap sensitivity is only guaranteed when more than five viral reads are present

## Evidence

- [intro] training on 48 bp fragments: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [intro] framework purpose: "we develop a framework, named viRNAtrap, that employs a deep learning model to accurately distinguish viral reads from RNA sequencing"
- [readme] TensorFlow model on fixed-length reads: "Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads."
- [intro] comparison scope: "We compared the performance of this model to previous models trained to identify viruses, namely DeepViFi16, DeepVirFinder15, ViraMiner21"
