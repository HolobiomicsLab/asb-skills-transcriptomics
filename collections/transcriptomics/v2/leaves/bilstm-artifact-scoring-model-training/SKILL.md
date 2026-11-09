---
name: bilstm-artifact-scoring-model-training
description: 'Use when building scFusion''s deep-learning artifact-scoring stage:
  chimeric/fusion candidates extracted from Smart-Seq scRNA-seq alignments must be
  classified as technical artifacts vs. real fusions before final fusion calls.'
when_to_use_negative:
- Input data is 10x Genomics rather than Smart-Seq scRNA-seq — scFusion is explicitly
  built for Smart-Seq data
- Input is bulk RNA-seq; scFusion's advantage relies on multiple single cells from
  one sample sharing the same fusion
- The target fusion is expected at low expression or is a rare fusion in a highly
  heterogeneous tumor sample, where all RNA-based methods (including scFusion) have
  reduced sensitivity/power
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: scFusion
  role: Pipeline housing the bi-LSTM artifact-scoring model; run via scFusion.py after
    training data preparation
  repo: https://github.com/ZijieJin/scFusion
- name: STAR >= 2.7.2d
  role: Aligns paired Smart-Seq FASTQ files to hg19 to generate chimeric-alignment
    evidence feeding candidate features
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Post-processing of STAR alignments (tested on version 1.10) before candidate
    extraction
- name: bedtools
  role: Extraction of chimeric/fusion-supporting candidate reads (tested on version
    2.29.2)
- name: pyensembl
  role: Annotates fusion candidates using GTF 'gene_name' and 'gene_type'/'gene_biotype'
    fields
  repo: https://github.com/openvax/pyensembl
