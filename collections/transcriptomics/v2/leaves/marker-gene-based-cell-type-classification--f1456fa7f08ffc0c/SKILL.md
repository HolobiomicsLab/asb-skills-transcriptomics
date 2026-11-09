---
name: marker-gene-based-cell-type-classification
description: Use when scRNA-seq clusters have been identified (e.g., via robustSingleCell
  PCA/clustering on LCMV1/LCMV2 10X data) and need to be assigned biologically interpretable
  cell-type labels from a curated marker-gene panel.
when_to_use_negative:
- Clusters have not yet been defined — marker annotation requires completed PCA/clustering
  and summarize differential expression output
- Data are not murine T cell scRNA-seq where the curated markers (Tcf7, Cxcr5, Bcl6,
  Foxp3, Mki67, etc.) are expected to discriminate subtypes; the marker panel must
  be adapted to the user's own data as the README states
- The goal is cross-dataset cluster integration/similarity rather than labeling —
  use assess.cluster.similarity / get.robust.cluster.similarity instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: robustSingleCell
  role: Pipeline providing get.cluster.names and set.cluster.names functions that
    test curated marker genes against per-cluster limma differential expression statistics
    and assign subtype labels
  repo: https://github.com/asmagen/robustSingleCell
- name: limma
  role: Backend for the differential expression (every cluster vs. all others) whose
    fold-change and FDR-corrected Q-values gate marker-based annotation
- name: R
  role: Runtime environment in which the types dataframe is built and the annotation
    functions are called
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:f3ed90f670595090ad424ede28444a3c6fb88a737ef6c81e30c41397cd0a8d54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/asmagen/robustSingleCell
  tools:
  - R
  - limma
  - robustSingleCell
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# marker-gene-based-cell-type-classification

## Summary

Annotates scRNA-seq clusters with curated T cell subtype names (Tfh, Th1, Tcmp, Treg, Tmem, CD8, CD4, Cycle) by testing curated marker genes against cluster differential expression statistics from robustSingleCell's limma-based summarize step.

## When to use

Use when scRNA-seq clusters have been identified (e.g., via robustSingleCell PCA/clustering on LCMV1/LCMV2 10X data) and need to be assigned biologically interpretable cell-type labels from a curated marker-gene panel.

## When NOT to use

- Clusters have not yet been defined — marker annotation requires completed PCA/clustering and summarize differential expression output
- Data are not murine T cell scRNA-seq where the curated markers (Tcf7, Cxcr5, Bcl6, Foxp3, Mki67, etc.) are expected to discriminate subtypes; the marker panel must be adapted to the user's own data as the README states
- The goal is cross-dataset cluster integration/similarity rather than labeling — use assess.cluster.similarity / get.robust.cluster.similarity instead

## Inputs

- robustSingleCell project environment after cluster.analysis and summarize (limma differential expression statistics per cluster)
- types dataframe mapping cell-type names to marker genes (e.g., data.frame(type='Tfh',gene=c('Tcf7','Cxcr5','Bcl6')))
- min.fold and max.Qval thresholds (1.0 and 0.01 in the tutorial)

## Outputs

- Cluster-name mapping from get.cluster.names (annotated labels per cluster)
- Updated project environment with cell-type cluster labels via set.cluster.names
- Regenerated summaries/plots in the analysis folder (tSNE colored by annotated cluster, marker-gene heatmaps)
- Per-cluster differential expression statistics with gene, fold-change and Q-value fields (cluster_defining_genes.tsv for the source task)

## How to apply

After running robustSingleCell's summarize function, which performs limma-based differential expression of every cluster against all remaining clusters per replicate, build a types dataframe mapping subtype names to marker genes (Tfh: Tcf7/Cxcr5/Bcl6; Th1: Cxcr6/Ifng/Tbx21; Tcmp: Ccr7/Bcl2/Tcf7; Treg: Foxp3/Il2ra; Tmem: Il7r/Ccr7; CD8: Cd8a; CD4: Cd4; Cycle: Mki67/Top2a/Birc5). Call get.cluster.names with min.fold = 1.0 and max.Qval = 0.01 so that only marker genes with fold change above 1.0 and Q-value below 0.01 support a cluster label. Assign the returned labels to clusters with set.cluster.names, then rerun summarize to regenerate tSNE and visualization outputs in the analysis folder with the annotated names. For the source task, cluster-defining genes must pass FDR-corrected (Benjamini–Hochberg) tests independently in both biological replicates with fold change > 1.25 and Q-value < 0.1.

## Related tools

- **robustSingleCell** (Pipeline providing get.cluster.names and set.cluster.names functions that test curated marker genes against per-cluster limma differential expression statistics and assign subtype labels) — https://github.com/asmagen/robustSingleCell
- **limma** (Backend for the differential expression (every cluster vs. all others) whose fold-change and FDR-corrected Q-values gate marker-based annotation)
- **R** (Runtime environment in which the types dataframe is built and the annotation functions are called)

## Examples

```
LCMV1_cluster_names <- get.cluster.names(LCMV1, types, min.fold = 1.0, max.Qval = 0.01); LCMV1 <- set.cluster.names(LCMV1, names = LCMV1_cluster_names); summarize(LCMV1, local = T)
```

## Evaluation signals

- cluster_defining_genes.tsv exists and contains one entry per annotated cluster with gene, fold-change and Q-value fields
- Each assigned cluster label matches one of the curated T-cell subtype marker sets within min.fold = 1.0 / max.Qval = 0.01 thresholds
- Cluster-defining genes pass FDR-corrected (Benjamini–Hochberg) tests independently in both biological replicates (fold change > 1.25, Q-value < 0.1)
- After set.cluster.names and rerunning summarize, plots (e.g., tSNE, marker heatmap) display subtype names instead of numeric cluster IDs
- Canonical marker heatmap (Cd8a, Cd4, Foxp3, Bcl6, Cxcr5, Tbx21, etc.) shows row-normalized enrichment consistent with the assigned labels

## Limitations

- Marker-gene types dataframe is example-specific; the README instructs users to 'adapt to their own data'
- robustSingleCell is not currently maintained, per the README warning
- Tutorial analyses are subsampled to 500 cells and require at least 8G of memory under slurm; results at reduced depth may not reflect full datasets

## Evidence

- [readme] After differential expression analysis, `get.cluster.names` assigns clusters with names using a customized set of marker genes which users should adapt to their own data.: "After differential expression analysis, `get.cluster.names` assigns clusters with names using a customized set of marker genes which users should adapt"
- [readme] Marker thresholds with get.cluster.names: "LCMV1_cluster_names <- get.cluster.names(LCMV1, types, min.fold = 1.0, max.Qval = 0.01)"
- [methods] Summarize performs limma-based DE against all other clusters: [evidence span withheld — anchor status: not_found]
- [methods] Replicate-level significance requirement: [evidence span withheld — anchor status: not_found]
- [intro] Pipeline purpose statement: "robustSingleCell is a pipeline designed to identify robust cell subpopulations using scRNAseq data"
