---
name: single-cell-rnaseq-fusion-benchmarking
description: Use when evaluating fusion gene detection performance on single-cell
  RNA-seq data with low sequencing depth, e.g. reproducing HVLD AUC values on K562
  or multiple-myeloma scRNA-seq cohorts against competing models.
when_to_use_negative:
- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion
  only works on paired-end RNA-seq data
- The analysis targets fusion genes not including the user-provided anchored target
  gene — these will not be detected
- The input dataset is too small to retrain the filter — use the default model parameters
  provided on GitHub instead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Anchored-fusion
  role: Primary tool whose HVLD filter model is retrained per dataset and evaluated;
    candidate chimeric reads generated via anchored gene mapping
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BWA-MEM
  role: Alignment tool used to generate artificial chimeric reads as negative training
    samples
- name: BLAT
  role: Local alignment on candidate chimeric reads to predict precise fusion locations
- name: scFusion
  role: Competing scRNA-seq fusion detection method trained on the same chimeric-read
    samples for benchmarking
  repo: https://github.com/XiDsLab/scFusion
- name: PyTorch
  role: Deep learning framework (>=1.10.2) for training the HVLD model
- name: scikit-learn
  role: Used (>=1.3) in training/evaluating the HVLD filter and computing AUC
- name: Biopython
  role: Sequence handling during HVLD model training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:f971e3cc7d6d5dd383ff202b261e80c27e2c3bbf06701e3a41b02d9d2e8b58aa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - BLAT
  - BWA-MEM
  - Biopython
  - PyTorch
  - scFusion
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# single-cell-rnaseq-fusion-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrains and benchmarks the HVLD deep learning filter of Anchored-fusion on scRNA-seq datasets (K562 SRA:SRP291312 and multiple myeloma SRA:SRP158590) to distinguish true from artificial fusion reads, comparing AUC against competing models such as scFusion.

## When to use

Use when evaluating fusion gene detection performance on single-cell RNA-seq data with low sequencing depth, e.g. reproducing HVLD AUC values on K562 or multiple-myeloma scRNA-seq cohorts against competing models.

## When NOT to use

- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion only works on paired-end RNA-seq data
- The analysis targets fusion genes not including the user-provided anchored target gene — these will not be detected
- The input dataset is too small to retrain the filter — use the default model parameters provided on GitHub instead

## Inputs

- Paired-end scRNA-seq FASTQ files for K562 cell line (SRA:SRP291312)
- Paired-end scRNA-seq FASTQ files for multiple myeloma patient cohort (SRA:SRP158590)
- PCAWG 3540 fusion genes as positive training samples
- Homologous gene file (included in the 'data' folder of Anchored-fusion)
- Target gene transcript reference fasta (from NCBI GENE)
- Genome fasta and gene annotation gtf (hg38 or hg19)

## Outputs

- Retrained HVLD filter model saved as the --model_file in the 'data' folder of Anchored-fusion
- Per-dataset AUC values for HVLD versus competing models (expected 91.0% K562, 94.5% multiple myeloma)
- Trained scFusion benchmark model on the same chimeric-read samples

## How to apply

Download paired-end scRNA-seq FASTQ data for the K562 cell line (SRA:SRP291312) and multiple myeloma patients (SRA:SRP158590) from SRA. Generate artificial chimeric reads as negative training samples by aligning reads with BWA-MEM, filtering out reads aligned to genes in the homologous gene file and removing all negative samples containing the anchored gene. Use the 3540 PCAWG fusion genes as positive training samples alongside the dataset-specific artificial chimeric reads, then train the HVLD hierarchical view learning and distillation model per dataset with PyTorch (>=1.10.2), numpy, and scikit-learn (>=1.3), using Biopython for sequence handling. Train the competing scFusion model on the same chimeric-read samples for benchmarking, save the retrained filter model to the 'data' folder of Anchored-fusion as the file specified by --model_file, and evaluate on held-out test samples. Correct application reproduces AUC values of 91.0% (K562) and 94.5% (multiple myeloma), improvements of 5.45% and 4.80% over the Transformer model.

## Related tools

- **Anchored-fusion** (Primary tool whose HVLD filter model is retrained per dataset and evaluated; candidate chimeric reads generated via anchored gene mapping) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BWA-MEM** (Alignment tool used to generate artificial chimeric reads as negative training samples)
- **BLAT** (Local alignment on candidate chimeric reads to predict precise fusion locations)
- **scFusion** (Competing scRNA-seq fusion detection method trained on the same chimeric-read samples for benchmarking) — https://github.com/XiDsLab/scFusion
- **PyTorch** (Deep learning framework (>=1.10.2) for training the HVLD model)
- **scikit-learn** (Used (>=1.3) in training/evaluating the HVLD filter and computing AUC)
- **Biopython** (Sequence handling during HVLD model training)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- HVLD AUC on held-out K562 scRNA-seq test samples reproduces 91.0%
- HVLD AUC on held-out multiple myeloma scRNA-seq test samples reproduces 94.5%
- Reported improvement over the Transformer model is approximately 5.45% (K562) and 4.80% (multiple myeloma)
- All negative training samples containing the anchored gene X are removed from the training set
- Retrained model file is stored in the 'data' folder and correctly referenced via --model_file

## Limitations

- Fusion genes that do not include the user-provided target gene will not be detected
- Anchored-fusion can only be used with paired-end RNA-seq data, not single-end, whole-genome, or whole-exome data
- Users must supply the reference sequence of their target gene
- If the input dataset is too small, the default model parameters on GitHub should be used rather than retrained HVLD parameters
- The value of the loss-balancing hyperparameter alpha in the HVLD loss function is not specified

## Evidence

- [results] the HVLD model achieved the best performance with AUC values of 91.0% and 94.5%, representing improvements of 5.45% and 4.80% over the Transformer model: [evidence span withheld — anchor status: not_found]
