---
name: homology-aware-read-filtering
description: Use when generating artificial chimeric-read training samples with BWA-MEM
  for the HVLD filter, or when the anchored gene has homologous regions that could
  create false fusion partner candidates.
when_to_use_negative:
- When the user's dataset is too small to retrain the filter — the article recommends
  using the default model parameter provided on GitHub instead
- When the user selected --not_filter_false_positive or --not_train_filter_model and
  supplied a pre-trained model, since training-time homology filtering is bypassed
- For single-end RNA-seq or whole-genome/whole-exome sequencing data, which Anchored-fusion
  cannot process
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Anchored-fusion
  role: Pipeline whose HVLD filter training consumes the homology-filtered artificial
    chimeric reads; provides the homologous gene file and 'data' folder for the retrained
    model
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BWA-MEM
  role: Alignment tool used to map reads when generating artificial chimeric reads
    and to identify reads aligned to homologous genes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:10d8c0134b5f003142da0f90b666adc136f832f97bce8f8454d79550b346ac6b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# homology-aware-read-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filter reads aligned to homologous genes and exclude negative training samples containing the anchored gene when generating artificial chimeric reads for Anchored-fusion's HVLD filter model, preventing sequence homology from contaminating fusion detection.

## When to use

Use when generating artificial chimeric-read training samples with BWA-MEM for the HVLD filter, or when the anchored gene has homologous regions that could create false fusion partner candidates.

## When NOT to use

- When the user's dataset is too small to retrain the filter — the article recommends using the default model parameter provided on GitHub instead
- When the user selected --not_filter_false_positive or --not_train_filter_model and supplied a pre-trained model, since training-time homology filtering is bypassed
- For single-end RNA-seq or whole-genome/whole-exome sequencing data, which Anchored-fusion cannot process

## Inputs

- Paired-end scRNA-seq or bRNA-seq FASTQ files (e.g. SRA:SRP291312 K562, SRA:SRP158590 multiple myeloma)
- Homologous gene file (included in the 'data' folder of the Anchored-fusion program)
- Positive training samples: 3540 PCAWG fusion genes
- BWA-MEM alignment of reads against the reference/transcript sequences
- Reference fasta of the target gene transcript and genome fasta with gene annotation gtf (hg38 or hg19)

## Outputs

- Cleaned artificial chimeric-read negative training set free of homologous-gene alignments and of samples containing the anchored gene
- Pruned potential fusion partner gene set Y (genes homologous to the anchored gene removed)
- Retrained HVLD filter model file placed in the 'data' folder for use as --model_file

## How to apply

Generate artificial chimeric reads as negative training samples by aligning reads with BWA-MEM, then apply three filters in order: first remove reads that map to homologous genes listed in the homologous gene file, then remove all negative training samples that contain the anchored gene X itself, and separately remove genes with homologous regions to gene X from the potential fusion partner gene set Y. The rationale is that sequence homology causes multi-mapping read pairs that most fusion tools discard, so unfiltered homologous alignments would bias the HVLD decision model's learned distribution of artificial fusions. The resulting cleaned negative samples, combined with the 3540 PCAWG fusion genes as positive samples, are used to train the dataset-specific HVLD filter, whose output model is saved to the 'data' folder and passed via --model_file. If the input dataset is too small for reliable retraining, fall back to the default pre-trained model parameters provided in the repository instead.

## Related tools

- **Anchored-fusion** (Pipeline whose HVLD filter training consumes the homology-filtered artificial chimeric reads; provides the homologous gene file and 'data' folder for the retrained model) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BWA-MEM** (Alignment tool used to map reads when generating artificial chimeric reads and to identify reads aligned to homologous genes)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- No negative training sample contains the anchored gene X after filtering
- No retained artificial chimeric read maps to a gene listed in the homologous gene file
- The retrained HVLD model, evaluated on held-out scRNA-seq test samples, reproduces AUC ≈ 91.0% on K562 (SRA:SRP291312) and ≈ 94.5% on the multiple myeloma cohort (SRA:SRP158590)
- The retrained model file resides in the 'data' folder and is accepted by the --model_file argument of Anchored_Fusion.py
- HVLD probability threshold of 0.1 applied downstream: candidate fusions scoring below 0.1 are removed

## Limitations

- If the user-provided dataset is too small, retrained HVLD parameters are unreliable and the default GitHub model should be used
- Fusion genes not including the user-provided target/anchored gene will not be detected
- The pre-trained filter has not been trained on the user's input data, so results may show deviations
- The value of the loss-balancing hyperparameter alpha in the HVLD loss function is not specified

## Evidence
