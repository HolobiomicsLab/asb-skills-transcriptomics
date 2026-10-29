---
name: viral-read-classification-model-training
description: Use when building or retraining an alignment-free viral-read classifier
  on fixed-length RNA-seq fragments (e.g., 48 bp), or when all models must be retrained
  on identical training fragments for a fair comparison on a shared test set.
when_to_use_negative:
- Reads are not from human RNA-seq samples — the article notes model predictions for
  sequences from organisms other than human/viral are not defined; models for other
  organisms are future work
- Only a handful of viral reads (<5) are expected in a sample — the sensitivity guarantee
  is stated only for cases with more than five viral reads present
- The goal is read alignment/mapping rather than alignment-free classification — map
  reads to the human genome first and apply this model only to unmapped reads
edam_operation: http://edamontology.org/operation_0463
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3520
tools:
- name: viRNAtrap
  role: Alignment-free pipeline whose TensorFlow model is trained on 48 bp fragments
    to discriminate viral RNAseq reads; provides the training code and precomputed
    48 bp model
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: TensorFlow
  role: Backend framework for building and training the viral-vs-human read classification
    model
- name: Keras
  role: High-level API used with TensorFlow to define and train the neural network
    and save it as .hdf5
  repo: keras-team/keras
- name: NumPy
  role: Array/numerical processing dependency used by viRNAtrap for sequence encoding
    and model I/O
- name: DeepVirFinder
  role: Baseline classifier retrained on the same 48 bp training fragments for the
    benchmark comparison
- name: DeepViFi
  role: Baseline classifier retrained on the same 48 bp training fragments for the
    benchmark comparison
- name: ViraMiner
  role: Baseline classifier retrained on the same 48 bp training fragments for the
    benchmark comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:ec5c997c59c5210aa77e21493293de28dbcd6966cfeb32db5cefe8b20e86ae00
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
  edam_operation: http://edamontology.org/operation_0463
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# viral-read-classification-model-training

## Summary

Train a TensorFlow/Keras deep learning model (viRNAtrap-style) to discriminate viral from human RNA-seq reads, using 48 bp fragments segmented from positive (viral) and negative (human) transcripts split into shared training and test sets, so the model can score unmapped reads and enable viral contig assembly and fair benchmarking against methods like DeepVirFinder, DeepViFi, ViraMiner, and Seq2Seq.

## When to use

Use when building or retraining an alignment-free viral-read classifier on fixed-length RNA-seq fragments (e.g., 48 bp), or when all models must be retrained on identical training fragments for a fair comparison on a shared test set.

## When NOT to use

- Reads are not from human RNA-seq samples — the article notes model predictions for sequences from organisms other than human/viral are not defined; models for other organisms are future work
- Only a handful of viral reads (<5) are expected in a sample — the sensitivity guarantee is stated only for cases with more than five viral reads present
- The goal is read alignment/mapping rather than alignment-free classification — map reads to the human genome first and apply this model only to unmapped reads

## Inputs

- Positive (viral) and negative (human) transcript sequences segmented into 48 bp fragments
- Shared training/test split of the 48 bp fragments
- Trained viRNAtrap-style model file model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5 (output of training; input to prediction)

## Outputs

- Trained TensorFlow/Keras model saved as .hdf5 (e.g., model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)
- Per-read viral/non-viral predictions on the shared 48 bp test set
- Metrics table benchmark_comparison_metrics.csv with accuracy, precision, recall, F1, and AUC for each model

## How to apply

Segment positive (viral) and negative (human) transcripts into 48 bp fragments and divide them into training and test sets shared across all models. Train a TensorFlow/Keras neural network on the 48 bp training fragments and save it as an .hdf5 file following the naming pattern model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5, which encodes the key hyperparameters (learning rate 0.005, pooling 5, embedding 25, L2 0.02). For benchmarking, retrain DeepVirFinder, DeepViFi, and ViraMiner on the exact same 48 bp training fragments following each method's own original training instructions, so the comparison is not confounded by different training data. Run all retrained models on the shared 48 bp test set to obtain per-read viral/non-viral predictions, then compute accuracy, precision, recall, F1, and AUC per model. The published result to reproduce is that viRNAtrap outperformed all baselines on all measures except precision, where DeepVirFinder was best.

## Related tools

- **viRNAtrap** (Alignment-free pipeline whose TensorFlow model is trained on 48 bp fragments to discriminate viral RNAseq reads; provides the training code and precomputed 48 bp model) — https://github.com/AuslanderLab/virnatrap.git
- **TensorFlow** (Backend framework for building and training the viral-vs-human read classification model)
- **Keras** (High-level API used with TensorFlow to define and train the neural network and save it as .hdf5) — keras-team/keras
- **NumPy** (Array/numerical processing dependency used by viRNAtrap for sequence encoding and model I/O)
- **DeepVirFinder** (Baseline classifier retrained on the same 48 bp training fragments for the benchmark comparison)
- **DeepViFi** (Baseline classifier retrained on the same 48 bp training fragments for the benchmark comparison)
- **ViraMiner** (Baseline classifier retrained on the same 48 bp training fragments for the benchmark comparison)

## Examples

```
virnatrap-predict --input input_fastq/ --output output_contigs/
```

## Evaluation signals

- Reproduced metric ranking matches the reported Fig. 1b/c finding: viRNAtrap outperforms all baselines on all measures except precision, where DeepVirFinder is best
- Saved model file exists and follows the naming pattern model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5
- benchmark_comparison_metrics.csv contains accuracy, precision, recall, F1, and AUC for all four models evaluated on the same 48 bp test set
- All four models (viRNAtrap, DeepVirFinder, DeepViFi, ViraMiner) were retrained on the identical 48 bp training fragments, not on each method's default data
- For installation sanity checks, viRNAtrap's demo output diff against expected_output/output_py.txt should show no differences or only small last-digit differences in scores in brackets

## Limitations

- It is not possible to model all contaminants of viruses that may have infected the samples during laboratory procedures
- Model predictions for sequences from organisms other than human/viral are not defined; models for other organisms are left to future work
- Sensitivity is only guaranteed when more than five viral reads are present in a sample

## Evidence

- [intro] positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [other] viRNAtrap outperformed baselines on all measures except precision: [evidence span withheld — anchor status: not_found]
- [readme] TensorFlow model precomputed on 48bp reads: "Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads."
- [abstract] alignment-free pipeline to identify viral reads and assemble viral contigs: "we develop viRNAtrap, an alignment-free pipeline to identify viral reads and assemble viral contigs"
