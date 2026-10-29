---
name: annotation-accuracy-evaluation-against-predefined-labels
description: Use when a new or existing cell type annotation method must be benchmarked
  for accuracy against predefined labels on scRNA-seq datasets, e.g. reproducing deCS
  vs. SingleR/CHETAH/scPred/SciBet/Cell BLAST comparisons.
when_to_use_negative:
- The query data are cancer cells — the paper notes deCS 'might be challenging for
  cancer cell annotations due to the expression profile difference between healthy
  and malignant cells'
- The goal is discovering novel cell types — as a mapping-based toolkit, deCS 'fails
  to characterize novel cell types'
- The input dataset lacks predefined cell labels, so no accuracy can be computed against
  a reference standard
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: Primary annotator evaluated in correlation (deCS.correlation) and Fisher (deCS.fisher)
    modes against multiple reference panels
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Preprocessing, clustering (FindNeighbors dims=1:10, FindClusters resolution=0.5)
    and marker gene identification (FindAllMarkers) before annotation
- name: SingleR
  role: Baseline annotator run on the same datasets for accuracy comparison
- name: CHETAH
  role: Baseline annotator run on the same datasets for accuracy comparison
- name: scPred
  role: Baseline annotator run on the same datasets for accuracy comparison
- name: SciBet
  role: Baseline annotator run on the same datasets for accuracy comparison
- name: Cell BLAST
  role: Baseline annotator run on the same datasets for accuracy comparison
- name: CellAssign
  role: Baseline annotator run on the same datasets for accuracy comparison
- name: scMatch
  role: Baseline annotator run on the same datasets for accuracy comparison
- name: scCATCH
  role: Baseline annotator (CellMatch-based) run on the same datasets for accuracy
    comparison
  repo: https://github.com/ZJUFanLab/scCATCH
- name: scVI
  role: Baseline annotator run on the same datasets for accuracy comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:9dbcb4c5641dc5c93b2db6deb75bccdf3ccf4c6126e2d9755c462e4eff482f01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - CHETAH
  - Cell BLAST
  - CellAssign
  - SciBet
  - Seurat
  - SingleR
  - deCS
  - scCATCH
  - scMatch
  - scPred
  - scVI
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# annotation-accuracy-evaluation-against-predefined-labels

## Summary

Benchmark an automatic scRNA-seq cell type annotation tool (deCS and baselines such as SingleR, CHETAH, scPred, SciBet, Cell BLAST) against predefined cell labels across eight test datasets, and compute per-tool annotation accuracy to populate a comparison table.

## When to use

Use when a new or existing cell type annotation method must be benchmarked for accuracy against predefined labels on scRNA-seq datasets, e.g. reproducing deCS vs. SingleR/CHETAH/scPred/SciBet/Cell BLAST comparisons.

## When NOT to use

- The query data are cancer cells — the paper notes deCS 'might be challenging for cancer cell annotations due to the expression profile difference between healthy and malignant cells'
- The goal is discovering novel cell types — as a mapping-based toolkit, deCS 'fails to characterize novel cell types'
- The input dataset lacks predefined cell labels, so no accuracy can be computed against a reference standard

## Inputs

- Test scRNA-seq datasets with predefined cell labels (e.g. PBMC 3k from 10X Genomics, pbmc_example.rda, bronchoalveolar immune cells from covid_balf, cardiac cells from GSE106118)
- deCS built-in reference panels (BlueprintEncode_main/fine, DICE_main/fine, MonacoImmune_main/fine, Human_cell_landscape, Human_cell_atlas_of_fetal) and CellMatch_markers
- Seurat-preprocessed query objects with cluster assignments and cluster-average z-score expression matrices and top-10 marker gene lists

## Outputs

- Per-cluster deCS annotation tables with Max_correlation, Top1/Top2/Top3 and Cell_labels columns
- Annotation assignments per tool (deCS correlation/Fisher and baseline methods) per dataset
- Per-tool and per-mode annotation accuracy values comparable to the published benchmark table (deCS correlation ~93.6% average)

