---
name: single-cell-rna-seq-annotation-benchmarking
description: Use when evaluating or comparing cell-type annotation methods on scRNA-seq
  data with simulated ground-truth labels, or when deciding between EasyCellType-style
  marker-database querying and supervised tools like scPred or SingleR.
when_to_use_negative:
- Input is real data without ground-truth labels — accuracy cannot be computed, so
  a quantitative benchmark is not the right framing
- The goal is a single dataset's annotation rather than a method comparison — run
  EasyCellType or SingleR directly instead of building a benchmark harness
- Input is already an annotation output (predicted labels table); this skill starts
  from count matrices, not post-hoc label reconciliation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: EasyCellType
  role: 'Method under test: marker-based annotation via GSEA and a modified Fisher''s
    exact test querying cell marker databases (soft and hard classifiers)'
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Standard preprocessing and marker identification pipeline (normalization,
    variable features, PCA, clustering, FindAllMarkers) applied to each simulated
    dataset
- name: Garnett
  role: Supervised comparison tool run in cell-level and cluster-level extension modes
- name: scPred
  role: Supervised comparison tool trained on 3500 sampled cells per dataset, evaluated
    in prediction and no-rejection variants
- name: SingleR
  role: Supervised comparison tool using HumanPrimaryCellAtlasData reference
- name: scCATCH
  role: Related cluster-based marker-matching annotator from the same method family
    (CellMatch database), not part of the primary benchmark
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:b0fca5ba16602a6b3a0a280fcd34d9f20b21ba91743bc364bd210fc661a69e65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - EasyCellType
  - Garnett
  - Seurat
  - SingleR
  - scCATCH
  - scPred
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# single-cell-rna-seq-annotation-benchmarking

## Summary

A benchmarking skill for comparing cell-type annotation tools on scRNA-seq data: simulate PBMC datasets with known ground-truth labels, run supervised tools (Garnett, scPred, SingleR) and marker-based tools (EasyCellType's GSEA and modified Fisher's exact test soft/hard classifiers), and score per-cell annotation accuracy. It exists to quantify when marker-database querying is comparable to supervised reference-based annotation.

## When to use

Use when evaluating or comparing cell-type annotation methods on scRNA-seq data with simulated ground-truth labels, or when deciding between EasyCellType-style marker-database querying and supervised tools like scPred or SingleR.

## When NOT to use

- Input is real data without ground-truth labels — accuracy cannot be computed, so a quantitative benchmark is not the right framing
- The goal is a single dataset's annotation rather than a method comparison — run EasyCellType or SingleR directly instead of building a benchmark harness
- Input is already an annotation output (predicted labels table); this skill starts from count matrices, not post-hoc label reconciliation

## Inputs

- Simulated PBMC scRNA-seq count matrices at multiple cell counts with ground-truth cell-type labels
- Seurat object (pbmc3k-style) after clustering, for marker extraction
- Cluster marker table from FindAllMarkers with Entrez IDs mapped via org.Hs.eg.db
- SingleR reference: HumanPrimaryCellAtlasData

## Outputs

- Combined per-cell annotation accuracy table (figure_s5_accuracy_table.csv) across tool variants and dataset sizes
- Per-tool/variant cell-type label predictions
- EasyCellType test statistics and annotation visualizations from GSEA and modified Fisher's exact test

## How to apply

Simulate PBMC scRNA-seq count matrices at several cell counts with known ground-truth cell-type labels, then preprocess each with the standard Seurat pipeline: CreateSeuratObject (min.cells = 3, min.features = 200), QC filtering (nFeature_RNA 200–2500, percent.mt < 5), NormalizeData, FindVariableFeatures (vst, nfeatures = 2000), ScaleData, RunPCA, FindNeighbors (dims 1:10), FindClusters (resolution = 0.5), and FindAllMarkers (only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25). Annotate cells with each tool and variant: Garnett (cell-level and cluster-level extension), scPred (trained on 3500 sampled cells, prediction and no-rejection variants), SingleR (HumanPrimaryCellAtlasData reference), and EasyCellType's two tests — GSEA and a Fisher's exact test combined with expression significance — which act as soft classifiers. Compute per-cell accuracy against the simulated ground truth for every tool/variant and dataset size, and write the combined table (figure_s5_accuracy_table.csv). The decision point to check is the soft-versus-hard classifier contrast: soft classifiers (GSEA/Fisher) are expected to be comparable to or better than supervised methods, while hard classifiers lag behind supervised tools except Garnett and SingleR.

## Related tools

- **EasyCellType** (Method under test: marker-based annotation via GSEA and a modified Fisher's exact test querying cell marker databases (soft and hard classifiers)) — https://github.com/rx-li/EasyCellType
- **Seurat** (Standard preprocessing and marker identification pipeline (normalization, variable features, PCA, clustering, FindAllMarkers) applied to each simulated dataset)
- **Garnett** (Supervised comparison tool run in cell-level and cluster-level extension modes)
- **scPred** (Supervised comparison tool trained on 3500 sampled cells per dataset, evaluated in prediction and no-rejection variants)
- **SingleR** (Supervised comparison tool using HumanPrimaryCellAtlasData reference)
- **scCATCH** (Related cluster-based marker-matching annotator from the same method family (CellMatch database), not part of the primary benchmark) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)
```

## Evaluation signals

- Per-cell accuracy values across all dataset sizes and tool/variants are internally consistent (no NaNs or duplicated tool/variant rows in figure_s5_accuracy_table.csv)
- Qualitative pattern matches Supplementary Figure S5: supervised methods except Garnett and SingleR outperform EasyCellType's hard classifiers
- EasyCellType's soft classifiers (GSEA, Fisher's exact test with expression significance) score comparable to or better than the supervised methods
- Accuracy computed strictly against simulated ground-truth labels, not cluster identities
- Marker lists derive from the Seurat FindAllMarkers settings used in the paper (only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)

## Limitations

- Supervised tools without rejection options can dominate EasyCellType's hard classifiers, so results depend on the classifier-variant pairing chosen
- Simply overlapping gene lists without considering gene ranking might lead to unreliable results — ranking-aware tests (GSEA) matter
- Software version numbers and full enumeration of marker databases are not documented in the article, limiting exact reproducibility

## Evidence

- [abstract] The simulation study and real-data applications demonstrate favorable results by the proposed method.: "The simulation study and real-data applications demonstrate favorable results by the proposed method."
- [intro] Two quantification approaches provided: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [methods] Standard Seurat workflow on PBMC data: "We followed the standard workflow provided by `Seurat` package[@seurat] to process the PBMC data set."
- [methods] FindAllMarkers parameters used for marker extraction: "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [readme] README description of EasyCellType's role: "devtools::install_github("rx-li/EasyCellType")"
