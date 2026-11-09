---
name: deep-neural-network-training
description: Use when you need to train a fixed-length read classifier (e.g., 48 bp
  fragments of viral vs. human transcripts) with binary labels, or reproduce viRNAtrap's
  reported test-set metrics (AUROC 0.81, AUPRC 0.82, accuracy 0.71, recall 0.83, precision
  0.67, F1 0.74).
when_to_use_negative:
- Sequences come from organisms other than human or virus — the paper states model
  predictions for such sequences are not defined and models for other organisms are
  left to future work
- Reads are already mapped to the reference human genome — the viRNAtrap workflow
  applies the model only to unmapped RNAseq reads to reduce running time
- The task requires alignment-based identification or typing of already-characterized
  viral genomes — blastn/blastx against curated viral databases is the appropriate
  step
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: viRNAtrap
  role: Alignment-free package whose Tensorflow component predicts whether fixed-length
    (48 bp) unmapped RNAseq reads are viral and assembles predicted viral reads into
    contigs
  repo: github.com/AuslanderLab/virnatrap
- name: TensorFlow
  role: Backend framework for building and training the 1D-convolutional deep learning
    model that discriminates viral from human 48 bp fragments
- name: Keras
  role: High-level API used to define the convolutional network with three fully connected
    layers, compile it with binary cross-entropy, and save HDF5 weights
  repo: keras-team/keras
- name: NumPy
  role: Array library used to handle the encoded 48 bp fragment matrices during training
    and evaluation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:604a2c05a8a4724302c4d0b8ab6f684bed7c509e1a88688443c5aaf9491569a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  - NumPy
  - TensorFlow
  - deep learning model
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# deep-neural-network-training

## Summary

Train a 1D-convolutional deep learning model in Keras/TensorFlow to classify 48 bp RNAseq fragments as viral or human, as done in viRNAtrap, an alignment-free pipeline for identifying viral reads and assembling viral contigs.

## When to use

Use when you need to train a fixed-length read classifier (e.g., 48 bp fragments of viral vs. human transcripts) with binary labels, or reproduce viRNAtrap's reported test-set metrics (AUROC 0.81, AUPRC 0.82, accuracy 0.71, recall 0.83, precision 0.67, F1 0.74).

## When NOT to use

- Sequences come from organisms other than human or virus — the paper states model predictions for such sequences are not defined and models for other organisms are left to future work
- Reads are already mapped to the reference human genome — the viRNAtrap workflow applies the model only to unmapped RNAseq reads to reduce running time
- The task requires alignment-based identification or typing of already-characterized viral genomes — blastn/blastx against curated viral databases is the appropriate step

## Inputs

- Positive (viral) transcript sequences
- Negative (human) transcript sequences
- 48 bp fragments segmented from transcripts, labeled viral/human and divided into training and test sets
- Pretrained checkpoint name encoding hyperparameters: model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5

## Outputs

- Trained Keras/TensorFlow model weights in HDF5 format (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)
- Held-out test-set metrics: AUROC, AUPRC, accuracy, recall, precision, F1 score
- Per-read viral probability scores usable for assembling predicted viral contigs

## How to apply

Segment positive (viral) and negative (human) transcript sequences into fixed 48 bp fragments and split them into training and test partitions. Build the 1D-convolutional network with three fully connected layers in Keras/TensorFlow, setting hyperparameters encoded in the pretrained checkpoint name model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5: learning rate 0.005, pooling 5, embedding 25, L2 regularization 0.02, and 64 units. Train with binary cross-entropy loss on the labeled 48 bp viral/human fragments. Evaluate on the held-out test partition, computing AUROC, AUPRC, accuracy, recall, precision, and F1, and save the fitted weights in HDF5 format. The rationale is to discriminate viral RNAseq reads without alignment, so that model scores can later be used to assemble predicted viral contigs from unmapped reads.

## Related tools

- **viRNAtrap** (Alignment-free package whose Tensorflow component predicts whether fixed-length (48 bp) unmapped RNAseq reads are viral and assembles predicted viral reads into contigs) — github.com/AuslanderLab/virnatrap
- **TensorFlow** (Backend framework for building and training the 1D-convolutional deep learning model that discriminates viral from human 48 bp fragments)
- **Keras** (High-level API used to define the convolutional network with three fully connected layers, compile it with binary cross-entropy, and save HDF5 weights) — keras-team/keras
- **NumPy** (Array library used to handle the encoded 48 bp fragment matrices during training and evaluation)

## Examples

```
virnatrap-predict --input input_fastq/ --output output_contigs/
```

## Evaluation signals

- Held-out test AUROC ≈ 0.81 and AUPRC ≈ 0.82, matching the published reference within reasonable tolerance
- Held-out test accuracy ≈ 0.71, recall ≈ 0.83, precision ≈ 0.67, and F1 ≈ 0.74
- Saved weights file name matches the hyperparameter encoding: model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5
- Prediction scores on the packaged demo fall within small least-significant-digit differences of expected_output/output_py.txt (e.g., contig2[0.8009399] vs 0.8009398)
- Model outputs one viral/human probability per fixed 48 bp read, suitable for downstream contig assembly

## Limitations

- Sensitivity is only guaranteed when more than five viral reads are present in a sample
- Not possible to model all contaminants of viruses that may have infected samples during laboratory procedures
- Model predictions for sequences from organisms other than human/viral are not defined; training models for other organisms is left to future work
- The pretrained model is specific to 48 bp reads; reads of other lengths are outside the trained regime
- Hyperparameters are only recoverable via the checkpoint-name convention (lr 0.005, pool 5, emb 25, l2 0.02, 64 units); deviations may break metric reproduction

## Evidence

- [abstract] Deep learning model trained to discriminate viral RNAseq reads: "which is based on a deep learning model trained to discriminate viral RNAseq reads"
- [intro] 48 bp fragment training/test design: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [readme] TensorFlow model on fixed-length 48bp reads: "Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads."
- [readme] Model applied to unmapped reads: "Use the Tensorflow model to predict which of the unmapped reads are viral; this is performed by the included software."
