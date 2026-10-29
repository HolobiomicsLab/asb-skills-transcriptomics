---
name: chimeric-read-negative-sample-generation
description: Use when retraining the HVLD false-positive filter of Anchored-fusion
  on a new RNA-seq dataset and dataset-specific artificial fusion reads are needed
  as negative training samples alongside PCAWG positive samples.
when_to_use_negative:
- The dataset provided by the user is too small — in that case the article recommends
  using the default model parameters provided on the GitHub repository instead of
  retraining on newly generated negative samples
- You only want to run fusion detection without filter retraining — use the '--not_train_filter_model'
  option with a pre-trained model instead of generating new negative samples
- The input is single-end RNA-seq or whole-genome/whole-exome data — Anchored-fusion
  only supports paired-end RNA-seq
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Anchored-fusion
  role: Host pipeline into which the generated negative samples are fed for per-dataset
    HVLD filter model training; retrained model is saved to its 'data' folder
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BWA-MEM
  role: Alignment tool used to construct artificial chimeric reads and to identify
    reads aligned to homologous genes during negative sample generation
- name: PyTorch
  role: Framework for training the HVLD decision model on the generated negative samples
    (>= 1.10.2)
- name: scikit-learn
  role: Supporting ML library for the HVLD model training (>= 1.3)
- name: Biopython
  role: Sequence handling during model training (>= 1.79)
- name: scFusion
  role: Competing scRNA-seq method trained on the same chimeric-read samples for benchmarking
  repo: https://github.com/XiDsLab/scFusion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:60e9591e2fd6d0cc17a1da902d7f230fd84568a3e9a1d58b344521b6d148ee98
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# chimeric-read-negative-sample-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates artificial chimeric reads as dataset-specific negative training samples for the Anchored-fusion HVLD filter model, by aligning reads with BWA-MEM and filtering out reads mapped to homologous genes of the anchored gene and any negative samples containing the anchored gene itself.

## When to use

Use when retraining the HVLD false-positive filter of Anchored-fusion on a new RNA-seq dataset and dataset-specific artificial fusion reads are needed as negative training samples alongside PCAWG positive samples.

## When NOT to use

- The dataset provided by the user is too small — in that case the article recommends using the default model parameters provided on the GitHub repository instead of retraining on newly generated negative samples
- You only want to run fusion detection without filter retraining — use the '--not_train_filter_model' option with a pre-trained model instead of generating new negative samples
- The input is single-end RNA-seq or whole-genome/whole-exome data — Anchored-fusion only supports paired-end RNA-seq

## Inputs

- Paired-end RNA-seq or scRNA-seq FASTQ files (e.g., SRA:SRP291312 K562 scRNA-seq, SRA:SRP158590 multiple myeloma scRNA-seq)
- Reference genome fasta (hg38 or hg19)
- Gene annotation GTF file compatible with the reference fasta
- Homologous gene file (included in the 'data' folder of Anchored-fusion)
- Target gene transcript reference fasta (from NCBI GENE)
- PCAWG positive sample set: 3540 fusion genes (Synapse:syn10003873)

## Outputs

- Artificial chimeric reads as negative training samples, with reads aligned to homologous genes and samples containing the anchored gene removed
- Retrained HVLD filter model file saved in the 'data' folder of Anchored-fusion (path passed via --model_file)

## How to apply

Align the input paired-end RNA-seq reads (e.g., scRNA-seq FASTQ from SRA:SRP291312 K562 or SRA:SRP158590 multiple myeloma) with BWA-MEM to construct artificial chimeric reads that mimic PCR-derived cDNA fusions. During generation, filter out reads that were aligned to homologous genes listed in the homologous gene file, and remove all negative training samples containing the anchored gene X to avoid harmful bias in the decision model. Combine the resulting artificial chimeric reads with the 3540 PCAWG fusion genes used as positive samples and train the HVLD model per dataset in PyTorch (>= 1.10.2) with scikit-learn (>= 1.3) and Biopython. Save the retrained filter model to the 'data' folder of Anchored-fusion as the file specified by --model_file. Correctness is judged by evaluating the retrained HVLD model on held-out test samples (expected AUC of 91.0% for K562 and 94.5% for multiple myeloma, improving over the Transformer baseline by 5.45% and 4.80% respectively).

## Related tools

- **Anchored-fusion** (Host pipeline into which the generated negative samples are fed for per-dataset HVLD filter model training; retrained model is saved to its 'data' folder) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BWA-MEM** (Alignment tool used to construct artificial chimeric reads and to identify reads aligned to homologous genes during negative sample generation)
- **PyTorch** (Framework for training the HVLD decision model on the generated negative samples (>= 1.10.2))
- **scikit-learn** (Supporting ML library for the HVLD model training (>= 1.3))
- **Biopython** (Sequence handling during model training (>= 1.79))
- **scFusion** (Competing scRNA-seq method trained on the same chimeric-read samples for benchmarking) — https://github.com/XiDsLab/scFusion

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- Retrained HVLD model reproduces reported AUCs on held-out scRNA-seq test samples: 91.0% (K562) and 94.5% (multiple myeloma)
- Improvement over the Transformer baseline confirmed: +5.45% (K562) and +4.80% (MM)
- No negative training samples containing the anchored gene X remain after filtering
- No reads aligned to homologous genes (per the homologous gene file) remain in the negative sample set
- Retrained model file is present in the 'data' folder of Anchored-fusion and loadable via --model_file

## Limitations

- If the user dataset is too small, retrained HVLD parameters may be unreliable; the article recommends using the default model parameters provided on GitHub
- All negative training samples containing the anchored gene must be removed, or the decision model may be harmed by biased training data
- Artificial chimeric reads model PCR-derived cDNA fusions; distribution mismatch with true sequencing artifacts may cause deviations when a pre-trained (not retrained) model is used

## Evidence
