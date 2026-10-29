---
name: viral-read-discrimination
description: Use when unmapped RNAseq reads must be classified as viral vs. human
  without alignment, or when predicted viral contigs need to be assembled from deep
  learning model scores.
when_to_use_negative:
- Reads can be confidently assigned by standard alignment to reference viral genomes;
  alignment-based methods are preferable when references exist
- The input sequences come from organisms other than human/viral — model predictions
  are not defined for other organisms
- Fewer than about five viral reads are expected in a sample; sensitivity is only
  stated for cases with more than five viral reads present
edam_operation: http://edamontology.org/operation_0465
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3474
tools:
- name: viRNAtrap
  role: Alignment-free pipeline that scores unmapped reads with a pretrained deep
    learning model and assembles predicted viral reads into contigs
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: TensorFlow
  role: Backend used to build, train, and run the 1D-convolutional read-classification
    model
  repo: https://github.com/tensorflow/tensorflow
- name: Keras
  role: High-level API used to construct the neural network with three fully connected
    layers and train it with binary cross-entropy
  repo: https://github.com/fchollet/keras
- name: blastn
  role: Post-processing comparison of assembled viral contigs against three curated
    viral databases
- name: blastx
  role: Targeted sequence analysis to verify presence and classification of divergent
    viral reads in samples
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:07e7bbcc3941f31f2ffe1d2b0c0c73a04c891fe86fb09a9ad7bbdda9f498e335
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - Keras
  - NumPy
  - TensorFlow
  - blastn
  - blastx
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0465
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# viral-read-discrimination

## Summary

Train and apply viRNAtrap, an alignment-free 1D-convolutional deep learning model (TensorFlow/Keras), to classify 48 bp RNAseq fragments as viral or human and use model scores to assemble predicted viral contigs.

## When to use

Use when unmapped RNAseq reads must be classified as viral vs. human without alignment, or when predicted viral contigs need to be assembled from deep learning model scores.

## When NOT to use

- Reads can be confidently assigned by standard alignment to reference viral genomes; alignment-based methods are preferable when references exist
- The input sequences come from organisms other than human/viral — model predictions are not defined for other organisms
- Fewer than about five viral reads are expected in a sample; sensitivity is only stated for cases with more than five viral reads present

## Inputs

- FASTQ files of unmapped RNAseq reads named *_unmapped.fastq (paired reads from a sample should be concatenated into one file)
- Positive (viral) and negative (human) transcript sequences segmented into 48 bp fragments for training/testing
- Curated viral databases for blastn post-processing of assembled contigs

## Outputs

- Per-sample FASTA-format contig files (suffix .txt) with predicted viral contigs and per-contig model scores in brackets
- Trained model weights in HDF5 format, e.g. model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5
- Held-out test-set metrics: AUROC, AUPRC, accuracy, recall, precision, F1

## How to apply

First map RNAseq reads to the human genome (hg19) independently and remove reads aligning to hg19 or the phiX phage contaminant. Segment positive (viral) and negative (human) transcripts into fixed 48 bp fragments split into training and test partitions, then build the 1D-convolutional network with three fully connected layers in Keras/TensorFlow using the reference hyperparameters encoded in the checkpoint name (learning rate 0.005, pooling 5, embedding 25, L2 regularization 0.02, 64 units) with binary cross-entropy loss. Run virnatrap-predict on a directory of *_unmapped.fastq files to score reads as viral and assemble predicted viral reads into contigs (slow native-Python mode or fast C mode after compiling src/assemble_read_c.c). Validate the trained model on the held-out test set against the reported metrics (AUROC 0.81, AUPRC 0.82, accuracy 0.71, recall 0.83, precision 0.67, F1 0.74), and compare assembled contig scores against expected_output files via diff. Post-process by aligning assembled contigs to curated viral databases with blastn and verifying divergent viruses with targeted blastn/blastx.

## Related tools

- **viRNAtrap** (Alignment-free pipeline that scores unmapped reads with a pretrained deep learning model and assembles predicted viral reads into contigs) — https://github.com/AuslanderLab/virnatrap.git
- **TensorFlow** (Backend used to build, train, and run the 1D-convolutional read-classification model) — https://github.com/tensorflow/tensorflow
- **Keras** (High-level API used to construct the neural network with three fully connected layers and train it with binary cross-entropy) — https://github.com/fchollet/keras
- **blastn** (Post-processing comparison of assembled viral contigs against three curated viral databases)
- **blastx** (Targeted sequence analysis to verify presence and classification of divergent viral reads in samples)

## Examples

```
virnatrap-predict --input input_fastq/ --output output_contigs/ --fastmode 1
```

## Evaluation signals

- Held-out test metrics match reported values (AUROC 0.81, AUPRC 0.82, accuracy 0.71, recall 0.83, precision 0.67, F1 0.74) within reasonable tolerance
- Installation demo: diff expected_output/output_py.txt output_contigs/example_contigs.txt returns no differences, or only small least-significant-digit score differences (e.g. 0.8009399 vs 0.8009398)
- Every input *_unmapped.fastq yields a corresponding output file; files with zero predicted viral contigs are created but empty
- Assembled contig scores in brackets are reproducible across slow (Python) and fast (C) assembly modes

## Limitations

- Not possible to model all contaminants of viruses that may have infected samples during laboratory procedures
- Sensitivity is only guaranteed when more than five viral reads are present
- Model predictions for sequences from organisms other than human/viral are not defined; models for other organisms are left to future work
- viRNAtrap does not overwrite existing output files — previous outputs must be removed before rerunning with the same input/output directories

## Evidence

- [abstract] alignment-free pipeline to identify viral reads and assemble viral contigs: "we develop viRNAtrap, an alignment-free pipeline to identify viral reads and assemble viral contigs"
- [intro] 48bp reads and hyperparameter-encoded checkpoint: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [intro] reads extracted as unmapped to hg19 or phiX: "we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23 that was identified as a frequent contaminant"
- [readme] input/output convention for unmapped FASTQ files: "a user must provide an input directory continaing one or more input FASTQ files of unmapped reads with file names ending in  *_unmapped.fastq"
- [readme] demo output comparison procedure: "The installation is correct if the above diff command retruns either no differences or small differences in the less significant digits for the scores in brackets"
- [discussion] sensitivity limitation for low viral read counts: "while not compromising sensitivity if more than five viral reads are present"
