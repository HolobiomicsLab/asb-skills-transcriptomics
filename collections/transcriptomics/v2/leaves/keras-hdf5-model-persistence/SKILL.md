---
name: keras-hdf5-model-persistence
description: Use when a trained Keras/TensorFlow binary classifier's weights must
  be persisted to disk in HDF5 format so it can be reloaded later for inference, metric
  reproduction, or distribution alongside a pipeline.
when_to_use_negative:
- 'Working in Keras 3 with a freshly saved model: the README states the up-to-date
  .keras format should be used, and .h5 is the legacy tf-keras path'
- You need to persist the full inference pipeline (read mapping and 48 bp segmentation
  happen independently of the model), not just network weights
- The network was not built with Keras/TensorFlow (e.g., a PyTorch or JAX model needs
  its own native serialization)
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: keras
  role: Provides the model-saving/weight-persistence API (HDF5 checkpointing) used
    to write and reload the trained viRNAtrap classifier
  repo: https://github.com/fchollet/keras
- name: TensorFlow
  role: Backend in which the viRNAtrap deep learning model is built and trained before
    checkpointing its weights
- name: viRNAtrap
  role: Pipeline whose precomputed Tensorflow model (48bp reads) is the artifact persisted
    as an HDF5 checkpoint
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:179640915a1535eb859d71807559c643b1473916106c4fe3973515a6df086ca2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fchollet/keras
  tools:
  - Keras
  - NumPy
  - TensorFlow
  - deep learning model
  - keras
  - viRNAtrap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# keras-hdf5-model-persistence

## Summary

Checkpoints the weights of a trained Keras/TensorFlow neural network (here, the viRNAtrap 48 bp viral-read classifier) to an HDF5 file whose filename encodes the training hyperparameters, enabling later reload for inference and reproduction of reported test metrics.

## When to use

Use when a trained Keras/TensorFlow binary classifier's weights must be persisted to disk in HDF5 format so it can be reloaded later for inference, metric reproduction, or distribution alongside a pipeline.

## When NOT to use

- Working in Keras 3 with a freshly saved model: the README states the up-to-date .keras format should be used, and .h5 is the legacy tf-keras path
- You need to persist the full inference pipeline (read mapping and 48 bp segmentation happen independently of the model), not just network weights
- The network was not built with Keras/TensorFlow (e.g., a PyTorch or JAX model needs its own native serialization)

## Inputs

- Trained Keras/TensorFlow model object (1D-convolutional network with three fully connected layers, trained on 48 bp viral/human fragments)
- Hyperparameter configuration (learning rate 0.005, pooling 5, embedding 25, L2 0.02, 64 units)

## Outputs

- HDF5 weight checkpoint named to encode hyperparameters, e.g. model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5

## How to apply

After building the viRNAtrap 1D-convolutional network (three fully connected layers) in Keras/TensorFlow and training it with binary cross-entropy on 48 bp fragments of viral and human transcripts, serialize the fitted weights in HDF5 format. Encode the reference hyperparameters in the checkpoint filename so the configuration is recoverable without a sidecar file, e.g. model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5 for learning rate 0.005, pooling 5, embedding 25, L2 regularization 0.02, and 64 units. Reload the checkpoint with the model-reconstruction API (load_weights) before evaluating on the held-out test set. Note that current Keras 3 favors the newer .keras save format; .hdf5/.h5 weight files are the legacy tf-keras persistence path, so pin a compatible TensorFlow/Keras environment when reloading.

## Related tools

- **keras** (Provides the model-saving/weight-persistence API (HDF5 checkpointing) used to write and reload the trained viRNAtrap classifier) — https://github.com/fchollet/keras
- **TensorFlow** (Backend in which the viRNAtrap deep learning model is built and trained before checkpointing its weights)
- **viRNAtrap** (Pipeline whose precomputed Tensorflow model (48bp reads) is the artifact persisted as an HDF5 checkpoint) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
model.save_weights('model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5')
```

## Evaluation signals

- The .hdf5 file reloads via model.load_weights into the re-instantiated architecture without shape or key errors
- Re-evaluating the reloaded model on the held-out test set reproduces AUROC 0.81, AUPRC 0.82, accuracy 0.71, recall 0.83, precision 0.67, F1 0.74 within reasonable tolerance
- Filename encodes all reference hyperparameters (lr 0.005, pool 5, emb 25, l2 0.02, 64 units) matching the training configuration
- Running the shipped demo after reinstall yields scores matching expected_output/output_py.txt up to small differences in less-significant digits (e.g., 0.8009399 vs 0.8009398)

## Limitations

- Keras 3 uses the newer .keras save format; .h5 is the legacy format available through the tf-keras package, so environment/version mismatches can block loading legacy .hdf5 checkpoints
- Model predictions for sequences from organisms other than human/viral are not defined; models for other organisms are left to future work
- Viral-read sensitivity is only guaranteed when more than five viral reads are present in the sample

## Evidence

- [readme] Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads.: "1. Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads."
- [intro] Train a neural network on 48 bp fragments of viral and human transcripts segmented into training and test sets: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [readme] Keras 3 .keras vs legacy tf.keras save format: [evidence span withheld — anchor status: not_found]
- [abstract] viRNAtrap alignment-free viral-read identification: "we develop viRNAtrap, an alignment-free pipeline to identify viral reads and assemble viral contigs"
