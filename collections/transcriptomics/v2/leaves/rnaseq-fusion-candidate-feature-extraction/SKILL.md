---
name: rnaseq-fusion-candidate-feature-extraction
description: Use when preparing spanning-read/split-read candidate fusion evidence
  (PCAWG positives, artificial chimeric negatives) as features for training or applying
  the HVLD filter in Anchored-fusion.
when_to_use_negative:
- Data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads — Anchored-fusion
  requires paired-end RNA-seq
- Fusions of interest do not include the user-provided anchored gene, since only target-gene
  fusions are reported
- Input dataset is too small to retrain the filter — use the default pre-trained model
  parameters from GitHub instead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Anchored-fusion
  role: Host pipeline that extracts anchored chimeric read candidates and trains the
    HVLD feature-based filter
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BLAT
  role: Performs local alignment on candidate chimeric reads to predict the precise
    fusion location
- name: BWA-MEM
  role: Alignment tool used to map reads to the anchored gene reference and the whole
    genome
- name: PyTorch
  role: Framework for training the HVLD model on extracted chimeric read features
    (>=1.10.2)
- name: scikit-learn
  role: Provides the SVM baseline classifier trained on the same candidate features
    (>=1.3)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:b7760b2439ed4dbbd61e9cdd4530b7e9ab5c453f49c59a351b90faa6fbbea16c
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
  - NumPy
  - PyTorch
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# rnaseq-fusion-candidate-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts classification features from candidate chimeric reads and spanning/split read evidence so the HVLD deep learning filter can distinguish true fusion reads from artificial chimeric fragments in paired-end RNA-seq data.

## When to use

Use when preparing spanning-read/split-read candidate fusion evidence (PCAWG positives, artificial chimeric negatives) as features for training or applying the HVLD filter in Anchored-fusion.

## When NOT to use

- Data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads — Anchored-fusion requires paired-end RNA-seq
- Fusions of interest do not include the user-provided anchored gene, since only target-gene fusions are reported
- Input dataset is too small to retrain the filter — use the default pre-trained model parameters from GitHub instead

## Inputs

- Paired-end RNA-seq FASTQ files (e.g., test_sample_1.fastq.gz)
- Target gene transcript reference FASTA from NCBI GENE
- Genome FASTA (hg38 or hg19, from GENCODE or Ensembl)
- Gene annotation GTF compatible with the genome FASTA
- PCAWG fusion gene positive samples (3540 fusions)
- Homologous gene file from the 'data' folder

## Outputs

- Feature-encoded candidate chimeric read sets for HVLD training/classification
- Trained HVLD filter model saved to the path given by '--model_file' (default under 'data')
- Filtered fusion candidate list (candidates with HVLD probability < 0.1 removed)

## How to apply

Collect paired-end reads where only one mate maps to the anchored gene X and align the unmapped mate to the whole genome, retaining spanning reads and split reads as fusion-supporting evidence. Build per-dataset training sets using the 3540 PCAWG fusion genes as positives and artificial chimeric reads as negatives, removing negative samples containing anchored gene X and reads aligned to homologous genes. Encode candidate chimeric reads as model inputs and train HVLD in PyTorch (>=1.10.2) with its hierarchical view learning and self-distillation loss, balancing the two loss types with the alpha hyperparameter using the positive samples and homologous gene file shipped in the 'data' folder. Filter candidates with HVLD probability below 0.1 and skip training via '--not_train_filter_model' or '--not_filter_false_positive' only when runtime must be saved.

## Related tools

- **Anchored-fusion** (Host pipeline that extracts anchored chimeric read candidates and trains the HVLD feature-based filter) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BLAT** (Performs local alignment on candidate chimeric reads to predict the precise fusion location)
- **BWA-MEM** (Alignment tool used to map reads to the anchored gene reference and the whole genome)
- **PyTorch** (Framework for training the HVLD model on extracted chimeric read features (>=1.10.2))
- **scikit-learn** (Provides the SVM baseline classifier trained on the same candidate features (>=1.3))

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- HVLD must reach approximately 82.1% average accuracy across the six CCLE datasets (K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12), max 83.4%, median 82.1%, min 80.8%
- HVLD accuracy must exceed the Transformer baseline by about 8.85% and the HVL no-distillation variant by about 0.75%
- Candidates with HVLD probability < 0.1 must be removed from the final fusion list
- No negative training samples containing the anchored gene X or reads aligned to homologous genes should remain in the training set
- The retrained filter model file must be written to the '--model_file' location (default 'data' folder)

## Limitations

- The value of the loss-balancing hyperparameter alpha in the HVLD loss function is not specified
- For small input datasets, the retrained HVLD parameters may be unreliable; the default model from GitHub is recommended
- Fusion genes not including the user-provided target gene cannot be detected
- Using the pre-trained model via '--not_train_filter_model' may produce deviating results because it was not trained on the user's input data

## Evidence
