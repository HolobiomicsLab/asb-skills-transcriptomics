---
name: virus-read-classification-from-rnaseq
description: Use when you have RNA-seq reads already unmapped to hg19/phiX (as *_unmapped.fastq
  files) and need to detect expressed viruses, including divergent ones missed by
  alignment-based methods, across tumor samples such as TCGA cancer types.
when_to_use_negative:
- When reads have not been pre-mapped to the human genome and phiX contaminant — mapping
  is performed independently by the user and must precede viRNAtrap
- When sequences may derive from organisms other than human/viral — the model's predictions
  for other organisms are undefined per the article's discussion
- When sensitivity depends on detecting very few viral reads — sensitivity is only
  guaranteed if more than five viral reads are present
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3673
tools:
- name: viRNAtrap
  role: Alignment-free pipeline that scores unmapped RNA-seq reads with a pretrained
    deep learning model and assembles predicted viral reads into contigs (Python slow
    mode or C fast mode)
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: TensorFlow
  role: Backend for the pretrained 48 bp read-classification model used to score reads
    as viral or non-viral
  repo: https://github.com/keras-team/keras
- name: blastn
  role: Post-processing comparison of assembled viral contigs against three curated
    viral databases (word size 15, E-value 0.01) to assign viral taxonomy per sample
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:eca5c6dfc071117c5bd4a14de81159596206a92b836bda5c26974cc6fcf77388
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - NumPy
  - TensorFlow
  - blastn
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# virus-read-classification-from-rnaseq

## Summary

Classify unmapped RNA-seq reads as viral using viRNAtrap's pretrained 48 bp TensorFlow deep learning model, then assemble predicted viral reads into contigs for taxonomic assignment — an alignment-free alternative for detecting viral expression in tumor transcriptomes.

## When to use

Use when you have RNA-seq reads already unmapped to hg19/phiX (as *_unmapped.fastq files) and need to detect expressed viruses, including divergent ones missed by alignment-based methods, across tumor samples such as TCGA cancer types.

## When NOT to use

- When reads have not been pre-mapped to the human genome and phiX contaminant — mapping is performed independently by the user and must precede viRNAtrap
- When sequences may derive from organisms other than human/viral — the model's predictions for other organisms are undefined per the article's discussion
- When sensitivity depends on detecting very few viral reads — sensitivity is only guaranteed if more than five viral reads are present

## Inputs

- Unmapped RNA-seq FASTQ files named *_unmapped.fastq in an input directory (one combined file per sample, paired mates concatenated)
- Pretrained 48 bp TensorFlow/Keras model file (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)
- Three curated viral reference databases for blastn-based taxonomy assignment (including a curated HERV database of intact retroviral genes)

## Outputs

- Per-input-file FASTA files of predicted viral contigs (suffix .txt) in the output directory, with one-to-one correspondence to input FASTQ files (empty files when zero contigs predicted)
- Per-sample viral taxonomy assignments from blastn against curated viral databases
- Aggregated per-sample and per-tumor-type virus detection counts (e.g., for TCGA CESC, HNSC, LIHC)

## How to apply

First reduce search space by extracting reads not aligned to hg19 or the phiX contaminant, writing them as *_unmapped.fastq files (concatenating paired-end mates per sample, since viRNAtrap treats each input file as one sample). Run virnatrap-predict with the pretrained model (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5) to score each read as viral or non-viral; the neural network was trained on 48 bp fragments of viral and human transcripts. Assemble model-predicted viral reads into contigs in fast mode by first compiling the C library assemble_read_c.c with gcc -O3 and running with --fastmode 1 (optionally --multi_proc 1 with up to 48 threads). Finally, compare the per-sample FASTA contigs against three curated viral databases using blastn with word size 15 and E-value 0.01 to assign viral taxonomy per sample.

## Related tools

- **viRNAtrap** (Alignment-free pipeline that scores unmapped RNA-seq reads with a pretrained deep learning model and assembles predicted viral reads into contigs (Python slow mode or C fast mode)) — https://github.com/AuslanderLab/virnatrap.git
- **TensorFlow** (Backend for the pretrained 48 bp read-classification model used to score reads as viral or non-viral) — https://github.com/keras-team/keras
- **blastn** (Post-processing comparison of assembled viral contigs against three curated viral databases (word size 15, E-value 0.01) to assign viral taxonomy per sample)

## Examples

```
gcc -o src/assemble_read_c.so -shared -fPIC -O3 src/assemble_read_c.c && virnatrap-predict --input input_fastq/ --output output_contigs/ --fastmode 1
```

## Evaluation signals

- Demo diff check: run virnatrap-predict on input_fastq/example_unmapped.fastq and diff output_contigs/example_contigs.txt against expected_output/output_py.txt (or output_c.txt in fast mode); only trailing-digit score differences (e.g., contig2[0.8009399] vs 0.8009398) are acceptable
- One-to-one correspondence: every *_unmapped.fastq input yields a per-sample contig FASTA (.txt) in the output directory, even if empty
- Benchmark TPR: detection of HR-ɑHPV in 288 CESC / 61 HNSC and HBV/HCV in 85/13 LIHC relative to TCGA clinical annotations should achieve TPR > 95%
- Re-run safety: virnatrap-predict does not overwrite existing output files — previous outputs must be removed before rerunning with the same directories

## Limitations

- It is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures (e.g., phiX, a frequent contaminant, must be filtered beforehand)
- Model predictions for sequences from organisms other than human/viral are not defined; models for other organisms are left to future work
- Sensitivity is only stated for cases where more than five viral reads are present
- Paired-end mates must be concatenated into a single file per sample; separate files would be treated as distinct samples, inflating sample counts

## Evidence

- [abstract] we develop viRNAtrap, an alignment-free pipeline to identify viral reads and assemble viral contigs: "we develop viRNAtrap, an alignment-free pipeline to identify viral reads and assemble viral contigs"
- [intro] positive (viral) and negative (human) transcripts segmented into 48 bp fragments: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [intro] blastn comparing assembled contigs to three curated viral databases: "in post-processing analysis, we used blastn24 to compare the assembled viral contigs to three curated viral databases"
- [readme] README: fast mode compiles the C assembler: "The fast mode calls a C library to assemble the viral contigs from the model-predicted viral reads."
- [readme] README: output correspondence and no-overwrite behavior: "If an input file leads to zero predicted viral contigs, then the corresponding output file will be created but will be empty."
- [discussion] Discussion: sensitivity caveat with >5 viral reads: "while not compromising sensitivity if more than five viral reads are present"