## How to apply

Download the test datasets (PBMC 3k from 10X Genomics or pbmc_example.rda from the deCS GitHub, bronchoalveolar immune cells from github.com/zhangzlab/covid_balf, cardiac cells from NCBI GEO GSE106118, plus remaining tissue datasets) with their predefined cell labels. Preprocess each dataset in Seurat using NormalizeData, FindVariableFeatures, ScaleData, RunPCA, then cluster with FindNeighbors(dims = 1:10) and FindClusters(resolution = 0.5). Run deCS in correlation mode (deCS.correlation on the cluster-average z-score matrix against reference panels such as BlueprintEncode, DICE, MonacoImmune, Human_cell_landscape, Human_cell_atlas_of_fetal) and Fisher mode (deCS.fisher on top-10 FindAllMarkers gene lists per cluster, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25), controlling mis-annotation with cor_threshold/p_threshold/cell_type_threshold. Run baseline annotators (SingleR, CHETAH, CellAssign, scMatch, scCATCH, scPred, SciBet, scVI, Cell BLAST) on the same preprocessed objects, then compare all annotations against the predefined labels and compute accuracy per tool and mode. Correct application is judged by reproducing the accuracy table, with deCS correlation mode averaging ~93.6% accuracy across the datasets and outperforming baselines on solid tissues.

## Related tools

- **deCS** (Primary annotator evaluated in correlation (deCS.correlation) and Fisher (deCS.fisher) modes against multiple reference panels) — https://github.com/bsml320/deCS
- **Seurat** (Preprocessing, clustering (FindNeighbors dims=1:10, FindClusters resolution=0.5) and marker gene identification (FindAllMarkers) before annotation)
- **SingleR** (Baseline annotator run on the same datasets for accuracy comparison)
- **CHETAH** (Baseline annotator run on the same datasets for accuracy comparison)
- **scPred** (Baseline annotator run on the same datasets for accuracy comparison)
- **SciBet** (Baseline annotator run on the same datasets for accuracy comparison)
- **Cell BLAST** (Baseline annotator run on the same datasets for accuracy comparison)
- **CellAssign** (Baseline annotator run on the same datasets for accuracy comparison)
- **scMatch** (Baseline annotator run on the same datasets for accuracy comparison)
- **scCATCH** (Baseline annotator (CellMatch-based) run on the same datasets for accuracy comparison) — https://github.com/ZJUFanLab/scCATCH
- **scVI** (Baseline annotator run on the same datasets for accuracy comparison)

## Examples

```
devtools::install_github("bsml320/deCS"); pbmc_deCS_cor_panel_A <- deCS.correlation(pbmc_cluster_marker_z_score, MonacoImmune_main_t_score)
```

## Evaluation signals

- Reproduced deCS correlation-mode accuracy averages ~93.6% across the eight test datasets, matching the published table
- deCS accuracy exceeds most baseline tools (SingleR, CHETAH, scPred, SciBet, Cell BLAST) on the six solid tissue benchmark datasets
- Every cluster receives a deCS Cell_labels assignment; clusters below cor_threshold/p_threshold are labeled 'Undetermined cells' rather than mis-assigned
- Same Seurat-preprocessed inputs (identical clustering and marker definitions) are used for all compared annotators
- Per-tool accuracy is computed by comparing each tool's annotations against the predefined cell labels per dataset

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells

## Evidence

- [abstract] Compared to many existing state-of-the-art annotation tools, deCS significantly reduced computation time and increased accuracy.: "Compared to many existing state-of-the-art annotation tools, deCS significantly reduced computation time and increased accuracy."
- [readme] Reference panel loading: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal et al."
- [methods] Filter thresholds: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [readme] Seurat pipeline parameters: "pbmc <- FindNeighbors(pbmc, dims = 1:10) 
pbmc <- FindClusters(pbmc, resolution = 0.5)"
- [discussion] Tissue-agnostic limitation: "First, deCS does not take tissue information (except for HCAF) into account which may bias the annotation results"
