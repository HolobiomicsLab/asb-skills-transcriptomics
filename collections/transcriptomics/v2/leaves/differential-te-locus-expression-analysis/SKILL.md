---
name: differential-te-locus-expression-analysis
description: Use when single-cell full-length scRNA-seq data (e.g., Smart-Seq2 glioblastoma
  BAMs) require locus-level TE quantification and downstream clustering/marker testing
  that subfamily-level TE counts cannot resolve.
when_to_use_negative:
- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand
  bases, where flanking coverage within the 2001 bp window is incomplete and alpha_i
  estimates are unreliable
- Distinguishing highly identical TE copies (e.g., HERVK isoforms) that lack uniquely
  mappable flanking regions
- Input is already a gene expression feature table with no TE/multi-mapping read context
  to exploit
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: MATES
  role: Deep-learning framework (AutoEncoder + MLP regressor) that allocates multi-mapping
    reads to specific TE loci and produces locus-level single-cell TE counts
  repo: https://github.com/mcgilldinglab/MATES
- name: bam_processor
  role: Splits STAR-aligned BAMs into unique- vs multi-mapping sets per cell and constructs
    TE-specific coverage vectors
  repo: https://github.com/mcgilldinglab/MATES
- name: data_processor
  role: Builds Vu/Vm coverage vectors, classifies U/M bins, and generates training/prediction
    data with 2500-sample caps and 50-sample family minimum
  repo: https://github.com/mcgilldinglab/MATES
- name: MATES_model
  role: Trains the AutoEncoder on unique-read coverage vectors with TE family one-hot
    encoding and estimates alpha_i via the MLP regressor
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: Computes locus-level counts as URi + ceil(alpha_i x MRi) and subfamily-level
    TE expression per cell
  repo: https://github.com/mcgilldinglab/MATES
- name: Squire
  role: Cited bulk-level TE quantification baseline that MATES improves upon
- name: TEtranscripts
  role: Cited bulk-level TE quantification baseline
- name: Telescope
  role: Cited bulk-level TE quantification baseline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:eaa4c76539740f4174a21cf065019d0e41bb900e8b5090b59216786d68ae315a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - MATES_model
  - Squire
  - TE_quantifier
  - TEtranscripts
  - Telescope
  - bam_processor
  - data_processor
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# differential-te-locus-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the MATES deep-learning pipeline to quantify locus-specific transposable element (TE) expression in Smart-Seq2 single-cell RNA-seq, then tests whether locus-level TE counts improve cell clustering and reveal marker TE associations with gene markers.

## When to use

Use when single-cell full-length scRNA-seq data (e.g., Smart-Seq2 glioblastoma BAMs) require locus-level TE quantification and downstream clustering/marker testing that subfamily-level TE counts cannot resolve.

## When NOT to use

- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand bases, where flanking coverage within the 2001 bp window is incomplete and alpha_i estimates are unreliable
- Distinguishing highly identical TE copies (e.g., HERVK isoforms) that lack uniquely mappable flanking regions
- Input is already a gene expression feature table with no TE/multi-mapping read context to exploit

## Inputs

- STAR-aligned Smart-Seq2 scRNA-seq BAM files (one BAM per cell, data_mode='Smart_seq')
- TE reference bed/CSV (TE_full.bed / TE_nooverlap.bed) and gene reference from build_reference.py
- Sample list and BAM path files
- Cell metadata / cluster labels for marker association testing

## Outputs

- Locus-level TE count matrix (counts = URi + ceil(alpha_i x MRi) per cell)
- Subfamily-level TE count matrix
- Alpha_i (locus-specific multi-mapping rate) estimates from the MLP regressor
- Cell clustering agreement statistics (ARI, NMI) for TE-only vs gene expression matrices
- Marker TE association table (e.g., LTR2B locus chr3|104522003|104522491 near CD166)

## How to apply

Import MATES and use bam_processor to segregate STAR-aligned reads into unique-mapping and multi-mapping BAM sets per cell. Use data_processor to build unique (Vu) and multi (Vm) coverage vectors spanning 2001 bp (1000 bp up- and downstream of each TE start), classifying bins as multi-dominant (M) or unique-dominant (U) with bin size W and threshold P, while capping training samples at 2500 per TE family and dropping families with fewer than 50 samples. Train the MATES_model AutoEncoder on unique-read coverage vectors with TE family one-hot encoding, then estimate locus-specific multi-mapping rate alpha_i with an MLP regressor, and use TE_quantifier to compute locus-level counts as URi + ceil(alpha_i x MRi) per cell. Cluster cells on the TE-only count matrix versus the gene expression matrix (reported median ARI 0.105, median NMI 0.161), test marker TE associations (e.g., HUERS-P1-int, HERVK-int with neoplastic markers; CD74 with LTR2B, LTR40A1), and compare locus-level vs subfamily-level clustering ARI (improvement P = 5.48e-7, Fig. 3j). Accept the run when these reported statistics and marker associations are recovered in the output table.

## Related tools

- **MATES** (Deep-learning framework (AutoEncoder + MLP regressor) that allocates multi-mapping reads to specific TE loci and produces locus-level single-cell TE counts) — https://github.com/mcgilldinglab/MATES
- **bam_processor** (Splits STAR-aligned BAMs into unique- vs multi-mapping sets per cell and constructs TE-specific coverage vectors) — https://github.com/mcgilldinglab/MATES
- **data_processor** (Builds Vu/Vm coverage vectors, classifies U/M bins, and generates training/prediction data with 2500-sample caps and 50-sample family minimum) — https://github.com/mcgilldinglab/MATES
- **MATES_model** (Trains the AutoEncoder on unique-read coverage vectors with TE family one-hot encoding and estimates alpha_i via the MLP regressor) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (Computes locus-level counts as URi + ceil(alpha_i x MRi) and subfamily-level TE expression per cell) — https://github.com/mcgilldinglab/MATES
- **Squire** (Cited bulk-level TE quantification baseline that MATES improves upon)
- **TEtranscripts** (Cited bulk-level TE quantification baseline)
- **Telescope** (Cited bulk-level TE quantification baseline)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- Reproduce median ARI of 0.105 and median NMI of 0.161 for TE-only vs gene-based clustering
- Locus-level vs subfamily-level clustering ARI improvement reaches P = 5.48e-7 (Fig. 3j)
- Marker associations recovered: neoplastic markers (EGFR, HUERS-P1-int, HERVK-int) and immune markers (CD74, LTR2B, LTR40A1)
- Locus-level counts equal URi + ceil(alpha_i x MRi) for every cell; verify formula on spot-checked loci
- Training data respects filters: <= 2500 samples per TE family, families with < 50 samples dropped, TE loci longer than 1000 bp truncated to first 1000 bp

## Limitations

- MATES requires more computational resources (GPUs) than scTE and SoloTE
- Limited ability to quantify long autonomously expressed TEs such as L1HS spanning several thousand bases; multi-mapping rate estimates may be inaccurate for TEs longer than 2001 bp
- Challenges distinguishing identical TE copies such as HERVK isoforms due to high sequence similarity and lack of uniquely mappable flanking regions
- Trimming TE-gene overlaps may unintentionally distort TE quantification, particularly where TEs intersect neighboring genes; no quantitative evaluation of when to opt out is provided
- Biological discoveries from MATES warrant further experimental validation

## Evidence

- [readme] Smart-seq2 downstream tutorial: [evidence span withheld — anchor status: external_doc]
