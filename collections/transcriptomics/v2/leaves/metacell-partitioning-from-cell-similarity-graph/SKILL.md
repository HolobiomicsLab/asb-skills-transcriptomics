---
name: metacell-partitioning-from-cell-similarity-graph
description: Use when a scRNA-seq dataset is too large or sparse for robust per-cell
  statistics and you need disjoint groups of cells of the same biological state as
  input for downstream analyses (trajectories, cell typing, differential analysis
  at the metacell level).
when_to_use_negative:
- The input is already aggregated at the metacell level or is a feature table — no
  per-cell profiles exist to partition.
- The analysis goal requires single-cell resolution (e.g. rare-state detection where
  each individual profile matters) — the article outlines how and when metacells should
  or should not be used.
- The data has not been pruned of technical artifacts (doublets, ruptured cells) and
  excluded-gene/cell lists have not been specified — the metacells package warns these
  decisions are crucial for high-quality metacells.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetacellAnalysisToolkit
  role: Integrated pipeline (MATK CLI + R package) to build, visualize and evaluate
    metacells with MC2, SuperCell or SEACells; also provides the CD34_sc/CD34_mc example
    datasets
  repo: github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: 'Preprocessing for the cell similarity space: LogNormalize normalization,
    FindVariableFeatures (2500 features), ScaleData, and RunPCA (npcs=50)'
- name: SEACells
  role: One of the supported metacell construction tools; identifies metacells via
    archetypal analysis on a nearest-neighbor kernel built from a low-dimensional
    embedding (X_pca for scRNA-seq)
  repo: dpeerlab/SEACells
- name: SuperCell
  role: One of the supported metacell construction tools invoked by the MATK pipeline
  repo: GfellerLab/SuperCell
- name: metacells (MC2)
  role: Python implementation of the improved metacell algorithm; partitions cell
    profiles of the same biological state with the minimal number of profiles needed
    for robust statistics
  repo: tanaylab/metacells
- name: MCView
  role: Shiny-app for interactive exploration and annotation of metacell models after
    partitioning
  repo: tanaylab/MCView
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:5bd226389b1c266790b6dd0128c3996d6416d6138b1087989640707651143c34
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MCView
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  - metacells (MC2)
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metacell-partitioning-from-cell-similarity-graph

## Summary

Partitions highly similar single cells into disjoint metacells using MC2, SuperCell, or SEACells, based on a cell similarity space defined by PCA of a normalized, variable-feature-selected scRNA-seq dataset, then aggregates raw counts into metacell profiles. Metacells reduce data size and complexity while preserving biologically relevant information and improving interpretability.

## When to use

Use when a scRNA-seq dataset is too large or sparse for robust per-cell statistics and you need disjoint groups of cells of the same biological state as input for downstream analyses (trajectories, cell typing, differential analysis at the metacell level).

## When NOT to use

- The input is already aggregated at the metacell level or is a feature table — no per-cell profiles exist to partition.
- The analysis goal requires single-cell resolution (e.g. rare-state detection where each individual profile matters) — the article outlines how and when metacells should or should not be used.
- The data has not been pruned of technical artifacts (doublets, ruptured cells) and excluded-gene/cell lists have not been specified — the metacells package warns these decisions are crucial for high-quality metacells.

## Inputs

- scRNA-seq raw count matrix (e.g. CD34_sc single-cell dataset from MetacellAnalysisToolkit) with per-cell UMI profiles
- Gene and cell exclusion lists (e.g. mitochondrial genes, low-UMI cells) for supervised filtering by the metacells algorithm
- Choice of partitioning tool: MC2, SuperCell, or SEACells

## Outputs

- Metacell profile matrix saved as metacell_profile_matrix.csv (summed raw counts per metacell)
- Cell-to-metacell membership table saved as cell_membership.csv (disjoint assignment vector, one metacell per cell)
- Seurat object with PCA embedding (npcs=50) defining the cell similarity space

## How to apply