- name: TensorFlow/Keras
  role: Implements and trains the bi-directional LSTM artifact classifier (tested
    on TF 2.8.0/2.3.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:02191879e9fd2e663f7b7022f8d309ad96567d717ca89edc4091d12e8e5d0eec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Keras
  - Python
  - R
  - STAR >= 2.7.2d
  - TensorFlow
  - TensorFlow/Keras
  - bedtools
  - numpy
  - pyensembl
  - pysam
  - samtools
  - scFusion
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# bilstm-artifact-scoring-model-training

## Summary

Train and validate a bi-directional LSTM (TensorFlow/Keras) model in scFusion that scores scRNA-seq fusion candidates as technical chimeric artifacts, supplementing statistical and ad hoc filters. Trained on six public Smart-Seq datasets, it reaches median ROC AUC 0.884 and median PR AUPR 0.913, and assigns >90% of PCAWG fusions artifact scores <0.5.

## When to use

Use when building scFusion's deep-learning artifact-scoring stage: chimeric/fusion candidates extracted from Smart-Seq scRNA-seq alignments must be classified as technical artifacts vs. real fusions before final fusion calls.

## When NOT to use

- Input data is 10x Genomics rather than Smart-Seq scRNA-seq — scFusion is explicitly built for Smart-Seq data
- Input is bulk RNA-seq; scFusion's advantage relies on multiple single cells from one sample sharing the same fusion
- The target fusion is expected at low expression or is a rare fusion in a highly heterogeneous tumor sample, where all RNA-based methods (including scFusion) have reduced sensitivity/power

## Inputs

- Chimeric/fusion-supporting candidate reads extracted from STAR (>= 2.7.2d) BAM alignments via samtools and bedtools
- Paired Smart-Seq scRNA-seq FASTQ files named *_1.fastq, *_2.fastq (fq suffix not allowed)
- hg19 reference genome FASTA (~3 GB) and GTF annotation (Ensembl/NCBI/UCSC) with 'gene_name' and 'gene_type'/'gene_biotype' fields for pyensembl
- 4-column hg19 UCSC mappability file (optional; enables the mappability filter)
- PCAWG fusion calls for external artifact-score evaluation

## Outputs

- Per-candidate artifact scores (probability that a fusion is a technical chimeric artifact)
- Per-dataset ROC AUC and precision-recall AUPR values for the six evaluated public scRNA-seq datasets
- PCAWG artifact-score summary (fraction with artifact score < 0.5) written to model_evaluation_metrics.csv

## How to apply

First produce fusion candidates by aligning paired Smart-Seq FASTQ files (*_1.fastq, *_2.fastq) to the hg19 reference with STAR (>= 2.7.2d), processing alignments with samtools, extracting chimeric candidates with bedtools, and annotating them with pyensembl using the GTF 'gene_name' and 'gene_type'/'gene_biotype' fields. Optionally apply the hg19 UCSC mappability filter (a 4-column UCSC file; omitting it turns the filter off). Build the bi-LSTM in TensorFlow/Keras (tested versions: tensorflow/keras 2.8.0 with numpy 1.22.3, or tensorflow 2.3.0/keras 2.4.3/numpy 1.18.5/scipy 1.4.1) and train on candidate fusion features to predict artifact vs. real fusions. Score held-out candidates per dataset and compute ROC AUC and PR AUPR; accept the model when the medians across the six datasets are 0.884 (ROC AUC) and 0.913 (PR AUPR) and >90% of PCAWG fusions receive artifact scores <0.5. The rationale is that Smart-Seq's heavy amplification step generates artificial chimeric reads, and the deep-learning model complements statistical and ad hoc filters — turning it off demonstrably reduces precision.

## Related tools

- **scFusion** (Pipeline housing the bi-LSTM artifact-scoring model; run via scFusion.py after training data preparation) — https://github.com/ZijieJin/scFusion
- **STAR >= 2.7.2d** (Aligns paired Smart-Seq FASTQ files to hg19 to generate chimeric-alignment evidence feeding candidate features) — https://github.com/alexdobin/STAR
- **samtools** (Post-processing of STAR alignments (tested on version 1.10) before candidate extraction)
- **bedtools** (Extraction of chimeric/fusion-supporting candidate reads (tested on version 2.29.2))
- **pyensembl** (Annotates fusion candidates using GTF 'gene_name' and 'gene_type'/'gene_biotype' fields) — https://github.com/openvax/pyensembl
- **TensorFlow/Keras** (Implements and trains the bi-directional LSTM artifact classifier (tested on TF 2.8.0/2.3.0))

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- Median ROC AUC across the six public scRNA-seq evaluation datasets equals 0.884
- Median precision-recall AUPR across the six datasets equals 0.913
- Fraction of PCAWG fusions receiving artifact score < 0.5 exceeds 90% (Fig 2c-f)
- model_evaluation_metrics.csv contains per-dataset ROC AUC, PR AUPR, and the PCAWG artifact-score summary
- Removing the deep-learning model from scFusion lowers precision, confirming its contribution

## Limitations

- scFusion has reduced sensitivity for fusions with low expression, common to all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; overcome only by sequencing more cells and/or deeper per cell
- Many potential artifacts are also filtered by ad hoc filters, so the deep-learning model is one of several False-positive control layers
- Recommend 64 GB memory and 8+ CPU cores per task; job scheduler (e.g. Slurm) optional

## Evidence

- [intro] deep-learning model role: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [intro] artifact rationale: "the heavy amplification step may generate artificial chimeric reads, leading to false-positive fusion candidates"
- [methods] alignment tooling: "STAR (https://github.com/alexdobin/STAR) >= 2.7.2d (tested on 2.7.2d and 2.7.8a)"
- [methods] mappability filter optionality: "If you do not provide this file, scFusion will turn off the mappability filter."
- [discussion] deep-learning contribution to precision: "When deep-learning model was turned off, the precisions of scFusion reduced"
- [readme] input format requirement: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq"
- [readme] DL framework versions: "python module: tensorflow, keras, and numpy (version 2.8.0, 2.8.0, and 1.22.3, respectively)"
- [readme] hardware recommendation: "64 GB memory or more for each task"
