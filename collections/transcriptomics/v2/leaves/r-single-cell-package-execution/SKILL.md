---
name: r-single-cell-package-execution
description: Use when the task requires running SingCellaR R functions (e.g. filter_cells_and_genes,
  identifyDifferentialGenes, runKBET, run_DAseq_comparison) on single-cell RNA-seq
  HSPC datasets in R.
when_to_use_negative:
- Input is already an annotated feature/count matrix outside the SingCellaR object
  model — SingCellaR lacks 'automatic object transformation' to interact with other
  packages such as Seurat.
- The analysis requires fully automatic cell-type annotation without curated signatures
  — the user must compile customized gene sets; only 75 pre-curated hematopoiesis
  gene sets are provided.
- Large-scale datasets where AUCell_buildRankings running time and memory issues make
  the workflow impractical.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: SingCellaR
  role: Primary R package whose functions (filter_cells_and_genes, identifyDifferentialGenes,
    runKBET, runScanorama, runFA2_ForceDirectedGraph) execute the workflow
  repo: github.com/supatt-lab/SingCellaR
- name: R
  role: Runtime environment in which SingCellaR objects and function calls are executed
- name: kBET
  role: Batch-effects acceptance-rate metric invoked via SingCellaR's runKBET wrapper
    with fixed seed and n.sample
- name: AUCell
  role: Signature-based cell-type scoring via AUCell_buildRankings within the annotation
    workflow
- name: SingleCellExperiment
  role: Underlying standard R object class used by SingCellaR to store single-cell
    experimental data
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:b56fed35b373dce7d491f7f0d2166bc7d862ee18b664c482007dcaad4e43ca3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AUCell
  - R
  - SingCellaR
  - SingleCellExperiment
  - kBET
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# r-single-cell-package-execution

## Summary

Executing the SingCellaR R package workflows for single-cell RNA-seq analysis of human hematopoietic stem and progenitor cells (HSPCs), from loading Cell Ranger matrices through QC filtering, normalization, clustering, integration benchmarking (LISI/kBET), and DAseq differential abundance testing.

## When to use

Use when the task requires running SingCellaR R functions (e.g. filter_cells_and_genes, identifyDifferentialGenes, runKBET, run_DAseq_comparison) on single-cell RNA-seq HSPC datasets in R.

## When NOT to use

- Input is already an annotated feature/count matrix outside the SingCellaR object model — SingCellaR lacks 'automatic object transformation' to interact with other packages such as Seurat.
- The analysis requires fully automatic cell-type annotation without curated signatures — the user must compile customized gene sets; only 75 pre-curated hematopoiesis gene sets are provided.
- Large-scale datasets where AUCell_buildRankings running time and memory issues make the workflow impractical.

## Inputs

- Cell Ranger gene expression matrices (10x outputs) loadable via load_matrices_from_cellranger
- Processed SingCellaR objects from upstream clustering and annotation workflows (e.g. eFL and FL HSPC objects)
- Deposited UMAP embeddings for each integration method (Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma) with per-cell batch and cell-type metadata
- GMT signature gene-set file for cell-cycle scoring and AUCell annotation

## Outputs

- Quality-controlled, normalized SingCellaR object with variable genes and cell-cycle scores
- Per-cell iLISI/cLISI scores and kBET acceptance-rate distributions per integration method
- DAseq log-fold-change DA scores per cell subpopulation and DA UMAP figures (e.g. eFL_vs_FL.pdf)
- Figure 3B iLISI/cLISI comparison plot and Figure 3C kBET acceptance-rate boxplot

## How to apply

Load gene expression matrices into a SingCellaR object with load_matrices_from_cellranger, then apply filter_cells_and_genes with min/max UMI, detected-gene and percent-mito thresholds, and normalize_UMIs with a scale.factor. Select variable genes via get_variable_genes_by_fitting_GLM_model (mean_expr_cutoff, disp_zscore_cutoff parameters) and cluster with identifyClusters or clustering_KMeansOnKMeans_Graph before annotation with AUCell-derived signature gene sets (75 curated hematopoiesis gene sets provided). For integrative benchmarking, run runLISI and runKBET on each integration embedding (Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma), calling set.seed beforehand and fixing the n.sample parameter so kBET acceptance rates are reproducible. For differential abundance between eFL and FL HSPC objects, call run_DAseq_comparison on the object pair to compute DA log-fold-change scores using the min.log2FC and min.expFraction conventions of the identifyDifferentialGenes family, then project per-cell DA scores onto the existing UMAP embedding and save the plot (e.g. eFL_vs_FL.pdf).

## Related tools

- **SingCellaR** (Primary R package whose functions (filter_cells_and_genes, identifyDifferentialGenes, runKBET, runScanorama, runFA2_ForceDirectedGraph) execute the workflow) — github.com/supatt-lab/SingCellaR
- **R** (Runtime environment in which SingCellaR objects and function calls are executed)
- **kBET** (Batch-effects acceptance-rate metric invoked via SingCellaR's runKBET wrapper with fixed seed and n.sample)
- **AUCell** (Signature-based cell-type scoring via AUCell_buildRankings within the annotation workflow)
- **SingleCellExperiment** (Underlying standard R object class used by SingCellaR to store single-cell experimental data)

## Examples

```
filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets)
```

## Evaluation signals

- The output eFL_vs_FL.pdf exists, renders the DA UMAP, and qualitatively matches Figure 7B (erythroid cells enriched in FL, megakaryocytic cells enriched in eFL).
- Both iLISI/cLISI and kBET plots are produced for all five integration methods.
- kBET acceptance rates are stable across repeated runs given a fixed set.seed and fixed n.sample (small run-to-run variation otherwise).
- Supervised Harmony shows higher kBET and iLISI scores with cLISI close to 1, indicating better integration for this HSPC dataset.
- identifyDifferentialGenes calls honor min.log2FC and min.expFraction filtering conventions.

## Limitations

- kBET scores vary slightly due to different seed-number settings and the number of subsampled cells; fix set.seed and n.sample to stabilize.
- runFA2_ForceDirectedGraph fails when the fa2 Python module is not installed or the R environmental path for the FA2 module is not found by R.
- runScanorama fails when the scanorama Python module is not installed.
- AUCell_buildRankings has running time and memory issues on large-scale datasets.
- UMAP/FDG plot rotations may differ from the protocol due to software-version and seed-number differences.
- SingCellaR lacks automatic object transformation to interact with other packages such as Seurat.

## Evidence

- [discussion] fix the number of cells for the kBET analysis using the n.sample parameter described in the 'runKBET' function: [evidence span withheld — anchor status: not_found]
- [methods] identifyDifferentialGenes(method, objectA, objectB, cellsA, cellsB, write.to.file, min.log2FC, min.expFraction): "`identifyDifferentialGenes(method, objectA, objectB, cellsA, cellsB, write.to.file, min.log2FC, min.expFraction)`"
- [methods] filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, ...): "`filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets)`"
- [discussion] SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R.: "SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R."
- [discussion] The user may encounter slightly different kBET scores from this protocol. This is due to the different seed number settings and the number of subsampling cells: "The user may encounter slightly different kBET scores from this protocol. This is due to the different seed number settings and the number of subsampling cells"
- [readme] The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