First define the cell similarity space: load the single-cell dataset (e.g. the CD34_sc dataset from MetacellAnalysisToolkit), normalize counts with Seurat's NormalizeData using normalization.method = 'LogNormalize', select 2500 variable features with FindVariableFeatures, scale with ScaleData, and compute a PCA embedding with RunPCA (npcs=50). Then partition highly similar cells into disjoint metacells using one of the supported tools — MC2 (tanaylab/metacells), SuperCell, or SEACells (runnable via the MATK command-line tool from MetacellAnalysisToolkit) — recording the single-cell membership vector so every cell is assigned to exactly one metacell. For metacells intended for comprehensive downstream analyses, sum the raw counts of all cells assigned to each metacell, because many downstream methods operate on raw counts or require particular normalization. Note that metacells reduce sampling variance by summing profiles of cells of the same state, but a metacell is not a cell type; multiple metacells may belong to the same type. The tanaylab metacells package additionally requires specifying excluded_gene/excluded_cell lists, which are crucial for creating high-quality metacells.

## Related tools

- **MetacellAnalysisToolkit** (Integrated pipeline (MATK CLI + R package) to build, visualize and evaluate metacells with MC2, SuperCell or SEACells; also provides the CD34_sc/CD34_mc example datasets) — github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (Preprocessing for the cell similarity space: LogNormalize normalization, FindVariableFeatures (2500 features), ScaleData, and RunPCA (npcs=50))
- **SEACells** (One of the supported metacell construction tools; identifies metacells via archetypal analysis on a nearest-neighbor kernel built from a low-dimensional embedding (X_pca for scRNA-seq)) — dpeerlab/SEACells
- **SuperCell** (One of the supported metacell construction tools invoked by the MATK pipeline) — GfellerLab/SuperCell
- **metacells (MC2)** (Python implementation of the improved metacell algorithm; partitions cell profiles of the same biological state with the minimal number of profiles needed for robust statistics) — tanaylab/metacells
- **MCView** (Shiny-app for interactive exploration and annotation of metacell models after partitioning) — tanaylab/MCView

## Examples

```
conda env create -f env/MetacellAnalysisToolkit_env.yml; MATK --input CD34_sc --tool SEACells
```

## Evaluation signals

- The partition is disjoint: every cell is assigned to exactly one metacell.
- The number of metacell profiles equals the number of unique membership values in cell_membership.csv.
- Every metacell profile is the sum of raw counts of the cells assigned to it (verify by re-aggregating counts from the membership vector).
- Metacell QC metrics (purity, compactness, separation, inner normalized variance) computed via mc_purity/mc_compactness/mc_separation/mc_INV fall in acceptable ranges and can be visualized with qc_boxplot(mc.obj = mc_data, qc.metrics = 'purity').
- Metacells project onto the single-cell UMAP consistently using mc_projection(), confirming the partition reflects the cell similarity space.

## Limitations

- A metacell is not a cell type and not a parametric model of cell state — multiple metacells may belong to the same type or share the same state if the data over-samples it.
- The raw data may suffer technical artifacts (two cells counted in one profile, ruptured cells, nuclear-only profiles) that must be pruned before partitioning; deciding the right excluded-gene and excluded-cell lists relies on analyst-supplied prior biological knowledge.
- Downstream choice matters: metacell profiles should be built by summing raw counts only when downstream methods operate on raw counts or require particular normalization; other aggregation choices may be needed.
- Version 0.9 of the tanaylab metacells package breaks compatibility with 0.8 in APIs and result annotation names/semantics; a conversion script is provided for migration.

## Evidence

- [methods] Partition highly similar cells into disjoint metacells using one of the supported tools (MC2, SuperCell, or SEACells), recording the single-cell membership vector.: [evidence span withheld — anchor status: not_found]
- [discussion] The authors recommend summing raw counts for comprehensive downstream analyses of metacells, since many downstream analysis methods operate on raw counts or require particular normalization.: [evidence span withheld — anchor status: not_found]
- [methods] sc_data <- NormalizeData(sc_data, normalization.method = "LogNormalize"): "sc_data <- NormalizeData(sc_data, normalization.method = "LogNormalize")"
- [readme] SEACells identifies metacells: groups of cells in the same biological state, found by archetypal analysis on a nearest-neighbor kernel built from a low-dimensional embedding (X_pca for scRNA-seq, X_svd for scATAC-seq).: "SEACells identifies **metacells**: groups of cells in the same biological state, found by
archetypal analysis on a nearest-neighbor kernel built from a low-dimensional embedding
(`X_pca` for"
- [readme] Deciding on the right list of excluded genes (and cells) is crucial for creating high-quality metacells.: [evidence span withheld — anchor status: external_doc]
- [abstract] a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods: "a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods"
