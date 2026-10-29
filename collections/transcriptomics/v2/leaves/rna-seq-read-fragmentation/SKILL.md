---
name: rna-seq-read-fragmentation
description: Use when training or fairly benchmarking fixed-length read classifiers
  (e.g., viRNAtrap vs DeepVirFinder, DeepViFi, ViraMiner, Seq2Seq) and you need viral
  and human transcripts converted into comparable 48 bp fragments with shared train/test
  splits.
when_to_use_negative:
- Input reads are already of a different fixed length (e.g., the 48 bp model expects
  48 bp; variable-length or much longer contigs should not be naively passed through
  this fragmentation without re-segmentation or a model retrained for that length)
- The goal is to run viRNAtrap inference on unmapped RNA-seq FASTQ files rather than
  to retrain or benchmark models — the packaged pipeline consumes *_unmapped.fastq
  input directly and does not require re-fragmenting transcripts
- Sequences come from organisms other than human/viral — the article notes model predictions
  for such sequences are not defined
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: viRNAtrap
  role: Source of the deep learning model whose fixed 48 bp read length dictates the
    fragmentation scheme; provides the TensorFlow/Keras training framework and .hdf5
    model format
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: TensorFlow
  role: Backend used to build and train the model on the 48 bp fragments
- name: Keras
  role: High-level API used with TensorFlow to train the viRNAtrap model and save
    it as .hdf5
  repo: https://github.com/fchollet/keras
- name: NumPy
  role: Array handling for fragment sequence encoding during training and prediction
- name: DeepVirFinder
  role: Baseline retrained on the same 48 bp training fragments for the shared test-set
    comparison
- name: DeepViFi
  role: Baseline retrained on the same 48 bp training fragments for the shared test-set
    comparison
- name: ViraMiner
  role: Baseline retrained on the same 48 bp training fragments for the shared test-set
    comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:00258bc14dc6ad3c129cd31a2c2020341723a3d8902ded336ec31b129ab7daea
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
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rna-seq-read-fragmentation

## Summary

Segments positive (viral) and negative (human) transcript sequences into fixed-length 48 bp fragments split into training and test sets, providing the fixed-length read unit required by the viRNAtrap TensorFlow model and its benchmark baselines.

## When to use

Use when training or fairly benchmarking fixed-length read classifiers (e.g., viRNAtrap vs DeepVirFinder, DeepViFi, ViraMiner, Seq2Seq) and you need viral and human transcripts converted into comparable 48 bp fragments with shared train/test splits.

## When NOT to use

- Input reads are already of a different fixed length (e.g., the 48 bp model expects 48 bp; variable-length or much longer contigs should not be naively passed through this fragmentation without re-segmentation or a model retrained for that length)
- The goal is to run viRNAtrap inference on unmapped RNA-seq FASTQ files rather than to retrain or benchmark models — the packaged pipeline consumes *_unmapped.fastq input directly and does not require re-fragmenting transcripts
- Sequences come from organisms other than human/viral — the article notes model predictions for such sequences are not defined

## Inputs

- Positive (viral) transcript sequences
- Negative (human) transcript sequences
- Fragment length parameter (48 bp as used by viRNAtrap)
- Train/test split specification shared across models

## Outputs

- 48 bp training fragments (viral and human)
- 48 bp test fragments (viral and human)
- Trained viRNAtrap Keras model saved as .hdf5 (e.g., model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)
- Per-read viral/non-viral predictions from each retrained model on the shared test set

## How to apply

Take positive (viral) and negative (human) transcript sequences and segment each into non-overlapping 48 bp fragments, since the viRNAtrap TensorFlow/Keras model was precomputed and trained on reads of this fixed length. Divide the resulting fragments into training and test sets that are shared across all models being compared, so that viRNAtrap, DeepVirFinder, DeepViFi, ViraMiner, and any off-the-shelf Seq2Seq baseline are retrained and evaluated on the identical fragment sets. Retrain each comparison method on the exact same 48 bp training fragments following each method's own original training instructions, then run all models on the shared 48 bp test set. Save the viRNAtrap-style Keras model as an .hdf5 file following the repo naming pattern (e.g., model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5). The rationale for fixed-length 48 bp segmentation is that the model predicts whether reads of a fixed length come from viruses or not, and identical fragmentation is what makes metric comparisons on accuracy, precision, recall, F1, and AUC valid.

## Related tools

- **viRNAtrap** (Source of the deep learning model whose fixed 48 bp read length dictates the fragmentation scheme; provides the TensorFlow/Keras training framework and .hdf5 model format) — https://github.com/AuslanderLab/virnatrap.git
- **TensorFlow** (Backend used to build and train the model on the 48 bp fragments)
- **Keras** (High-level API used with TensorFlow to train the viRNAtrap model and save it as .hdf5) — https://github.com/fchollet/keras
- **NumPy** (Array handling for fragment sequence encoding during training and prediction)
- **DeepVirFinder** (Baseline retrained on the same 48 bp training fragments for the shared test-set comparison)
- **DeepViFi** (Baseline retrained on the same 48 bp training fragments for the shared test-set comparison)
- **ViraMiner** (Baseline retrained on the same 48 bp training fragments for the shared test-set comparison)

## Examples

```
git clone https://github.com/AuslanderLab/virnatrap.git; cd virnatrap; pip install .; python train_model.py --fraglen 48 --save model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5
```

## Evaluation signals

- All fragments are exactly 48 bp in length (no variable-length reads remain in train/test sets)
- The training and test fragment sets are identical across viRNAtrap and all baselines (DeepVirFinder, DeepViFi, ViraMiner, Seq2Seq)
- The trained viRNAtrap model is saved as an .hdf5 file matching the pattern model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5
- Compiled metrics table (accuracy, precision, recall, F1, AUC) reproduces the Fig. 1b/c ranking: viRNAtrap outperforms all baselines on all measures except precision, where DeepVirFinder is best
- Retraining of each baseline follows that method's own original training instructions (no cross-method hyperparameter mixing)

## Limitations

- The pretrained model is fixed to 48 bp reads; predictions for sequences of other lengths or from organisms other than human/viral are not defined and would require training new models
- Comparisons are only fair if all methods are retrained on the exact same 48 bp training fragments and evaluated on the shared test set; differing fragment provenance invalidates the ranking
- A precision-only failure mode is expected: DeepVirFinder outperforms all other methods on precision even though viRNAtrap wins on other measures

## Evidence

- [intro] positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [readme] Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads.: "Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads."
- [intro] We compared the performance of this model to previous models trained to identify viruses, namely DeepViFi16, DeepVirFinder15, ViraMiner21: "We compared the performance of this model to previous models trained to identify viruses, namely DeepViFi16, DeepVirFinder15, ViraMiner21"
- [other] When all models were evaluated on the shared test set, viRNAtrap outperformed the comparison methods on all evaluation measures except precision, on which DeepVirFinder outperformed all other methods.: [evidence span withheld — anchor status: not_found]
- [discussion] Future work could train models to identify viruses from a variety of other organisms, and, with the viRNAtrap framework, achieve higher sensitivity for viral detection: "Future work could train models to identify viruses from a variety of other organisms, and, with the viRNAtrap framework, achieve higher sensitivity for viral detection"
