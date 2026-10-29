---
name: rna-read-sequence-classification
description: Use when you have unmapped RNAseq reads (e.g., *_unmapped.fastq after
  removal of hg19/phiX alignments) and need an alignment-free binary viral-vs-human
  read classifier or predicted viral contigs.
when_to_use_negative:
- Input reads are already mapped/aligned or you require alignment-based taxonomic
  assignments — viRNAtrap is deliberately alignment-free and expects *_unmapped.fastq
  after user-performed human-genome mapping
- Sequences come from organisms other than human/viral — model predictions for such
  sequences are not defined; the authors note models for other organisms are future
  work
- You need guaranteed sensitivity when very few viral reads are present — the stated
  sensitivity guarantee only holds if more than five viral reads are present
edam_operation: http://edamontology.org/operation_3461
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3474
tools:
- name: viRNAtrap
  role: Alignment-free pipeline that scores unmapped RNAseq reads with a pretrained
    deep learning model and assembles predicted viral reads into contigs
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: TensorFlow
  role: Backend framework for building and running the 48 bp viral-vs-human read classifier
- name: Keras
  role: High-level API used with TensorFlow to construct the 1D-convolutional network
    with three fully connected layers
  repo: https://github.com/fchollet/keras
- name: NumPy
  role: Array processing for sequence fragment encoding during model inference/training
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:21b0d90fc56fd1b4f2e4ad4d56ec99aaa53e84dfa79a0dcb2b83e4ab0fe9fb5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - Keras
  - NumPy
  - TensorFlow
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_3461
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# rna-read-sequence-classification

## Summary

Train and apply viRNAtrap, an alignment-free 1D-convolutional deep learning model built in Keras/TensorFlow, to classify unmapped RNAseq reads as viral or human based on fixed 48 bp sequence fragments, then use model scores to assemble predicted viral contigs.

## When to use

Use when you have unmapped RNAseq reads (e.g., *_unmapped.fastq after removal of hg19/phiX alignments) and need an alignment-free binary viral-vs-human read classifier or predicted viral contigs.

## When NOT to use

- Input reads are already mapped/aligned or you require alignment-based taxonomic assignments — viRNAtrap is deliberately alignment-free and expects *_unmapped.fastq after user-performed human-genome mapping
- Sequences come from organisms other than human/viral — model predictions for such sequences are not defined; the authors note models for other organisms are future work
- You need guaranteed sensitivity when very few viral reads are present — the stated sensitivity guarantee only holds if more than five viral reads are present

## Inputs

- Input directory of FASTQ files of unmapped RNAseq reads named *_unmapped.fastq (paired-end files should be concatenated into one file per sample)
- Pretrained 48 bp viRNAtrap TensorFlow/Keras checkpoint (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)

## Outputs

- One FASTA-format file (suffix .txt) per input FASTQ in the output directory, containing predicted viral contigs with per-contig model scores in brackets, e.g. >contig2[0.8009399]
- Optionally, held-out test metrics (AUROC 0.81, AUPRC 0.82, accuracy 0.71, recall 0.83, precision 0.67, F1 0.74) when retraining/evaluating the model

## How to apply

First have the user map RNAseq reads to the human genome independently and supply only unmapped FASTQ files named *_unmapped.fastq. viRNAtrap then applies a precomputed TensorFlow model trained on positive (viral) and negative (human) transcripts segmented into 48 bp fragments (training/test split; binary cross-entropy loss) to score each read as viral or not, with reference hyperparameters encoded in the checkpoint name model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5 (learning rate 0.005, pooling 5, embedding 25, L2 0.02, 64 units). Predicted viral reads are assembled into contigs via either a native Python implementation or a compiled C library (gcc -shared -fPIC -O3 on src/assemble_read_c.c) selected with --fastmode 1. Output is a FASTA-formatted contig file per input file with model scores in brackets in the FASTA headers. Validate by diffing against the bundled expected_output (small score differences in the last significant digits are acceptable).

## Related tools

- **viRNAtrap** (Alignment-free pipeline that scores unmapped RNAseq reads with a pretrained deep learning model and assembles predicted viral reads into contigs) — https://github.com/AuslanderLab/virnatrap.git
- **TensorFlow** (Backend framework for building and running the 48 bp viral-vs-human read classifier)
- **Keras** (High-level API used with TensorFlow to construct the 1D-convolutional network with three fully connected layers) — https://github.com/fchollet/keras
- **NumPy** (Array processing for sequence fragment encoding during model inference/training)

## Examples

```
virnatrap-predict --input input_fastq/ --output output_contigs/ --fastmode 1
```

## Evaluation signals

- Held-out test metrics reproduce the published reference values: AUROC 0.81, AUPRC 0.82, accuracy 0.71, recall 0.83, precision 0.67, F1 0.74 (within reasonable tolerance)
- Demo run: diff of output_contigs/example_contigs.txt against expected_output/output_py.txt returns no differences or only last-digit score differences (e.g., 0.8009399 vs 0.8009398)
- One-to-one correspondence: each input *_unmapped.fastq yields exactly one output file; zero-viral-contig inputs still produce an (empty) output file
- Contig scores in FASTA headers fall in the [0,1] model-probability range
- Saved weights are written in HDF5 format with hyperparameters encoded in the filename (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)

## Limitations

- viRNAtrap treats each distinct input file as a distinct sample — paired-end FASTQs must be concatenated into one file per sample or they will be processed as separate samples
- virnatrap-predict will not overwrite existing output files; previous outputs must be removed before rerunning with the same input/output directories
- It is not possible to model all laboratory contaminants; contamination scenarios may evade the model
- Sensitivity is only guaranteed when more than five viral reads are present; predictions for non-human/non-viral organisms are undefined

## Evidence

- [intro] positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [abstract] we develop viRNAtrap, an alignment-free pipeline to identify viral reads and assemble viral contigs: "we develop viRNAtrap, an alignment-free pipeline to identify viral reads and assemble viral contigs"
- [readme] virnatrap-predict --input input_fastq/ --output output_contigs/: "virnatrap-predict --input input_fastq/ --output output_contigs/ --fastmode 1"
- [methods] map reads to the human genome; this is performed independently by the user: "map reads to the human genome; this is performed independently by the user."
- [readme] gcc -o src/assemble_read_c.so -shared -fPIC -O3 src/assemble_read_c.c: "gcc -o src/assemble_read_c.so -shared -fPIC -O3 src/assemble_read_c.c"
