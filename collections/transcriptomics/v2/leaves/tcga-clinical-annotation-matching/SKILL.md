---
name: tcga-clinical-annotation-matching
description: Use when you have per-sample viral detection calls from RNA-seq (e.g.,
  viRNAtrap contigs classified via blastn) and need to quantify detection accuracy
  against TCGA clinical viral-status annotations for the same samples.
when_to_use_negative:
- The sample set has no corresponding TCGA clinical viral-status annotations to serve
  as ground truth, so no TPR can be computed
- The goal is discovery of novel or divergent viruses rather than benchmarking detection
  of known cancer-associated viruses (HR-ɑHPV, HBV, HCV) already annotated in TCGA
- Input reads are total raw RNA-seq reads rather than unmapped reads, since viRNAtrap
  expects reads pre-filtered against hg19 and phiX
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0781
tools:
- name: viRNAtrap
  role: Alignment-free deep learning pipeline that scores unmapped 48 bp reads as
    viral/non-viral and assembles predicted viral contigs per sample (virnatrap-predict,
    fast mode with assemble_read_c.so)
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: blastn
  role: Compares assembled viral contigs against three curated viral databases (word
    size 15, E-value 0.01) to assign viral taxonomy per sample for matching against
    clinical annotations
- name: TensorFlow
  role: Backend for the pretrained 48 bp model used to predict which unmapped reads
    are viral
- name: Keras
  role: Deep learning framework used to build and load the pretrained 48 bp viral-read
    classification model
  repo: https://github.com/fchollet/keras
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:d4755e7df6fd6eedcaca3d13c1d92f9327cc49b8b8026ca53b05e91dc90212b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - Keras
  - NumPy
  - TensorFlow
  - blastn
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# tcga-clinical-annotation-matching

## Summary

Validates computational virus-detection calls from viRNAtrap by matching per-sample viral detections against TCGA clinical annotations of known cancer-associated viruses (HR-ɑHPV in CESC/HNSC, HBV/HCV in LIHC) and computing a true positive rate that must exceed 95%.

## When to use

Use when you have per-sample viral detection calls from RNA-seq (e.g., viRNAtrap contigs classified via blastn) and need to quantify detection accuracy against TCGA clinical viral-status annotations for the same samples.

## When NOT to use

- The sample set has no corresponding TCGA clinical viral-status annotations to serve as ground truth, so no TPR can be computed
- The goal is discovery of novel or divergent viruses rather than benchmarking detection of known cancer-associated viruses (HR-ɑHPV, HBV, HCV) already annotated in TCGA
- Input reads are total raw RNA-seq reads rather than unmapped reads, since viRNAtrap expects reads pre-filtered against hg19 and phiX

## Inputs

- Per-sample *_unmapped.fastq files of reads not aligned to hg19 or phiX from TCGA CESC/HNSC/LIHC RNA-seq (paired-end files concatenated per sample)
- Pretrained viRNAtrap 48 bp TensorFlow model (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)
- Three curated viral reference databases for blastn classification
- TCGA clinical annotations of viral status (HPV positivity for CESC/HNSC, HBV/HCV status for LIHC)

## Outputs

- Per-sample FASTA files of predicted viral contigs in the output directory
- Per-sample viral taxonomy assignments from blastn against curated viral databases
- Aggregated per-tumor-type detection counts for CESC, HNSC, and LIHC
- True positive rate of viRNAtrap detections versus TCGA clinical annotations (expected > 95%)

## How to apply

Run viRNAtrap on the *_unmapped.fastq files (reads not aligned to hg19 or phiX, with paired-end files concatenated per sample) using the pretrained 48 bp TensorFlow model, then assemble predicted viral reads with the compiled C library (assemble_read_c.so) in fast mode. Classify each sample by blastn comparison of the assembled contigs against the three curated viral databases (word size 15, E-value 0.01) to assign viral taxonomy. Aggregate per-sample and per-tumor-type detection counts for CESC, HNSC, and LIHC, then match each detected virus-sample call to the TCGA clinical annotation of viral status for that sample. Compute the true positive rate as the fraction of clinically annotated virus-positive samples correctly identified; the pipeline is considered validated when TPR > 95% for HR-ɑHPV in CESC (288 samples) and HNSC (61 samples), HBV in LIHC (85 samples), and HCV in LIHC (13 samples), reproducing the Supplementary Fig. 1 pattern.

## Related tools

- **viRNAtrap** (Alignment-free deep learning pipeline that scores unmapped 48 bp reads as viral/non-viral and assembles predicted viral contigs per sample (virnatrap-predict, fast mode with assemble_read_c.so)) — https://github.com/AuslanderLab/virnatrap.git
- **blastn** (Compares assembled viral contigs against three curated viral databases (word size 15, E-value 0.01) to assign viral taxonomy per sample for matching against clinical annotations)
- **TensorFlow** (Backend for the pretrained 48 bp model used to predict which unmapped reads are viral)
- **Keras** (Deep learning framework used to build and load the pretrained 48 bp viral-read classification model) — https://github.com/fchollet/keras

## Examples

```
virnatrap-predict --input input_fastq/ --output output_contigs/ --fastmode 1 --multi_proc 1
```

## Evaluation signals

- Per-sample FASTA contig files are generated in the output directory, with a one-to-one correspondence between input *_unmapped.fastq files and output files (empty outputs allowed for zero viral contigs)
- True positive rate exceeds 95% for HR-ɑHPV detection in CESC (288 samples) and HNSC (61 samples) and for HBV (85 samples) and HCV (13 samples) in LIHC
- Aggregated per-tumor-type detection counts reproduce the Supplementary Fig. 1 pattern of known cancer-virus detections
- Demo sanity check: diff of output_contigs/example_contigs.txt against expected_output/output_c.txt (fast mode) shows no differences or only small last-digit score differences (e.g., contig2 score 0.8009399 vs 0.8009398)
- Each detected virus-sample call matches a TCGA clinical annotation of the corresponding viral type for that sample

## Limitations

- Virus detections without a corresponding TCGA clinical annotation cannot be validated by TPR, so this skill only benchmarks known cancer-associated viruses (HR-ɑHPV, HBV, HCV)
- It is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures, which can confound detection calls
- Sensitivity guarantee applies only when more than five viral reads are present in a sample
- virnatrap-predict does not overwrite existing output files; prior outputs must be removed before rerunning the same input/output directories

## Evidence

- [results] viRNAtrap achieves a true positive rate above 95% for HR-ɑHPV detection in CESC and HNSC and for HBV and HCV detection in LIHC: [evidence span withheld — anchor status: not_found]
- [intro] we used blastn24 to compare the assembled viral contigs to three curated viral databases: "in post-processing analysis, we used blastn24 to compare the assembled viral contigs to three curated viral databases"
- [intro] we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23: "we extracted reads that were not aligned to the human genome (hg19) or to the phiX phage23 that was identified as a frequent contaminant"
- [readme] the user is advised to concatenate the two files into one combined file for input to viRNAtrap: "For pairs of files with paired reads from the same sample, which may be stored separately for other sequence analysis, the user is advised to concatenate the two files into one combined file for"
- [readme] virnatrap-predict will not overwrite output files that already exist: "If one wants to rerun the command with the same  input files and the same output_contigs/ output directory, one should first remove the previous output files. virrnatrap-predict will not overwrite"
