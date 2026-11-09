---
name: fold-change-qvalue-threshold-filtering
description: Use when assigning reproducible cell-cluster identities from scRNAseq
  differential expression output, e.g. selecting cluster-defining genes across LCMV1/LCMV2
  replicates before naming clusters with curated T-cell markers.
when_to_use_negative:
- Input is a single replicate only — the FC > 1.25 / Q < 0.1 rule requires genes to
  pass DE tests independently in both LCMV1 and LCMV2, so one-replicate results cannot
  satisfy the reproducibility criterion
- Data is not from the robustSingleCell summarize/limma output format (per-cluster
  fold change and Q-value tables), e.g. already-annotated feature tables
- The goal is de novo subtype discovery rather than naming clusters against the curated
  T-cell marker types dataframe
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: robustSingleCell
  role: Provides summarize, get.cluster.names, and set.cluster.names functions that
    compute limma-based DE per cluster and apply fold-change/Q-value cutoffs for cluster
    naming
  repo: https://github.com/asmagen/robustSingleCell
- name: limma
  role: Backend method for differential expression of every cluster against all remaining
    clusters within each dataset
  repo: https://github.com/asmagen/robustSingleCell
- name: R
  role: Runtime environment in which the robustSingleCell filtering and annotation
    functions are executed
  repo: https://github.com/asmagen/robustSingleCell
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:ac6e798b46b387dccc80da356a540637c7753028f8df5413a06a0a2ac1a92c4f
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# fold-change-qvalue-threshold-filtering

## Summary

A two-stage significance filtering procedure in the robustSingleCell pipeline: limma-based per-cluster differential expression statistics are retained only when they pass fold-change and Benjamini-Hochberg Q-value cutoffs (FC > 1.25, Q < 0.1 in both replicates for cluster-defining genes; FC > 1.0, Q < 0.01 for marker-based cluster naming).

## When to use

Use when assigning reproducible cell-cluster identities from scRNAseq differential expression output, e.g. selecting cluster-defining genes across LCMV1/LCMV2 replicates before naming clusters with curated T-cell markers.

## When NOT to use

- Input is a single replicate only — the FC > 1.25 / Q < 0.1 rule requires genes to pass DE tests independently in both LCMV1 and LCMV2, so one-replicate results cannot satisfy the reproducibility criterion
- Data is not from the robustSingleCell summarize/limma output format (per-cluster fold change and Q-value tables), e.g. already-annotated feature tables
- The goal is de novo subtype discovery rather than naming clusters against the curated T-cell marker types dataframe

## Inputs

- Per-cluster limma differential expression statistics (fold change, Q-value) from robustSingleCell summarize for LCMV1 and LCMV2
- Curated marker-gene types dataframe (type/gene pairs for Tfh, Th1, Tcmp, Treg, Tmem, CD8, CD4, Cycle)
- Cluster assignments from cluster.analysis

## Outputs

- Filtered cluster-defining gene set per cluster (gene, fold change, Q-value), reproducible across two biological replicates
- Annotated cluster labels (e.g., Tfh, Th1, Tcmp, Treg, Tmem, CD8, CD4, Cycle) via set.cluster.names
- cluster_defining_genes.tsv with one entry per annotated cluster

## How to apply

For each replicate project (LCMV1, LCMV2), run the robustSingleCell summarize function, which performs limma differential expression of every cluster against all remaining clusters and writes statistics to the analysis folder. Extract per-cluster statistics and retain cluster-defining genes that pass FDR-corrected (Benjamini-Hochberg) tests independently in two biological replicates with fold change > 1.25 and Q-value < 0.1. Separately, when annotating clusters via get.cluster.names, use the curated marker types dataframe (Tfh: Tcf7/Cxcr5/Bcl6; Th1: Cxcr6/Ifng/Tbx21; Tcmp: Ccr7/Bcl2/Tcf7; Treg: Foxp3/Il2ra; Tmem: Il7r/Ccr7; CD8: Cd8a; CD4: Cd4; Cycle: Mki67/Top2a/Birc5) with min.fold = 1.0 and max.Qval = 0.01, then apply the labels with set.cluster.names. The stricter FC > 1.25 / Q < 0.1 cutoff enforces replicate reproducibility, while the looser min.fold = 1.0 / Q < 0.01 cutoff governs marker-based name assignment.

## Related tools

- **robustSingleCell** (Provides summarize, get.cluster.names, and set.cluster.names functions that compute limma-based DE per cluster and apply fold-change/Q-value cutoffs for cluster naming) — https://github.com/asmagen/robustSingleCell
- **limma** (Backend method for differential expression of every cluster against all remaining clusters within each dataset) — https://github.com/asmagen/robustSingleCell
- **R** (Runtime environment in which the robustSingleCell filtering and annotation functions are executed) — https://github.com/asmagen/robustSingleCell

## Examples

```
LCMV1_cluster_names <- get.cluster.names(LCMV1, types, min.fold = 1.0, max.Qval = 0.01); LCMV1 <- set.cluster.names(LCMV1, names = LCMV1_cluster_names)
```

## Evaluation signals

- cluster_defining_genes.tsv exists and contains one entry per annotated cluster with gene, fold-change, and Q-value fields
- Every retained cluster-defining gene passes FDR-corrected (Benjamini-Hochberg) DE tests independently in both LCMV1 and LCMV2 with fold change > 1.25 and Q-value < 0.1
- All assigned cluster labels correspond to one of the curated T-cell subtype marker sets (Tfh, Th1, Tcmp, Treg, Tmem, CD8, CD4, Cycle) within min.fold = 1.0 / max.Qval = 0.01 thresholds
- get.cluster.names was called with min.fold = 1.0 and max.Qval = 0.01 (not the stricter 1.25/0.1 DE cutoffs) before set.cluster.names
- Marker genes for each annotated cluster (e.g., Cxcr5/Bcl6 for Tfh, Foxp3/Il2ra for Treg) show enriched expression in the labeled cluster

## Limitations

- The package is not currently maintained, so fixes or adaptations must be handled manually
- Marker-based naming via get.cluster.names requires users to adapt the curated marker types dataframe to their own data; markers tuned for CD44+ LCMV T cells may not transfer to other tissues or models
- The workflow analyses each dataset (LCMV1, LCMV2) separately to account for dataset-specific technical and biological differences before requiring cross-replicate concordance

## Evidence

- [other] DE thresholds from source card finding: [evidence span withheld — anchor status: not_found]
- [readme] summarize performs DE and get.cluster.names applies marker cutoffs: "The `summarize` function which performs differential expression analysis, computes tSNE and visualizes the results in the analysis folder. After differential expression analysis, `get.cluster.names`"
- [readme] get.cluster.names call with min.fold and max.Qval: "LCMV1_cluster_names <- get.cluster.names(LCMV1, types, min.fold = 1.0, max.Qval = 0.01)"
- [readme] per-replicate analysis rationale: "We cluster each dataset separately to account for dataset-specific technical and biological differences. Then, we measure the transcriptional similarity and divergence between the clusters identified"
