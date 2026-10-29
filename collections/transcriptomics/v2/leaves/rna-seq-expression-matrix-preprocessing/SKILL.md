---
name: rna-seq-expression-matrix-preprocessing
description: Use when raw TCGA RNA-seq data for multiple cancer types must be assembled
  into a single tumor/normal gene-expression matrix before feature selection or classification,
  e.g. to compute per-gene DI scores over 20,501 genes.
when_to_use_negative:
- Input is already a preprocessed/normalized expression matrix or a feature table
  — go straight to feature selection instead of re-downloading TCGA data
- The analysis targets a single small dataset (e.g. one GEO series like GSE40419 with
  164 samples) where the TCGA-Assembler pan-cancer assembly overhead is unnecessary
- Data source is not TCGA RNA-seq (e.g. single-cell or microarray data), since the
  Module_A.R pipeline and sample counts are TCGA-specific
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: TCGA-Assembler
  role: Module_A.R tool used to download TCGA RNA-seq data into the TCGA_DATAS folder
  repo: http://www.compgenome.org/TCGA-Assembler/index.php
- name: R
  role: Runs tcga_download.R and its dependencies (curl, httr, stringr, r) to fetch
    TCGA data
- name: Python 3.4
  role: Runs src/wx_tcga.py to normalize samples into the 6,226-sample pan-cancer
    matrix
  repo: https://github.com/deargen/DearWXpub
- name: Wx
  role: Downstream consumer of the preprocessed matrix; computes per-gene DI scores
    for feature selection
  repo: https://github.com/deargen/DearWXpub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:b8a712d465190f833352424e0fd29c0eb4fc16be9fb52d73084f772835e95750
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Python 3.4
  - R
  - TCGA-Assembler
  - TensorFlow GPU
  - Wx
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# rna-seq-expression-matrix-preprocessing

## Summary

Download and preprocess TCGA pan-cancer RNA-seq (illuminahiseq_rnaseqv2-RSEM_genes_normalized) data for 12 cancer types into a normalized 6,226-sample gene-expression matrix (5,609 tumor / 617 normal) suitable for downstream feature selection such as the Wx DI-score algorithm.

## When to use

Use when raw TCGA RNA-seq data for multiple cancer types must be assembled into a single tumor/normal gene-expression matrix before feature selection or classification, e.g. to compute per-gene DI scores over 20,501 genes.

## When NOT to use

- Input is already a preprocessed/normalized expression matrix or a feature table — go straight to feature selection instead of re-downloading TCGA data
- The analysis targets a single small dataset (e.g. one GEO series like GSE40419 with 164 samples) where the TCGA-Assembler pan-cancer assembly overhead is unnecessary
- Data source is not TCGA RNA-seq (e.g. single-cell or microarray data), since the Module_A.R pipeline and sample counts are TCGA-specific

## Inputs

- TCGA RNA-seq illuminahiseq_rnaseqv2-RSEM_genes_normalized data for 12 cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, THCA)
- TCGA-Assembler Module_A.R download tool
- tcga_download.R driver script with R packages curl, httr, stringr, r

## Outputs

- TCGA_DATAS folder of downloaded per-cancer-type RNA-seq files
- Preprocessed normalized pan-cancer expression matrix of 6,226 samples (5,609 tumor / 617 normal) across 20,501 genes
- wx_tcga_di_ranked_genes.csv with the full DI-ranked gene list (after feature selection)

## How to apply

Download the TCGA RNA-seq (illuminahiseq_rnaseqv2-RSEM_genes_normalized) data for the 12 cancer types using the TCGA-Assembler Module_A.R tool driven by tcga_download.R, first installing the R dependencies (curl, httr, stringr, r) and OpenSSL/libcurl system libraries; this populates a TCGA_DATAS folder in the repository path. Then preprocess the cohort with `python src/wx_tcga.py`, which normalizes samples into the 6,226-sample pan-cancer matrix (5,609 tumor / 617 normal, 90.09% tumor ratio) covering BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, and THCA. The resulting matrix feeds directly into Wx feature selection (Keras 2.1.2 / TensorFlow GPU 1.4.0, Python 3.4) to compute per-gene DI scores. Validate cohort composition against the expected per-type sample counts (e.g. BRCA 1,214 = 1,101 tumor + 113 normal) before proceeding to feature ranking.

## Related tools

- **TCGA-Assembler** (Module_A.R tool used to download TCGA RNA-seq data into the TCGA_DATAS folder) — http://www.compgenome.org/TCGA-Assembler/index.php
- **R** (Runs tcga_download.R and its dependencies (curl, httr, stringr, r) to fetch TCGA data)
- **Python 3.4** (Runs src/wx_tcga.py to normalize samples into the 6,226-sample pan-cancer matrix) — https://github.com/deargen/DearWXpub
- **Wx** (Downstream consumer of the preprocessed matrix; computes per-gene DI scores for feature selection) — https://github.com/deargen/DearWXpub

## Examples

```
git clone https://github.com/deargen/DearWXpub.git && cd DearWXpub && Rscript tcga_download.R && python src/wx_tcga.py
```

## Evaluation signals

- TCGA_DATAS folder is populated after tcga_download.R completes
- Preprocessed matrix totals 6,226 samples: 5,609 tumor and 617 normal (90.09% tumor ratio)
- Per-type counts match the README table, e.g. BRCA 1,214 (1,101/113), LUAD 576 (517/59), KICH 90 (65/25)
- The matrix contains 20,501 genes evaluated, sufficient to rank GAPDH in the top 14 and ACTB at rank 14 in the downstream DI ranking
- Downstream Wx classification accuracy on the 14-gene set reaches ~96.72% on 3,119 validation samples, indicating the matrix was normalized correctly

## Limitations

- Sample counts in the public repo are larger than in the paper because some TCGA samples were updated, so selected features can differ from the published list
- Selected features can differ depending on the neural network backend framework used (Keras vs the original naive TensorFlow)
- Neural network-based approaches require a large number of samples for good classification or regression performance

## Evidence

- [methods] Download the TCGA RNA-seq (illuminahiseq_rnaseqv2-RSEM_genes_normalized) data for the 12 cancer types using the TCGA-Assembler Module_A.R tool driven by tcga_download.R, populating the TCGA_DATAS folder.: "Download the TCGA RNA-seq data"
- [readme] we downloaded tool (`Module_A.R`) via  http://www.compgenome.org/TCGA-Assembler/index.php: "we downloaded tool (`Module_A.R`) via  http://www.compgenome.org/TCGA-Assembler/index.php"
- [readme] It will generate preprocessed TCGA data set. And Select features, Get the scores.: "It will generate preprocessed TCGA data set. And Select features, Get the scores."
- [results] a pan-cancer cohort from TCGA RNA-seq data consisting of 12 different types of cancer and normal (control) samples: "a pan-cancer cohort from TCGA RNA-seq data consisting of 12 different types of cancer and normal (control) samples"
