---
name: simulated-scrna-seq-count-matrix-generation
description: Use when benchmarking cell-type annotation tools requires scRNA-seq datasets
  with known ground-truth labels (e.g. simulated PBMC count matrices at varying cell
  counts) so per-cell annotation accuracy can be computed without manual curation.
when_to_use_negative:
- Input is real experimental scRNA-seq data without verified ground-truth labels —
  per-cell accuracy cannot be computed, so annotation tools should instead be compared
  on qualitative/soft-evidence grounds
- The goal is annotation of a single user dataset with EasyCellType (GSEA or modified
  Fisher's exact test on marker lists), not benchmarking against supervised tools
  like Garnett, scPred or SingleR
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: EasyCellType
  role: Tool under evaluation; its hard and soft classifiers (GSEA and modified Fisher's
    exact test) are benchmarked for annotation accuracy on the simulated PBMC datasets
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Standard preprocessing pipeline applied to each simulated dataset before annotation
    (normalization, variable feature selection, scaling, PCA, clustering)
- name: Garnett
  role: Supervised comparison tool; run in both cell-level and cluster-level extension
    modes using a pre-trained PBMC classifier
- name: scPred
  role: Supervised comparison tool; trained on 3500 sampled cells per dataset with
    prediction and no-rejection prediction variants
- name: SingleR
  role: Supervised comparison tool; annotates cells using the HumanPrimaryCellAtlasData
    reference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:b842a4d8952a2fffe1f03c0f04fe452c87f76be48fcac2d8a4d3d91fc2b81818
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - EasyCellType
  - Garnett
  - Seurat
  - SingleR
  - scPred
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Simulated scRNA-seq count matrix generation

## Summary

Generates simulated PBMC scRNA-seq count matrices at several cell counts with known ground-truth cell-type labels, used as benchmark datasets to quantitatively compare cell-type annotation tools such as EasyCellType, Garnett, scPred and SingleR.

## When to use

Use when benchmarking cell-type annotation tools requires scRNA-seq datasets with known ground-truth labels (e.g. simulated PBMC count matrices at varying cell counts) so per-cell annotation accuracy can be computed without manual curation.

## When NOT to use

- Input is real experimental scRNA-seq data without verified ground-truth labels — per-cell accuracy cannot be computed, so annotation tools should instead be compared on qualitative/soft-evidence grounds
- The goal is annotation of a single user dataset with EasyCellType (GSEA or modified Fisher's exact test on marker lists), not benchmarking against supervised tools like Garnett, scPred or SingleR

## Inputs

- Simulation specification: target PBMC cell counts (several dataset sizes) and ground-truth cell-type label sets
- PBMC scRNA-seq reference data (e.g. pbmc3k filtered gene-barcode matrices from 10X Genomics) to inform simulated expression profiles

## Outputs

- Simulated PBMC scRNA-seq count matrices at several cell counts, each paired with known ground-truth cell-type labels
- Preprocessed Seurat objects (normalized, variable features, scaled, PCA, clusters) ready for annotation-tool comparison
- Per-cell accuracy values feeding figure_s5_accuracy_table.csv (combined accuracy table across tool variants and dataset sizes)

## How to apply

Simulate PBMC scRNA-seq count matrices at several cell counts, embedding known ground-truth cell-type labels that serve as the reference for accuracy evaluation. For each simulated dataset, run the standard Seurat workflow (CreateSeuratObject with min.cells=3, min.features=200; NormalizeData; FindVariableFeatures with selection.method='vst' and nfeatures=2000; ScaleData; RunPCA; FindNeighbors on dims 1:10; FindClusters at resolution 0.5) so that downstream annotation tools receive comparably preprocessed input. The decision point is dataset size: generate multiple cell counts so that accuracy trends across dataset scale (as in Supplementary Figure S5) can be observed. The rationale is that simulations provide exact per-cell truth labels, letting you compute per-cell annotation accuracy for every tool/variant — EasyCellType hard vs soft classifiers, Garnett (cell-level and cluster-level extension), scPred (prediction and no-rejection variants), SingleR with HumanPrimaryCellAtlasData — under identical conditions.

## Related tools

- **EasyCellType** (Tool under evaluation; its hard and soft classifiers (GSEA and modified Fisher's exact test) are benchmarked for annotation accuracy on the simulated PBMC datasets) — https://github.com/rx-li/EasyCellType
- **Seurat** (Standard preprocessing pipeline applied to each simulated dataset before annotation (normalization, variable feature selection, scaling, PCA, clustering))
- **Garnett** (Supervised comparison tool; run in both cell-level and cluster-level extension modes using a pre-trained PBMC classifier)
- **scPred** (Supervised comparison tool; trained on 3500 sampled cells per dataset with prediction and no-rejection prediction variants)
- **SingleR** (Supervised comparison tool; annotates cells using the HumanPrimaryCellAtlasData reference)

## Evaluation signals

- Per-cell annotation accuracy computed against the simulated ground-truth labels for every tool/variant and dataset size
- Reproduced accuracy values across datasets and tool variants are internally consistent (no contradictions between runs of the same configuration)
- Results qualitatively reproduce the Supplementary Figure S5 pattern: supervised methods except Garnett and SingleR generally outperform EasyCellType's hard classifiers, while EasyCellType's soft classifiers are comparable to or better than the supervised methods
- Combined accuracy table is written to figure_s5_accuracy_table.csv covering all tool/variant and dataset-size combinations

## Limitations

- Software version numbers for the annotation tools are not documented in the article, so exact reproduction of the simulation benchmark may vary with package versions
- The article's abstract only reports that the 'simulation study and real-data applications demonstrate favorable results'; detailed simulation parameters must come from the supplementary reproduction recipe (task_002), not the main text

## Evidence

- [abstract] Simulation study demonstrates favorable results for EasyCellType: "The simulation study and real-data applications demonstrate favorable results by the proposed method."
- [methods] Standard Seurat workflow applied to PBMC data: "We followed the standard workflow provided by `Seurat` package[@seurat] to process the PBMC data set."
- [intro] Two annotation quantification approaches benchmarked: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [readme] EasyCellType package provides simulation/processed PBMC data: "EasyCellType is a R package developed to automatically examine the input marker lists obtained from existing software such as Seurat over the cell marker databases."
