---
name: hvld-self-distillation-model-training
description: 'Use when reproducing or adapting Anchored-fusion''s false-positive filtering:
  paired-end RNA-seq (bulk or scRNA-seq, e.g. K562 SRP291312, MM SRP158590) with a
  retrained per-dataset HVLD filter model.'
when_to_use_negative:
- The user's input dataset is too small to retrain reliably — use the default model
  parameters provided on the GitHub repository instead
- The user selects '--not_train_filter_model' with a pre-trained model, or '--not_filter_false_positive'
  to skip filtering entirely for speed
- Data are single-end RNA-seq, whole-genome, or whole-exome sequencing — Anchored-fusion
  only supports paired-end RNA-seq
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Anchored-fusion
  role: Host pipeline whose HVLD module is trained and whose --model_file flag receives
    the retrained filter model
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BWA-MEM
  role: Aligner used to generate artificial chimeric reads as negative training samples
- name: scFusion
  role: Competing scRNA-seq fusion model trained on the same chimeric-read samples
    for benchmarking
  repo: https://github.com/XiDsLab/scFusion
- name: PyTorch
  role: Deep learning framework used to train the HVLD self-distillation model (>=1.10.2)
- name: Biopython
  role: Sequence handling during training sample preparation (>=1.79)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:25d8538fcadcb99f521b024f54b6ad7e4749de86f7085e3c0b2b8d0d1ed009e7
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hvld-self-distillation-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train the hierarchical view learning and distillation (HVLD) self-distillation deep learning filter of Anchored-fusion per dataset, using artificial chimeric reads as negative samples and 3540 PCAWG fusion genes as positive samples, to discriminate true fusions from PCR-derived artificial cDNA chimeras.

## When to use

Use when reproducing or adapting Anchored-fusion's false-positive filtering: paired-end RNA-seq (bulk or scRNA-seq, e.g. K562 SRP291312, MM SRP158590) with a retrained per-dataset HVLD filter model.

## When NOT to use

- The user's input dataset is too small to retrain reliably — use the default model parameters provided on the GitHub repository instead
- The user selects '--not_train_filter_model' with a pre-trained model, or '--not_filter_false_positive' to skip filtering entirely for speed
- Data are single-end RNA-seq, whole-genome, or whole-exome sequencing — Anchored-fusion only supports paired-end RNA-seq

## Inputs

- Paired-end RNA-seq FASTQ files (bRNA-seq or scRNA-seq, e.g. SRA:SRP291312 K562, SRA:SRP158590 multiple myeloma)
- Artificial chimeric reads generated as negative training samples (BWA-MEM-aligned, homologous-gene-filtered)
- 3540 PCAWG fusion genes as positive training samples
- Homologous gene file and positive training samples from the 'data' folder of the Anchored-fusion program
- Target gene transcript fasta (NCBI GENE), genome fasta (GENCODE/Ensembl, hg38 or hg19), and compatible gene annotation gtf

## Outputs

- Retrained HVLD filter model file stored in the 'data' folder of Anchored-fusion (path given via --model_file)
- Per-dataset filtering decisions with HVLD probabilities for candidate fusions (candidates with probability < 0.1 removed)
- Benchmark metrics such as AUC on held-out test samples (reported: 91.0% K562, 94.5% MM)

## How to apply

Download paired-end FASTQ data (e.g. SRA:SRP291312 for K562 scRNA-seq or SRA:SRP158590 for multiple myeloma), generate artificial chimeric reads as negative samples by aligning with BWA-MEM, filtering reads aligned to homologous genes and removing all negative samples containing the anchored gene. Use the 3540 PCAWG fusion genes as positive training samples alongside dataset-specific artificial chimeric reads. Train the HVLD model with PyTorch (>=1.10.2), numpy, scikit-learn (>=1.3) and Biopython per dataset; benchmark against scFusion on the same chimeric-read samples if desired. Save the retrained filter model to the 'data' folder of the Anchored-fusion program as the file specified by --model_file, or fall back to the default GitHub model parameters for small datasets. At inference, candidate fusions with HVLD probability less than 0.1 are filtered out.

## Related tools

- **Anchored-fusion** (Host pipeline whose HVLD module is trained and whose --model_file flag receives the retrained filter model) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BWA-MEM** (Aligner used to generate artificial chimeric reads as negative training samples)
- **scFusion** (Competing scRNA-seq fusion model trained on the same chimeric-read samples for benchmarking) — https://github.com/XiDsLab/scFusion
- **PyTorch** (Deep learning framework used to train the HVLD self-distillation model (>=1.10.2))
- **Biopython** (Sequence handling during training sample preparation (>=1.79))

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- Held-out test AUC reproduces the reported values: 91.0% on K562 scRNA-seq and 94.5% on the multiple myeloma cohort (improvements of 5.45% and 4.80% over Transformer)
- No negative training sample contains the anchored gene X after the removal step
- Reads aligned to homologous genes are excluded from the negative sample set
- Saved model file exists in the 'data' folder and is loadable via --model_file
- At filtering time, candidates with HVLD probability < 0.1 are absent from final fusion calls

## Limitations

- The value of the loss-balancing hyperparameter alpha in the HVLD loss function is not specified in the article
- For small input datasets, retrained HVLD parameters may be unreliable; the default GitHub model parameters are recommended
- Using the pre-trained model (--not_train_filter_model) on untrained data may give results with some deviations
- Training is per-dataset; the model learns the artificial fusion gene distribution of the current input dataset only

## Evidence
